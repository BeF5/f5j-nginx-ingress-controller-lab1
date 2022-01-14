想定外の動作となった場合の切り分け方法
####

事象の切り分け方法
====

各パートでKubernetes環境にデプロイしたアプリケーションの動作が期待した通りとならない場合の切り分けについてご紹介します。

1.	各リソースのデプロイに失敗する
----

- ``kubectl describe <resource type> <resource name> (-n <namespace>)`` コマンドの結果を確認し、リソースのデプロイでエラーが発生していないか確認してください
- ``kubectl logs <pod name> (-n <namespace>)``コマンドの結果を確認し、PODのログメッセージを確認し、エラーが発生していないか確認してください

2.	リソースは正しく作成できたが設定が反映されない
----

- １．の内容を参考にコマンドを実行してください
- 特にリソースが正しく生成された場合でも、NGINXのコンフィグロードでエラーとなる場合があります。その場合には、``kubectl logs`` コマンドでコンフィグロードに失敗する理由が表示されておりますのでそちらでご確認ください
- 一度設定の反映に成功し、その後リソースの内容変更によりエラーが発生した場合には、リソースの変更前の設定で動作いたします。注意深くログを確認し、問題箇所を特定してください

3.	期待した疎通ができない
----

- ``kubectl logs <NGINX Ingress Controller pod> -n nginx-ingress`` コマンドの結果を確認し、Access LogとError Logの内容を確認してください
- NGINX Ingress Controllerで通信を受け付けた場合には以下のようにログが出力されます。ログが出力されること、ログの内容をもとに期待した通信となっているか確認してください
::

    10.244.0.1 - - [24/Aug/2021:14:30:51 +0000] "GET / HTTP/1.1" 500 178 "-" "curl/7.68.0" "-"
    10.244.0.1 - - [25/Aug/2021:05:47:17 +0000] "POST / HTTP/1.1" 200 246 "-" "curl/7.68.0" "-"
    10.244.0.1 - - [25/Aug/2021:05:47:09 +0000] "POST / HTTP/1.1" 200 246 "-" "curl/7.68.0" "-"

4.	設定の内容を確認したい
----

- NGINX Ingress Controllerは本書で紹介のVirtualServer / VirtualServerRoute / PolicyやIngressにより設定を行うと、それらの内容からNGINXの設定ファイルに変換し、その内容を反映しています
- 「kubectl exec -it <NGINX Ingress Controller pod> -n nginx-ingress」 -- bash」コマンドを実行し、NGINX Ingress ControllerのShellを操作することが可能です
- 以下フォルダの各ファイルが適切な設定となっているかご確認ください。意図したファイルが生成されていない場合にはリソースの作成に失敗している可能性がありますのでログをご確認ください。

.. list-table:: File
    :widths: 20 40 20 
    :header-rows: 1
    :stub-columns: 1

    * - **PATH**
      - **Content**
      - **File Name**
    * - /etc/nginx/conf.d/
      - HTTP/HTTPSの主な設定が格納されます。複数のVirtualServerをデプロイした場合には複数のファイルが生成されます。
      - vs_<namespace>_<object name>.conf
    * - /etc/nginx/stream-conf.d/
      - TCP/UDPの主な設定が格納されます。複数のTransportServerをデプロイした場合には複数のファイルが生成されます。合わせて必要となるGlobalConfigurationの作成も完了していることを確認してください。
      - ts_<namespace>_<object name>.conf
    * - /etc/nginx/secrets/
      - 証明書・鍵のファイルが格納されます。複数のSecretをデプロイした場合には複数のファイルが生成されます。参照先のオブジェクトの生成が成功した際に、本ファイルが生成されます。
      - <namespace>-<object name>
    * - /etc/nginx/waf/nac-policies/
      - WAFのセキュリティポリシーが格納されます。複数のAPPolicyをデプロイした場合には複数のファイルが生成されます。
      - <namespace>_<object name>
    * - /etc/nginx/waf/nac-logconfs/
      - WAFのログポリシーが格納されます。複数のAPLogConfをデプロイした場合には複数のファイルが生成されます。ログポリシーの参照先となるWAFセキュリティポリシーの生成が成功した際に、本ファイルが生成されます。
      - <namespace>_<object name>
    * - /etc/nginx/waf/nac-usersigs/
      - WAFのユーザ定義Signatureが格納されます。複数のAPUserSigをデプロイした場合には複数のファイルが生成されます。ログポリシーの参照先となるWAFセキュリティポリシーの生成が成功した際に、本ファイルが生成されます。
      - <namespace>_<object name>
    * - /etc/nginx/oidc/
      - OIDCで参照するファイルが格納されています。
      - (各種JSファイル等)
	

	
