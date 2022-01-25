NICによるWebアプリの通信制御
####

この章では、実際にデプロイしたNGINX Ingress Controllerを使い、様々なサンプルアプリケーションを動作させ、その設定方法や動きを確認いただきます。
設定例は `NGINX Inc GitHubの examples/custom-resources/ <https://github.com/nginxinc/kubernetes-ingress/tree/v2.1.0/examples/custom-resources>`__ に管理されております

シンプルなWebアプリケーションのデプロイ
====

シンプルなWebアプリケーションをデプロイします。
Kubernetes環境で、Webアプリケーションをデプロイします。そのアプリケーションに対し通信制御を行うVirtualServer、及びHTTPSに必要な証明書をデプロイします。

https://github.com/nginxinc/kubernetes-ingress/tree/v2.1.0/examples/custom-resources/basic-configuration

サンプルアプリケーションをデプロイ
----

.. code-block:: cmdin
 
  cd ~/kubernetes-ingress/examples/custom-resources/basic-configuration/
  kubectl create -f cafe.yaml
  kubectl create -f cafe-secret.yaml
  kubectl create -f cafe-virtual-server.yaml

リソースを確認
----

以下の通り、各リソースを適切に作成されていることを確認します。

.. code-block:: cmdin
 
  kubectl get pod

.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル

  NAME                       READY   STATUS    RESTARTS  AGE
  coffee-7c86d7d67c-wjxss    1/1     Running   0         1m
  coffee-7c86d7d67c-8jm9z    1/1     Running   0         1m
  tea-5c457db9-dc4cs         1/1     Running   0         1m


.. code-block:: cmdin
 
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


動作確認
----

curlコマンドでリクエストを送信します。作成したWebアプリケーションから応答があることを確認します。 ``/coffee`` 、 ``/tea`` というURLに応じて異なるアプリケーションに転送されていることが確認できます

.. code-block:: cmdin
 
  curl -H "Host:cafe.example.com" http://localhost/coffee

.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル
  :emphasize-lines: 2,4

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
  :emphasize-lines: 2,4

  Server address: 192.168.127.20:8080
  Server name: tea-5c457db9-dc4cs
  Date: 17/Jan/2022:00:14:08 +0000
  URI: /tea
  Request ID: 6fd58877d9e85903300df7ceb0f81eb2

同様に、HTTPSの接続を確認します。

.. code-block:: cmdin
 
  curl -kv -H "Host:cafe.example.com" https://localhost/coffee

.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル
  :emphasize-lines: 26,42,44

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
  :emphasize-lines: 26,42,44

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


リソースの削除
----

.. code-block:: cmdin
 
  ## cd ~/kubernetes-ingress/examples/custom-resources/basic-configuration/
  kubectl delete -f cafe-secret.yaml
  kubectl delete -f cafe-virtual-server.yaml
  kubectl delete -f cafe.yaml


複数アプリケーション・チームを想定した VS / VSR 設定
====

https://github.com/nginxinc/kubernetes-ingress/tree/v2.1.0/examples/custom-resources/cross-namespace-configuration

NGINX Ingress Controller はCRDを用い、Virtual Server / Virtual Server Router / Policy といったリソースを使うことで、権限と設定範囲を適切に管理することが可能です。
ここでは、通信を待ち受けるため ``cafe`` namespace に、VirtualServer をデプロイします。そして ``tea`` / ``coffee`` namespace に アプリケーションと、アプリケーション宛に通信を転送するための VirtualServerRoute をデプロイします。 

サンプルアプリケーションをデプロイ
----

.. code-block:: cmdin
 
  cd ~/kubernetes-ingress/examples/custom-resources/cross-namespace-configuration]
  kubectl create -f namespaces.yaml
  kubectl create -f tea.yaml
  kubectl create -f coffee.yaml
  kubectl create -f tea-virtual-server-route.yaml
  kubectl create -f coffee-virtual-server-route.yaml
  kubectl create -f cafe-secret.yaml
  kubectl create -f cafe-virtual-server.yaml

リソースを確認
----

以下の通り、各リソースを適切に作成されていることを確認します。

.. code-block:: cmdin
    
  kubectl get ns --sort-by=.metadata.creationTimestamp
  
.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル

  NAME               STATUS   AGE
  **省略**
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
----

curlコマンドでリクエストを送信します。作成したWebアプリケーションから応答があることを確認します。 ``/coffee`` 、 ``/tea`` というURLに応じて異なるアプリケーションに転送されていることが確認できます

.. code-block:: cmdin
   
  curl -H "Host: cafe.example.com" http://localhost/coffee
  
.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル
  :emphasize-lines: 2,4

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
  :emphasize-lines: 2,4

  Server address: 192.168.127.24:8080
  Server name: tea-5c457db9-h5sm9
  Date: 17/Jan/2022:05:44:29 +0000
  URI: /tea
  Request ID: 698ab29da633f24a9bf5384c1499b056

同様にHTTPSの接続を確認します。HTTPSの結果は ``/tea`` にアクセスした結果のみ掲載します。 ``/coffee`` の結果も合わせて確認ください。

.. code-block:: cmdin
 
  curl -vk -H "Host: cafe.example.com" https://localhost/tea
  
.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル
  :emphasize-lines: 26,42,44

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
----

.. code-block:: cmdin
 
  ## cd ~/kubernetes-ingress/examples/custom-resources/cross-namespace-configuration]
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

| 通信の内容に応じて転送するサービスを制御するサンプルです。
| ユーザの属性毎に転送するサービスを変更したり、開発中アプリケーションに対するリクエストを識別し通信を制御したりする場合などに利用します

