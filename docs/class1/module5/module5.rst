NICによるTCP/UDPの通信制御
####

この章では、実際にデプロイしたNGINX Ingress Controllerを使い、TCP/UDPのアプリケーションに対する通信制御方法を確認します
設定例は `NGINX Inc GitHubの examples/custom-resources/ <https://github.com/nginxinc/kubernetes-ingress/tree/v3.1.1/examples/custom-resources>`__ に管理されております

TCP / UDP の分散設定
====

TCP / UDP を分散するためアプリケーションをデプロイします

https://github.com/nginxinc/kubernetes-ingress/tree/v3.1.1/examples/custom-resources/basic-tcp-udp


Deployment、NodePortの内容を修正しデプロイ
----

TCP / UDP を分散するため、NGINX Ingress Controller のDeployment、NodePortの内容を修正しデプロイします。
Deploymentのargsで指定するパラメータの詳細は [Command-line Arguments](https://docs.nginx.com/nginx-ingress-controller/configuration/global-configuration/command-line-arguments)を参照してください

.. code-block:: cmdin
  
  cd ~/kubernetes-ingress/deployments
  cp deployment/nginx-plus-ingress.yaml deployment/nginx-plus-ingress-tcpudp.yaml
  vi deployment/nginx-plus-ingress-tcpudp.yaml

コメントを付与した行を参考に適切な内容に修正してください

.. code-block:: yaml
  :linenos:
  :caption: deployment/nginx-plus-ingress-tcpudp.yaml
  :emphasize-lines: 9,10,11,12,13,14,15,16,17,29
 
  ** 省略 **
  spec:
     serviceAccountName: nginx-ingress
     containers:
     - image: registry.example.com/root/nic/nginxplus-ingress-nap-dos:2.1.0  
     imagePullPolicy: IfNotPresent
     name: nginx-plus-ingress
     ports:
     - name: dns-tcp         # DNS TCP を有効にします
       containerPort: 5353   
     - name: dns-udp         # DNS UDP を有効にします
       containerPort: 53 
       protocol: UDP         
     #- name: http
     #  containerPort: 80
     #- name: https
     #  containerPort: 443
  ** 省略 **
      args:
        - -nginx-plus
        - -nginx-configmaps=$(POD_NAMESPACE)/nginx-config
        - -default-server-tls-secret=$(POD_NAMESPACE)/default-server-secret
       #- -enable-app-protect
       #- -enable-app-protect-dos
       #- -v=3 # Enables extensive logging. Useful for troubleshooting.
       #- -report-ingress-status
       #- -external-service=nginx-ingress
       #- -enable-prometheus-metrics
        - -global-configuration=$(POD_NAMESPACE)/nginx-configuration # Global Configuration を有効にします

単一のNodePortでTCP/UDP双方を設定できないため、2つのファイルに分けて設定します。

.. code-block:: cmdin

  ## cd ~/kubernetes-ingress/deployments
  cp service/nodeport.yaml  service/nodeport-tcp.yaml
  vi service/nodeport-tcp.yaml

以下の内容を参考に修正してください。

.. code-block:: yaml
  :linenos:
  :caption: service/nodeport-tcp.yaml
  :emphasize-lines: 4,9,10,11,12,13

  apiVersion: v1
  kind: Service
  metadata:
    name: nginx-ingress
    namespace: nginx-ingress
  spec:
    type: NodePort
    ports:
    - port: 5353        # DNS TCP を有効にします (ファイルからDNS UDPを削除します)
      targetPort: 5353
      protocol: TCP
      name: dns-tcp
    selector:
      app: nginx-ingress

.. code-block:: cmdin

  ## cd ~/kubernetes-ingress/deployments
  cp service/nodeport.yaml  service/nodeport-udp.yaml
  vi service/nodeport-udp.yaml

以下の内容を参考に修正してください。

.. code-block:: yaml
  :linenos:
  :caption: service/nodeport-udp.yaml
  :emphasize-lines: 4,9,10,11,12,13

  apiVersion: v1
  kind: Service
  metadata:
    name: nginx-ingress-udp
    namespace: nginx-ingress
  spec:
    type: NodePort
    ports:
    - port: 53        # DNS UDP のポート番号を変更します (ファイルからDNS TCPを削除します)
      targetPort: 53
      protocol: UDP
      name: dns-udp
    selector:
      app: nginx-ingress

TCP/UDPではGlobal Configurationにより外部から通信を待ち受ける設定を行います

.. code-block:: cmdin
 
  cd ~/kubernetes-ingress/examples/custom-resources/basic-tcp-udp/
  cp global-configuration.yaml  global-configuration.yaml-bak
  vi global-configuration.yaml

以下の内容を参考に修正してください。

.. code-block:: yaml
  :linenos:
  :caption: global-configuration.yaml

  apiVersion: k8s.nginx.org/v1alpha1
  kind: GlobalConfiguration
  metadata:
    name: nginx-configuration
    namespace: nginx-ingress
  spec:
    listeners:
    - name: dns-udp
      port: 53       # DNS UDP のポート番号を 5353 -> 53 に変更
      protocol: UDP
    - name: dns-tcp
      port: 5353
      protocol: TCP


その他、NGINX Ingress ControlleでTCP/UDPを転送するための設定、サンプルアプリケーションを確認します。

TCPの設定です。Kindで ``TransportServer`` を指定します。

.. code-block:: yaml
  :linenos:
  :caption: transport-server-tcp.yaml
  :emphasize-lines: 2

  apiVersion: k8s.nginx.org/v1alpha1
  kind: TransportServer
  metadata:
    name: dns-tcp
  spec:
    listener:
      name: dns-tcp
      protocol: TCP
    upstreams:
    - name: dns-app
      service: coredns
      port: 5353
    action:
      pass: dns-app

UDPの設定です。TCPとほぼ同様です

.. code-block:: yaml
  :linenos:
  :caption: transport-server-udp.yaml
  :emphasize-lines: 2

  apiVersion: k8s.nginx.org/v1alpha1
  kind: TransportServer
  metadata:
    name: dns-udp
  spec:
    listener:
      name: dns-udp
      protocol: UDP
    upstreams:
    - name: dns-app
      service: coredns
      port: 5353
    upstreamParameters:
      udpRequests: 1
      udpResponses: 1
    action:
      pass: dns-app


サンプルアプリケーションです。TCP/UDP共に5353で通信を待ち受けます。またDNSを 8.8.8.8:53 へと転送する動作となります。


.. code-block:: yaml
  :linenos:
  :caption: dns.yaml
  :emphasize-lines: 6,7,8,9,10,

  apiVersion: v1
  kind: ConfigMap
  metadata:
    name: coredns
  data:
    Corefile: |
      .:5353 {
        forward . 8.8.8.8:53
        log
      }
  ---
  apiVersion: apps/v1
  kind: Deployment
  metadata:
    name: coredns
  spec:
    replicas: 2
    selector:
      matchLabels:
        app: coredns
    template:
      metadata:
        labels:
          app: coredns
      spec:
        containers:
        - name: coredns
          image: coredns/coredns:1.8.6
          args: [ "-conf", "/etc/coredns/Corefile" ]
          volumeMounts:
          - name: config-volume
            mountPath: /etc/coredns
            readOnly: true
          ports:
          - containerPort: 5353
            name: dns
            protocol: UDP
          - containerPort: 5353
            name: dns-tcp
            protocol: TCP
          securityContext:
            readOnlyRootFilesystem: true
        volumes:
          - name: config-volume
            configMap:
              name: coredns
              items:
              - key: Corefile
                path: Corefile
  ---
  apiVersion: v1
  kind: Service
  metadata:
    name: coredns
  spec:
    selector:
     app: coredns
    ports:
    - name: dns
      port: 5353
      protocol: UDP
    - name: dns-tcp
      port: 5353
      protocol: TCP

修正した内容、各種リソースをデプロイします。
デプロイする内容に応じてディレクトリが変更となりますので注意ください

``~/kubernetes-ingress/deployments`` 配下の内容を設定してください。(NICのDeployment, NodePort)

.. code-block:: cmdin
  
  cd ~/kubernetes-ingress/deployments
  kubectl apply -f service/nodeport-tcp.yaml
  kubectl apply -f service/nodeport-udp.yaml
  kubectl apply -f deployment/nginx-plus-ingress-tcpudp.yaml

``~/kubernetes-ingress/examples/custom-resources/basic-tcp-udp`` 配下の内容を設定してください。(サンプルアプリケーション、分散設定等)

.. code-block:: cmdin

  cd ~/kubernetes-ingress/examples/custom-resources/basic-tcp-udp
  kubectl apply -f global-configuration.yaml
  kubectl apply -f dns.yaml
  kubectl apply -f transport-server-tcp.yaml
  kubectl apply -f transport-server-udp.yaml


デプロイしたNodePortの内容を確認し、TCP/UDPの待ち受けているポートに対してリクエストを転送するようNGINXの設定を変更します。

.. code-block:: cmdin

  kubectl get svc -n nginx-ingress

.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル
  :emphasize-lines: 2,3

  NAME                     TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)          AGE
  nginx-ingress            NodePort    10.108.250.160   <none>        5353:30292/TCP   6d14h
  nginx-ingress-udp        NodePort    10.99.147.245    <none>        53:31482/UDP     16m

