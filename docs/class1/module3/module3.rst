NICによるWebアプリの通信制御
####

この章では、実際にデプロイしたNGINX Ingress Controllerを使い、様々なサンプルアプリケーションを動作させ、その設定方法や動きを確認いただきます。
設定例は `NGINX Inc GitHubの examples/custom-resources/ <https://github.com/nginxinc/kubernetes-ingress/tree/v2.1.0/examples/custom-resources>`__ に管理されております

シンプルなWebアプリケーションのデプロイ
====

https://github.com/nginxinc/kubernetes-ingress/tree/v2.1.0/examples/custom-resources/basic-configuration

#. サンプルアプリケーション、NGINX Ingress Controller の設定をデプロイ

.. code-block:: cmdin
 
  cd ~/kubernetes-ingress/examples/custom-resources/basic-configuration/
  kubectl create -f cafe.yaml
  kubectl create -f cafe-secret.yaml
  kubectl create -f cafe-virtual-server.yaml

#. 作成したリソースを確認

.. code-block:: cmdin
 
  ## cd ~/kubernetes-ingress/examples/custom-resources/basic-configuration/
  kubectl get deployment

.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル

  NAME     READY   UP-TO-DATE   AVAILABLE   AGE
  coffee   2/2     2            2           1m
  tea      1/1     1            1           1m

.. code-block:: cmdin
 
  kubectl get secret  | grep cafe-secret

.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル

  cafe-secret           kubernetes.io/tls                     2      1m

.. code-block:: cmdin
 
  kubectl get vs
  
.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル

  NAME   STATE   HOST               IP    PORTS   AGE
  cafe   Valid   cafe.example.com                 94s


#. 動作確認

.. code-block:: cmdin
 
  curl -H "Host:cafe.example.com" http://localhost/coffee

.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル
   
  Server address: 192.168.127.25:8080
  Server name: coffee-7c86d7d67c-wjxss
  Date: 17/Jan/2022:00:14:03 +0000
  URI: /coffee
  Request ID: 069567120c306da6f92e16e5d73e5040

.. code-block:: cmdin
 
  curl -H "Host:cafe.example.com" http://localhost/tea

.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル

  Server address: 192.168.127.20:8080
  Server name: tea-5c457db9-dc4cs
  Date: 17/Jan/2022:00:14:08 +0000
  URI: /tea
  Request ID: 6fd58877d9e85903300df7ceb0f81eb2

.. code-block:: cmdin
 
  curl -kv -H "Host:cafe.example.com" https://localhost/coffee