サンプルアプリケーションをデプロイ
----

.. code-block:: cmdin
 
  cd ~/kubernetes-ingress/examples/custom-resources/advanced-routing
  kubectl create -f cafe.yaml
  kubectl create -f cafe-virtual-server.yaml

リソースを確認
----

ポイントとなるファイルの内容を確認します。

``cafe-virtual-server.yaml`` で通信制御の条件を指定しています。条件は ``matches`` というパラメータで指定します。このサンプルの条件は以下の内容です。

* path: /tea

  * リクエストのHTTPメソッド($request_method)が、POSTの場合、 ``tea-post`` へ転送する。 それ以外は ``tea`` へ転送する。

* path: /coffee

  * cookie の version の値が v2 の場合、 ``coffee-v2`` へ転送する。それ以外は ``coffee-v1`` へ転送する。


それぞれの記述内容を以下で確認してください。

.. code-block:: yaml
  :linenos:
  :caption: cafe-virtual-server.yaml
  :emphasize-lines: 21,24,25,27,29,32,33,34,36,38

  apiVersion: k8s.nginx.org/v1
  kind: VirtualServer
  metadata:
    name: cafe
  spec:
    host: cafe.example.com
    upstreams:
    - name: tea-post
      service: tea-post-svc
      port: 80
    - name: tea
      service: tea-svc
      port: 80
    - name: coffee-v1
      service: coffee-v1-svc
      port: 80
    - name: coffee-v2
      service: coffee-v2-svc
      port: 80
    routes:
    - path: /tea
      matches:
      - conditions:
        - variable: $request_method
          value: POST
        action:
          pass: tea-post
      action:
        pass: tea
    - path: /coffee
      matches:
      - conditions:
        - cookie: version
          value: v2
        action:
          pass: coffee-v2
      action:
        pass: coffee-v1

以下の通り、各リソースを適切に作成されていることを確認します。

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
----

先程、設定ファイルから確認した条件を再度記載します。

* path: /tea

  * リクエストのHTTPメソッド($request_method)が、POSTの場合、 ``tea-post`` へ転送する。 それ以外は ``tea`` へ転送する。

* path: /coffee

  * cookie の version の値が v2 の場合、 ``coffee-v2`` へ転送する。それ以外は ``coffee-v1`` へ転送する。


Curlコマンドで動作を確認します。 

``/tea`` 宛でHTTPメソッドを指定しない(GET)の場合の動作は以下の通りです

.. code-block:: cmdin
 
  curl -H "Host: cafe.example.com" http://localhost/tea

.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル
  :emphasize-lines: 2

  Server address: 192.168.127.33:8080
  Server name: tea-5c457db9-dcswc
  Date: 17/Jan/2022:09:00:56 +0000
  URI: /tea
  Request ID: 00e9eb4d61f7afdb8c5656da94d15b98

``/tea`` 宛でHTTP POSTメソッドを指定した場合の動作は以下の通りです。

.. code-block:: cmdin
 
  curl -H "Host: cafe.example.com" http://localhost/tea -X POST

.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル
  :emphasize-lines: 2

  Server address: 192.168.127.32:8080
  Server name: tea-post-7db8cd8bf-m5gbz
  Date: 17/Jan/2022:09:01:02 +0000
  URI: /tea
  Request ID: 4deeb82434a6f799ffc894a229ac361a

``/coffee`` 宛でCookieの値を指定しない場合の動作は以下の通りです。

.. code-block:: cmdin
 
  curl -H "Host: cafe.example.com" http://localhost/coffee

.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル
  :emphasize-lines: 2

  Server address: 192.168.127.23:8080
  Server name: coffee-v1-6b78998db9-8cv49
  Date: 17/Jan/2022:09:01:25 +0000
  URI: /coffee
  Request ID: 8d182c9c060d5a4d4dec226292ac2820

``/coffee`` 宛でCookieに"version=v2"と指定した場合の動作は以下の通りです。

.. code-block:: cmdin
 
  curl -H "Host: cafe.example.com" http://localhost/coffee --cookie "version=v2"

.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル
  :emphasize-lines: 2

  Server address: 192.168.127.27:8080
  Server name: coffee-v2-748cbbb49f-mbxpr
  Date: 17/Jan/2022:09:01:35 +0000
  URI: /coffee
  Request ID: befacc5e7ca56a1a09e5982315c74fa0

リソースの削除
----

.. code-block:: cmdin
 
  ## cd ~/kubernetes-ingress/examples/custom-resources/advanced-routing
  kubectl delete  -f cafe-virtual-server.yaml
  kubectl delete  -f cafe.yaml


割合を指定した分散 (Traffic Split)
====

https://github.com/nginxinc/kubernetes-ingress/tree/v2.1.0/examples/custom-resources/traffic-splitting

割合を指定し、トラフィックを分散することができます。


サンプルアプリケーションをデプロイ
----

.. code-block:: cmdin
 
  cd ~/kubernetes-ingress/examples/custom-resources/traffic-splitting    
  kubectl create -f cafe.yaml
  kubectl create -f cafe-virtual-server.yaml


Virtual Serverの内容を確認します。 ``cafe-virtual-server.yaml`` の ``/coffee`` に ``splits`` を指定し、更に ``weight`` でサービスへ転送する割合を指定しています。

.. code-block:: yaml
   :linenos:
   :caption: cafe-virtual-server.yaml
   :emphasize-lines: 16,17,20

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
----

