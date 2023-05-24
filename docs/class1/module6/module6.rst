カスタム設定の実施
####

この章では、様々なカスタム設定を行う方法について確認をいただきます。
まずベースとなるアプリケーションとして `シンプルなアプリケーション <https://f5j-nginx-ingress-controller-lab1.readthedocs.io/en/latest/class1/module3/module3.html#web>`__ の手順に従ってアプリケーションをデプロイしてください

ConfigMapによる設定
====

ConfigMapによる、設定内容の追加を確認します。

https://github.com/nginxinc/kubernetes-ingress/tree/v3.1.1/examples/shared-examples/custom-log-format

| 様々な項目を設定することが可能ですが、このサンプルでは ``log-format`` に関する動作を確認します。その他パラメータについては以下の記事を参照してください。
| `ConfigMap Resource <https://docs.nginx.com/nginx-ingress-controller/configuration/global-configuration/configmap-resource/>`__


設定変更前のLogFormatの確認
----

NGINX Ingress Controller の POD名を確認します

.. code-block:: cmdin

  kubectl get pod -n nginx-ingress | grep ingress

.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル

  nginx-ingress-5ddc7f4f-4xhpn          1/1     Running   4 (4d6h ago)   5d19h

POD名を指定し、現在の設定を確認します

.. code-block:: cmdin

  # kubectl exec -it <対象のPOD名> -n nginx-ingress -- grep -A3 "log_format  main" /etc/nginx/nginx.conf
  kubectl exec -it nginx-ingress-5ddc7f4f-4xhpn -n nginx-ingress -- grep -A3 "log_format  main" /etc/nginx/nginx.conf

.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル

    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

設定の追加
----


ファイルを作成し、デプロイします

.. code-block:: cmdin

  cd ~/kubernetes-ingress/examples/shared-examples/custom-log-format
  cat << EOF > log-configmap.yaml
  kind: ConfigMap
  apiVersion: v1
  metadata:
    name: nginx-config
    namespace: nginx-ingress
  data:
    log-format: 'CONFIGMAP \$remote_addr - \$remote_user [\$time_local] "\$request" \$status \$body_bytes_sent "\$http_referer"  "\$http_user_agent" "\$http_x_forwarded_for" "\$resource_name" "\$resource_type" "\$resource_namespace" "\$service"'
  EOF
  
  kubectl apply -f log-configmap.yaml

リソースを確認
----

ConfigMapがデプロイされていることを確認します

.. code-block:: cmdin

  kubectl get configmap -n nginx-ingress | grep nginx-config

.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル

  nginx-config                    1      1m


LogFormatが変更されていることを確認します

.. code-block:: cmdin

  # kubectl exec -it <対象のPOD名> -n nginx-ingress -- grep -A3 "log_format  main" /etc/nginx/nginx.conf
  kubectl exec -it nginx-ingress-5ddc7f4f-4xhpn -n nginx-ingress -- grep -A3 "log_format" /etc/nginx/nginx.conf

.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル

    log_format  main
                     'CONFIGMAP $remote_addr - $remote_user [$time_local] "$request" $status $body_bytes_sent "$http_referer"  "$http_user_agent" "$http_x_forwarded_for" "$resource_name" "$resource_type" "$resource_namespace" "$service"'
                     ;


変更したLogFormatの内容となっており、先頭に ``CONFIGMAP`` という文字列が追加されていることが確認できます


動作確認
----

アプリケーションに対しリクエストを送ります

.. code-block:: cmdin

  curl -H "Host:cafe.example.com" http://localhost/tea

.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル

  Server name: tea-5c457db9-rfpxs
  Date: 31/Jan/2022:06:55:55 +0000
  URI: /tea
  Request ID: c91d025f4089dcf3db6f6127099c6965

ログを確認します

.. code-block:: cmdin

  # kubectl logs  <対象のPOD名> -n nginx-ingress --tail=1
  kubectl logs nginx-ingress-5ddc7f4f-4xhpn -n nginx-ingress --tail=1

.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル

  CONFIGMAP 10.1.1.9 - - [31/Jan/2022:06:55:55 +0000] "GET /tea HTTP/1.1" 200 156 "-"  "curl/7.68.0" "-" "cafe" "virtualserver" "default" "tea-svc"


