Kubernetes Gateway による通信制御
####

NGINXをDataplaneとしてKubernetes Gatewayを利用する方法を紹介します。
Kubernetes Gatewayの情報は `GitHub: nginx-kubernetes-gateway <https://github.com/nginxinc/nginx-kubernetes-gateway>`__ を参照してください

.. NOTE::
  本資料作成時点では、NGINX OSSを利用した動作確認となります

この章では、実際にデプロイしたNGINX Kubernetes Gatewayを使った通信制御方法を確認します
設定例は `GitHub nginx-kubernetes-gateway/examples <https://github.com/nginxinc/nginx-kubernetes-gateway/tree/main/examples>`__ の内容となります


シンプルなWebアプリケーション
====

シンプルなWebアプリケーションをデプロイします。

https://github.com/nginxinc/nginx-kubernetes-gateway/tree/main/examples/cafe-example

サンプルアプリケーションをデプロイ
----

.. code-block:: cmdin
 
  cd ~/nginx-kubernetes-gateway/examples/cafe-example/
  kubectl apply -f cafe.yaml
  kubectl apply -f gateway.yaml
  kubectl apply -f cafe-routes.yaml

リソースの確認
----

作成したリソースの内容を確認します

.. code-block:: cmdin
 
  cat gateway.yaml

.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル

  apiVersion: gateway.networking.k8s.io/v1beta1
  kind: Gateway
  metadata:
    name: gateway
    labels:
      domain: k8s-gateway.nginx.org
  spec:
    gatewayClassName: nginx
    listeners:
    - name: http
      port: 80
      protocol: HTTP

.. code-block:: cmdin
 
  cat gateway.yaml

.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル

  apiVersion: gateway.networking.k8s.io/v1beta1
  kind: HTTPRoute
  metadata:
    name: coffee
  spec:
    parentRefs:
    - name: gateway
      sectionName: http
    hostnames:
    - "cafe.example.com"
    rules:
    - matches:
      - path:
          type: PathPrefix
          value: /coffee
      backendRefs:
      - name: coffee
        port: 80
  ---
  apiVersion: gateway.networking.k8s.io/v1beta1
  kind: HTTPRoute
  metadata:
    name: tea
  spec:
    parentRefs:
    - name: gateway
      sectionName: http
    hostnames:
    - "cafe.example.com"
    rules:
    - matches:
      - path:
          type: Exact
          value: /tea
      backendRefs:
      - name: tea


正しくリソースが作成されたことを確認します

.. code-block:: cmdin
 
  kubectl get gateway

.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル

  NAME      CLASS   ADDRESS         PROGRAMMED   AGE
  gateway   nginx   192.168.127.2                54s

httproute を確認します。 ``cafe.example.com`` のHostnameに対し、 ``coffee`` と ``tea`` がデプロイされています

.. code-block:: cmdin
 
  kubectl get httproute

.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル

  NAME     HOSTNAMES              AGE
  coffee   ["cafe.example.com"]   57s
  tea      ["cafe.example.com"]   57s

coffee と tea の Podがデプロイされていることが確認できます

.. code-block:: cmdin
 
  kubectl get pod

.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル

  NAME                      READY   STATUS    RESTARTS        AGE
  coffee-7c86d7d67c-dxw9s   1/1     Running   1 (152m ago)    11h
  tea-5c457db9-wrtn6        1/1     Running   1 (3h34m ago)   11h


リソースの詳細を確認します

Gatewayは通信の待ち受けに関する設定です

.. code-block:: cmdin
 
  kubectl describe gateway gateway