以下の通り、各リソースを適切に作成されていることを確認します。

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
----

 Curlコマンドで複数回リクエストを送ると、 ``coffee-v1`` 、 ``coffee-v2`` のそれぞれに転送されていることが確認できます。

.. code-block:: cmdin
 
  curl -s -H "Host: cafe.example.com" http://localhost/coffee
  
.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル
  :emphasize-lines: 2

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
  :emphasize-lines: 2

  Server address: 192.168.127.45:8080
  Server name: coffee-v2-748cbbb49f-llpb6
  Date: 17/Jan/2022:12:26:37 +0000
  URI: /coffee
  Request ID: 357237a3fea498b6efd90c929d526e64

以下コマンドを参考に複数回Curlを実行し、その結果をファイルに記録します。記録の内容より ``coffee-v1`` に ``coffee-v2`` 転送した数を確認できます。
分散する割合は少しばらつきが発生しますが、参考として分散した数の結果を確認してください。

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
----

.. code-block:: cmdin
 
  ## cd ~/kubernetes-ingress/examples/custom-resources/traffic-splitting
  kubectl delete -f cafe-virtual-server.yaml
  kubectl delete -f cafe.yaml
  rm split.txt



IPアドレスによる通信の制御 (Access Control)
====

https://github.com/nginxinc/kubernetes-ingress/tree/v2.1.0/examples/custom-resources/access-control

Policyにより通信制御を行う方法を確認します。リクエストの送信元IPアドレスに応じて通信の許可・拒否を行う方法を確認します。

サンプルアプリケーションをデプロイ
----

.. code-block:: cmdin
 
  cd ~/kubernetes-ingress/examples/custom-resources/access-control
  kubectl apply -f webapp.yaml
  kubectl apply -f access-control-policy-deny.yaml
  kubectl apply -f virtual-server.yaml

リソースを確認
----

以下の通り、各リソースを適切に作成されていることを確認します。

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

VirtualServerに ``webapp-policy`` が割り当てられていることが確認できます。

.. code-block:: cmdin
 
  kubectl describe vs
  
.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル
  :emphasize-lines: 13,12

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


| コマンドを実行しPolicyの内容を確認します。Policyの内容が ``Spec`` に記載されています。

.. code-block:: cmdin
 
  kubectl describe policy
  
.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル
  :emphasize-lines: 10,11,12,13

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


動作確認
----

curlコマンドで動作を確認します。以下のように通信が ``拒否`` されていることが確認できます

.. code-block:: cmdin
 
  curl -H "Host:webapp.example.com" http://localhost/

.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル
  :emphasize-lines: 2

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


コマンドを実行しPolicyの内容を確認します。Policyの内容が ``Spec`` に記載されています。

.. code-block:: cmdin
 
  kubectl describe policy
  
.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル
  :emphasize-lines: 10,11,12,13

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
----

.. code-block:: cmdin
 
  ## cd ~/kubernetes-ingress/examples/custom-resources/access-control
  kubectl delete -f access-control-policy-allow.yaml
  kubectl delete -f virtual-server.yaml
  kubectl delete -f webapp.yaml

URL Path の 変換 (Rewrite)
====

https://github.com/nginxinc/kubernetes-ingress/tree/v2.1.0/examples/custom-resources/rewrites


Rewrite を用いて、URL Path を書換えることが可能です。

サンプルアプリケーションをデプロイ
----

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
----

Virtual Serverの定義内容を確認します。route に 3つのPathを定義し、rewritePath でURLの書換えを行います。該当のPathでそれぞれのサービスに適したPathの書換えルールを定義します。

.. code-block:: yaml
   :linenos:
   :caption: 作成する rewrite-virtual-server.yaml の内容
   :name: 作成する rewrite-virtual-server.yaml の内容
  :emphasize-lines: 15,19,20,24,25,29

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



| 正規表現のルールは、以下サイトを利用し確認いただけます
| `debuggex <https://www.debuggex.com/>`__
| ``PCRE`` をプルダウンより選択し、上部に ``正規表現のルール`` 、下部に ``評価する文字列`` を入力し、結果を確認できます


以下の通り、各リソースを適切に作成されていることを確認します。

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
----

先程定義を確認したとおり、URLが書換えられていることが確認できます。

.. code-block:: cmdin
 
  curl -H "Host:cafe.example.com" http://localhost/tea/

.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル
  :emphasize-lines: 4

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
  :emphasize-lines: 4

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
  :emphasize-lines: 4

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
  :emphasize-lines: 4

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
  :emphasize-lines: 4
 
  Server address: 192.168.127.40:8080
  Server name: tea-5c457db9-ksljs
  Date: 17/Jan/2022:14:23:02 +0000
  URI: /service/cafe/image/top.jpg
  Request ID: 38c3cf24e3f5e0cdfe451b0d646c0e1d
   

リソースの削除
----

.. code-block:: cmdin
 
  ## cd ~/kubernetes-ingress/examples/custom-resources/rewrites
  kubectl delete -f ../basic-configuration/cafe.yaml
  kubectl delete -f rewrite-virtual-server.yaml


Ingress Controller で JWT Validation のデプロイ
====

https://github.com/nginxinc/kubernetes-ingress/tree/v2.1.0/examples/custom-resources/jwt


NGINX Ingress Controller で JWT の Validation を行い、通信制御を行うことが可能です。

サンプルアプリケーションをデプロイ
----

.. code-block:: cmdin
 
  cd ~/kubernetes-ingress/examples/custom-resources/jwt/
  kubectl apply -f webapp.yaml    
  kubectl apply -f jwk-secret.yaml
  kubectl apply -f jwt.yaml
  kubectl apply -f virtual-server.yaml


