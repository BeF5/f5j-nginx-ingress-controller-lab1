NICによるWebアプリの通信制御
####

この章では、実際にデプロイしたNGINX Ingress Controllerを使い、様々なサンプルアプリケーションを動作させ、その設定方法や動きを確認いただきます。
設定例は `NGINX Inc GitHubの examples/custom-resources/ <https://github.com/nginxinc/kubernetes-ingress/tree/v2.1.0/examples/custom-resources>`__ に管理されております

シンプルなWebアプリケーションのデプロイ
====

https://github.com/nginxinc/kubernetes-ingress/tree/v2.1.0/examples/custom-resources/basic-configuration

サンプルアプリケーション、NGINX Ingress Controller の設定をデプロイ

::
    
    cd ~/kubernetes-ingress/examples/custom-resources/basic-configuration/
    kubectl create -f cafe.yaml
    kubectl create -f cafe-secret.yaml
    kubectl create -f cafe-virtual-server.yaml

作成したリソースを確認

::

    ## cd ~/kubernetes-ingress/examples/custom-resources/basic-configuration/
    kubectl get deployment

    ** 実行結果サンプル **
    NAME     READY   UP-TO-DATE   AVAILABLE   AGE
    coffee   2/2     2            2           1m
    tea      1/1     1            1           1m

    kubectl get secret  | grep cafe-secret

    ** 実行結果サンプル **
    cafe-secret           kubernetes.io/tls                     2      1m

    kubectl get vs
    
    ** 実行結果サンプル **
    NAME   STATE   HOST               IP    PORTS   AGE
    cafe   Valid   cafe.example.com                 94s


動作確認

::

    curl -H "Host:cafe.example.com" http://localhost/coffee

    ** 実行結果サンプル **    
    Server address: 192.168.127.25:8080
    Server name: coffee-7c86d7d67c-wjxss
    Date: 17/Jan/2022:00:14:03 +0000
    URI: /coffee
    Request ID: 069567120c306da6f92e16e5d73e5040


    curl -H "Host:cafe.example.com" http://localhost/tea

    ** 実行結果サンプル **
    Server address: 192.168.127.20:8080
    Server name: tea-5c457db9-dc4cs
    Date: 17/Jan/2022:00:14:08 +0000
    URI: /tea
    Request ID: 6fd58877d9e85903300df7ceb0f81eb2

    curl -kv -H "Host:cafe.example.com" https://localhost/coffee

    ** 実行結果サンプル **
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

    curl -kv -H "Host:cafe.example.com" https://localhost/tea

    ** 実行結果サンプル **
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

::

    ## cd ~/kubernetes-ingress/examples/custom-resources/basic-configuration/
    kubectl delete -f cafe-secret.yaml
    kubectl delete -f cafe-virtual-server.yaml
    kubectl delete -f cafe.yaml


複数アプリケーション・チームを想定した VS / VSR 設定
====

https://github.com/nginxinc/kubernetes-ingress/tree/v2.1.0/examples/custom-resources/cross-namespace-configuration

この章ではシンプルなWebアプリケーションをデプロイします。
NGINXはCRDを用い、Virtual Server / Virtual Server Router / Policy といったリソースを使うことで、権限と設定範囲を適切に管理することが可能です。

サンプルアプリケーションをデプロイ

::
    
    kubectl create -f namespaces.yaml
    kubectl create -f tea.yaml
    kubectl create -f coffee.yaml
    kubectl create -f tea-virtual-server-route.yaml
    kubectl create -f coffee-virtual-server-route.yaml
    kubectl create -f cafe-secret.yaml
    kubectl create -f cafe-virtual-server.yaml

リソースを確認