リソースの削除
----

作成したリソース、ファイルを削除します

.. code-block:: cmdin

  ## cd ~/kubernetes-ingress/examples/custom-resources/basic-configuration/
  kubectl delete -f log-configmap.yaml
  rm log-configmap.yaml

設定を確認し、反映前の状態となっていることを確認します

.. code-block:: cmdin

  # kubectl exec -it <対象のPOD名> -n nginx-ingress -- grep -A3 "log_format  main" /etc/nginx/nginx.conf
  kubectl exec -it nginx-ingress-5ddc7f4f-4xhpn -n nginx-ingress -- grep -A3 "log_format  main" /etc/nginx/nginx.conf

.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル

    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';


Snippetsによる設定
====

Snippetsの機能を利用することにより、VirtualServer/VirtualServerRouteなどで対応していないパラメータをNGINXの設定ファイル記述方式のまま設定に反映することが可能です。

`<https://docs.nginx.com/nginx-ingress-controller/configuration/virtualserver-and-virtualserverroute-resources/#using-snippets>`__


設定変更前の確認
----

Snippetsを利用する場合、予めDeploymentのコマンドラインオプションで ``-enable-snippets`` を指定する必要があります。正しく設定されていることを確認します。

`Command-line Arguments <https://docs.nginx.com/nginx-ingress-controller/configuration/global-configuration/command-line-arguments/>`__

.. code-block:: cmdin

  kubectl describe deployment nginx-ingress -n nginx-ingress

.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル
  :emphasize-lines: 14

  Name:                   nginx-ingress
  Namespace:              nginx-ingress
  CreationTimestamp:      Tue, 25 Jan 2022 11:29:13 +0000
  
  ** 省略 **
  
      Args:
        -nginx-plus
        -nginx-configmaps=$(POD_NAMESPACE)/nginx-config
        -default-server-tls-secret=$(POD_NAMESPACE)/default-server-secret
        -enable-app-protect
        -enable-app-protect-dos
        -enable-oidc
        -enable-snippets
   
  ** 省略 **
  
有効となってないない場合、 `NGINX Ingress Controllerの実行 <https://f5j-nginx-ingress-controller-lab1.readthedocs.io/en/latest/class1/module2/module2.html#id3>`__ を参考に再度デプロイを行ってください


設定の追加
----

.. code-block:: cmdin

  cd ~/kubernetes-ingress/examples/custom-resources/basic-configuration
  cat << EOF > snippets-cafe-virtual-server.yaml
  apiVersion: k8s.nginx.org/v1
  kind: VirtualServer
  metadata:
    name: cafe
  spec:
    http-snippets: |
      limit_req_zone \$binary_remote_addr zone=mylimit:10m rate=1r/s;
    host: cafe.example.com
    tls:
      secret: cafe-secret
    server-snippets: |
          limit_req zone=mylimit;
    upstreams:
    - name: tea
      service: tea-svc
      port: 80
    - name: coffee
      service: coffee-svc
      port: 80
    routes:
    - path: /tea
      location-snippets:
        limit_req_log_level warn;
      action:
        pass: tea
    - path: /coffee
      action:
        pass: coffee
  EOF

設定した内容を確認します。以下の通り指定し、各Snipettsにより設定を追加しています

.. code-block:: cmdin

  ## cd ~/kubernetes-ingress/examples/custom-resources/basic-configuration/
  cat snippet-cafe-virtual-server.yaml

.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル
  :emphasize-lines: 6,7,11,12,22,23

  apiVersion: k8s.nginx.org/v1
  kind: VirtualServer
  metadata:
    name: cafe
  spec:
    http-snippets: |
      limit_req_zone $binary_remote_addr zone=mylimit:10m rate=1r/s;
    host: cafe.example.com
    tls:
      secret: cafe-secret
    server-snippets: |
          limit_req zone=mylimit;
    upstreams:
    - name: tea
      service: tea-svc
      port: 80
    - name: coffee
      service: coffee-svc
      port: 80
    routes:
    - path: /tea
      location-snippets:
        limit_req_log_level warn;
      action:
        pass: tea
    - path: /coffee
      action:
        pass: coffee

