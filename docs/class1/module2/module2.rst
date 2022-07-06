NGINX Ingress Controller(NIC) 環境のセットアップ
####


| 以下の手順に従ってNGINX Ingress Controllerのイメージを作成します  
| 参考： `Installation with Manifests <https://docs.nginx.com/nginx-ingress-controller/installation/installation-with-manifests/>`__

1. 環境セットアップ
====

ファイルを取得します

.. code-block:: cmdin

  cd ~/
  git clone https://github.com/nginxinc/kubernetes-ingress/
  cd ~/kubernetes-ingress
  git checkout v2.1.0


ライセンスファイルをコピーしてください
ファイルが配置されていない場合、トライアルを申請し証明書と鍵を取得してください

.. code-block:: cmdin
   
  cp ~/nginx-repo* .
  ls nginx-repo.*

2. コンテナイメージの作成
====

| NGINX Plus ＋ NGINX App Protectのコンテナイメージを作成します
| 参考： `Building the Ingress Controller Image <https://docs.nginx.com/nginx-ingress-controller/installation/building-ingress-controller-image>`__

.. code-block:: cmdin
  
  make debian-image-nap-dos-plus PREFIX=registry.example.com/root/nic/nginxplus-ingress-nap-dos TARGET=container TAG=2.1.0
  # Image の Build は数分(約5分)必要となります
  docker images | grep nginxplus-ingress-nap-dos

.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル

  registry.example.com/root/nic/nginxplus-ingress-nap-dos   2.1.0     5b5cdc61cf76   31 seconds ago   611MB


Container Image のPushのためにレジストリへログイン

.. code-block:: cmdin
  
  # registry.example.com にログイン
  docker login registry.example.com
  Username: root       << 左の文字列を入力
  Password: password   << 左の文字列を入力

.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル

  WARNING! Your password will be stored unencrypted in /home/ubuntu/.docker/config.json.
  Configure a credential helper to remove this warning. See
  https://docs.docker.com/engine/reference/commandline/login/#credentials-store

  Login Succeeded

Container Image のPush

.. code-block:: cmdin
  
  docker push registry.example.com/root/nic/nginxplus-ingress-nap-dos:2.1.0


3. NGINX Ingress Controller環境のセットアップ
====

先程の手順で取得したGitHubのフォルダへ移動し、必要となるリソースをデプロイします。

.. code-block:: cmdin
  
  cd ~/kubernetes-ingress/deployments
  kubectl apply -f common/ns-and-sa.yaml
  kubectl apply -f rbac/rbac.yaml
  kubectl apply -f rbac/ap-rbac.yaml
  kubectl apply -f rbac/apdos-rbac.yaml
  kubectl apply -f common/default-server-secret.yaml
  kubectl apply -f common/nginx-config.yaml
  kubectl apply -f common/ingress-class.yaml
  kubectl apply -f common/crds/k8s.nginx.org_virtualservers.yaml
  kubectl apply -f common/crds/k8s.nginx.org_virtualserverroutes.yaml
  kubectl apply -f common/crds/k8s.nginx.org_transportservers.yaml
  kubectl apply -f common/crds/k8s.nginx.org_policies.yaml
  kubectl apply -f common/crds/k8s.nginx.org_globalconfigurations.yaml
  kubectl apply -f common/crds/appprotect.f5.com_aplogconfs.yaml
  kubectl apply -f common/crds/appprotect.f5.com_appolicies.yaml
  kubectl apply -f common/crds/appprotect.f5.com_apusersigs.yaml
  kubectl apply -f common/crds/appprotectdos.f5.com_apdoslogconfs.yaml
  kubectl apply -f common/crds/appprotectdos.f5.com_apdospolicy.yaml
  kubectl apply -f common/crds/appprotectdos.f5.com_dosprotectedresources.yaml


4. NGINX App Protect Dosで利用するArbitratorを実行
====

Deploymentの内容を確認

``deployment/appprotect-dos-arb.yaml`` の内容を確認します。

.. code-block:: bash
  :linenos:
  :caption: deployment/appprotect-dos-arb.yaml

  apiVersion: apps/v1
  kind: Deployment
  metadata:
    name: appprotect-dos-arb
    namespace: nginx-ingress
  spec:
    replicas: 1
    selector:
      matchLabels:
        app: appprotect-dos-arb
    template:
      metadata:
        labels:
          app: appprotect-dos-arb
      spec:
        containers:
        - name: appprotect-dos-arb
          image: docker-registry.nginx.com/nap-dos/app_protect_dos_arb:1.1.0
          imagePullPolicy: IfNotPresent
          resources:
            limits:
              memory: "128Mi"
              cpu: "500m"
          ports:
            - containerPort: 3000
          securityContext:
            allowPrivilegeEscalation: false
            runAsUser: 1001
            capabilities:
              drop:
                - ALL

``service/appprotect-dos-arb-svc.yaml`` の内容を確認します。

.. code-block:: bash
  :linenos:
  :caption: service/appprotect-dos-arb-svc.yaml

  apiVersion: v1
  kind: Service
  metadata:
    name: svc-appprotect-dos-arb
    namespace: nginx-ingress
  spec:
    selector:
      app: appprotect-dos-arb
    ports:
      - name: arb
        port: 3000
        protocol: TCP
        targetPort: 3000

デプロイします。

.. code-block:: cmdin
  
  kubectl apply -f deployment/appprotect-dos-arb.yaml
  kubectl apply -f service/appprotect-dos-arb-svc.yaml


デプロイ結果を確認します。

.. code-block:: cmdin

  kubectl get deployment -n nginx-ingress

.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル

  NAME                 READY   UP-TO-DATE   AVAILABLE   AGE
  appprotect-dos-arb   1/1     1            1           4m32s