::
        
    kubectl get ns --sort-by=.metadata.creationTimestamp
    
    ** 実行結果サンプル **
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
    
    
    kubectl get vsr -A
    
    ** 実行結果サンプル **
    NAMESPACE   NAME     STATE   HOST               IP    PORTS   AGE
    coffee      coffee   Valid   cafe.example.com                 89s
    tea         tea      Valid   cafe.example.com                 93s
    
    
    kubectl get vs -A
    
    ** 実行結果サンプル **
    NAMESPACE   NAME   STATE   HOST               IP    PORTS   AGE
    cafe        cafe   Valid   cafe.example.com                 85s
    
    
    kubectl get secret -A | grep cafe
    
    ** 実行結果サンプル **
    cafe               cafe-secret                                      kubernetes.io/tls                     2      101s
    cafe               default-token-94nrl                              kubernetes.io/service-account-token   3      2m3s
    
    
    kubectl get secret -A | grep cafe-secret
    
    ** 実行結果サンプル **
    NAME                  TYPE                                  DATA   AGE
    cafe-secret           kubernetes.io/tls                     2      2m5s
    
    
    kubectl get pod -o wide -A|grep -e coffee -e tea
    
    ** 実行結果サンプル **
    coffee             coffee-7c86d7d67c-pq5w2                    1/1     Running   0                88s   192.168.127.22   ip-10-1-1-9   <none>           <none>
    tea                tea-5c457db9-h5sm9                         1/1     Running   0                14m   192.168.127.24   ip-10-1-1-9   <none>           <none>


動作確認

::
        
    curl -H "Host: cafe.example.com" http://localhost/coffee
    
    ** 実行結果サンプル **
    Server address: 192.168.127.22:8080
    Server name: coffee-7c86d7d67c-pq5w2
    Date: 17/Jan/2022:05:44:25 +0000
    URI: /coffee
    Request ID: 1414627aac091b5a7897bac37d046cea
    
    
    curl -H "Host: cafe.example.com" http://localhost/tea
    
    ** 実行結果サンプル **
    Server address: 192.168.127.24:8080
    Server name: tea-5c457db9-h5sm9
    Date: 17/Jan/2022:05:44:29 +0000
    URI: /tea
    Request ID: 698ab29da633f24a9bf5384c1499b056
    
    
    curl -vk -H "Host: cafe.example.com" https://localhost/tea
    
    ** 実行結果サンプル **
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

::

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

::
    
    cd ~/kubernetes-ingress/examples/custom-resources/advanced-routing
    kubectl create -f cafe.yaml
    kubectl create -f cafe-virtual-server.yaml

リソースを確認

::

    kubectl get deployment

    ** 実行結果サンプル **
    NAME        READY   UP-TO-DATE   AVAILABLE   AGE
    coffee-v1   1/1     1            1           16s
    coffee-v2   1/1     1            1           15s
    tea         1/1     1            1           15s
    tea-post    1/1     1            1           15s

    kubectl get pod -o wide

    ** 実行結果サンプル **
    NAME                         READY   STATUS    RESTARTS   AGE   IP               NODE          NOMINATED NODE   READINESS GATES
    coffee-v1-6b78998db9-8cv49   1/1     Running   0          26s   192.168.127.23   ip-10-1-1-9   <none>           <none>
    coffee-v2-748cbbb49f-mbxpr   1/1     Running   0          26s   192.168.127.27   ip-10-1-1-9   <none>           <none>
    tea-5c457db9-dcswc           1/1     Running   0          26s   192.168.127.33   ip-10-1-1-9   <none>           <none>
    tea-post-7db8cd8bf-m5gbz     1/1     Running   0          26s   192.168.127.32   ip-10-1-1-9   <none>           <none>

    kubectl get vs

    ** 実行結果サンプル **
    NAME   STATE   HOST               IP    PORTS   AGE
    cafe   Valid   cafe.example.com                 28s



動作確認