.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル

  Name:         gateway
  Namespace:    default
  Labels:       domain=k8s-gateway.nginx.org
  Annotations:  <none>
  API Version:  gateway.networking.k8s.io/v1beta1
  Kind:         Gateway
  Metadata:
    Creation Timestamp:  2023-05-25T01:18:02Z
    Generation:          1
    Managed Fields:
      API Version:  gateway.networking.k8s.io/v1beta1
      Fields Type:  FieldsV1
      fieldsV1:
        f:metadata:
          f:annotations:
            .:
            f:kubectl.kubernetes.io/last-applied-configuration:
          f:labels:
            .:
            f:domain:
        f:spec:
          .:
          f:gatewayClassName:
          f:listeners:
            .:
            k:{"name":"http"}:
              .:
              f:allowedRoutes:
                .:
                f:namespaces:
                  .:
                  f:from:
              f:name:
              f:port:
              f:protocol:
      Manager:      kubectl-client-side-apply
      Operation:    Update
      Time:         2023-05-25T01:18:02Z
      API Version:  gateway.networking.k8s.io/v1beta1
      Fields Type:  FieldsV1
      fieldsV1:
        f:status:
          f:addresses:
          f:conditions:
            k:{"type":"Accepted"}:
              f:lastTransitionTime:
              f:message:
              f:observedGeneration:
              f:reason:
              f:status:
          f:listeners:
            .:
            k:{"name":"http"}:
              .:
              f:attachedRoutes:
              f:conditions:
                .:
                k:{"type":"Accepted"}:
                  .:
                  f:lastTransitionTime:
                  f:message:
                  f:observedGeneration:
                  f:reason:
                  f:status:
                  f:type:
                k:{"type":"Conflicted"}:
                  .:
                  f:lastTransitionTime:
                  f:message:
                  f:observedGeneration:
                  f:reason:
                  f:status:
                  f:type:
                k:{"type":"ResolvedRefs"}:
                  .:
                  f:lastTransitionTime:
                  f:message:
                  f:observedGeneration:
                  f:reason:
                  f:status:
                  f:type:
              f:name:
              f:supportedKinds:
      Manager:         gateway
      Operation:       Update
      Subresource:     status
      Time:            2023-05-25T01:18:03Z
    Resource Version:  204507
    UID:               b680933d-a7a2-4780-a89e-c5e751abb971
  Spec:
    Gateway Class Name:  nginx
    Listeners:
      Allowed Routes:
        Namespaces:
          From:  Same
      Name:      http
      Port:      80
      Protocol:  HTTP
  Status:
    Addresses:
      Type:   IPAddress
      Value:  192.168.127.2
    Conditions:
      Last Transition Time:  2023-05-25T01:18:08Z
      Message:               Gateway is accepted
      Observed Generation:   1
      Reason:                Accepted
      Status:                True
      Type:                  Accepted
    Listeners:
      Attached Routes:  2
      Conditions:
        Last Transition Time:  2023-05-25T01:18:08Z
        Message:               Listener is accepted
        Observed Generation:   1
        Reason:                Accepted
        Status:                True
        Type:                  Accepted
        Last Transition Time:  2023-05-25T01:18:08Z
        Message:               All references are resolved
        Observed Generation:   1
        Reason:                ResolvedRefs
        Status:                True
        Type:                  ResolvedRefs
        Last Transition Time:  2023-05-25T01:18:08Z
        Message:               No conflicts
        Observed Generation:   1
        Reason:                NoConflicts
        Status:                False
        Type:                  Conflicted
      Name:                    http
      Supported Kinds:
        Group:  gateway.networking.k8s.io
        Kind:   HTTPRoute
  Events:       <none>

HTTP RouteはHTTP通信の転送に関連するリソースです。
``coffee`` の HTTP Routeの内容が以下です

.. code-block:: cmdin
 
  kubectl describe httproute coffee