リソースを確認
----

利用するファイルの内容を確認します

まず、JWK(Json Web Key)としてVirtual ServerのPolicy内で指定するsecretの内容を確認します

.. code-block:: yaml
  :linenos:
  :caption: jwk-secret.yaml
  :name: jwk-secret.yaml
  :emphasize-lines: 7

  apiVersion: v1
  kind: Secret
  metadata:
    name: jwk-secret
  type: nginx.org/jwk
  data:
    jwk: eyJrZXlzIjoKICAgIFt7CiAgICAgICAgImsiOiJabUZ1ZEdGemRHbGphbmQwIiwKICAgICAgICAia3R5Ijoib2N0IiwKICAgICAgICAia2lkIjoiMDAwMSIKICAgIH1dCn0K

``jwk`` というKeyに対し、 ``値`` として文字列が指定されていることが確認できます。
文字列の内容をbase64デコードします

.. code-block:: cmdin
 
  # echo -n <jwk に指定された文字列> | base64 -d
  echo -n "eyJrZXlzIjoKICAgIFt7CiAgICAgICAgImsiOiJabUZ1ZEdGemRHbGphbmQwIiwKICAgICAgICAia3R5Ijoib2N0IiwKICAgICAgICAia2lkIjoiMDAwMSIKICAgIH1dCn0K" | base64 -d

出力結果が以下となります。

.. code-block:: json
  :linenos:
  :caption: jwk を base64 デコードした結果
  :emphasize-lines: 3

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


kty "oct" で利用する Keyの内容をBase64デコードした結果は以下の通り

.. code-block:: cmdin

  echo -n "ZmFudGFzdGljand0" | base64 -d

.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル

  fantasticjwt

この結果により、このサンプルでは ``fantasticjwt`` という文字列がKeyとして使用されていることが確認できます。

今回サンプルリクエストに利用するJWTがこの文字列で署名されたものであるか確認します。 ``token.jwt`` の内容を表示します。

.. code-block:: bash
  :linenos:
  :caption: token.jwt

  eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiIsImtpZCI6IjAwMDEifQ.eyJuYW1lIjoiUXVvdGF0aW9uIFN5c3RlbSIsInN1YiI6InF1b3RlcyIsImlzcyI6Ik15IEFQSSBHYXRld2F5In0.ggVOHYnVFB8GVPE-VOIo3jD71gTkLffAY0hQOGXPL2I

| `JWT.io <https://jwt.io/>`__ を開き、 **Algorithm** が ``HS256`` で有ることを確認します。
| **VERIFY SIGNATURE** 欄の ``your-256-bit-secret`` に先程 ``jwk`` の内容をデコードして確認した文字列 ``fantasticjwt`` を入力してください。
| 画面左側 **Eocoded** 欄に、``token.jwt`` の内容を貼り付け、左下の表示が ``Signature Verified`` となることを確認してください。
| この結果より、クライアントリクエストで利用するJWTは、検証可能なものであることが確認できます。またこのJWTに含まれる情報が右側に表示されますので合わせて確認ください。

   .. image:: ./media/jwtio_verify.jpg
      :width: 500

その他、NGINX Plus / JWT に関する詳細は 
`Blog:Authenticating API Clients with JWT and NGINX Plus <https://www.nginx.com/blog/authenticating-api-clients-jwt-nginx-plus/>`__ 
を参照してください


VirtualServerで利用するPolicyについて確認します。まずVirtualServerの内容は以下です

.. code-block:: yaml
  :linenos:
  :caption: virtual-server.yaml
  :name: virtual-server.yaml
  :emphasize-lines: 7,8

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
  :emphasize-lines: 4,8,9

  apiVersion: k8s.nginx.org/v1
  kind: Policy
  metadata:
    name: jwt-policy
  spec:
    jwt:
      realm: MyProductAPI
      secret: jwk-secret
      token: $http_token

| 先程VirtualServerの内容で確認したように、 ``jwt-policy`` という名前のPolicyとなります。
| specにPolicyの設定が記述されています。secretに先程作成した ``jwt-secret`` が指定されており、
| tokenとして参照する内容は、 ``token`` というhttp headerの値とするため、 ``$http_token`` を指定しています。


以下の通り、各リソースを適切に作成されていることを確認します。

.. code-block:: cmdin
   
  kubectl get deployment
    
.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル

  NAME     READY   UP-TO-DATE   AVAILABLE   AGE
  webapp   1/1     1            1           23s

.. code-block:: cmdin
   
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


.. code-block:: cmdin
   
  kubectl get vs
    
.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル

  NAME     STATE   HOST                 IP    PORTS   AGE
  webapp   Valid   webapp.example.com                 35s
    

動作確認
----

Policyが適用されたVirtualServerにJWTをHeaderに付与していないため、通信に対し ``401 Authorization required`` が応答されていることを確認します

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

リソースの削除
----

.. code-block:: cmdin
  
  ## cd ~/kubernetes-ingress/examples/custom-resources/jwt/
  kubectl delete -f virtual-server.yaml
  kubectl delete -f jwt.yaml
  kubectl delete -f jwk-secret.yaml
  kubectl delete -f webapp.yaml

Ingress Controller で OIDC RPのデプロイ
====

https://github.com/nginxinc/kubernetes-ingress/tree/v2.1.0/examples/custom-resources/oidc

| NGINX Ingress Controller による JWT の制御に加え、NGINXより提供するJavaScript Moduleを利用することにより、OIDCのRPとして動作することが可能です。
| このサンプルでは、KeycloakをIDPとして動作させ、クライアントのリクエストを適切に認証することを確認いただけます。