作成した内容を反映します

.. code-block:: cmdin

  ## cd ~/kubernetes-ingress/examples/custom-resources/basic-configuration/
  kubectl apply -f snippets-cafe-virtual-server.yaml

リソースを確認
----

VSの設定を変更しましたので、実際に生成されるNGINXの設定ファイルに正しく snippets で指定した内容が追加されていることを確認します

.. code-block:: cmdin

  # kubectl exec -it  <対象のPOD名> -n nginx-ingress -- grep -e "server {" -e location -e limit_req /etc/nginx/conf.d/vs_default_cafe.conf
  kubectl exec -it nginx-ingress-5ddc7f4f-4xhpn -n nginx-ingress -- grep -e "server {" -e location -e limit_req /etc/nginx/conf.d/vs_default_cafe.conf

.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル
  :emphasize-lines: 1,3,5

  limit_req_zone $binary_remote_addr zone=mylimit:10m rate=1r/s;
  server {
      limit_req zone=mylimit;
      location /tea {
          limit_req_log_level warn;
      location /coffee {


少し恣意的な出力結果となりますが、こちらを元に設定内容を確認します。

- 1行目
    - conf.d ディレクトリの設定ファイルは http block で include される内容となります
    - 2行目の server block より前、同じ階層で表示されることから、こちらの内容は http block に追加された設定となります
- 3行目
    - server block 内、4行目の location /tea の前に表示されています
    - こちらの内容は server block に追加された内容となります
- 5行目
    - location block 内、location /tea の中に表示されています
    - こちらの内容は location /tea に追加された内容となります

動作確認
----

forを用いて、HTTPリクエストを連続して２回送ります。まず、 ``/coffee`` 宛のリクエストを確認します

.. code-block:: cmdin

  for i in {1..2}; do echo "==$i==" ; curl -s -H "Host: cafe.example.com" http://localhost/coffee; done;

.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル

  ==1==
  Server address: 192.168.127.53:8080
  Server name: coffee-7c86d7d67c-ss2j8
  Date: 31/Jan/2022:09:55:01 +0000
  URI: /coffee
  Request ID: 0bfa4fe0baf1f0437756a448ab815d03
  ==2==
  <html>
  <head><title>503 Service Temporarily Unavailable</title></head>
  <body>
  <center><h1>503 Service Temporarily Unavailable</h1></center>
  <hr><center>nginx/1.21.3</center>
  </body>
  </html>


| 1つ目のリクエストは正しく結果が表示されています。2つ目のリクエストは 503 が応答されています。
| ログを確認します。

.. code-block:: cmdin

  # kubectl logs  <対象のPOD名> -n nginx-ingress --tail=3
  kubectl logs nginx-ingress-5ddc7f4f-4xhpn -n nginx-ingress --tail=3

.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル

  10.1.1.9 - - [31/Jan/2022:09:55:01 +0000] "GET /coffee HTTP/1.1" 200 164 "-" "curl/7.68.0" "-"
  2022/01/31 09:55:01 [error] 205#205: *50 limiting requests, excess: 0.972 by zone "mylimit", client: 10.1.1.9, server: cafe.example.com, request: "GET /coffee HTTP/1.1", host: "cafe.example.com"
  10.1.1.9 - - [31/Jan/2022:09:55:01 +0000] "GET /coffee HTTP/1.1" 503 197 "-" "curl/7.68.0" "-"


| ログを確認すると、1行目が1つ目のリクエストの結果となります。
| 2行目がrate limitのエラー、そして3行目がrate limitが発生した通信のアクセスログとなります。
| 2行目のログレベルを見ると ``[error]`` となっていることが確認できます

次に、 ``/coffee`` 宛のリクエストを確認します

.. code-block:: cmdin

  for i in {1..2}; do echo "==$i==" ; curl -s -H "Host: cafe.example.com" http://localhost/tea; done;

.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル

  ==1==
  Server address: 192.168.127.55:8080
  Server name: tea-5c457db9-rfpxs
  Date: 31/Jan/2022:09:55:30 +0000
  URI: /tea
  Request ID: 3d14ac59fd88c1b507a611283045be98
  ==2==
  <html>
  <head><title>503 Service Temporarily Unavailable</title></head>
  <body>
  <center><h1>503 Service Temporarily Unavailable</h1></center>
  <hr><center>nginx/1.21.3</center>
  </body>
  </html>

| 先程と同様に、1つ目のリクエストは正しく結果が表示されています。2つ目のリクエストは 503 が応答されています。
| ログを確認します

.. code-block:: cmdin

  # kubectl logs  <対象のPOD名> -n nginx-ingress --tail=3
  kubectl logs nginx-ingress-5ddc7f4f-4xhpn -n nginx-ingress --tail=3

.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル

  10.1.1.9 - - [31/Jan/2022:09:55:30 +0000] "GET /tea HTTP/1.1" 200 156 "-" "curl/7.68.0" "-"
  2022/01/31 09:55:30 [warn] 205#205: *53 limiting requests, excess: 0.984 by zone "mylimit", client: 10.1.1.9, server: cafe.example.com, request: "GET /tea HTTP/1.1", host: "cafe.example.com"
  10.1.1.9 - - [31/Jan/2022:09:55:30 +0000] "GET /tea HTTP/1.1" 503 197 "-" "curl/7.68.0" "-"


| 基本的な内容は先程と同じです。
| 一点異なるのが、2行目のログレベルを見ると ``[warn]`` となっていることが確認できます。
| これは ``location-snippets`` で指定した ``limit_req_log_level`` により、ログレベルを変更した結果となります

リソースの削除
----

こちらでは ``snippets`` を追加したVSへと変更したので、元の ``snippets`` の指定がない設定を再度反映します。また、ファイルを削除します

.. code-block:: cmdin

  ## cd ~/kubernetes-ingress/examples/custom-resources/basic-configuration/
  kubectl apply -f cafe-virtual-server.yaml
  rm snippets-cafe-virtual-server.yaml

Templateの変更
====

| NGINX Ingress Controller は Template で各リソースで指定されたパラメータを元に、NGINX の設定ファイルを生成しています。
| お客様が求める通信要件やアプリケーションの内容によってはこのTemplateで生成される設定ファイルでは要件を十分に満たせない場合があります。
| この章では、Template を意図した内容へ変更し、プラットフォームのベースとなる設定を変更する方法を確認します。

https://docs.nginx.com/nginx-ingress-controller/configuration/global-configuration/custom-templates/

Templateファイルは以下フォルダに格納されています。

https://github.com/nginxinc/kubernetes-ingress/tree/v3.1.1/internal/configs

- version1 : NGINX ( main ``nginx.tmpl`` 、Ingress ``nginx.ingress.tmpl`` ) 、NGINX Plus ( main ``nginx-plus.tmpl`` 、 Ingress ``nginx-plus.ingress.tmpl`` )のTemplateが格納されています 
- version2 : NGINX ( ``nginx.virtualserver.tmpl`` ) 、 NGINX Plus ( ``nginx-plus.virtualserver.tmpl`` )の VirtualServer Templateが格納されています


設定の追加
----

Template 用 ConfigMapの作成

.. code-block:: cmdin

  ## cd ~/kubernetes-ingress/examples/custom-resources/basic-configuration/

  # ベースとなるファイルを作成します
  cat << EOF > vs-custom-template.yaml
  kind: ConfigMap
  apiVersion: v1
  metadata:
    name: nginx-config
    namespace: nginx-ingress
  data:
    virtualserver-template: |
  EOF

  # ファイル末尾に nginx-plus.virtualserver.tmpl の内容を追加します
  sed "s/^/    /"  ~/kubernetes-ingress/internal/configs/version2/nginx-plus.virtualserver.tmpl >> vs-custom-template.yaml

以下の内容を参考に、Directive を追加してください。

.. code-block:: cmdin

  vi vs-custom-template.yaml

.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル
  :emphasize-lines: 7

  ※省略※
  541                 {{ if not $l.GRPCPass }}
  542             proxy_http_version 1.1;
  543             set $default_connection_header {{ if $l.HasKeepalive }}""{{ else }}close{{ end }};
  544             proxy_set_header Upgrade $http_upgrade;
  545             proxy_set_header Connection $vs_connection_header;
  546             proxy_set_header X-App-Authentication $http_x_authtype:$arg_userapikey;
  547             proxy_pass_request_headers {{ if $l.ProxyPassRequestHeaders }}on{{ else }}off{{ end }};
  548                 {{ end }}
  ※省略※

.. NOTE::

  Templateで ``$http_x_authtype`` と指定しています。これはHTTP Headerの値を参照しており、 ``$http_<name>`` という書式で指定します。HTTP Headerの名称(<name>)はダッシュ( ``-`` )をアンダースコア( ``_`` )に置換して指定する必要があります。

今回のサンプルは、NGINX Ingress Controller を経由する通信全てに新たなHTTP Header ``X-App-Authentication $http_x_authtype:$arg_userapikey;`` を追加する例となります

ConfigMapをデプロイします。

.. code-block:: cmdin

  ## cd ~/kubernetes-ingress/examples/custom-resources/basic-configuration/
  kubectl apply -f vs-custom-template.yaml


反映した結果を確認します。ConfigMapの反映エラーは ``kubectl logs <NIC Pod>`` で確認いただけます。正しく反映されない場合はエラーの内容をよく確認して適宜対応してください。
以下の場合、エラーなくコンフィグが正しく反映された例となります

.. code-block:: cmdin

  # kubectl logs  <対象のPOD名> -n nginx-ingress --tail=5
  kubectl logs nginx-ingress-5ddc7f4f-4xhpn -n nginx-ingress --tail=5

.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル

  2022/01/31 11:19:45 [notice] 20#20: worker process 261 exited with code 0
  2022/01/31 11:19:45 [notice] 20#20: cache manager process 262 exited with code 0
  2022/01/31 11:19:45 [notice] 20#20: signal 29 (SIGIO) received
  2022/01/31 11:19:45 [notice] 20#20: signal 17 (SIGCHLD) received from 260
  2022/01/31 11:19:45 [notice] 20#20: worker process 260 exited with code 0
  2022/01/31 11:19:45 [notice] 20#20: signal 29 (SIGIO) received


今回のサンプルではバックエンドに到達した通信の情報を確認するため、以下のコンテナイメージをデプロイしますサービスとして以下を利用します。

`rteller/nginx_echo <https://hub.docker.com/r/rteller/nginx_echo>`__

バックエンドのアプリケーションの内容を以下コマンドで変更します

.. code-block:: cmdin

  ## cd ~/kubernetes-ingress/examples/custom-resources/basic-configuration/
  sed -e "s#nginxdemos/nginx-hello:plain-text#rteller/nginx_echo:latest#g" -e "s#8080#8000#g" cafe.yaml  > echo-cafe.yaml

変更した内容を確認します。

.. code-block:: cmdin

  diff -u cafe.yaml echo-cafe.yaml

.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル
  :emphasize-lines: 7,8,10,11,19,20,28,29,31,32,40,41

  --- cafe.yaml   2022-01-25 11:17:45.239371139 +0000
  +++ echo-cafe.yaml      2022-01-31 11:25:29.065695861 +0000
  @@ -14,9 +14,9 @@
       spec:
         containers:
         - name: coffee
  -        image: nginxdemos/nginx-hello:plain-text
  +        image: rteller/nginx_echo:latest
           ports:
  -        - containerPort: 8080
  +        - containerPort: 8000
   ---
   apiVersion: v1
   kind: Service
  @@ -25,7 +25,7 @@
   spec:
     ports:
     - port: 80
  -    targetPort: 8080
  +    targetPort: 8000
       protocol: TCP
       name: http
     selector:
  @@ -47,9 +47,9 @@
       spec:
         containers:
         - name: tea
  -        image: nginxdemos/nginx-hello:plain-text
  +        image: rteller/nginx_echo:latest
           ports:
  -        - containerPort: 8080
  +        - containerPort: 8000
   ---
   apiVersion: v1
   kind: Service
  @@ -58,7 +58,7 @@
   spec:
     ports:
     - port: 80
  -    targetPort: 8080
  +    targetPort: 8000
       protocol: TCP
       name: http
     selector:


変更した内容をデプロイします。

.. code-block:: cmdin

  ## cd ~/kubernetes-ingress/examples/custom-resources/basic-configuration/
  kubectl apply -f echo-cafe.yaml

新たにコンテナイメージを取得するため、デプロイに1分ほど必要となります。以下のように各Podが正しく動作していることを確認してください

.. code-block:: cmdin

  kubectl get pod

.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル

  NAME                      READY   STATUS    RESTARTS   AGE
  coffee-57ffcb58cc-66fdq   1/1     Running   0          22s
  coffee-57ffcb58cc-b8cgm   1/1     Running   0          8s
  tea-56b4985bd5-lwgtb      1/1     Running   0          22s


動作確認
----

curlコマンドを用いて、サンプルリクエストを送信します。 ``jq`` コマンドを用いて、レスポンスのJSONデータからリクエストに含まれるHTTP Header情報を表示しています

.. code-block:: cmdin

  curl -s -H "Host: cafe.example.com" http://localhost/tea | jq .request.uri.headers

.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル
  :emphasize-lines: 8

  {
    "Connection": "close",
    "X-Real-IP": "10.1.1.9",
    "X-Forwarded-For": "10.1.1.9",
    "X-Forwarded-Host": "cafe.example.com",
    "X-Forwarded-Port": "80",
    "X-Forwarded-Proto": "http",
    "X-App-Authentication": ":",
    "Host": "cafe.example.com",
    "User-Agent": "curl/7.68.0",
    "Accept": "*/*"
  }

curlコマンドでは指定していないHTTP Headerがいくつか表示されています。これらは、NGINX Ingress Controllerによって新たに追加された内容となります。
今回設定で追加した内容は、 ``X-App-Authentication`` で、正しくバックエンドのアプリケーションまで到達していることが確認できます。


次に、対象の ``X-App-Authentication`` というHeaderに値が表示されるよう、サンプルリクエストを送ります。Templateに追加した内容の通り、Headerに表示されていることが確認できます。

.. code-block:: cmdin

  curl -s -H "Host: cafe.example.com" -H "x-authtype: APIKEY" "http://localhost/tea?userapikey=ABCD1234EFGH" | jq .request.uri.headers

.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル
  :emphasize-lines: 10

  {
    "User-Agent": "curl/7.68.0",
    "x-authtype": "APIKEY",
    "X-Forwarded-Proto": "http",
    "Connection": "close",
    "Host": "cafe.example.com",
    "Accept": "*/*",
    "X-Forwarded-Host": "cafe.example.com",
    "X-Forwarded-For": "10.1.1.9",
    "X-App-Authentication": "APIKEY:ABCD1234EFGH",
    "X-Real-IP": "10.1.1.9",
    "X-Forwarded-Port": "80"
  }

通信のログを確認します。

.. code-block:: cmdin

  # kubectl logs  <対象のPOD名> -n nginx-ingress --tail=5
  kubectl logs nginx-ingress-5ddc7f4f-4xhpn -n nginx-ingress --tail=5

.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル

  10.1.1.9 - - [31/Jan/2022:11:27:26 +0000] "GET /tea HTTP/1.1" 200 849 "-" "curl/7.68.0" "-"
  10.1.1.9 - - [31/Jan/2022:11:30:33 +0000] "GET /tea?userapikey=ABCD1234EFGH HTTP/1.1" 200 957 "-" "curl/7.68.0" "-"

リソースの削除
----

.. code-block:: cmdin

  # ConfigMap を初期化します
  kubectl apply -f  ~/kubernetes-ingress/deployments/common/nginx-config.yaml
  # 再度 Pod をデプロイします
  kubectl replace --force -f ~/kubernetes-ingress/deployments/deployment/nginx-plus-ingress.yaml
  
  ## cd ~/kubernetes-ingress/examples/custom-resources/basic-configuration/
  # 不要なファイルを削除します
  rm vs-custom-template.yaml
  rm echo-cafe.yaml