.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル

  Name:         coffee
  Namespace:    default
  Labels:       <none>
  Annotations:  <none>
  API Version:  gateway.networking.k8s.io/v1beta1
  Kind:         HTTPRoute
  Metadata:
    Creation Timestamp:  2023-05-25T01:18:06Z
    Generation:          1
    Managed Fields:
      API Version:  gateway.networking.k8s.io/v1beta1
      Fields Type:  FieldsV1
      fieldsV1:
        f:metadata:
          f:annotations:
            .:
            f:kubectl.kubernetes.io/last-applied-configuration:
        f:spec:
          .:
          f:hostnames:
          f:parentRefs:
          f:rules:
      Manager:      kubectl-client-side-apply
      Operation:    Update
      Time:         2023-05-25T01:18:06Z
      API Version:  gateway.networking.k8s.io/v1beta1
      Fields Type:  FieldsV1
      fieldsV1:
        f:status:
          .:
          f:parents:
      Manager:         gateway
      Operation:       Update
      Subresource:     status
      Time:            2023-05-25T01:18:07Z
    Resource Version:  204508
    UID:               126217a6-b7d4-4dc4-bceb-b969bdb94194
  Spec:
    Hostnames:
      cafe.example.com
    Parent Refs:
      Group:         gateway.networking.k8s.io
      Kind:          Gateway
      Name:          gateway
      Section Name:  http
    Rules:
      Backend Refs:
        Group:
        Kind:    Service
        Name:    coffee
        Port:    80
        Weight:  1
      Matches:
        Path:
          Type:   PathPrefix
          Value:  /coffee
  Status:
    Parents:
      Conditions:
        Last Transition Time:  2023-05-25T01:18:08Z
        Message:               The route is accepted
        Observed Generation:   1
        Reason:                Accepted
        Status:                True
        Type:                  Accepted
        Last Transition Time:  2023-05-25T01:18:08Z
        Message:               All references are resolved
        Observed Generation:   1
        Reason:                ResolvedRefs
        Status:                True
        Type:                  ResolvedRefs
      Controller Name:         k8s-gateway.nginx.org/nginx-gateway-controller
      Parent Ref:
        Group:         gateway.networking.k8s.io
        Kind:          Gateway
        Name:          gateway
        Namespace:     default
        Section Name:  http
  Events:              <none>

``tea`` の HTTP Routeの内容が以下です

.. code-block:: cmdin
 
  kubectl describe httproute tea

.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル

  Name:         tea
  Namespace:    default
  Labels:       <none>
  Annotations:  <none>
  API Version:  gateway.networking.k8s.io/v1beta1
  Kind:         HTTPRoute
  Metadata:
    Creation Timestamp:  2023-05-25T01:18:06Z
    Generation:          1
    Managed Fields:
      API Version:  gateway.networking.k8s.io/v1beta1
      Fields Type:  FieldsV1
      fieldsV1:
        f:metadata:
          f:annotations:
            .:
            f:kubectl.kubernetes.io/last-applied-configuration:
        f:spec:
          .:
          f:hostnames:
          f:parentRefs:
          f:rules:
      Manager:      kubectl-client-side-apply
      Operation:    Update
      Time:         2023-05-25T01:18:06Z
      API Version:  gateway.networking.k8s.io/v1beta1
      Fields Type:  FieldsV1
      fieldsV1:
        f:status:
          .:
          f:parents:
      Manager:         gateway
      Operation:       Update
      Subresource:     status
      Time:            2023-05-25T01:18:08Z
    Resource Version:  204509
    UID:               901df757-fc2a-4d2d-9e9f-c36253cbdd19
  Spec:
    Hostnames:
      cafe.example.com
    Parent Refs:
      Group:         gateway.networking.k8s.io
      Kind:          Gateway
      Name:          gateway
      Section Name:  http
    Rules:
      Backend Refs:
        Group:
        Kind:    Service
        Name:    tea
        Port:    80
        Weight:  1
      Matches:
        Path:
          Type:   Exact
          Value:  /tea
  Status:
    Parents:
      Conditions:
        Last Transition Time:  2023-05-25T01:18:08Z
        Message:               The route is accepted
        Observed Generation:   1
        Reason:                Accepted
        Status:                True
        Type:                  Accepted
        Last Transition Time:  2023-05-25T01:18:08Z
        Message:               All references are resolved
        Observed Generation:   1
        Reason:                ResolvedRefs
        Status:                True
        Type:                  ResolvedRefs
      Controller Name:         k8s-gateway.nginx.org/nginx-gateway-controller
      Parent Ref:
        Group:         gateway.networking.k8s.io
        Kind:          Gateway
        Name:          gateway
        Namespace:     default
        Section Name:  http
  Events:              <none>

動作確認
----

``cafe.example.com`` の ``/coffee`` に対してリクエストを送ります

.. code-block:: cmdin
 
  curl -H "Host:cafe.example.com" http://localhost/coffee