サンプルアプリケーションをデプロイ
----

リソースをデプロイします。ここでIDPとして動作させる ``KeyCloak`` をデプロイします

.. code-block:: cmdin
  
  cd ~/kubernetes-ingress/examples/custom-resources/oidc
  kubectl apply -f tls-secret.yaml
  kubectl apply -f webapp.yaml
  kubectl apply -f keycloak.yaml
  kubectl apply -f virtual-server-idp.yaml

このサンプルで利用するFQDNを確認します。ラボ環境のJumpHostでは予め双方のFQDNを登録しています。

.. code-block:: cmdin
  
  ## cd ~/kubernetes-ingress/examples/custom-resources/oidc
  grep host virtual-server*yaml

.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル

  virtual-server-idp.yaml:  host: keycloak.example.com
  virtual-server.yaml:  host: webapp.example.com

| ブラウザからKeyCloakにアクセスし、設定を行います。
| Chromeを開き、 ``https://keycloak.example.com`` へアクセスしてください。

.. NOTE::
  Keycloak はデプロイから起動まで数分(2～3分)かかります。正しく疎通ができない場合は一定時間おいて再度接続してください
  


   .. image:: ./media/keycloak_top.jpg
      :width: 500

**Administration Console** を開きます。ログイン画面が表示されますので以下の情報でログインしてください。

* ログイン情報
=========== ============
**usename** **password**  
=========== ============
admin       admin
=========== ============

  .. image:: ./media/keycloak_login.jpg
     :width: 500

左メニューより **Clients** を開き、 **Create** から新規作成を行います。

  .. image:: ./media/keycloak_clients.jpg
     :width: 500

Client ID: ``nginx-plus`` を指定し、 **Save** します。

  .. image:: ./media/keycloak_clients_new.jpg
     :width: 500

SettingsタブのAccess Type: ``confidential`` を選択し、Valid Redirect URIs: ``https://webapp.example.com:443/_codexch`` を入力し、 **Save** します。

  .. image:: ./media/keycloak_clients_setting.jpg
     :width: 500

Credentialsタブを開きます。後ほどSecretの値を利用しますので表示されている文字列を記録しておきます。

  .. image:: ./media/keycloak_clients_secret.jpg
     :width: 500

Rolesタブを開き、 **Add Role** から追加を行います。

  .. image:: ./media/keycloak_clients_role.jpg
     :width: 500

Role Name: ``nginx-keycloak-role`` を指定し、 **Save** します。

  .. image:: ./media/keycloak_clients_role2.jpg
     :width: 500

左メニュー **Users** を開き、 **Add user** からユーザの新規作成を行います。

  .. image:: ./media/keycloak_clients_users.jpg
     :width: 500

Username: ``nginx-user`` を指定し、 **Save** します。

  .. image:: ./media/keycloak_clients_users_new.jpg
     :width: 500

Credentialsタブを開き、Password: ``test`` を入力、Temporary: ``Off`` を選択し、nginx-userのパスワードを設定します。

  .. image:: ./media/keycloak_clients_users_pass.jpg
     :width: 500

Role Mappingsタブを開き、Client Roles: ``nginx-plus`` を選択し、Available Rolesに表示される ``nginx-keycloak-role`` を選択し、 **Add selected** でRoleをAssignします。

  .. image:: ./media/keycloak_clients_users_role_mapping.jpg
     :width: 500

これでKeycloakの準備は完了しました。

先程、Keycloakで確認したSecretの内容をbase64エンコードします

.. code-block:: cmdin

  echo -n "f0558674-70a1-45a9-8c90-02245628b8f1" | base64

.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル

  ZjA1NTg2NzQtNzBhMS00NWE5LThjOTAtMDIyNDU2MjhiOGYx

``client-secret.yaml``  に設定します

.. code-block:: cmdin
  
  ## cd ~/kubernetes-ingress/examples/custom-resources/oidc
  vi client-secret.yaml
  
.. code-block:: yaml
  :linenos:
  :caption: Client Secret の指定
  :emphasize-lines: 7

  apiVersion: v1
  kind: Secret
  metadata:
    name: oidc-secret
  type: nginx.org/oidc
  data:
    client-secret: ***BASE64 EncodeしたSECRET情報***

OIDC PolicyとClientSecretをデプロイします。

.. code-block:: cmdin
  
  ## cd ~/kubernetes-ingress/examples/custom-resources/oidc
  kubectl apply -f client-secret.yaml
  kubectl apply -f oidc.yaml

本書の環境では単一のPodでNGINX Ingress Controllerを動作させているためZone Synchronizationの設定はしません。必要となる方は手順を参考に実施してください。

最後にNGINX Ingress ControllerをWebアプリケーション用のOIDC RP として動作させるため、VirtualServerを作成します。kbue-dnsのIPアドレスを確認し、virtual-server.yamlに設定を追加します。

.. code-block:: cmdin
  
  kubectl get svc -n kube-system | grep kube-dns

.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル

  kube-dns   ClusterIP   10.96.0.10   <none>        53/UDP,53/TCP,9153/TCP   12d

.. code-block:: cmdin
  
  ## cd ~/kubernetes-ingress/examples/custom-resources/oidc
  vi virtual-server.yaml

.. code-block:: yaml
  :linenos:
  :caption: server-snippetsにkube-dnsのIPをresolverとして指定
  :emphasize-lines: 11,12

  apiVersion: k8s.nginx.org/v1
  kind: VirtualServer
  metadata:
    name: webapp
  spec:
    host: webapp.example.com
    tls:
      secret: tls-secret
      redirect:
        enable: true
    server-snippets: |
      resolver 10.96.0.10;                     # kube-dnsのIPアドレスを指定します
    upstreams:
      - name: webapp
        service: webapp-svc
        port: 80
    routes:
      - path: /
        policies:
        - name: oidc-policy
        action:
          pass: webapp

