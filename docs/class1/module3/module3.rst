NGINX Ingress Controller の動作確認
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
    
    ** 実行結果サンプル **
    deployment.apps/coffee created
    service/coffee-svc created
    deployment.apps/tea created
    service/tea-svc created


    kubectl create -f cafe-secret.yaml
    
    ** 実行結果サンプル **
    secret/cafe-secret created
    
    
    kubectl create -f cafe-virtual-server.yaml
    
    ** 実行結果サンプル **
    virtualserver.k8s.nginx.org/cafe created


作成したリソースを確認

::

    ## cd ~/kubernetes-ingress/examples/custom-resources/basic-configuration/
    kubectl get virtualserver cafe
    
    ** 実行結果サンプル **
    NAME   STATE   HOST               IP    PORTS   AGE
    cafe   Valid   cafe.example.com                 16s

    kubectl get deployment

    ** 実行結果サンプル **
    NAME     READY   UP-TO-DATE   AVAILABLE   AGE
    coffee   2/2     2            2           1m
    tea      1/1     1            1           1m


    kubectl get secret  | grep cafe

    ** 実行結果サンプル **
    cafe-secret           kubernetes.io/tls                     2      1m



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
    
    ** 実行結果サンプル **
    namespace/cafe created
    namespace/tea created
    namespace/coffee created
    
    
    kubectl create -f tea.yaml
    
    ** 実行結果サンプル **
    deployment.apps/tea created
    service/tea-svc created
    
    
    kubectl create -f coffee.yaml
    
    ** 実行結果サンプル **
    deployment.apps/coffee created
    service/coffee-svc created
    
    
    kubectl create -f tea-virtual-server-route.yaml
    
    ** 実行結果サンプル **
    virtualserverroute.k8s.nginx.org/tea created
    
    
    kubectl create -f coffee-virtual-server-route.yaml
    
    ** 実行結果サンプル **
    virtualserverroute.k8s.nginx.org/coffee created
    
    
    kubectl create -f cafe-secret.yaml
    
    ** 実行結果サンプル **
    secret/cafe-secret created
    
    
    kubectl create -f cafe-virtual-server.yaml
    
    ** 実行結果サンプル **
    virtualserver.k8s.nginx.org/cafe created

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

    ** 実行結果サンプル **
    deployment.apps/coffee-v1 created
    service/coffee-v1-svc created
    deployment.apps/coffee-v2 created
    service/coffee-v2-svc created
    deployment.apps/tea-post created
    service/tea-post-svc created
    deployment.apps/tea created
    service/tea-svc created

    kubectl create -f cafe-virtual-server.yaml

    ** 実行結果サンプル **
    virtualserver.k8s.nginx.org/cafe created

リソースを確認
** 実行結果サンプル **

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
リソースを確認
動作確認
リソースの削除
** 実行結果サンプル **


IPアドレスによる通信の制御 (Access Control)
====

https://github.com/nginxinc/kubernetes-ingress/tree/v2.1.0/examples/custom-resources/access-control



URL Path の 変換 (Rewrite)
====

https://github.com/nginxinc/kubernetes-ingress/tree/v2.1.0/examples/custom-resources/rewrites

TCP / UDP の分散設定
====

https://github.com/nginxinc/kubernetes-ingress/tree/v2.1.0/examples/custom-resources/basic-tcp-udp

Ingress Controller で JWT Validation のデプロイ
====

https://github.com/nginxinc/kubernetes-ingress/tree/v2.1.0/examples/custom-resources/jwt

gRPC
====

https://github.com/nginxinc/kubernetes-ingress/tree/v2.1.0/examples/grpc-services


Custom Log Format
====

https://github.com/nginxinc/kubernetes-ingress/tree/v2.1.0/examples/custom-log-format


Ingress Controller で OIDC RPのデプロイ
====

https://github.com/nginxinc/kubernetes-ingress/tree/v2.1.0/examples/custom-resources/oidc


Ingress Controller で WAF機能(NGINX App Protect WAF) のデプロイ
====

https://github.com/nginxinc/kubernetes-ingress/tree/v2.1.0/examples/custom-resources/waf


Ingress Controller で 高度なDoS対策機能(NGINX App Protect DoS) のデプロイ
====

https://github.com/nginxinc/kubernetes-ingress/tree/v2.1.0/examples/custom-resources/dos



サンプルアプリケーションをデプロイ
リソースを確認
動作確認
リソースの削除
** 実行結果サンプル **