確認したNode Portで割り当てられたポート番号宛に通信を転送するように、NGINXを設定します。
転送するポート番号は、環境に合わせて適切に変更してください。

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
     upstream tcp5353_backend {
        server node1:30292;    # DNS TCP(TCP/5353)に割り当てられたポート番号
     }
     upstream udp53_backend {
        server node1:31482;     # DNS UDP(UDP/53)に割り当てられたポート番号
     }

     server {
        listen 5353;            # DNS TCPを(TCP/5353)で待ち受けます
        proxy_pass tcp5353_backend;
     }
     server {
        listen 53 udp;          # DNS UDPを(UDP/53)で待ち受けます
        proxy_pass udp53_backend;
     }
  }
  EOF
  sudo cp nginx.conf /etc/nginx/nginx.conf
  sudo nginx -s reload


デプロイした結果の確認
----

.. code-block:: cmdin

  kubectl get pod -n nginx-ingress | grep nginx-ingress

.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル

  nginx-ingress-68949d7f46-qh9kp        1/1     Running   0             4s

NGINX Ingress ControllerのPodの詳細を確認します


.. code-block:: cmdin

  ## pod名は、kuebctl get pod -n nginx-ingress の出力結果を参照してください
  kubectl describe pod nginx-ingress-68949d7f46-qh9kp -n nginx-ingress