.. code-block:: cmdin
  
  ## cd ~/kubernetes-ingress/examples/custom-resources/oidc
  kubectl apply -f virtual-server.yaml



リソースを確認
----

以下の通り、各リソースを適切に作成されていることを確認します。


.. code-block:: cmdin

  kubectl get secret | grep -e oidc -e tls-secret

.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル

  oidc-secret           nginx.org/oidc                        1      4m29s
  tls-secret            kubernetes.io/tls                     2      21m
  
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
----

Chromeブラウザを開き、 ``Secret Tab (New Incognito Window)`` を開いてください。

  .. image:: ./media/chrome_secret_tab.jpg
     :width: 500

``https://webapp.example.com`` へアクセスしてください。

  .. image:: ./media/chrome_webapp.jpg
     :width: 500

Keycloakのログイン画面が表示されます。先程設定を行った ``nginx-user`` でログインしてください。

* ログイン情報
=========== ============
**usename** **password**  
=========== ============
nginx-user  test
=========== ============

  .. image:: ./media/chrome_webapp_keycloak_login.jpg
     :width: 500

ログインが正常に行われた場合、Webアプリケーションの結果をブラウザで確認いただけます。

  .. image:: ./media/chrome_webapp_logined.jpg
     :width: 500

リソースの削除
----

.. code-block:: cmdin

  ## cd ~/kubernetes-ingress/examples/custom-resources/oidc
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
----

.. code-block:: cmdin

  cd ~/kubernetes-ingress/examples/custom-resources/ingress-mtls
  kubectl apply -f webapp.yaml
  kubectl apply -f ingress-mtls-secret.yaml
  kubectl apply -f ingress-mtls.yaml
  kubectl create -f tls-secret.yaml
  kubectl apply -f virtual-server.yaml


リソースを確認
----

ポイントとなるファイルの内容を確認します。

``ingress-mtls-secret.yaml`` でクライアント証明書の評価に用いる証明書を作成します。

.. code-block:: bash
  :linenos:
  :caption: ingress-mtls-secret.yaml

  kind: Secret
  metadata:
    name: ingress-mtls-secret
  apiVersion: v1
  type: nginx.org/ca
  data:
    ca.crt: **省略**

``ingress-mtls.yaml`` は別途作成した ``ingress-mtls-secret`` をclientCertSecretに指定し(7)、Virtual Serverで利用するPolicyを作成します。

.. code-block:: bash
  :linenos:
  :caption: ingress-mtls.yaml
  :emphasize-lines: 7

  apiVersion: k8s.nginx.org/v1
  kind: Policy
  metadata:
    name: ingress-mtls-policy
  spec:
    ingressMTLS:
      clientCertSecret: ingress-mtls-secret
      verifyClient: "on"
      verifyDepth: 1

作成した ``ingress-mtls-policy`` というPolicyをリクエストに対し適用するため、TLSの指定(7,8)と、Policyの指定(9,10)を行っています

.. code-block:: bash
  :linenos:
  :caption: virtual-server.yaml
  :emphasize-lines: 7,8,9,10

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

通信に利用される証明書の内容は、以下コマンドを参考に確認してください