::

    curl -H "Host: cafe.example.com" http://localhost/tea

    ** 実行結果サンプル **
    Server address: 192.168.127.33:8080
    Server name: tea-5c457db9-dcswc
    Date: 17/Jan/2022:09:00:56 +0000
    URI: /tea
    Request ID: 00e9eb4d61f7afdb8c5656da94d15b98

    curl -H "Host: cafe.example.com" http://localhost/tea -X POST

    ** 実行結果サンプル **
    Server address: 192.168.127.32:8080
    Server name: tea-post-7db8cd8bf-m5gbz
    Date: 17/Jan/2022:09:01:02 +0000
    URI: /tea
    Request ID: 4deeb82434a6f799ffc894a229ac361a

    curl -H "Host: cafe.example.com" http://localhost/coffee

    ** 実行結果サンプル **
    Server address: 192.168.127.23:8080
    Server name: coffee-v1-6b78998db9-8cv49
    Date: 17/Jan/2022:09:01:25 +0000
    URI: /coffee
    Request ID: 8d182c9c060d5a4d4dec226292ac2820

    curl -H "Host: cafe.example.com" http://localhost/coffee --cookie "version=v2"

    ** 実行結果サンプル **
    Server address: 192.168.127.27:8080
    Server name: coffee-v2-748cbbb49f-mbxpr
    Date: 17/Jan/2022:09:01:35 +0000
    URI: /coffee
    Request ID: befacc5e7ca56a1a09e5982315c74fa0

リソースの削除

::

    kubectl delete  -f cafe-virtual-server.yaml
    kubectl delete  -f cafe.yaml


割合を指定した分散 (Traffic Split)
====

https://github.com/nginxinc/kubernetes-ingress/tree/v2.1.0/examples/custom-resources/traffic-splitting

サンプルアプリケーションをデプロイ

::

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

::
    
    kubectl get deployment
    
    ** 実行結果サンプル **
    NAME        READY   UP-TO-DATE   AVAILABLE   AGE
    coffee-v1   2/2     2            2           19s
    coffee-v2   2/2     2            2           19s
    
    
    kubectl get pod -o wide
    
    ** 実行結果サンプル **
    NAME                         READY   STATUS    RESTARTS   AGE   IP               NODE          NOMINATED NODE   READINESS GATES
    coffee-v1-6b78998db9-h4jkb   1/1     Running   0          25s   192.168.127.47   ip-10-1-1-9   <none>           <none>
    coffee-v1-6b78998db9-nn42z   1/1     Running   0          25s   192.168.127.44   ip-10-1-1-9   <none>           <none>
    coffee-v2-748cbbb49f-llpb6   1/1     Running   0          25s   192.168.127.45   ip-10-1-1-9   <none>           <none>
    coffee-v2-748cbbb49f-vrpzx   1/1     Running   0          25s   192.168.127.46   ip-10-1-1-9   <none>           <none>
    
    
    kubectl get vs
    
    ** 実行結果サンプル **
    NAME   STATE   HOST               IP    PORTS   AGE
    cafe   Valid   cafe.example.com                 26s


動作確認

::
    
    curl -s -H "Host: cafe.example.com" http://localhost/coffee
    
    ** 実行結果サンプル **
    Server address: 192.168.127.44:8080
    Server name: coffee-v1-6b78998db9-nn42z
    Date: 17/Jan/2022:12:26:49 +0000
    URI: /coffee
    Request ID: c127f0f724eb1b3becd57603b6d603ea
    
    curl -s -H "Host: cafe.example.com" http://localhost/coffee
    
    ** 実行結果サンプル **
    Server address: 192.168.127.45:8080
    Server name: coffee-v2-748cbbb49f-llpb6
    Date: 17/Jan/2022:12:26:37 +0000
    URI: /coffee
    Request ID: 357237a3fea498b6efd90c929d526e64


::

    ## cd ~/kubernetes-ingress/examples/custom-resources/traffic-splitting
    > split.txt ;\
    for i in {1..20}; \
    do curl -s -H "Host: cafe.example.com" http://localhost/coffee | grep "Server name" >> split.txt ; \
    done ; \
    echo -n "v1:" ; grep v1 split.txt  | wc -l ; echo -n "v2:"  ; grep v2 split.txt  | wc -l
    
    ** 実行結果サンプル **
    v1:18
    v2:2


リソースの削除

::
    
    ## cd ~/kubernetes-ingress/examples/custom-resources/traffic-splitting
    kubectl delete -f cafe-virtual-server.yaml
    kubectl delete -f cafe.yaml
    rm split.txt



IPアドレスによる通信の制御 (Access Control)
====

https://github.com/nginxinc/kubernetes-ingress/tree/v2.1.0/examples/custom-resources/access-control


サンプルアプリケーションをデプロイ