.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル
  :emphasize-lines: 8,14

  ** 省略 **

  Containers:
    nginx-plus-ingress:
      Container ID:  docker://f1aa9d4434ebda4817f9ff957987120c85808bcdcb9d64978e76814c20e422fe
      Image:         registry.example.com/root/nic/nginxplus-ingress-nap-dos:2.1.0
      Image ID:      docker-pullable://registry.example.com/root/nic/nginxplus-ingress-nap-dos@sha256:8c9a8ce1cdda45c2a289cb20ce37a60e25b4d4669c2c9c9d4d0831c353c6c668
      Ports:         5353/TCP, 53/UDP, 8081/TCP, 9113/TCP
      Host Ports:    0/TCP, 0/UDP, 0/TCP, 0/TCP
      Args:
        -nginx-plus
        -nginx-configmaps=$(POD_NAMESPACE)/nginx-config
        -default-server-tls-secret=$(POD_NAMESPACE)/default-server-secret
        -global-configuration=$(POD_NAMESPACE)/nginx-configuration
      State:          Running

  ** 省略 **


TCPは ``5353/TCP`` 、UDP ``53/UDP`` となっていることが確認できます。また ``global-configuration`` のargsが正しく記載されています

.. code-block:: cmdin

  kubectl get globalconfiguration -n nginx-ingress

.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル

  NAME                  AGE
  nginx-configuration   8m3s


.. code-block:: cmdin

  kubectl get svc coredns

.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル

  NAME      TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)             AGE
  coredns   ClusterIP   10.101.115.180   <none>        5353/UDP,5353/TCP   2m41s

.. code-block:: cmdin

  kubectl get pod | grep coredns

.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル

  coredns-75466b69b7-8b7r5   1/1     Running   0              3m29s
  coredns-75466b69b7-8tgrc   1/1     Running   0              3m29s

先程確認したとおり、TCP/UDPそれぞれのNodePortの内容が確認できます

.. code-block:: cmdin

  kubectl get svc -n nginx-ingress