.. code-block:: cmdin

  # echo -n <ca.crt に指定された文字列> | base64 -d > ca-crt.txt
  echo -n "LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUQvVENDQXVXZ0F3SUJBZ0lVSzdhbU14OFlLWG1BVG51SkZETDlWS2ZUR2ZNd0RRWUpLb1pJaHZjTkFRRUwKQlFBd2dZMHhDekFKQmdOVkJBWVRBbFZUTVFzd0NRWURWUVFJREFKRFFURVdNQlFHQTFVRUJ3d05VMkZ1SUVaeQpZVzVqYVhOamJ6RU9NQXdHQTFVRUNnd0ZUa2RKVGxneEREQUtCZ05WQkFzTUEwdEpRekVXTUJRR0ExVUVBd3dOCmEybGpMbTVuYVc1NExtTnZiVEVqTUNFR0NTcUdTSWIzRFFFSkFSWVVhM1ZpWlhKdVpYUmxjMEJ1WjJsdWVDNWoKYjIwd0hoY05NakF3T1RFNE1qQXlOVEkyV2hjTk16QXdPVEUyTWpBeU5USTJXakNCalRFTE1Ba0dBMVVFQmhNQwpWVk14Q3pBSkJnTlZCQWdNQWtOQk1SWXdGQVlEVlFRSERBMVRZVzRnUm5KaGJtTnBjMk52TVE0d0RBWURWUVFLCkRBVk9SMGxPV0RFTU1Bb0dBMVVFQ3d3RFMwbERNUll3RkFZRFZRUUREQTFyYVdNdWJtZHBibmd1WTI5dE1TTXcKSVFZSktvWklodmNOQVFrQkZoUnJkV0psY201bGRHVnpRRzVuYVc1NExtTnZiVENDQVNJd0RRWUpLb1pJaHZjTgpBUUVCQlFBRGdnRVBBRENDQVFvQ2dnRUJBTmFINVRzaTZzaUFsU085dEJnYmY3VVRwcWowMUhRTlQ2UjhtQy9pCjhLYXFaSW9XSUdvN2xhTW9xTDYydTc4ay9WOHM2Z0FJaU1DSzBjekFvTFhNSnlJQkxQeTg4Yzdtc2xwZXgxTkEKVmRtMkVTVkN6bVlERE1TT3FpVmszWmpYeC9URmo2QzhNRFhhRkZUWFg1dWdtbWdscnFCWlh0OVI5VVBwVTJMNwo1bEZ0NlJ2R3VGczgvbVZORVR5c1A0SFhCWlh2ZE9mdG1YWUkvK01hOW5CMzIzNjdmcTI0L0RKZ2YvK2xRbUsxCkJLR3poSTZSc1pSSmdWOXdpK1VuZTBYNjlaS2lLOFdXU3lZS252YnRrcHZuTDA2dGNJaXJZNi80UzZ4Sm1HRVQKZEJUNmVxc0NoSUpQUStWSEp5dTROdnV6WmVCUXpGdmMwNytnUGZkVWZra1FXODhDQXdFQUFhTlRNRkV3SFFZRApWUjBPQkJZRUZKUGdhcnFYa00rdEJ0djVhdndTUWhUQmpTU2VNQjhHQTFVZEl3UVlNQmFBRkpQZ2FycVhrTSt0CkJ0djVhdndTUWhUQmpTU2VNQThHQTFVZEV3RUIvd1FGTUFNQkFmOHdEUVlKS29aSWh2Y05BUUVMQlFBRGdnRUIKQUl3WXpoY0s4OWtRL0xGWjZFRHgrQWp2bnJTVSs1cmdwQkgrRjVTNUUyY3pXOE5rNXhySnl0Y0ZUbUtlKzZScwpENHlxeTZSVVFEeWNYaDlPelBjbzgzYTBoeFlCZ1M5MWtJa25wYWF4dndLRDJleWc3UGNnK1lkS1FhZFlMcUY0CmI3cWVtc1FVVkpOWHdkZS9VanRBejlEOTh4dngwM2hQY2Qwb2dzUUhWZ21BZVpFd2l3UzFmTy9WNUE4dTl3MEkKcHlJRTVReXlHcHNpS2dpalpiMmhrS05RVHVJcEhiVnFydVA4eEV6TlFnamhkdS9uUW5OYy9lRUltVUlrQkFUVQpiSHdQc2xwYzVhdVV1TXJxR3lEQ0p2QUJpV3J2SmE3Yi9XcmtDT3FUWVhtR2NGM0w1ZU9FeTBhYkp0M2NNcSs5CnJLTUNVQWlkNG0yNEthWnc3OUk2anNBPQotLS0tLUVORCBDRVJUSUZJQ0FURS0tLS0tCg==" | base64 -d > ca-crt.txt
  openssl x509 -text -noout -in ca-crt.txt

.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル

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
        **省略**

.. code-block:: cmdin

  openssl x509 -text -noout -in client-cert.pem

.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル

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
          **省略**




以下の通り、各リソースを適切に作成されていることを確認します。

.. code-block:: cmdin

  kubectl get deployment

.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル

  NAME     READY   UP-TO-DATE   AVAILABLE   AGE
  webapp   1/1     1            1           8s

.. code-block:: cmdin

  kubectl get secret  | grep -e tls-secret

.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル

  ingress-mtls-secret   nginx.org/ca                          1      32s
  tls-secret            kubernetes.io/tls                     2      31s

.. code-block:: cmdin

  kubectl get policy

.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル

  NAME                  STATE   AGE
  ingress-mtls-policy   Valid   44s

.. code-block:: cmdin

  kubectl get vs

.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル

  NAME     STATE   HOST                 IP    PORTS   AGE
  webapp   Valid   webapp.example.com                 48s



動作確認
----

| curlコマンドの結果を確認します。
| クライアント証明書が要求(13)に対し、応答をしますが(17)、証明書が無いため署名データのやり取りは確認できません。サーバ証明書の内容がServer certificateに表示されています(24-28)。
| クライアント証明書が提示されなかったため、リクエストが拒否され、Webサーバから400 Bad Requestと共にエラーが応答されています(36,44)。

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

| curlコマンドの結果を確認します。
| クライアント証明書が要求され(13)、その要求に対し証明書を応答します(17,19)。サーバ証明書の内容がServer certificateに表示されています(25-29)。
| 正しくクライアントの証明書が評価できたため、リクエストが許可され、Webサーバから200 OKが応答されており(37)、正しいレスポンスが表示されています(46-50)

.. code-block:: cmdin

  curl -v -k --resolve webapp.example.com:443:127.0.0.1 https://webapp.example.com:443/ --cert ./client-cert.pem --key ./client-key.pem

.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル
  :emphasize-lines: 15,17,19,25,26,27,28,29,37,46,47,48,49,50

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
----

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
----

.. code-block:: cmdin

  cd ~/kubernetes-ingress/examples/custom-resources/egress-mtls
  kubectl apply -f secure-app.yaml
  kubectl apply -f egress-mtls-secret.yaml
  kubectl apply -f egress-trusted-ca-secret.yaml
  kubectl apply -f egress-mtls.yaml
  kubectl apply -f virtual-server.yaml

リソースを確認
----

| ファイルの内容を確認します。
| ``secure-app.yaml`` は、Kubernetes環境内で動作するアプリケーションで、クライアント証明書の評価を行います。ポイントとなる箇所を以下に示します。

- volumeMountsでそれぞれのPathにVolumeをマウントしています。/etc/nginx/sslに ``app-tls-secret`` というSecret(22,29)、/etc/nginx/conf.d/に ``secure-config`` というConfigMap(24,32)の内容がそれぞれマウントされます
- ``secure-config`` というConfigMapではNGINXの設定を指定します。SSLの終端(58,59)及び、クライアント証明書(61,62)の評価を行うよう設定を記述しています