::

    cd ~/kubernetes-ingress/examples/custom-resources/access-control
    kubectl apply -f webapp.yaml
    kubectl apply -f access-control-policy-deny.yaml
    kubectl apply -f virtual-server.yaml

リソースを確認

::

    kubectl get pod
    
    ** 実行結果サンプル **
    NAME                     READY   STATUS    RESTARTS   AGE
    webapp-64d444885-j4q7z   1/1     Running   0          2m7s
    
    kubectl get deployment
    
    ** 実行結果サンプル **
    NAME     READY   UP-TO-DATE   AVAILABLE   AGE
    webapp   1/1     1            1           2m13s
    
    kubectl get vs
    
    ** 実行結果サンプル **
    NAME     STATE   HOST                 IP    PORTS   AGE
    webapp   Valid   webapp.example.com                 2m8s
    
    kubectl get policy
    
    ** 実行結果サンプル **
    NAME            STATE   AGE
    webapp-policy   Valid   2m18s
    
    kubectl describe vs
    
    ** 実行結果サンプル **
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

::
    
    kubectl describe policy
    
    ** 実行結果サンプル **
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

::

    curl -H "Host:webapp.example.com" http://localhost/

    ** 実行結果サンプル **
    <html>
    <head><title>403 Forbidden</title></head>
    <body>
    <center><h1>403 Forbidden</h1></center>
    <hr><center>nginx/1.21.3</center>
    </body>
    </html>

``webapp-policy`` の内容を変更します

::
    
    ## cd ~/kubernetes-ingress/examples/custom-resources/access-control
    kubectl apply -f access-control-policy-allow.yaml

    ** 実行結果サンプル **
    policy.k8s.nginx.org/webapp-policy configured


コマンドを実行しPolicyの内容を確認します。Policyの内容が ``Spec`` に記載されています。

::
    
    kubectl describe policy
    
    ** 実行結果サンプル **
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

::
    
    curl -H "Host:webapp.example.com" http://localhost/

    ** 実行結果サンプル **
    Server address: 192.168.127.48:8080
    Server name: webapp-64d444885-j4q7z
    Date: 17/Jan/2022:12:48:51 +0000
    URI: /
    Request ID: 752997339b21d94210fc911cb41f7216
    

リソースの削除

::
    
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

::
    
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

::

    kubectl get pod

    ** 実行結果サンプル **
    NAME                      READY   STATUS    RESTARTS   AGE
    coffee-7c86d7d67c-ws2t8   1/1     Running   0          39m
    coffee-7c86d7d67c-zt5tr   1/1     Running   0          39m
    tea-5c457db9-ksljs        1/1     Running   0          39m

    kubectl get deployment

    ** 実行結果サンプル **
    NAME     READY   UP-TO-DATE   AVAILABLE   AGE
    coffee   2/2     2            2           39m
    tea      1/1     1            1           39m

    kubectl get vs

    ** 実行結果サンプル **
    NAME   STATE   HOST               IP    PORTS   AGE
    cafe   Valid   cafe.example.com                 39m


動作確認

::

    curl -H "Host:cafe.example.com" http://localhost/tea/

    ** 実行結果サンプル **
    Server address: 192.168.127.40:8080
    Server name: tea-5c457db9-ksljs
    Date: 17/Jan/2022:14:22:46 +0000
    URI: /
    Request ID: 2576a16546e7d17467e04da2ab794109

    curl -H "Host:cafe.example.com" http://localhost/tea/abc

    ** 実行結果サンプル **
    Server address: 192.168.127.40:8080
    Server name: tea-5c457db9-ksljs
    Date: 17/Jan/2022:14:22:14 +0000
    URI: /abc
    Request ID: 5ce49a600fb24a40340ba6edad91ffb2

    curl -H "Host:cafe.example.com" http://localhost/coffee

    ** 実行結果サンプル **
    Server address: 192.168.127.39:8080
    Server name: coffee-7c86d7d67c-zt5tr
    Date: 17/Jan/2022:14:22:40 +0000
    URI: /beans
    Request ID: 9b15d10a624faee145b875b8f83460e3

    curl -H "Host:cafe.example.com" http://localhost/coffee/def/ghi

    ** 実行結果サンプル **
    Server address: 192.168.127.39:8080
    Server name: coffee-7c86d7d67c-zt5tr
    Date: 17/Jan/2022:14:22:27 +0000
    URI: /beans/def/ghi
    Request ID: f70d98547c615a145b2a40ddfe5884a4
    
    curl -H "Host:cafe.example.com" http://localhost/cafe/top.jpg
    
    ** 実行結果サンプル **
    Server address: 192.168.127.40:8080
    Server name: tea-5c457db9-ksljs
    Date: 17/Jan/2022:14:23:02 +0000
    URI: /service/cafe/image/top.jpg
    Request ID: 38c3cf24e3f5e0cdfe451b0d646c0e1d
   