.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル

  NAME                     TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)          AGE
  nginx-ingress            NodePort    10.108.250.160   <none>        5353:30292/TCP   6d14h
  nginx-ingress-udp        NodePort    10.99.147.245    <none>        53:31482/UDP     16m


動作確認
----

TCP/UDPのそれぞれで ``kubernetes.io`` の名前解決を行います。NGINX Ingress ControllerへQueryを送信します。

TCPでDNS Queryを送信します。Portは ``5353`` です。

.. code-block:: cmdin

  dig node1 -p 5353  kubernetes.io +tcp

.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル
  :emphasize-lines: 30

  ; <<>> DiG 9.16.1-Ubuntu <<>> node1 -p 5353 kubernetes.io +tcp
  ;; global options: +cmd
  ;; connection timed out; no servers could be reached
  
  ;; Got answer:
  ;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 28171
  ;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1
  
  ;; OPT PSEUDOSECTION:
  ; EDNS: version: 0, flags:; udp: 4096
  ;; QUESTION SECTION:
  ;kubernetes.io.                 IN      A
  
  ;; ANSWER SECTION:
  kubernetes.io.          2474    IN      A       147.75.40.148
  
  ;; Query time: 23 msec
  ;; SERVER: 127.0.0.53#5353(127.0.0.53)
  ;; WHEN: Fri Jan 21 06:46:38 UTC 2022
  ;; MSG SIZE  rcvd: 71
  
  .. code-block:: cmdin


UDPでDNS Queryを送信します。Portは ``53`` です。

.. code-block:: cmdin

  dig node1 -p 53  kubernetes.io

.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル
  :emphasize-lines: 30

  ; <<>> DiG 9.16.1-Ubuntu <<>> node1 -p 53 kubernetes.io
  ;; global options: +cmd
  ;; Got answer:
  ;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 2308
  ;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1
  
  ;; OPT PSEUDOSECTION:
  ; EDNS: version: 0, flags:; udp: 65494
  ;; QUESTION SECTION:
  ;node1.                         IN      A
  
  ;; ANSWER SECTION:
  node1.                  0       IN      A       10.1.1.9
  
  ;; Query time: 3 msec
  ;; SERVER: 127.0.0.53#53(127.0.0.53)
  ;; WHEN: Fri Jan 21 06:46:48 UTC 2022
  ;; MSG SIZE  rcvd: 50
  
  ;; Got answer:
  ;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 22877
  ;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1
  
  ;; OPT PSEUDOSECTION:
  ; EDNS: version: 0, flags:; udp: 65494
  ;; QUESTION SECTION:
  ;kubernetes.io.                 IN      A
  
  ;; ANSWER SECTION:
  kubernetes.io.          210     IN      A       147.75.40.148
  
  ;; Query time: 0 msec
  ;; SERVER: 127.0.0.53#53(127.0.0.53)
  ;; WHEN: Fri Jan 21 06:46:48 UTC 2022
  ;; MSG SIZE  rcvd: 58



リソースの削除
----

サンプルアプリケーション、及び分散設定を削除します

.. code-block:: cmdin

  cd ~/kubernetes-ingress/examples/custom-resources/basic-tcp-udp
  kubectl delete -f global-configuration.yaml
  kubectl delete -f dns.yaml
  kubectl delete -f transport-server-tcp.yaml
  kubectl delete -f transport-server-udp.yaml

HTTP/HTTPSを待ち受ける設定に戻す場合、以下の操作を参考に実施してください。

| NGINXの設定については、再度デプロイの後、待受のポート番号を確認して適切にnginx.confを変更してください。
| 手順は以下を参照してください。
| `5. NGINX Ingress Controller を外部へ NodePort で公開する <https://f5j-nginx-ingress-controller-lab1.readthedocs.io/en/latest/class1/module2/module2.html#nginx-ingress-controller-nodeport>`__

.. code-block:: cmdin
  
  cd ~/kubernetes-ingress/deployments
  kubectl delete -f service/nodeport-tcp.yaml
  kubectl delete -f service/nodeport-udp.yaml
  kubectl delete -f deployment/nginx-plus-ingress-tcpudp.yaml

  kubectl apply -f service/nodeport.yaml
  kubectl apply -f deployment/nginx-plus-ingress.yaml
  ## 手順を参考にnginx.confを変更してください