.. code-block:: yaml
  :linenos:
  :caption: secure-app.yaml
  :emphasize-lines: 22,24,29,32,58,59,61,62

  apiVersion: apps/v1
  kind: Deployment
  metadata:
    name: secure-app
  spec:
    replicas: 1
    selector:
      matchLabels:
        app: secure-app
    template:
      metadata:
        labels:
          app: secure-app
      spec:
        containers:
          - name: secure-app
            image: nginxdemos/nginx-hello:plain-text
            ports:
              - containerPort: 8443
            volumeMounts:
              - name: secret
                mountPath: /etc/nginx/ssl
                readOnly: true
              - name: config-volume
                mountPath: /etc/nginx/conf.d
        volumes:
          - name: secret
            secret:
              secretName: app-tls-secret
          - name: config-volume
            configMap:
              name: secure-config
  ---
  apiVersion: v1
  kind: Service
  metadata:
    name: secure-app
  spec:
    ports:
      - port: 8443
        targetPort: 8443
        protocol: TCP
        name: https
    selector:
      app: secure-app
  ---
  apiVersion: v1
  kind: ConfigMap
  metadata:
    name: secure-config
  data:
    app.conf: |-
      server {
        listen 8443 ssl;
  
        server_name secure-app.example.com;
  
        ssl_certificate /etc/nginx/ssl/tls.crt;
        ssl_certificate_key /etc/nginx/ssl/tls.key;
  
        ssl_verify_client on;
        ssl_client_certificate /etc/nginx/ssl/ca.crt;
  
        default_type text/plain;
  
        location / {
          return 200 "hello from pod $hostname\n";
        }
      }
  ---
  apiVersion: v1
  kind: Secret
  metadata:
    name: app-tls-secret
  type: Opaque
  data:
    tls.crt: **省略**
    tls.key: **省略**
    ca.crt: **省略**


``egress-mtls.yaml`` は、VirtualServerに適用する EgressTlsのPolicyとなります。アプリケーションへ転送する際にの証明書として ``egress-mtls-secret`` 、 ``egress-trusted-ca-secret`` として作成したSecretを参照します(7,8)。

.. code-block:: yaml
  :linenos:
  :caption: egress-mtls.yaml
  :emphasize-lines: 7,8

  apiVersion: k8s.nginx.org/v1
  kind: Policy
  metadata:
    name: egress-mtls-policy
  spec:
    egressMTLS:
      tlsSecret: egress-mtls-secret
      trustedCertSecret: egress-trusted-ca-secret
      verifyServer: on
      verifyDepth: 2
      serverName: on
      sslName: secure-app.example.com

``virtual-server.yaml`` は、upstreamへtlsを有効にし(11,12)、routesで、 ``egress-mtls-policy`` を指定しています(14,15)。

.. code-block:: yaml
  :linenos:
  :caption: virtual-server.yaml
  :emphasize-lines: 11,12,14,15

  apiVersion: k8s.nginx.org/v1
  kind: VirtualServer
  metadata:
    name: webapp
  spec:
    host: webapp.example.com
    upstreams:
    - name: secure-app
      service: secure-app
      port: 8443
      tls:
        enable: true
    routes:
    - path: /
      policies:
      - name: egress-mtls-policy
      action:
        pass: secure-app


以下の通り、各リソースを適切に作成されていることを確認します。

.. code-block:: cmdin
    
  kubectl get deployment

.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル

  NAME         READY   UP-TO-DATE   AVAILABLE   AGE
  secure-app   1/1     1            1           73s

.. code-block:: cmdin
    
  kubectl get pod

.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル

  NAME                          READY   STATUS    RESTARTS   AGE
  secure-app-6dc947cc5f-8855b   1/1     Running   0          75s

.. code-block:: cmdin
    
  kubectl get svc | grep secure-app

.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル

  secure-app   ClusterIP   10.101.84.115   <none>        8443/TCP   5m17s
  

.. code-block:: cmdin
    
  kubectl get secret | grep -e app-tls -e egress

.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル

  app-tls-secret             Opaque                                3      6m53s
  egress-mtls-secret         kubernetes.io/tls                     2      6m48s
  egress-trusted-ca-secret   nginx.org/ca                          1      6m42s



動作確認
----

.. code-block:: cmdin

  curl -v -H "Host:webapp.example.com" http://localhost/

.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル

  *   Trying 127.0.0.1:80...
  * TCP_NODELAY set
  * Connected to localhost (127.0.0.1) port 80 (#0)
  > GET / HTTP/1.1
  > Host:webapp.example.com
  > User-Agent: curl/7.68.0
  > Accept: */*
  >
  * Mark bundle as not supporting multiuse
  < HTTP/1.1 200 OK
  < Server: nginx/1.21.3
  < Date: Wed, 19 Jan 2022 15:14:03 GMT
  < Content-Type: text/plain
  < Content-Length: 43
  < Connection: keep-alive
  <
  hello from pod secure-app-6dc947cc5f-8855b
  * Connection #0 to host localhost left intact


リソースの削除
----

.. code-block:: cmdin

  ## cd ~/kubernetes-ingress/examples/custom-resources/egress-mtls
  kubectl delete -f secure-app.yaml
  kubectl delete -f egress-mtls-secret.yaml
  kubectl delete -f egress-trusted-ca-secret.yaml
  kubectl delete -f egress-mtls.yaml
  kubectl delete -f virtual-server.yaml