.. code-block:: cmdin
   
  kubectl get pod -n nginx-ingress

.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル

  NAME                                  READY   STATUS    RESTARTS   AGE
  appprotect-dos-arb-5d89486bbc-pkbrg   1/1     Running   0          4m43s

.. code-block:: cmdin
  
  kubectl get svc -n nginx-ingress

.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル

  NAME                     TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)    AGE
  svc-appprotect-dos-arb   ClusterIP   None         <none>        3000/TCP   6s


5. NGINX Ingress Controllerの実行
====

NGINX Ingress Controllerのpodを実行します。DeploymentとDaemonSetによる実行が可能ですが、のこの記事ではDeploymentで実行します。DaemonSetで実行したい場合にはマニュアルを参照して適切に読み替えて進めてください。

argsで指定するパラメータの詳細は `Command-line Arguments <https://docs.nginx.com/nginx-ingress-controller/configuration/global-configuration/command-line-arguments>`__ を参照してください

.. code-block:: cmdin
  
  ## cd ~/kubernetes-ingress/deployments
  vi deployment/nginx-plus-ingress.yaml

コメントを付与した行を適切な内容に修正してください

.. code-block:: yaml
  :linenos:
  :caption: deployment/nginx-plus-ingress.yaml
  :emphasize-lines: 5,13,14,20,21

  ** 省略 **
  spec:
     serviceAccountName: nginx-ingress
     containers:
     - image: registry.example.com/root/nic/nginxplus-ingress-nap-dos:2.1.0  # 対象のレジストリを指定してください
     imagePullPolicy: IfNotPresent
     name: nginx-plus-ingress
  ** 省略 **
     args:
        - -nginx-plus
        - -nginx-configmaps=$(POD_NAMESPACE)/nginx-config
        - -default-server-tls-secret=$(POD_NAMESPACE)/default-server-secret
        - -enable-app-protect                            # App Protect WAFを有効にします
        - -enable-app-protect-dos                        # App Protect DoSを利用する場合、有効にします
        #- -v=3 # Enables extensive logging. Useful for troubleshooting.
        #- -report-ingress-status
        #- -external-service=nginx-ingress
        #- -enable-prometheus-metrics
        #- -global-configuration=$(POD_NAMESPACE)/nginx-configuration
        - -enable-preview-policies                       # OIDCに必要となるArgsを有効にします
        - -enable-snippets                               # OIDCで一部設定を追加するためsnippetsを有効にします


修正したマニフェストを指定しPodを作成します。

.. code-block:: cmdin
   
  ## cd ~/kubernetes-ingress/deployments
  kubectl apply -f deployment/nginx-plus-ingress.yaml
  
.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル

  deployment.apps/nginx-ingress created

.. code-block:: cmdin
   
  kubectl get pods --namespace=nginx-ingress | grep nginx-ingress
   
.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル

  nginx-ingress-7f67968b56-d8gf5       1/1     Running   0          3s

.. code-block:: cmdin
   
  kubectl get deployment -n nginx-ingress | grep nginx-ingress

.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル

  nginx-ingress   1/1     1            1           2m52s


6. NGINX Ingress Controller を外部へ NodePort で公開する
====

本ラボの環境ではKubernetesへのアクセスを受けるため、NGINX Ingress Controllerを外部へNodePortで公開します。
以下コマンドで設定の内容を確認します。type NodePortでHTTP、HTTPSで待ち受ける設定であることを確認します。

.. code-block:: yaml
  :linenos:
  :caption: service/nodeport.yaml

  apiVersion: v1
  kind: Service
  metadata:
    name: nginx-ingress
    namespace: nginx-ingress
  spec:
    type: NodePort
    ports:
    - port: 80
      targetPort: 80
      protocol: TCP
      name: http
    - port: 443
      targetPort: 443
      protocol: TCP
      name: https
    selector:
      app: nginx-ingress



NodePortをデプロイします。

.. code-block:: cmdin
   
  ## cd ~/kubernetes-ingress/deployments
  kubectl apply -f service/nodeport.yaml

.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル

  service/nginx-ingress created

.. code-block:: cmdin
   
  kubectl get svc -n nginx-ingress | grep nginx-ingress

.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル

  nginx-ingress   NodePort   10.108.250.160   <none>        80:32692/TCP,443:31957/TCP   5s

このコマンドを実行した結果、Kubernetes の Worker Nodeでそれぞれのサービスに対しポートが割り当てられています。
図の内容を確認してください。

   .. image:: ./media/kube_nodeport.jpg
       :width: 400

| クライアントからアクセスするため、HTTP(TCP/80)、HTTPS(TCP/443)を待ち受け、それぞれNodePortで公開するポート番号へ転送するLBを用意します。
| 今回のラボ環境では同Linux Host上にNGINX Plusをインストールし以下nginx.confとしました。NGINX OSSでも同様の設定で問題ありません

.. NOTE::
   NGINX Plusをインストールする場合、こちらの手順「 `NGINX Plusのインストール (15min) <https://f5j-nginx-plus-lab1.readthedocs.io/en/latest/class1/module2/module2.html#nginx-plus-15min>`__」を参考に、NGINX Plusをインストールしてください。

先程確認したNoder Portで割り当てられたポート番号宛に通信を転送するように、NGINXを設定します。

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
        server node1:32692;    # HTTP(TCP/80)に割り当てられたポート番号
     }
     upstream tcp443_backend {
        server node1:31957;     # HTTPS(TCP/443)に割り当てられたポート番号
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

   
現在の状態は以下となり、サービスを外部に公開する準備が完了しました。

   .. image:: ./media/kube_external_nginx.jpg
       :width: 400