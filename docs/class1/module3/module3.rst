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



複数アプリケーション・チームを想定した VS / VSR 設定
====

https://github.com/nginxinc/kubernetes-ingress/tree/v2.1.0/examples/custom-resources/cross-namespace-configuration

この章ではシンプルなWebアプリケーションをデプロイします。
NGINXはCRDを用い、Virtual Server / Virtual Server Router / Policy といったリソースを使うことで、権限と設定範囲を適切に管理することが可能です。

通信内容による条件分岐・サービスへの転送
====

https://github.com/nginxinc/kubernetes-ingress/tree/v2.1.0/examples/custom-resources/advanced-routing


割合を指定した分散 (Traffic Split)
====

https://github.com/nginxinc/kubernetes-ingress/tree/v2.1.0/examples/custom-resources/traffic-splitting

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


Ingress Controller で OIDC RPのデプロイ
====

https://github.com/nginxinc/kubernetes-ingress/tree/v2.1.0/examples/custom-resources/oidc


Ingress Controller で WAF機能(NGINX App Protect WAF) のデプロイ
====

https://github.com/nginxinc/kubernetes-ingress/tree/v2.1.0/examples/custom-resources/waf


Ingress Controller で 高度なDoS対策機能(NGINX App Protect DoS) のデプロイ
====

https://github.com/nginxinc/kubernetes-ingress/tree/v2.1.0/examples/custom-resources/dos