.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル

  Server address: 192.168.127.60:8080
  Server name: coffee-7c86d7d67c-dxw9s
  Date: 25/May/2023:01:21:02 +0000
  URI: /coffee
  Request ID: 9fb7dcfd60d04a9dbb510ab7bda6583a

``cafe.example.com`` の ``/tea`` に対してリクエストを送ります

.. code-block:: cmdin
 
  curl -H "Host:cafe.example.com" http://localhost/tea

.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル

  Server address: 192.168.127.62:8080
  Server name: tea-5c457db9-wrtn6
  Date: 25/May/2023:01:21:16 +0000
  URI: /tea
  Request ID: d2caeeaa2fe6722b3df9b8cbf145b382


リソースの削除
----

.. code-block:: cmdin
 
  cd ~/nginx-kubernetes-gateway/examples/cafe-example/
  kubectl delete -f cafe.yaml
  kubectl delete -f gateway.yaml
  kubectl delete -f cafe-routes.yaml

HTTPSの処理
====

HTTPSの終端とWebアプリケーションをデプロイします。

https://github.com/nginxinc/nginx-kubernetes-gateway/tree/main/examples/https-termination

サンプルアプリケーションをデプロイ
----

.. code-block:: cmdin

  cd ~/nginx-kubernetes-gateway/examples/https-termination
  kubectl apply -f cafe.yaml
  kubectl apply -f cafe-secret.yaml
  kubectl apply -f gateway.yaml
  kubectl apply -f cafe-routes.yaml

リソースの確認
----

主要なリソースの内容を確認します


.. code-block:: cmdin
 
  cat gateway.yaml

.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル

  apiVersion: gateway.networking.k8s.io/v1beta1
  kind: Gateway
  metadata:
    name: gateway
    labels:
      domain: k8s-gateway.nginx.org
  spec:
    gatewayClassName: nginx
    listeners:
    - name: http
      port: 80
      protocol: HTTP
    - name: https
      port: 443
      protocol: HTTPS
      tls:
        mode: Terminate
        certificateRefs:
        - kind: Secret
          name: cafe-secret
          namespace: default

``listeners`` 待ち受ける通信を記述しています。また、httpsの配下に ``tls`` を記述し、TLSを終端すること(Terminate)や、利用する証明書(certificateRefs)を記述しています。

HTTPRouteの内容を確認します

.. code-block:: cmdin
 
  cat cafe-routes.yaml

.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル

  apiVersion: gateway.networking.k8s.io/v1beta1
  kind: HTTPRoute
  metadata:
    name: cafe-tls-redirect
  spec:
    parentRefs:
    - name: gateway
      sectionName: http
    hostnames:
    - "cafe.example.com"
    rules:
    - filters:
      - type: RequestRedirect
        requestRedirect:
          scheme: https
          port: 443
  ---
  apiVersion: gateway.networking.k8s.io/v1beta1
  kind: HTTPRoute
  metadata:
    name: coffee
  spec:
    parentRefs:
    - name: gateway
      sectionName: https
    hostnames:
    - "cafe.example.com"
    rules:
    - matches:
      - path:
          type: PathPrefix
          value: /coffee
      backendRefs:
      - name: coffee
        port: 80
  ---
  apiVersion: gateway.networking.k8s.io/v1beta1
  kind: HTTPRoute
  metadata:
    name: tea
  spec:
    parentRefs:
    - name: gateway
      sectionName: https
    hostnames:
    - "cafe.example.com"
    rules:
    - matches:
      - path:
          type: PathPrefix
          value: /tea
      backendRefs:
      - name: tea
        port: 80

HTTPRouteを3つ指定しています。
1つ目のHTTPRouteはHTTPのりクストをHTTPSにリダイレクトします。parentRefsでGatewayの ``http`` を参照しています。
2つ目が ``/coffee`` に関する設定、3つ目が ``/tea`` に関する設定となります。parentRefsでGatewayの ``https`` を参照しています。

作成されたリソースの情報を確認します


.. code-block:: cmdin
 
  kubectl get pod

.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル

  NAME                      READY   STATUS    RESTARTS   AGE
  coffee-7c86d7d67c-x8rc6   1/1     Running   0          62s
  tea-5c457db9-gbxlp        1/1     Running   0          62s