.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル

  *   Trying 127.0.0.1:443...
  * TCP_NODELAY set
  * Connected to localhost (127.0.0.1) port 443 (#0)
  * ALPN, offering h2
  * ALPN, offering http/1.1
  * successfully set certificate verify locations:
  *   CAfile: /etc/ssl/certs/ca-certificates.crt
    CApath: /etc/ssl/certs
  * TLSv1.3 (OUT), TLS handshake, Client hello (1):
  * TLSv1.3 (IN), TLS handshake, Server hello (2):
  * TLSv1.2 (IN), TLS handshake, Certificate (11):
  * TLSv1.2 (IN), TLS handshake, Server key exchange (12):
  * TLSv1.2 (IN), TLS handshake, Server finished (14):
  * TLSv1.2 (OUT), TLS handshake, Client key exchange (16):
  * TLSv1.2 (OUT), TLS change cipher, Change cipher spec (1):
  * TLSv1.2 (OUT), TLS handshake, Finished (20):
  * TLSv1.2 (IN), TLS handshake, Finished (20):
  * SSL connection using TLSv1.2 / ECDHE-RSA-AES256-GCM-SHA384
  * ALPN, server accepted to use http/1.1
  * Server certificate:
  *  subject: CN=NGINXIngressController
  *  start date: Sep 12 18:03:35 2018 GMT
  *  expire date: Sep 11 18:03:35 2023 GMT
  *  issuer: CN=NGINXIngressController
  *  SSL certificate verify result: self signed certificate (18), continuing anyway.
  > GET /coffee HTTP/1.1
  > Host:cafe.example.com
  > User-Agent: curl/7.68.0
  > Accept: */*
  >
  * Mark bundle as not supporting multiuse
  < HTTP/1.1 200 OK
  < Server: nginx/1.21.3
  < Date: Mon, 17 Jan 2022 00:14:34 GMT
  < Content-Type: text/plain
  < Content-Length: 164
  < Connection: keep-alive
  < Expires: Mon, 17 Jan 2022 00:14:33 GMT
  < Cache-Control: no-cache
  <
  Server address: 192.168.127.26:8080
  Server name: coffee-7c86d7d67c-8jm9z
  Date: 17/Jan/2022:00:14:34 +0000
  URI: /coffee
  Request ID: 3af5bd62d9756c934b4c731d0cadfcb1
  * Connection #0 to host localhost left intact

.. code-block:: cmdin
 
  curl -kv -H "Host:cafe.example.com" https://localhost/tea

.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル

  *   Trying 127.0.0.1:443...
  * TCP_NODELAY set
  * Connected to localhost (127.0.0.1) port 443 (#0)
  * ALPN, offering h2
  * ALPN, offering http/1.1
  * successfully set certificate verify locations:
  *   CAfile: /etc/ssl/certs/ca-certificates.crt
    CApath: /etc/ssl/certs
  * TLSv1.3 (OUT), TLS handshake, Client hello (1):
  * TLSv1.3 (IN), TLS handshake, Server hello (2):
  * TLSv1.2 (IN), TLS handshake, Certificate (11):
  * TLSv1.2 (IN), TLS handshake, Server key exchange (12):
  * TLSv1.2 (IN), TLS handshake, Server finished (14):
  * TLSv1.2 (OUT), TLS handshake, Client key exchange (16):
  * TLSv1.2 (OUT), TLS change cipher, Change cipher spec (1):
  * TLSv1.2 (OUT), TLS handshake, Finished (20):
  * TLSv1.2 (IN), TLS handshake, Finished (20):
  * SSL connection using TLSv1.2 / ECDHE-RSA-AES256-GCM-SHA384
  * ALPN, server accepted to use http/1.1
  * Server certificate:
  *  subject: CN=NGINXIngressController
  *  start date: Sep 12 18:03:35 2018 GMT
  *  expire date: Sep 11 18:03:35 2023 GMT
  *  issuer: CN=NGINXIngressController
  *  SSL certificate verify result: self signed certificate (18), continuing anyway.
  > GET /tea HTTP/1.1
  > Host:cafe.example.com
  > User-Agent: curl/7.68.0
  > Accept: */*
  >
  * Mark bundle as not supporting multiuse
  < HTTP/1.1 200 OK
  < Server: nginx/1.21.3
  < Date: Mon, 17 Jan 2022 00:14:39 GMT
  < Content-Type: text/plain
  < Content-Length: 156
  < Connection: keep-alive
  < Expires: Mon, 17 Jan 2022 00:14:38 GMT
  < Cache-Control: no-cache
  <
  Server address: 192.168.127.20:8080
  Server name: tea-5c457db9-dc4cs
  Date: 17/Jan/2022:00:14:39 +0000
  URI: /tea
  Request ID: af1466d1fc1b7481cb82352885f9cbc2


#. リソースの削除

.. code-block:: cmdin
 
  ## cd ~/kubernetes-ingress/examples/custom-resources/basic-configuration/
  kubectl delete -f cafe-secret.yaml
  kubectl delete -f cafe-virtual-server.yaml
  kubectl delete -f cafe.yaml


複数アプリケーション・チームを想定した VS / VSR 設定
====

https://github.com/nginxinc/kubernetes-ingress/tree/v2.1.0/examples/custom-resources/cross-namespace-configuration

この章ではシンプルなWebアプリケーションをデプロイします。
NGINXはCRDを用い、Virtual Server / Virtual Server Router / Policy といったリソースを使うことで、権限と設定範囲を適切に管理することが可能です。

#. サンプルアプリケーションをデプロイ

.. code-block:: cmdin
 
  kubectl create -f namespaces.yaml
  kubectl create -f tea.yaml
  kubectl create -f coffee.yaml
  kubectl create -f tea-virtual-server-route.yaml
  kubectl create -f coffee-virtual-server-route.yaml
  kubectl create -f cafe-secret.yaml
  kubectl create -f cafe-virtual-server.yaml

#. リソースを確認

.. code-block:: cmdin
    
  kubectl get ns --sort-by=.metadata.creationTimestamp
  
.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル

  NAME               STATUS   AGE
  kube-public        Active   10d
  kube-system        Active   10d
  kube-node-lease    Active   10d
  default            Active   10d
  tigera-operator    Active   10d
  calico-system      Active   10d
  calico-apiserver   Active   10d
  nginx-ingress      Active   2d18h
  coffee             Active   75s
  cafe               Active   75s
  tea                Active   75s
  
.. code-block:: cmdin
 
  kubectl get vsr -A
  
.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル

  NAMESPACE   NAME     STATE   HOST               IP    PORTS   AGE
  coffee      coffee   Valid   cafe.example.com                 89s
  tea         tea      Valid   cafe.example.com                 93s
  
.. code-block:: cmdin
 
  kubectl get vs -A
  
.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル

  NAMESPACE   NAME   STATE   HOST               IP    PORTS   AGE
  cafe        cafe   Valid   cafe.example.com                 85s
  
.. code-block:: cmdin
 
  kubectl get secret -A | grep cafe
  
.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル

  cafe               cafe-secret                                      kubernetes.io/tls                     2      101s
  cafe               default-token-94nrl                              kubernetes.io/service-account-token   3      2m3s
  
.. code-block:: cmdin
 
  kubectl get secret -A | grep cafe-secret
  
.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル

  NAME                  TYPE                                  DATA   AGE
  cafe-secret           kubernetes.io/tls                     2      2m5s
  
.. code-block:: cmdin
 
  kubectl get pod -o wide -A|grep -e coffee -e tea
  
.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル

  coffee             coffee-7c86d7d67c-pq5w2                    1/1     Running   0                88s   192.168.127.22   ip-10-1-1-9   <none>           <none>
  tea                tea-5c457db9-h5sm9                         1/1     Running   0                14m   192.168.127.24   ip-10-1-1-9   <none>           <none>


動作確認

.. code-block:: cmdin
   
  curl -H "Host: cafe.example.com" http://localhost/coffee
  
.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル

  Server address: 192.168.127.22:8080
  Server name: coffee-7c86d7d67c-pq5w2
  Date: 17/Jan/2022:05:44:25 +0000
  URI: /coffee
  Request ID: 1414627aac091b5a7897bac37d046cea
  
.. code-block:: cmdin
 
  curl -H "Host: cafe.example.com" http://localhost/tea
  
.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル

  Server address: 192.168.127.24:8080
  Server name: tea-5c457db9-h5sm9
  Date: 17/Jan/2022:05:44:29 +0000
  URI: /tea
  Request ID: 698ab29da633f24a9bf5384c1499b056
  
.. code-block:: cmdin
 
  curl -vk -H "Host: cafe.example.com" https://localhost/tea
  
.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル

  *   Trying 127.0.0.1:443...
  * TCP_NODELAY set
  * Connected to localhost (127.0.0.1) port 443 (#0)
  * ALPN, offering h2
  * ALPN, offering http/1.1
  * successfully set certificate verify locations:
  *   CAfile: /etc/ssl/certs/ca-certificates.crt
    CApath: /etc/ssl/certs
  * TLSv1.3 (OUT), TLS handshake, Client hello (1):
  * TLSv1.3 (IN), TLS handshake, Server hello (2):
  * TLSv1.2 (IN), TLS handshake, Certificate (11):
  * TLSv1.2 (IN), TLS handshake, Server key exchange (12):
  * TLSv1.2 (IN), TLS handshake, Server finished (14):
  * TLSv1.2 (OUT), TLS handshake, Client key exchange (16):
  * TLSv1.2 (OUT), TLS change cipher, Change cipher spec (1):
  * TLSv1.2 (OUT), TLS handshake, Finished (20):
  * TLSv1.2 (IN), TLS handshake, Finished (20):
  * SSL connection using TLSv1.2 / ECDHE-RSA-AES256-GCM-SHA384
  * ALPN, server accepted to use http/1.1
  * Server certificate:
  *  subject: CN=NGINXIngressController
  *  start date: Sep 12 18:03:35 2018 GMT
  *  expire date: Sep 11 18:03:35 2023 GMT
  *  issuer: CN=NGINXIngressController
  *  SSL certificate verify result: self signed certificate (18), continuing anyway.
  > GET /tea HTTP/1.1
  > Host: cafe.example.com
  > User-Agent: curl/7.68.0
  > Accept: */*
  >
  * Mark bundle as not supporting multiuse
  < HTTP/1.1 200 OK
  < Server: nginx/1.21.3
  < Date: Mon, 17 Jan 2022 05:44:42 GMT
  < Content-Type: text/plain
  < Content-Length: 156
  < Connection: keep-alive
  < Expires: Mon, 17 Jan 2022 05:44:41 GMT
  < Cache-Control: no-cache
  <
  Server address: 192.168.127.24:8080
  Server name: tea-5c457db9-h5sm9
  Date: 17/Jan/2022:05:44:42 +0000
  URI: /tea
  Request ID: 8ec25fd33d381df7261fda9f9da66558
  * Connection #0 to host localhost left intact


リソースの削除

.. code-block:: cmdin
 
  kubectl delete -f tea-virtual-server-route.yaml
  kubectl delete -f cafe-virtual-server.yaml
  kubectl delete -f coffee-virtual-server-route.yaml
  kubectl delete -f cafe-secret.yaml
  kubectl delete -f tea.yaml
  kubectl delete -f coffee.yaml
  kubectl delete -f namespaces.yaml


通信内容による条件分岐・サービスへの転送
====

https://github.com/nginxinc/kubernetes-ingress/tree/v2.1.0/examples/custom-resources/advanced-routing

サンプルアプリケーションをデプロイ

.. code-block:: cmdin
 
  cd ~/kubernetes-ingress/examples/custom-resources/advanced-routing
  kubectl create -f cafe.yaml
  kubectl create -f cafe-virtual-server.yaml

リソースを確認

.. code-block:: cmdin
 
  kubectl get deployment

.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル

  NAME        READY   UP-TO-DATE   AVAILABLE   AGE
  coffee-v1   1/1     1            1           16s
  coffee-v2   1/1     1            1           15s
  tea         1/1     1            1           15s
  tea-post    1/1     1            1           15s

.. code-block:: cmdin
 
  kubectl get pod -o wide

.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル

  NAME                         READY   STATUS    RESTARTS   AGE   IP               NODE          NOMINATED NODE   READINESS GATES
  coffee-v1-6b78998db9-8cv49   1/1     Running   0          26s   192.168.127.23   ip-10-1-1-9   <none>           <none>
  coffee-v2-748cbbb49f-mbxpr   1/1     Running   0          26s   192.168.127.27   ip-10-1-1-9   <none>           <none>
  tea-5c457db9-dcswc           1/1     Running   0          26s   192.168.127.33   ip-10-1-1-9   <none>           <none>
  tea-post-7db8cd8bf-m5gbz     1/1     Running   0          26s   192.168.127.32   ip-10-1-1-9   <none>           <none>

.. code-block:: cmdin
 
  kubectl get vs

.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル

  NAME   STATE   HOST               IP    PORTS   AGE
  cafe   Valid   cafe.example.com                 28s



動作確認

.. code-block:: cmdin
 
  curl -H "Host: cafe.example.com" http://localhost/tea

.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル

  Server address: 192.168.127.33:8080
  Server name: tea-5c457db9-dcswc
  Date: 17/Jan/2022:09:00:56 +0000
  URI: /tea
  Request ID: 00e9eb4d61f7afdb8c5656da94d15b98

.. code-block:: cmdin
 
  curl -H "Host: cafe.example.com" http://localhost/tea -X POST

.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル

  Server address: 192.168.127.32:8080
  Server name: tea-post-7db8cd8bf-m5gbz
  Date: 17/Jan/2022:09:01:02 +0000
  URI: /tea
  Request ID: 4deeb82434a6f799ffc894a229ac361a

.. code-block:: cmdin
 
  curl -H "Host: cafe.example.com" http://localhost/coffee

.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル

  Server address: 192.168.127.23:8080
  Server name: coffee-v1-6b78998db9-8cv49
  Date: 17/Jan/2022:09:01:25 +0000
  URI: /coffee
  Request ID: 8d182c9c060d5a4d4dec226292ac2820

.. code-block:: cmdin
 
  curl -H "Host: cafe.example.com" http://localhost/coffee --cookie "version=v2"

.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル

  Server address: 192.168.127.27:8080
  Server name: coffee-v2-748cbbb49f-mbxpr
  Date: 17/Jan/2022:09:01:35 +0000
  URI: /coffee
  Request ID: befacc5e7ca56a1a09e5982315c74fa0

リソースの削除

.. code-block:: cmdin
 
  kubectl delete  -f cafe-virtual-server.yaml
  kubectl delete  -f cafe.yaml


割合を指定した分散 (Traffic Split)
====

https://github.com/nginxinc/kubernetes-ingress/tree/v2.1.0/examples/custom-resources/traffic-splitting

サンプルアプリケーションをデプロイ

.. code-block:: cmdin
 
  cd ~/kubernetes-ingress/examples/custom-resources/traffic-splitting    
  kubectl create -f cafe.yaml
  kubectl create -f cafe-virtual-server.yaml


Virtual Serverの内容を確認

.. code-block:: yaml
   :linenos:
   :caption: cafe-virtual-server.yaml
   :name: cafe-virtual-server.yaml
    
  apiVersion: k8s.nginx.org/v1
  kind: VirtualServer
  metadata:
    name: cafe
  spec:
    host: cafe.example.com
    upstreams:
    - name: coffee-v1
      service: coffee-v1-svc
      port: 80
    - name: coffee-v2
      service: coffee-v2-svc
      port: 80
    routes:
    - path: /coffee
      splits:
      - weight: 90
        action:
          pass: coffee-v1
      - weight: 10
        action:
          pass: coffee-v2


リソースを確認

.. code-block:: cmdin
 
  kubectl get deployment
  
.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル

  NAME        READY   UP-TO-DATE   AVAILABLE   AGE
  coffee-v1   2/2     2            2           19s
  coffee-v2   2/2     2            2           19s
  
.. code-block:: cmdin
 
  kubectl get pod -o wide
  
.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル

  NAME                         READY   STATUS    RESTARTS   AGE   IP               NODE          NOMINATED NODE   READINESS GATES
  coffee-v1-6b78998db9-h4jkb   1/1     Running   0          25s   192.168.127.47   ip-10-1-1-9   <none>           <none>
  coffee-v1-6b78998db9-nn42z   1/1     Running   0          25s   192.168.127.44   ip-10-1-1-9   <none>           <none>
  coffee-v2-748cbbb49f-llpb6   1/1     Running   0          25s   192.168.127.45   ip-10-1-1-9   <none>           <none>
  coffee-v2-748cbbb49f-vrpzx   1/1     Running   0          25s   192.168.127.46   ip-10-1-1-9   <none>           <none>
  
.. code-block:: cmdin
 
  kubectl get vs
  
.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル

  NAME   STATE   HOST               IP    PORTS   AGE
  cafe   Valid   cafe.example.com                 26s


動作確認

.. code-block:: cmdin
 
  curl -s -H "Host: cafe.example.com" http://localhost/coffee
  
.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル

  Server address: 192.168.127.44:8080
  Server name: coffee-v1-6b78998db9-nn42z
  Date: 17/Jan/2022:12:26:49 +0000
  URI: /coffee
  Request ID: c127f0f724eb1b3becd57603b6d603ea

.. code-block:: cmdin
 
  curl -s -H "Host: cafe.example.com" http://localhost/coffee
  
.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル

  Server address: 192.168.127.45:8080
  Server name: coffee-v2-748cbbb49f-llpb6
  Date: 17/Jan/2022:12:26:37 +0000
  URI: /coffee
  Request ID: 357237a3fea498b6efd90c929d526e64


.. code-block:: cmdin
 
  ## cd ~/kubernetes-ingress/examples/custom-resources/traffic-splitting
  > split.txt ;\
  for i in {1..20}; \
  do curl -s -H "Host: cafe.example.com" http://localhost/coffee | grep "Server name" >> split.txt ; \
  done ; \
  echo -n "v1:" ; grep v1 split.txt  | wc -l ; echo -n "v2:"  ; grep v2 split.txt  | wc -l
  
.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル

  v1:18
  v2:2


リソースの削除

.. code-block:: cmdin
 
  ## cd ~/kubernetes-ingress/examples/custom-resources/traffic-splitting
  kubectl delete -f cafe-virtual-server.yaml
  kubectl delete -f cafe.yaml
  rm split.txt



IPアドレスによる通信の制御 (Access Control)
====

https://github.com/nginxinc/kubernetes-ingress/tree/v2.1.0/examples/custom-resources/access-control


サンプルアプリケーションをデプロイ

.. code-block:: cmdin
 
  cd ~/kubernetes-ingress/examples/custom-resources/access-control
  kubectl apply -f webapp.yaml
  kubectl apply -f access-control-policy-deny.yaml
  kubectl apply -f virtual-server.yaml

リソースを確認

.. code-block:: cmdin
 
  kubectl get pod
  
.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル

  NAME                     READY   STATUS    RESTARTS   AGE
  webapp-64d444885-j4q7z   1/1     Running   0          2m7s

.. code-block:: cmdin
 
  kubectl get deployment
  
.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル

  NAME     READY   UP-TO-DATE   AVAILABLE   AGE
  webapp   1/1     1            1           2m13s

.. code-block:: cmdin
 
  kubectl get vs
  
.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル

  NAME     STATE   HOST                 IP    PORTS   AGE
  webapp   Valid   webapp.example.com                 2m8s

.. code-block:: cmdin
 
  kubectl get policy
  
.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル

  NAME            STATE   AGE
  webapp-policy   Valid   2m18s

.. code-block:: cmdin
 
  kubectl describe vs
  
.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル

  Name:         webapp
  Namespace:    default
  Labels:       <none>
  Annotations:  <none>
  API Version:  k8s.nginx.org/v1
  Kind:         VirtualServer
  
  ** 省略 **
  
  Spec:
    Host:  webapp.example.com
    Policies:
      Name:  webapp-policy
    Routes:
      Action:
        Pass:  webapp
      Path:    /
    Upstreams:
      Name:     webapp
      Port:     80
      Service:  webapp-svc
  Status:
    External Endpoints:
      Ip:
      Ports:
    Message:  Configuration for default/webapp was added or updated
    Reason:   AddedOrUpdated
    State:    Valid


| VSに ``webapp-policy`` が割り当てられていることが確認できます。
| コマンドを実行しPolicyの内容を確認します。Policyの内容が ``Spec`` に記載されています。

.. code-block:: cmdin
 
  kubectl describe policy
  
.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル

  Name:         webapp-policy
  Namespace:    default
  Labels:       <none>
  Annotations:  <none>
  API Version:  k8s.nginx.org/v1
  Kind:         Policy
  
  ** 省略 **
  
  Spec:
    Access Control:
      Deny:
        10.0.0.0/8
  Status:
    Message:  Policy default/webapp-policy was added or updated
    Reason:   AddedOrUpdated
    State:    Valid
  Events:
    Type    Reason          Age                  From                      Message
    ----    ------          ----                 ----                      -------
    Normal  AddedOrUpdated  61s (x3 over 2m31s)  nginx-ingress-controller  Policy default/webapp-policy was added or updated


curlコマンドで動作を確認します。以下のように通信が ``拒否`` されていることが確認できます

.. code-block:: cmdin
 
  curl -H "Host:webapp.example.com" http://localhost/

.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル

  <html>
  <head><title>403 Forbidden</title></head>
  <body>
  <center><h1>403 Forbidden</h1></center>
  <hr><center>nginx/1.21.3</center>
  </body>
  </html>

``webapp-policy`` の内容を変更します

.. code-block:: cmdin
 
  ## cd ~/kubernetes-ingress/examples/custom-resources/access-control
  kubectl apply -f access-control-policy-allow.yaml

.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル

  policy.k8s.nginx.org/webapp-policy configured


コマンドを実行しPolicyの内容を確認します。Policyの内容が ``Spec`` に記載されています。

.. code-block:: cmdin
 
  kubectl describe policy
  
.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル

  Name:         webapp-policy
  Namespace:    default
  Labels:       <none>
  Annotations:  <none>
  API Version:  k8s.nginx.org/v1
  Kind:         Policy
  
  ** 省略 **
  
  Spec:
    Access Control:
      Allow:
        10.0.0.0/8
  Status:
    Message:  Policy default/webapp-policy was added or updated
    Reason:   AddedOrUpdated
    State:    Valid


curlコマンドで動作を確認します。以下のように通信が ``許可`` されていることが確認できます

.. code-block:: cmdin
 
  curl -H "Host:webapp.example.com" http://localhost/

.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル

  Server address: 192.168.127.48:8080
  Server name: webapp-64d444885-j4q7z
  Date: 17/Jan/2022:12:48:51 +0000
  URI: /
  Request ID: 752997339b21d94210fc911cb41f7216
    

リソースの削除

.. code-block:: cmdin
 
  ## cd ~/kubernetes-ingress/examples/custom-resources/access-control
  kubectl delete -f access-control-policy-allow.yaml
  kubectl delete -f virtual-server.yaml
  kubectl delete -f webapp.yaml

URL Path の 変換 (Rewrite)
====

https://github.com/nginxinc/kubernetes-ingress/tree/v2.1.0/examples/custom-resources/rewrites


| Rewrite を用いて、URL Path を書換え、後段のサービスに転送することが可能です。
| まずVirtual Serverの定義内容を確認します。
| route に 3つのPathを定義し、rewritePath でURLの書換えを行います。
| 該当のPathでそれぞれのサービスに適したPathの書換えルールを定義します。


.. code-block:: yaml
   :linenos:
   :caption: 作成する rewrite-virtual-server.yaml の内容
   :name: 作成する rewrite-virtual-server.yaml の内容

  apiVersion: k8s.nginx.org/v1
  kind: VirtualServer
  metadata:
    name: cafe
  spec:
    host: cafe.example.com
    upstreams:
    - name: tea
      service: tea-svc
      port: 80
    - name: coffee
      service: coffee-svc
      port: 80
    routes:
    - path: /tea/
      action:
        proxy:
          upstream: tea
          rewritePath: /
    - path: /coffee
      action:
        proxy:
          upstream: coffee
          rewritePath: /beans
    - path: ~ /(\w+)/(.+\.(?:gif|jpg|png)$)
      action:
        proxy:
          upstream: tea
          rewritePath: /service/$1/image/$2


書換えのルールを表にまとめます。

.. list-table::
    :widths: 4 1 1 4
    :header-rows: 1
    :stub-columns: 1

    * - **Path**
      - **一致タイプ**
      - **Rewrite**
      - **結果**
    * - /tea/
      - 完全一致
      - /
      - /tea/abc -> /abc
    * - /coffee 
      - 完全一致
      - /beans
      - /coffee/def/ghi -> /beans/def/ghi
    * - ~ /(\w+)/(.+\.(?:gif|jpg|png)$)
      - 正規表現
      - /service/$1/image/$2
      - /cafe/top.jpg -> /service/cafe/image/top.jpg



正規表現のルールは、以下サイトを利用し確認いただけます
`debuggex <https://www.debuggex.com/>`__
``PCRE`` をプルダウンより選択し、上部に ``正規表現のルール`` 、下部に ``評価する文字列`` を入力し、結果を確認できます


サンプルアプリケーションをデプロイ

.. code-block:: cmdin
  
  cd ~/kubernetes-ingress/examples/custom-resources/rewrites
  cat << EOF > rewrite-virtual-server.yaml
  apiVersion: k8s.nginx.org/v1
  kind: VirtualServer
  metadata:
    name: cafe
  spec:
    host: cafe.example.com
    upstreams:
    - name: tea
      service: tea-svc
      port: 80
    - name: coffee
      service: coffee-svc
      port: 80
    routes:
    - path: /tea/
      action:
        proxy:
          upstream: tea
          rewritePath: /
    - path: /coffee
      action:
        proxy:
          upstream: coffee
          rewritePath: /beans
    - path: ~ /(\w+)/(.+\.(?:gif|jpg|png)$)
      action:
        proxy:
          upstream: tea
          rewritePath: /service/$1/image/$2
  EOF

  kubectl apply -f ../basic-configuration/cafe.yaml
  kubectl apply -f rewrite-virtual-server.yaml

リソースを確認

.. code-block:: cmdin
 
  kubectl get pod

.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル

  NAME                      READY   STATUS    RESTARTS   AGE
  coffee-7c86d7d67c-ws2t8   1/1     Running   0          39m
  coffee-7c86d7d67c-zt5tr   1/1     Running   0          39m
  tea-5c457db9-ksljs        1/1     Running   0          39m

.. code-block:: cmdin
 
  kubectl get deployment

.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル

  NAME     READY   UP-TO-DATE   AVAILABLE   AGE
  coffee   2/2     2            2           39m
  tea      1/1     1            1           39m

.. code-block:: cmdin
 
  kubectl get vs

.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル

  NAME   STATE   HOST               IP    PORTS   AGE
  cafe   Valid   cafe.example.com                 39m


動作確認

.. code-block:: cmdin
 
  curl -H "Host:cafe.example.com" http://localhost/tea/

.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル

  Server address: 192.168.127.40:8080
  Server name: tea-5c457db9-ksljs
  Date: 17/Jan/2022:14:22:46 +0000
  URI: /
  Request ID: 2576a16546e7d17467e04da2ab794109

.. code-block:: cmdin
 
  curl -H "Host:cafe.example.com" http://localhost/tea/abc

.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル

  Server address: 192.168.127.40:8080
  Server name: tea-5c457db9-ksljs
  Date: 17/Jan/2022:14:22:14 +0000
  URI: /abc
  Request ID: 5ce49a600fb24a40340ba6edad91ffb2

.. code-block:: cmdin
 
  curl -H "Host:cafe.example.com" http://localhost/coffee

.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル

  Server address: 192.168.127.39:8080
  Server name: coffee-7c86d7d67c-zt5tr
  Date: 17/Jan/2022:14:22:40 +0000
  URI: /beans
  Request ID: 9b15d10a624faee145b875b8f83460e3

.. code-block:: cmdin
 
  curl -H "Host:cafe.example.com" http://localhost/coffee/def/ghi

.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル

  Server address: 192.168.127.39:8080
  Server name: coffee-7c86d7d67c-zt5tr
  Date: 17/Jan/2022:14:22:27 +0000
  URI: /beans/def/ghi
  Request ID: f70d98547c615a145b2a40ddfe5884a4
  
.. code-block:: cmdin

   curl -H "Host:cafe.example.com" http://localhost/cafe/top.jpg

.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル
  
  Server address: 192.168.127.40:8080
  Server name: tea-5c457db9-ksljs
  Date: 17/Jan/2022:14:23:02 +0000
  URI: /service/cafe/image/top.jpg
  Request ID: 38c3cf24e3f5e0cdfe451b0d646c0e1d
   

リソースの削除

.. code-block:: cmdin
 
  ## cd ~/kubernetes-ingress/examples/custom-resources/rewrites
  kubectl delete -f ../basic-configuration/cafe.yaml
  kubectl delete -f rewrite-virtual-server.yaml


Ingress Controller で JWT Validation のデプロイ
====

https://github.com/nginxinc/kubernetes-ingress/tree/v2.1.0/examples/custom-resources/jwt

サンプルアプリケーションをデプロイ

.. code-block:: cmdin
 
  cd ~/kubernetes-ingress/examples/custom-resources/jwt/
  kubectl apply -f webapp.yaml    
  kubectl apply -f jwk-secret.yaml
  kubectl apply -f jwt.yaml
  kubectl apply -f virtual-server.yaml

利用するファイルの内容を確認します

まず、JWK(Json Web Key)としてVirtual ServerのPolicy内で指定するsecretの内容を確認します

.. code-block:: yaml
  :linenos:
  :caption: jwk-secret.yaml
  :name: jwk-secret.yaml

  apiVersion: v1
  kind: Secret
  metadata:
    name: jwk-secret
  type: nginx.org/jwk
  data:
    jwk: eyJrZXlzIjoKICAgIFt7CiAgICAgICAgImsiOiJabUZ1ZEdGemRHbGphbmQwIiwKICAgICAgICAia3R5Ijoib2N0IiwKICAgICAgICAia2lkIjoiMDAwMSIKICAgIH1dCn0K

``jwk`` というKeyに対し、 ``値`` として文字列が指定されていることが確認できます。
文字列の内容をbase64 decodeします

.. code-block:: cmdin
 
  # echo -n <jwk に指定された文字列> | base64 -d
  echo -n "eyJrZXlzIjoKICAgIFt7CiAgICAgICAgImsiOiJabUZ1ZEdGemRHbGphbmQwIiwKICAgICAgICAia3R5Ijoib2N0IiwKICAgICAgICAia2lkIjoiMDAwMSIKICAgIH1dCn0K" | base64 -d


出力結果が以下となります

.. code-block:: json
  :lineos:
  :caption: jwk
  :name: jwk

  {"keys":
      [{
          "k":"ZmFudGFzdGljand0",
          "kty":"oct",
          "kid":"0001"
      }]
  }
 

各パラメータ内容は以下の通り

.. list-table::
    :widths: 2 6 2 
    :header-rows: 1
    :stub-columns: 1

    * - **Parameter**
      - **意味**
      - **Link**
    * - k
      - k (key value) パラメータは, kty octで利用する base64url encodeされたKey文字列をもつ
      - `JSON Web Algorithms (JWA) 6.4.1 "k" <https://www.rfc-editor.org/rfc/rfc7518.txt>`__
    * - kty
      - kty (key type) パラメータは, RSA や EC といった暗号アルゴリズムファミリーを示す
      - `JSON Web Key (JWK) 4.1 "kty" <https://openid-foundation-japan.github.io/rfc7517.ja.html#ktyDef>`__
    * - kid
      - kid (key ID) パラメータは特定の鍵を識別するために用いられる
      - `JSON Web Key (JWK) 4.5 "kid" <https://openid-foundation-japan.github.io/rfc7517.ja.html#kidDef>`__


kty "oct" で利用する Keyの内容をBase64 Decodeした結果は以下の通り

.. code-block:: cmdin

  echo -n "ZmFudGFzdGljand0" | base64 -d

.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル

  fantasticjwt


VSで利用するPolicyについて確認します。まずVSの内容は以下です

.. code-block:: yaml
  :linenos:
  :caption: virtual-server.yaml
  :name: virtual-server.yaml

  apiVersion: k8s.nginx.org/v1
  kind: VirtualServer
  metadata:
    name: webapp
  spec:
    host: webapp.example.com
    policies:
    - name: jwt-policy
    upstreams:
    - name: webapp
      service: webapp-svc
      port: 80
    routes:
    - path: /
      action:
        pass: webapp

hostに対し ``jwt-policy`` というポリシーが適用されていることが確認できます。
では次に、Policyの内容を確認します

.. code-block:: yaml
  :linenos:
  :caption: jwt.yaml
  :name: jwt.yaml
    
  apiVersion: k8s.nginx.org/v1
  kind: Policy
  metadata:
    name: jwt-policy
  spec:
    jwt:
      realm: MyProductAPI
      secret: jwk-secret
      token: $http_token

| 先程VSの内容で確認したように、 ``jwt-policy`` という名前のPolicyとなります。
| specにPolicyの設定が記述されています。secretに先程作成した ``jwt-secret`` が指定されており、
| tokenとして参照する内容は、 ``token`` というhttp headerの値とするため、 ``$http_token`` を指定しています。


クライアントがリクエストする際に利用するJWTのサンプルの内容を確認します。


リソースを確認

.. code-block:: cmdin
   
  kubectl get deployment
    
.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル

  NAME     READY   UP-TO-DATE   AVAILABLE   AGE
  webapp   1/1     1            1           23s
  
  kubectl get secret | grep jwk
    
.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル

  jwk-secret            nginx.org/jwk                         1      40s

.. code-block:: cmdin
  
  kubectl get policy
    
.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル

  NAME         STATE   AGE
  jwt-policy   Valid   38s
  
  kubectl get vs
    
.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル

  NAME     STATE   HOST                 IP    PORTS   AGE
  webapp   Valid   webapp.example.com                 35s
    

動作確認

Policyが適用されたVSにJWTをHeaderに付与していないため、通信に対し ``401 Authorization required`` が応答されていることを確認します

.. code-block:: cmdin
  
  curl -H "Host:webapp.example.com" http://localhost/
    
.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル
  
  <html>
  <head><title>401 Authorization Required</title></head>
  <body>
  <center><h1>401 Authorization Required</h1></center>
  <hr><center>nginx/1.21.3</center>
  </body>
  </html>

curlコマンドで動作を確認します。以下のように通信が ``許可`` されていることが確認できます

.. code-block:: cmdin

  curl -H "Host:webapp.example.com" http://localhost/ -H "Token: `cat token.jwt`"
    
.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル

  Server address: 192.168.127.57:8080
  Server name: webapp-64d444885-r5fnt
  Date: 18/Jan/2022:12:49:59 +0000
  URI: /
  Request ID: 86182122eec0392769b4d86d64653419
  cat token.jwt
  eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiIsImtpZCI6IjAwMDEifQ.eyJuYW1lIjoiUXVvdGF0aW9uIFN5c3RlbSIsInN1YiI6InF1b3RlcyIsImlzcyI6Ik15IEFQSSBHYXRld2F5In0.ggVOHYnVFB8GVPE-VOIo3jD71gTkLffAY0hQOGXPL2I


リソースの削除

.. code-block:: cmdin
  
  ## cd ~/kubernetes-ingress/examples/custom-resources/jwt/
  kubectl delete -f virtual-server.yaml
  kubectl delete -f jwt.yaml
  kubectl delete -f jwk-secret.yaml
  kubectl delete -f webapp.yaml

Ingress Controller で OIDC RPのデプロイ
====

https://github.com/nginxinc/kubernetes-ingress/tree/v2.1.0/examples/custom-resources/oidc


サンプルアプリケーションをデプロイ

.. code-block:: cmdin
  
  cd ~/kubernetes-ingress/examples/custom-resources/oidc
  kubectl apply -f tls-secret.yaml
  kubectl apply -f webapp.yaml
  kubectl apply -f keycloak.yaml
  kubectl apply -f virtual-server-idp.yaml

.. code-block:: cmdin
  
  grep host virtual-server*yaml

.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル

  virtual-server-idp.yaml:  host: keycloak.example.com
  virtual-server.yaml:  host: webapp.example.com
  
  echo -n "f0558674-70a1-45a9-8c90-02245628b8f1" | base64

.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル

  ZjA1NTg2NzQtNzBhMS00NWE5LThjOTAtMDIyNDU2MjhiOGYx

.. code-block:: cmdin
  
  vi client-secret.yaml
  
.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル

  ああああ

.. code-block:: cmdin
   
  kubectl apply -f client-secret.yaml
  kubectl apply -f oidc.yaml

.. code-block:: cmdin
   
  kubectl get svc -n kube-system | grep kube-dns

.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル

  kube-dns   ClusterIP   10.96.0.10   <none>        53/UDP,53/TCP,9153/TCP   12d

.. code-block:: cmdin
  
  vi virtual-server.yaml

.. code-block:: cmdin
  
  kubectl apply -f virtual-server.yaml



リソースを確認


.. code-block:: cmdin

  kubectl get secret | grep -e oidc -e tls-secret

.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル

  oidc-secret           nginx.org/oidc                        1      4m29s
  tls-secret            kubernetes.io/tls                     2      21m

.. code-block:: cmdin
  kubectl get pod

.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル

  NAME                        READY   STATUS    RESTARTS   AGE
  keycloak-5cc8d76bd4-zpj87   1/1     Running   0          22m
  webapp-6c9689bbf4-qws2b     1/1     Running   0          22m
  
.. code-block:: cmdin

  kubectl get deployment

.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル

  NAME       READY   UP-TO-DATE   AVAILABLE   AGE
  keycloak   1/1     1            1           22m
  webapp     1/1     1            1           22m
  
.. code-block:: cmdin

  kubectl get svc

.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル

  NAME         TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
  keycloak     ClusterIP   10.97.4.138     <none>        8080/TCP   22m
  kubernetes   ClusterIP   10.96.0.1       <none>        443/TCP    12d
  webapp-svc   ClusterIP   10.104.69.230   <none>        80/TCP     22m
  
.. code-block:: cmdin

  kubectl get policy

.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル

  NAME          STATE   AGE
  oidc-policy   Valid   9m28s
  
.. code-block:: cmdin

  kubectl get vs

.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル

  NAME       STATE   HOST                   IP    PORTS   AGE
  keycloak   Valid   keycloak.example.com                 23m
  webapp     Valid   webapp.example.com                   7m40s


動作確認

Chromeブラウザを開き、 ``Secret Tab`` を開いてください。
そして、webapp.example.com を開いてください


リソースの削除

.. code-block:: cmdin

  kubectl delete -f webapp.yaml
  kubectl delete -f keycloak.yaml
  kubectl delete -f virtual-server-idp.yaml
  kubectl delete -f client-secret.yaml
  kubectl delete -f oidc.yaml
  kubectl delete -f virtual-server.yaml


Ingress MTLS
====

https://github.com/nginxinc/kubernetes-ingress/tree/v2.1.0/examples/custom-resources/ingress-mtls

サンプルアプリケーションをデプロイ

.. code-block:: cmdin

  cd ~/kubernetes-ingress/examples/custom-resources/ingress-mtls
  kubectl apply -f webapp.yaml
  kubectl apply -f ingress-mtls-secret.yaml
  kubectl apply -f ingress-mtls.yaml
  kubectl create -f tls-secret.yaml
  kubectl apply -f virtual-server.yaml


.. code-block:: bash
  :linenos:
  :caption: ingress-mtls-secret.yaml

  kind: Secret
  metadata:
    name: ingress-mtls-secret
  apiVersion: v1
  type: nginx.org/ca
  data:
    ca.crt: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUQvVENDQXVXZ0F3SUJBZ0lVSzdhbU14OFlLWG1BVG51SkZETDlWS2ZUR2ZNd0RRWUpLb1pJaHZjTkFRRUwKQlFBd2dZMHhDekFKQmdOVkJBWVRBbFZUTVFzd0NRWURWUVFJREFKRFFURVdNQlFHQTFVRUJ3d05VMkZ1SUVaeQpZVzVqYVhOamJ6RU9NQXdHQTFVRUNnd0ZUa2RKVGxneEREQUtCZ05WQkFzTUEwdEpRekVXTUJRR0ExVUVBd3dOCmEybGpMbTVuYVc1NExtTnZiVEVqTUNFR0NTcUdTSWIzRFFFSkFSWVVhM1ZpWlhKdVpYUmxjMEJ1WjJsdWVDNWoKYjIwd0hoY05NakF3T1RFNE1qQXlOVEkyV2hjTk16QXdPVEUyTWpBeU5USTJXakNCalRFTE1Ba0dBMVVFQmhNQwpWVk14Q3pBSkJnTlZCQWdNQWtOQk1SWXdGQVlEVlFRSERBMVRZVzRnUm5KaGJtTnBjMk52TVE0d0RBWURWUVFLCkRBVk9SMGxPV0RFTU1Bb0dBMVVFQ3d3RFMwbERNUll3RkFZRFZRUUREQTFyYVdNdWJtZHBibmd1WTI5dE1TTXcKSVFZSktvWklodmNOQVFrQkZoUnJkV0psY201bGRHVnpRRzVuYVc1NExtTnZiVENDQVNJd0RRWUpLb1pJaHZjTgpBUUVCQlFBRGdnRVBBRENDQVFvQ2dnRUJBTmFINVRzaTZzaUFsU085dEJnYmY3VVRwcWowMUhRTlQ2UjhtQy9pCjhLYXFaSW9XSUdvN2xhTW9xTDYydTc4ay9WOHM2Z0FJaU1DSzBjekFvTFhNSnlJQkxQeTg4Yzdtc2xwZXgxTkEKVmRtMkVTVkN6bVlERE1TT3FpVmszWmpYeC9URmo2QzhNRFhhRkZUWFg1dWdtbWdscnFCWlh0OVI5VVBwVTJMNwo1bEZ0NlJ2R3VGczgvbVZORVR5c1A0SFhCWlh2ZE9mdG1YWUkvK01hOW5CMzIzNjdmcTI0L0RKZ2YvK2xRbUsxCkJLR3poSTZSc1pSSmdWOXdpK1VuZTBYNjlaS2lLOFdXU3lZS252YnRrcHZuTDA2dGNJaXJZNi80UzZ4Sm1HRVQKZEJUNmVxc0NoSUpQUStWSEp5dTROdnV6WmVCUXpGdmMwNytnUGZkVWZra1FXODhDQXdFQUFhTlRNRkV3SFFZRApWUjBPQkJZRUZKUGdhcnFYa00rdEJ0djVhdndTUWhUQmpTU2VNQjhHQTFVZEl3UVlNQmFBRkpQZ2FycVhrTSt0CkJ0djVhdndTUWhUQmpTU2VNQThHQTFVZEV3RUIvd1FGTUFNQkFmOHdEUVlKS29aSWh2Y05BUUVMQlFBRGdnRUIKQUl3WXpoY0s4OWtRL0xGWjZFRHgrQWp2bnJTVSs1cmdwQkgrRjVTNUUyY3pXOE5rNXhySnl0Y0ZUbUtlKzZScwpENHlxeTZSVVFEeWNYaDlPelBjbzgzYTBoeFlCZ1M5MWtJa25wYWF4dndLRDJleWc3UGNnK1lkS1FhZFlMcUY0CmI3cWVtc1FVVkpOWHdkZS9VanRBejlEOTh4dngwM2hQY2Qwb2dzUUhWZ21BZVpFd2l3UzFmTy9WNUE4dTl3MEkKcHlJRTVReXlHcHNpS2dpalpiMmhrS05RVHVJcEhiVnFydVA4eEV6TlFnamhkdS9uUW5OYy9lRUltVUlrQkFUVQpiSHdQc2xwYzVhdVV1TXJxR3lEQ0p2QUJpV3J2SmE3Yi9XcmtDT3FUWVhtR2NGM0w1ZU9FeTBhYkp0M2NNcSs5CnJLTUNVQWlkNG0yNEthWnc3OUk2anNBPQotLS0tLUVORCBDRVJUSUZJQ0FURS0tLS0tCg==

.. code-block:: cmdin

  echo -n "LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUQvVENDQXVXZ0F3SUJBZ0lVSzdhbU14OFlLWG1BVG51SkZETDlWS2ZUR2ZNd0RRWUpLb1pJaHZjTkFRRUwKQlFBd2dZMHhDekFKQmdOVkJBWVRBbFZUTVFzd0NRWURWUVFJREFKRFFURVdNQlFHQTFVRUJ3d05VMkZ1SUVaeQpZVzVqYVhOamJ6RU9NQXdHQTFVRUNnd0ZUa2RKVGxneEREQUtCZ05WQkFzTUEwdEpRekVXTUJRR0ExVUVBd3dOCmEybGpMbTVuYVc1NExtTnZiVEVqTUNFR0NTcUdTSWIzRFFFSkFSWVVhM1ZpWlhKdVpYUmxjMEJ1WjJsdWVDNWoKYjIwd0hoY05NakF3T1RFNE1qQXlOVEkyV2hjTk16QXdPVEUyTWpBeU5USTJXakNCalRFTE1Ba0dBMVVFQmhNQwpWVk14Q3pBSkJnTlZCQWdNQWtOQk1SWXdGQVlEVlFRSERBMVRZVzRnUm5KaGJtTnBjMk52TVE0d0RBWURWUVFLCkRBVk9SMGxPV0RFTU1Bb0dBMVVFQ3d3RFMwbERNUll3RkFZRFZRUUREQTFyYVdNdWJtZHBibmd1WTI5dE1TTXcKSVFZSktvWklodmNOQVFrQkZoUnJkV0psY201bGRHVnpRRzVuYVc1NExtTnZiVENDQVNJd0RRWUpLb1pJaHZjTgpBUUVCQlFBRGdnRVBBRENDQVFvQ2dnRUJBTmFINVRzaTZzaUFsU085dEJnYmY3VVRwcWowMUhRTlQ2UjhtQy9pCjhLYXFaSW9XSUdvN2xhTW9xTDYydTc4ay9WOHM2Z0FJaU1DSzBjekFvTFhNSnlJQkxQeTg4Yzdtc2xwZXgxTkEKVmRtMkVTVkN6bVlERE1TT3FpVmszWmpYeC9URmo2QzhNRFhhRkZUWFg1dWdtbWdscnFCWlh0OVI5VVBwVTJMNwo1bEZ0NlJ2R3VGczgvbVZORVR5c1A0SFhCWlh2ZE9mdG1YWUkvK01hOW5CMzIzNjdmcTI0L0RKZ2YvK2xRbUsxCkJLR3poSTZSc1pSSmdWOXdpK1VuZTBYNjlaS2lLOFdXU3lZS252YnRrcHZuTDA2dGNJaXJZNi80UzZ4Sm1HRVQKZEJUNmVxc0NoSUpQUStWSEp5dTROdnV6WmVCUXpGdmMwNytnUGZkVWZra1FXODhDQXdFQUFhTlRNRkV3SFFZRApWUjBPQkJZRUZKUGdhcnFYa00rdEJ0djVhdndTUWhUQmpTU2VNQjhHQTFVZEl3UVlNQmFBRkpQZ2FycVhrTSt0CkJ0djVhdndTUWhUQmpTU2VNQThHQTFVZEV3RUIvd1FGTUFNQkFmOHdEUVlKS29aSWh2Y05BUUVMQlFBRGdnRUIKQUl3WXpoY0s4OWtRL0xGWjZFRHgrQWp2bnJTVSs1cmdwQkgrRjVTNUUyY3pXOE5rNXhySnl0Y0ZUbUtlKzZScwpENHlxeTZSVVFEeWNYaDlPelBjbzgzYTBoeFlCZ1M5MWtJa25wYWF4dndLRDJleWc3UGNnK1lkS1FhZFlMcUY0CmI3cWVtc1FVVkpOWHdkZS9VanRBejlEOTh4dngwM2hQY2Qwb2dzUUhWZ21BZVpFd2l3UzFmTy9WNUE4dTl3MEkKcHlJRTVReXlHcHNpS2dpalpiMmhrS05RVHVJcEhiVnFydVA4eEV6TlFnamhkdS9uUW5OYy9lRUltVUlrQkFUVQpiSHdQc2xwYzVhdVV1TXJxR3lEQ0p2QUJpV3J2SmE3Yi9XcmtDT3FUWVhtR2NGM0w1ZU9FeTBhYkp0M2NNcSs5CnJLTUNVQWlkNG0yNEthWnc3OUk2anNBPQotLS0tLUVORCBDRVJUSUZJQ0FURS0tLS0tCg==" | base64 -d > ca-crt.txt
  openssl x509 -text -noout -in ca-crt.txt
  Certificate:
    Data:
        Version: 3 (0x2)
        Serial Number:
            2b:b6:a6:33:1f:18:29:79:80:4e:7b:89:14:32:fd:54:a7:d3:19:f3
        Signature Algorithm: sha256WithRSAEncryption
        Issuer: C = US, ST = CA, L = San Francisco, O = NGINX, OU = KIC, CN = kic.nginx.com, emailAddress = kubernetes@nginx.com
        Validity
            Not Before: Sep 18 20:25:26 2020 GMT
            Not After : Sep 16 20:25:26 2030 GMT
        Subject: C = US, ST = CA, L = San Francisco, O = NGINX, OU = KIC, CN = kic.nginx.com, emailAddress = kubernetes@nginx.com
        Subject Public Key Info:
            Public Key Algorithm: rsaEncryption
                RSA Public-Key: (2048 bit)
                Modulus:
                    00:d6:87:e5:3b:22:ea:c8:80:95:23:bd:b4:18:1b:
                    7f:b5:13:a6:a8:f4:d4:74:0d:4f:a4:7c:98:2f:e2:
                    f0:a6:aa:64:8a:16:20:6a:3b:95:a3:28:a8:be:b6:
                    bb:bf:24:fd:5f:2c:ea:00:08:88:c0:8a:d1:cc:c0:
                    a0:b5:cc:27:22:01:2c:fc:bc:f1:ce:e6:b2:5a:5e:
                    c7:53:40:55:d9:b6:11:25:42:ce:66:03:0c:c4:8e:
                    aa:25:64:dd:98:d7:c7:f4:c5:8f:a0:bc:30:35:da:
                    14:54:d7:5f:9b:a0:9a:68:25:ae:a0:59:5e:df:51:
                    f5:43:e9:53:62:fb:e6:51:6d:e9:1b:c6:b8:5b:3c:
                    fe:65:4d:11:3c:ac:3f:81:d7:05:95:ef:74:e7:ed:
                    99:76:08:ff:e3:1a:f6:70:77:db:7e:bb:7e:ad:b8:
                    fc:32:60:7f:ff:a5:42:62:b5:04:a1:b3:84:8e:91:
                    b1:94:49:81:5f:70:8b:e5:27:7b:45:fa:f5:92:a2:
                    2b:c5:96:4b:26:0a:9e:f6:ed:92:9b:e7:2f:4e:ad:
                    70:88:ab:63:af:f8:4b:ac:49:98:61:13:74:14:fa:
                    7a:ab:02:84:82:4f:43:e5:47:27:2b:b8:36:fb:b3:
                    65:e0:50:cc:5b:dc:d3:bf:a0:3d:f7:54:7e:49:10:
                    5b:cf
                Exponent: 65537 (0x10001)
        X509v3 extensions:
            X509v3 Subject Key Identifier:
                93:E0:6A:BA:97:90:CF:AD:06:DB:F9:6A:FC:12:42:14:C1:8D:24:9E
            X509v3 Authority Key Identifier:
                keyid:93:E0:6A:BA:97:90:CF:AD:06:DB:F9:6A:FC:12:42:14:C1:8D:24:9E

            X509v3 Basic Constraints: critical
                CA:TRUE
    Signature Algorithm: sha256WithRSAEncryption
         8c:18:ce:17:0a:f3:d9:10:fc:b1:59:e8:40:f1:f8:08:ef:9e:
         b4:94:fb:9a:e0:a4:11:fe:17:94:b9:13:67:33:5b:c3:64:e7:
         1a:c9:ca:d7:05:4e:62:9e:fb:a4:6c:0f:8c:aa:cb:a4:54:40:
         3c:9c:5e:1f:4e:cc:f7:28:f3:76:b4:87:16:01:81:2f:75:90:
         89:27:a5:a6:b1:bf:02:83:d9:ec:a0:ec:f7:20:f9:87:4a:41:
         a7:58:2e:a1:78:6f:ba:9e:9a:c4:14:54:93:57:c1:d7:bf:52:
         3b:40:cf:d0:fd:f3:1b:f1:d3:78:4f:71:dd:28:82:c4:07:56:
         09:80:79:91:30:8b:04:b5:7c:ef:d5:e4:0f:2e:f7:0d:08:a7:
         22:04:e5:0c:b2:1a:9b:22:2a:08:a3:65:bd:a1:90:a3:50:4e:
         e2:29:1d:b5:6a:ae:e3:fc:c4:4c:cd:42:08:e1:76:ef:e7:42:
         73:5c:fd:e1:08:99:42:24:04:04:d4:6c:7c:0f:b2:5a:5c:e5:
         ab:94:b8:ca:ea:1b:20:c2:26:f0:01:89:6a:ef:25:ae:db:fd:
         6a:e4:08:ea:93:61:79:86:70:5d:cb:e5:e3:84:cb:46:9b:26:
         dd:dc:32:af:bd:ac:a3:02:50:08:9d:e2:6d:b8:29:a6:70:ef:
         d2:3a:8e:c0

openssl x509 -text -noout -in client-cert.pem
Certificate:
    Data:
        Version: 1 (0x0)
        Serial Number: 1 (0x1)
        Signature Algorithm: sha256WithRSAEncryption
        Issuer: C = US, ST = CA, L = San Francisco, O = NGINX, OU = KIC, CN = kic.nginx.com, emailAddress = kubernetes@nginx.com
        Validity
            Not Before: Sep 18 20:27:15 2020 GMT
            Not After : Sep 16 20:27:15 2030 GMT
        Subject: C = US, ST = CA, L = San Francisco, O = NGINX
        Subject Public Key Info:
            Public Key Algorithm: rsaEncryption
                RSA Public-Key: (2048 bit)
                Modulus:
                    00:b2:46:4a:d2:37:66:5d:97:63:3a:78:2b:b5:7d:
                    3b:a7:b6:7c:c4:ef:0a:89:2d:a7:1b:2c:4c:31:e3:
                    d2:ae:3c:02:6e:d3:56:5c:18:f7:13:f7:d6:4d:56:
                    45:cf:26:4e:cd:b9:ad:09:71:ff:ef:fb:bf:a6:6c:
                    fc:a4:bc:4d:fe:e6:33:18:53:18:df:f3:de:b3:a0:
                    d2:eb:9e:67:da:cb:9d:c3:e5:14:64:05:c0:15:8d:
                    f0:ff:2e:9b:3b:7e:58:63:e2:8c:dd:69:95:ed:ed:
                    ba:b7:4f:48:44:56:9f:a5:02:7c:70:e2:b6:b7:55:
                    3a:a8:a9:c0:31:1d:f6:15:c7:75:52:a1:99:6d:45:
                    94:98:ab:37:89:d3:2f:40:aa:24:08:fb:89:5f:45:
                    04:f7:57:62:cf:89:ca:3e:68:69:4d:b4:f1:7e:74:
                    3c:22:c2:b3:85:40:a6:66:d6:39:fd:e4:ea:94:1f:
                    f6:14:72:8f:1a:98:c6:9d:47:2e:04:2d:bc:12:b7:
                    b0:df:19:3c:74:be:79:e1:72:3e:fa:80:31:8e:b9:
                    d0:6c:d4:01:bb:4f:cb:5c:26:ec:bd:d7:f7:20:02:
                    40:ec:ac:45:6c:fe:4a:cb:ae:81:54:46:53:9d:19:
                    7a:64:01:4e:7d:0f:f1:e5:38:ac:be:62:a9:28:69:
                    d6:43
                Exponent: 65537 (0x10001)
    Signature Algorithm: sha256WithRSAEncryption
         0f:22:2c:eb:ca:51:ee:a2:8f:e5:41:2e:b0:26:fc:39:27:61:
         7e:da:3d:f4:f1:6f:f0:21:c0:9d:68:f5:04:ec:13:30:29:5f:
         eb:c0:36:cb:79:85:97:76:6a:7f:f5:2b:6b:d8:96:eb:46:98:
         20:87:df:cf:16:c3:93:19:55:ab:5f:b8:cd:60:af:d5:08:0c:
         28:3e:67:33:c4:f9:a6:6c:f1:e0:bb:dd:0c:3c:43:1a:88:62:
         dc:4e:8d:cd:1b:82:14:e9:b3:13:16:80:d1:f6:a9:13:b5:ee:
         1c:0f:d9:9e:c7:21:61:10:f0:d4:e7:fb:a3:d3:6f:a4:c5:4d:
         08:a4:62:e1:9e:25:eb:45:76:e1:23:c8:92:01:6e:85:db:1d:
         7f:a5:07:10:4a:0b:08:92:f7:9f:e6:dd:51:60:e0:1a:a6:3f:
         f2:f4:f2:ce:13:87:6f:ed:03:21:81:0a:5a:2a:ed:00:48:cf:
         48:78:7c:fc:b8:d4:b1:94:0b:d9:7a:84:2f:26:b1:4a:19:52:
         d9:96:63:3b:48:70:b6:a7:9a:e9:26:43:1f:8d:e7:29:af:ac:
         87:54:69:24:4e:72:d7:c0:e0:76:10:5b:5c:2f:42:be:4a:9f:
         37:4a:6e:57:22:03:f9:29:e0:bd:17:81:f9:25:62:39:bc:f4:
         08:60:23:e3


.. code-block:: bash
  :linenos:
  :caption: ingress-mtls.yaml

  apiVersion: k8s.nginx.org/v1
  kind: Policy
  metadata:
    name: ingress-mtls-policy
  spec:
    ingressMTLS:
      clientCertSecret: ingress-mtls-secret
      verifyClient: "on"
      verifyDepth: 1

.. code-block:: bash
  :linenos:
  :caption: virtual-server.yaml

  apiVersion: k8s.nginx.org/v1
  kind: VirtualServer
  metadata:
    name: webapp
  spec:
    host: webapp.example.com
    tls:
      secret: tls-secret
    policies:
    - name: ingress-mtls-policy
    upstreams:
    - name: webapp
      service: webapp-svc
      port: 80
    routes:
    - path: /
      action:
        pass: webapp


動作確認

.. code-block:: cmdin
  curl -v -k --resolve webapp.example.com:443:127.0.0.1 https://webapp.example.com:443/

.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル
  :emphasize-lines: 15,17,24,25,26,27,28,36,44

  * Added webapp.example.com:443:127.0.0.1 to DNS cache
  * Hostname webapp.example.com was found in DNS cache
  *   Trying 127.0.0.1:443...
  * TCP_NODELAY set
  * Connected to webapp.example.com (127.0.0.1) port 443 (#0)
  * ALPN, offering h2
  * ALPN, offering http/1.1
  * successfully set certificate verify locations:
  *   CAfile: /etc/ssl/certs/ca-certificates.crt
    CApath: /etc/ssl/certs
  * TLSv1.3 (OUT), TLS handshake, Client hello (1):
  * TLSv1.3 (IN), TLS handshake, Server hello (2):
  * TLSv1.2 (IN), TLS handshake, Certificate (11):
  * TLSv1.2 (IN), TLS handshake, Server key exchange (12):
  * TLSv1.2 (IN), TLS handshake, Request CERT (13):
  * TLSv1.2 (IN), TLS handshake, Server finished (14):
  * TLSv1.2 (OUT), TLS handshake, Certificate (11):
  * TLSv1.2 (OUT), TLS handshake, Client key exchange (16):
  * TLSv1.2 (OUT), TLS change cipher, Change cipher spec (1):
  * TLSv1.2 (OUT), TLS handshake, Finished (20):
  * TLSv1.2 (IN), TLS handshake, Finished (20):
  * SSL connection using TLSv1.2 / ECDHE-RSA-AES256-GCM-SHA384
  * ALPN, server accepted to use http/1.1
  * Server certificate:
  *  subject: CN=webapp.example.com
  *  start date: Sep 29 22:19:59 2020 GMT
  *  expire date: Sep 27 22:19:59 2030 GMT
  *  issuer: CN=webapp.example.com
  *  SSL certificate verify result: self signed certificate (18), continuing anyway.
  > GET / HTTP/1.1
  > Host: webapp.example.com
  > User-Agent: curl/7.68.0
  > Accept: */*
  >
  * Mark bundle as not supporting multiuse
  < HTTP/1.1 400 Bad Request
  < Server: nginx/1.21.3
  < Date: Wed, 19 Jan 2022 12:16:54 GMT
  < Content-Type: text/html
  < Content-Length: 237
  < Connection: close
  <
  <html>
  <head><title>400 No required SSL certificate was sent</title></head>
  <body>
  <center><h1>400 Bad Request</h1></center>
  <center>No required SSL certificate was sent</center>
  <hr><center>nginx/1.21.3</center>
  </body>
  </html>
  * Closing connection 0
  * TLSv1.2 (OUT), TLS alert, close notify (256):

.. code-block:: cmdin
  curl -v -k --resolve webapp.example.com:443:127.0.0.1 https://webapp.example.com:443/ --cert ./client-cert.pem --key ./client-key.pem

.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル
  :emphasize-lines: 15,17,19,25,26,27,28,29,37

  * Added webapp.example.com:443:127.0.0.1 to DNS cache
  * Hostname webapp.example.com was found in DNS cache
  *   Trying 127.0.0.1:443...
  * TCP_NODELAY set
  * Connected to webapp.example.com (127.0.0.1) port 443 (#0)
  * ALPN, offering h2
  * ALPN, offering http/1.1
  * successfully set certificate verify locations:
  *   CAfile: /etc/ssl/certs/ca-certificates.crt
    CApath: /etc/ssl/certs
  * TLSv1.3 (OUT), TLS handshake, Client hello (1):
  * TLSv1.3 (IN), TLS handshake, Server hello (2):
  * TLSv1.2 (IN), TLS handshake, Certificate (11):
  * TLSv1.2 (IN), TLS handshake, Server key exchange (12):
  * TLSv1.2 (IN), TLS handshake, Request CERT (13):
  * TLSv1.2 (IN), TLS handshake, Server finished (14):
  * TLSv1.2 (OUT), TLS handshake, Certificate (11):
  * TLSv1.2 (OUT), TLS handshake, Client key exchange (16):
  * TLSv1.2 (OUT), TLS handshake, CERT verify (15):
  * TLSv1.2 (OUT), TLS change cipher, Change cipher spec (1):
  * TLSv1.2 (OUT), TLS handshake, Finished (20):
  * TLSv1.2 (IN), TLS handshake, Finished (20):
  * SSL connection using TLSv1.2 / ECDHE-RSA-AES256-GCM-SHA384
  * ALPN, server accepted to use http/1.1
  * Server certificate:
  *  subject: CN=webapp.example.com
  *  start date: Sep 29 22:19:59 2020 GMT
  *  expire date: Sep 27 22:19:59 2030 GMT
  *  issuer: CN=webapp.example.com
  *  SSL certificate verify result: self signed certificate (18), continuing anyway.
  > GET / HTTP/1.1
  > Host: webapp.example.com
  > User-Agent: curl/7.68.0
  > Accept: */*
  >
  * Mark bundle as not supporting multiuse
  < HTTP/1.1 200 OK
  < Server: nginx/1.21.3
  < Date: Wed, 19 Jan 2022 12:16:56 GMT
  < Content-Type: text/plain
  < Content-Length: 157
  < Connection: keep-alive
  < Expires: Wed, 19 Jan 2022 12:16:55 GMT
  < Cache-Control: no-cache
  <
  Server address: 192.168.127.22:8080
  Server name: webapp-64d444885-x5d4p
  Date: 19/Jan/2022:12:16:56 +0000
  URI: /
  Request ID: c1b1c9c9b30331cbc7f034e026b939fc
  * Connection #0 to host webapp.example.com left intact

リソースの削除

.. code-block:: cmdin

  ## cd ~/kubernetes-ingress/examples/custom-resources/ingress-mtls
  kubectl delete -f webapp.yaml
  kubectl delete -f ingress-mtls-secret.yaml
  kubectl delete -f ingress-mtls.yaml
  kubectl delete -f tls-secret.yaml
  kubectl delete -f virtual-server.yaml
  rm ca-crt.txt


Egress MTLS
====

https://github.com/nginxinc/kubernetes-ingress/tree/v2.1.0/examples/custom-resources/egress-mtls



サンプルアプリケーションをデプロイ
リソースを確認
動作確認
リソースの削除
.. code-block:: cmdin

.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル