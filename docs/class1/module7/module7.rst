Kubernetes Gateway のセットアップ
####

NGINXをDataplaneとしてKubernetes Gatewayを利用する方法を紹介します。
Kubernetes Gatewayの情報は `GitHub: nginx-kubernetes-gateway <https://github.com/nginxinc/nginx-kubernetes-gateway>`__ を参照してください

.. NOTE::
  本資料作成時点では、NGINX OSSを利用した動作確認となります

Gateway APIについて
====

Gateway APIについて簡単にご説明します

Gateway APIはKubernetesにおいて、既存のIngressに比べより柔軟な設定方法の実現を目的としています。
``GatewayClass``, ``Gateway``, ``HTTPRoute``, ``TCPRoute``, ``Service`` などのリソースを用いて、様々なベンダーや担当者などが自由に設定できるインタフェースを提供します。

   .. image:: https://gateway-api.sigs.k8s.io/images/api-model.png
       :width: 400

各コンセプトについて紹介します

- ``Role-oriented`` : Gatewayは複数のAPIリソースで構成されます。各Roleに応じたKubernetesネットワークの利用や設定を提供します
- ``Portable(移植可能性)`` : 以前の実装から引き継がれる性質。Ingressは汎用的な仕様を備え、Gateway APIは移植可能な多くの機能が実装されています
- ``Expressive(多様な設定)`` : Gatweay APIはヘッダーベースの処理、トラフィックのウェイト等、IngressではCustom Annotationが必要であったような機能がサポートされます
- ``Extensible(拡張性)`` : Gateway APIは様々な層のAPIに接続可能で、より適切な箇所でより細かな要件の実現が可能となります

更に以下のような特徴を持ちます

- ``GatewayCalss`` :  GatewayClass はロードバランシングを形式化したものです。Kubernetesのリソースを利用し、簡単に且つそれぞれ個別にユーザが設定の管理をすることができます
- ``Shared Gateways and cross-Namespace support(複数ネームスペース間でのGatewayの共有)`` : ロードバランサーや仮想IP(VIP)として通信を待ち受ける単一のゲートウェイに、それぞれ個別の Route リソースを割り当てることができます。これにより複数のチームが互いに密な連携をすることなくインフラストラクチャを安全に共有できます
- ``Typed Routes and typed backends(予め定義されたRouteとBackend)`` : Gateway APIは予め役割が定義されたRouteとBackendがあります。これはHTTPやgRPCなどのプロトコルや、ServiceやStorage Bucketなどバックエンドを柔軟にサポートできます

これらの特徴から、複数の組織で多彩な設定と、各設定の柔軟な可搬性を同時に実現します

   .. image:: https://gateway-api.sigs.k8s.io/images/gateway-roles.png
       :width: 400

環境のセットアップ
====

ファイルの取得

.. code-block:: cmdin

  cd ~/ 
  git clone https://github.com/nginxinc/nginx-kubernetes-gateway.git
  cd nginx-kubernetes-gateway

Gateway APIリソースをデプロイします

.. code-block:: cmdin

  ## cd nginx-kubernetes-gateway
  kubectl kustomize "https://github.com/nginx/nginx-gateway-fabric/config/crd/gateway-api/standard?ref=v1.6.1" | kubectl apply -f -
  kubectl apply -f https://raw.githubusercontent.com/nginx/nginx-gateway-fabric/v1.6.1/deploy/crds.yaml

必要となるリソースをデプロイします

.. code-block:: cmdin

  ## cd nginx-kubernetes-gateway
  kubectl apply -f ../nginx-kubernetes-gateway-conf/deploy.yaml

Kubernetes Gateway 用のNGINXが起動していることを確認します

.. code-block:: cmdin

  ## cd nginx-kubernetes-gateway
  kubectl get pods -n nginx-gateway

.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル

  NAME                             READY   STATUS    RESTARTS   AGE
  nginx-gateway-67cb7f7d65-gq8jp   2/2     Running   0          7m5s

外部へNodePortで公開します。
一分内容を追記し、NodePort Serviceをデプロイします

.. code-block:: cmdin

  ## cd nginx-kubernetes-gateway
  cat << EOF > nodeport-config.yaml
  apiVersion: v1
  kind: Service
  metadata:
    name: nginx-gateway
    namespace: nginx-gateway
    labels:
      app.kubernetes.io/name: nginx-gateway-fabric
      app.kubernetes.io/instance: ngf
      app.kubernetes.io/version: "1.6.1"
  spec:
    type: NodePort
    selector:
      app.kubernetes.io/name: nginx-gateway-fabric
      app.kubernetes.io/instance: ngf
    ports:
      - name: http
        port: 80
        protocol: TCP
        targetPort: 80
        nodePort: 31437
      - name: https
        port: 443
        protocol: TCP
        targetPort: 443
        nodePort: 31438
  EOF
  kubectl apply -f nodeport-config.yaml

以下コマンドでポート確認します

.. code-block:: cmdin

  kubectl get svc -n nginx-gateway

.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル

  NAME            TYPE       CLUSTER-IP       EXTERNAL-IP   PORT(S)                      AGE
  nginx-gateway   NodePort   10.110.155.239   <none>        80:32203/TCP,443:31483/TCP   28m



このコマンドを実行した結果、Kubernetes の Worker Nodeでそれぞれのサービスに対しポートが割り当てられています。
`6. NGINX Ingress Controller を外部へ NodePort で公開する <https://f5j-nginx-ingress-controller-lab1.readthedocs.io/en/latest/class1/module2/module2.html#nginx-ingress-controller-nodeport>`__ の内容を参考に、NGINXのConfigを作成し、設定を反映します。

.. code-block:: cmdin

  cd ~/
  sudo cp /etc/nginx/nginx.conf /etc/nginx/nginx.conf-
  cat << EOF > nginx.conf
  user  nginx;
  worker_processes  auto;
  
  error_log  /var/log/nginx/error.log notice;
  pid        /var/run/nginx.pid;
  
  
  events {
     worker_connections  1024;
  }
  
  
  # TCP/UDP load balancing
  #
  stream {
     upstream tcp80_backend {
        server node1:32203;    # HTTP(TCP/80)に割り当てられたポート番号
     }
     upstream tcp443_backend {
        server node1:31483;     # HTTPS(TCP/443)に割り当てられたポート番号
     }
  
     server {
        listen 80;
        proxy_pass tcp80_backend;
     }
     server {
        listen 443;
        proxy_pass tcp443_backend;
     }
  }
  EOF
  sudo cp nginx.conf /etc/nginx/nginx.conf
  sudo nginx -s reload
  