.. code-block:: cmdin
 
  kubectl get secret | grep cafe

.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル

  cafe-secret           kubernetes.io/tls                     2      75s


.. code-block:: cmdin
 
  kubectl get gateway

.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル

  NAME      CLASS   ADDRESS         PROGRAMMED   AGE
  gateway   nginx   192.168.127.2                95s

.. code-block:: cmdin
 
  kubectl get httproute

.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル

  NAME                HOSTNAMES              AGE
  cafe-tls-redirect   ["cafe.example.com"]   99s
  coffee              ["cafe.example.com"]   99s
  tea                 ["cafe.example.com"]   99s


.. code-block:: cmdin
 
  kubectl get pod

.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル
 
.. code-block:: cmdin
 
  kubectl get pod

.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル
 


動作確認
----

``http`` で ``cafe.example.com`` の ``/coffee`` に対してリクエストを送ります

.. code-block:: cmdin
 
  curl -v --resolve cafe.example.com:80:127.0.0.1 http://cafe.example.com:80/coffee

.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル

  *   Trying 127.0.0.1:80...
  * TCP_NODELAY set
  * Connected to localhost (127.0.0.1) port 80 (#0)
  > GET /coffee HTTP/1.1
  > Host:cafe.example.com
  > User-Agent: curl/7.68.0
  > Accept: */*
  >
  * Mark bundle as not supporting multiuse
  < HTTP/1.1 302 Moved Temporarily
  < Server: nginx/1.23.4
  < Date: Thu, 25 May 2023 04:08:09 GMT
  < Content-Type: text/html
  < Content-Length: 145
  < Connection: keep-alive
  < Location: https://cafe.example.com:443/coffee
  <
  <html>
  <head><title>302 Found</title></head>
  <body>
  <center><h1>302 Found</h1></center>
  <hr><center>nginx/1.23.4</center>
  </body>
  </html>
  * Connection #0 to host localhost left intact

httpでアクセスした場合には ``302 Moved Temporarily`` が応答され、Location Header が ``Location: https://cafe.example.com:443/coffee`` と返されていることがわかります

次にHTTPSで通信ができることを確認します

``https`` で ``cafe.example.com`` の ``/tea`` に対してリクエストを送ります

.. code-block:: cmdin
 
  curl -kv --resolve cafe.example.com:443:127.0.0.1 https://cafe.example.com:443/coffee

.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル

  * Added cafe.example.com:443:127.0.0.1 to DNS cache
  * Hostname cafe.example.com was found in DNS cache
  *   Trying 127.0.0.1:443...
  * TCP_NODELAY set
  * Connected to cafe.example.com (127.0.0.1) port 443 (#0)
  * ALPN, offering h2
  * ALPN, offering http/1.1
  * successfully set certificate verify locations:
  *   CAfile: /etc/ssl/certs/ca-certificates.crt
    CApath: /etc/ssl/certs
  * TLSv1.3 (OUT), TLS handshake, Client hello (1):
  * TLSv1.3 (IN), TLS handshake, Server hello (2):
  * TLSv1.3 (IN), TLS handshake, Encrypted Extensions (8):
  * TLSv1.3 (IN), TLS handshake, Certificate (11):
  * TLSv1.3 (IN), TLS handshake, CERT verify (15):
  * TLSv1.3 (IN), TLS handshake, Finished (20):
  * TLSv1.3 (OUT), TLS change cipher, Change cipher spec (1):
  * TLSv1.3 (OUT), TLS handshake, Finished (20):
  * SSL connection using TLSv1.3 / TLS_AES_256_GCM_SHA384
  * ALPN, server accepted to use http/1.1
  * Server certificate:
  *  subject: CN=cafe.example.com
  *  start date: Jul 14 21:52:39 2022 GMT
  *  expire date: Jul 14 21:52:39 2023 GMT
  *  issuer: CN=cafe.example.com
  *  SSL certificate verify result: self signed certificate (18), continuing anyway.
  > GET /coffee HTTP/1.1
  > Host: cafe.example.com
  > User-Agent: curl/7.68.0
  > Accept: */*
  >
  * TLSv1.3 (IN), TLS handshake, Newsession Ticket (4):
  * TLSv1.3 (IN), TLS handshake, Newsession Ticket (4):
  * old SSL session ID is stale, removing
  * Mark bundle as not supporting multiuse
  < HTTP/1.1 200 OK
  < Server: nginx/1.23.4
  < Date: Thu, 25 May 2023 04:21:02 GMT
  < Content-Type: text/plain
  < Content-Length: 163
  < Connection: keep-alive
  < Expires: Thu, 25 May 2023 04:21:01 GMT
  < Cache-Control: no-cache
  <
  Server address: 192.168.127.9:8080
  Server name: coffee-7c86d7d67c-x8rc6
  Date: 25/May/2023:04:21:02 +0000
  URI: /coffee
  Request ID: f82492f218b7b865c2a9745e859cf394
  * Connection #0 to host cafe.example.com left intact

``200 OK`` が応答されており、正しく通信ができることが確認できます

同様に ``https`` で ``cafe.example.com`` の ``/tea`` に対してリクエストを送ります

.. code-block:: cmdin
 
  curl -kv --resolve cafe.example.com:443:127.0.0.1 https://cafe.example.com:443/tea

.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル

  * Added cafe.example.com:443:127.0.0.1 to DNS cache
  * Hostname cafe.example.com was found in DNS cache
  *   Trying 127.0.0.1:443...
  * TCP_NODELAY set
  * Connected to cafe.example.com (127.0.0.1) port 443 (#0)
  * ALPN, offering h2
  * ALPN, offering http/1.1
  * successfully set certificate verify locations:
  *   CAfile: /etc/ssl/certs/ca-certificates.crt
    CApath: /etc/ssl/certs
  * TLSv1.3 (OUT), TLS handshake, Client hello (1):
  * TLSv1.3 (IN), TLS handshake, Server hello (2):
  * TLSv1.3 (IN), TLS handshake, Encrypted Extensions (8):
  * TLSv1.3 (IN), TLS handshake, Certificate (11):
  * TLSv1.3 (IN), TLS handshake, CERT verify (15):
  * TLSv1.3 (IN), TLS handshake, Finished (20):
  * TLSv1.3 (OUT), TLS change cipher, Change cipher spec (1):
  * TLSv1.3 (OUT), TLS handshake, Finished (20):
  * SSL connection using TLSv1.3 / TLS_AES_256_GCM_SHA384
  * ALPN, server accepted to use http/1.1
  * Server certificate:
  *  subject: CN=cafe.example.com
  *  start date: Jul 14 21:52:39 2022 GMT
  *  expire date: Jul 14 21:52:39 2023 GMT
  *  issuer: CN=cafe.example.com
  *  SSL certificate verify result: self signed certificate (18), continuing anyway.
  > GET /tea HTTP/1.1
  > Host: cafe.example.com
  > User-Agent: curl/7.68.0
  > Accept: */*
  >
  * TLSv1.3 (IN), TLS handshake, Newsession Ticket (4):
  * TLSv1.3 (IN), TLS handshake, Newsession Ticket (4):
  * old SSL session ID is stale, removing
  * Mark bundle as not supporting multiuse
  < HTTP/1.1 200 OK
  < Server: nginx/1.23.4
  < Date: Thu, 25 May 2023 04:22:10 GMT
  < Content-Type: text/plain
  < Content-Length: 155
  < Connection: keep-alive
  < Expires: Thu, 25 May 2023 04:22:09 GMT
  < Cache-Control: no-cache
  <
  Server address: 192.168.127.6:8080
  Server name: tea-5c457db9-gbxlp
  Date: 25/May/2023:04:22:10 +0000
  URI: /tea
  Request ID: dd548205c65fbbab524ccb3d0cce1ba8
  * Connection #0 to host cafe.example.com left intact


リソースの削除
----

.. code-block:: cmdin
 
  cd ~/nginx-kubernetes-gateway/examples/cafe-example/
  kubectl delete -f cafe.yaml
  kubectl delete -f cafe-secret.yaml
  kubectl delete -f gateway.yaml
  kubectl delete -f cafe-routes.yaml