リソースの削除

::
    
    ## cd ~/kubernetes-ingress/examples/custom-resources/rewrites
 
    kubectl delete -f ../basic-configuration/cafe.yaml
    kubectl delete -f rewrite-virtual-server.yaml


Ingress Controller で JWT Validation のデプロイ
====

https://github.com/nginxinc/kubernetes-ingress/tree/v2.1.0/examples/custom-resources/jwt

サンプルアプリケーションをデプロイ

::

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

::

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

::

    echo -n "ZmFudGFzdGljand0" | base64 -d

    ** 実行結果サンプル **
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

::
   
    kubectl get deployment
    
    ** 実行結果サンプル **
    NAME     READY   UP-TO-DATE   AVAILABLE   AGE
    webapp   1/1     1            1           23s
    
    kubectl get secret | grep jwk
    
    ** 実行結果サンプル **
    jwk-secret            nginx.org/jwk                         1      40s
    
    kubectl get policy
    
    ** 実行結果サンプル **
    NAME         STATE   AGE
    jwt-policy   Valid   38s
    
    kubectl get vs
    
    ** 実行結果サンプル **
    NAME     STATE   HOST                 IP    PORTS   AGE
    webapp   Valid   webapp.example.com                 35s
    

動作確認

Policyが適用されたVSにJWTをHeaderに付与していないため、通信に対し ``401 Authorization required`` が応答されていることを確認します

::

    curl -H "Host:webapp.example.com" http://localhost/
    
    ** 実行結果サンプル **
    <html>
    <head><title>401 Authorization Required</title></head>
    <body>
    <center><h1>401 Authorization Required</h1></center>
    <hr><center>nginx/1.21.3</center>
    </body>
    </html>

curlコマンドで動作を確認します。以下のように通信が ``許可`` されていることが確認できます

::

    curl -H "Host:webapp.example.com" http://localhost/ -H "Token: `cat token.jwt`"
    
    ** 実行結果サンプル **
    Server address: 192.168.127.57:8080
    Server name: webapp-64d444885-r5fnt
    Date: 18/Jan/2022:12:49:59 +0000
    URI: /
    Request ID: 86182122eec0392769b4d86d64653419
    cat token.jwt
    eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiIsImtpZCI6IjAwMDEifQ.eyJuYW1lIjoiUXVvdGF0aW9uIFN5c3RlbSIsInN1YiI6InF1b3RlcyIsImlzcyI6Ik15IEFQSSBHYXRld2F5In0.ggVOHYnVFB8GVPE-VOIo3jD71gTkLffAY0hQOGXPL2I


リソースの削除

::

    ## cd ~/kubernetes-ingress/examples/custom-resources/jwt/

    kubectl delete -f virtual-server.yaml
    kubectl delete -f jwt.yaml
    kubectl delete -f jwk-secret.yaml
    kubectl delete -f webapp.yaml

Ingress Controller で OIDC RPのデプロイ
====

https://github.com/nginxinc/kubernetes-ingress/tree/v2.1.0/examples/custom-resources/oidc



Ingress MTLS
====

https://github.com/nginxinc/kubernetes-ingress/tree/v2.1.0/examples/custom-resources/ingress-mtls


Egress MTLS
====

https://github.com/nginxinc/kubernetes-ingress/tree/v2.1.0/examples/custom-resources/egress-mtls



サンプルアプリケーションをデプロイ
リソースを確認
動作確認
リソースの削除
** 実行結果サンプル **

