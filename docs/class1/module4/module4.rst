NICによるWebアプリのセキュリティ
####

この章では、実際にデプロイしたNGINX Ingress Controllerを使い、Webアプリケーションに対するWAF・DoS対策の実施方法を確認します。
設定例は `NGINX Inc GitHubの examples/custom-resources/ <https://github.com/nginxinc/kubernetes-ingress/tree/v2.1.0/examples/custom-resources>`__ に管理されております

Syslog Serverのデプロイ
====

NGINX App Protect WAF、NGINX App Protect DoS 双方のセキュリティモジュールのログをSyslogサーバに転送し、結果を確認します。
まずはじめにSyslogサーバをデプロイします。

| ``syslog.yaml`` は、NAP WAF、NAP DoSで利用します。
| ``syslog2.yaml`` は、NAP DoSで利用します 

.. NOTE::

  GitHub上で公開されている、syslog.yaml , syslog2.yaml のイメージTagが現在は存在しないTagとなっています。
  ``取得するSyslogイメージのタグを変更`` は暫定処置となります。

(暫定処置) 取得するSyslogイメージのタグを変更

.. code-block:: cmdin

  cd ~/kubernetes-ingress/examples/custom-resources/dos
  cp syslog.yaml syslog.yaml-bak
  cp syslog2.yaml syslog2.yaml-bak
  sed -i -e 's/3.31.2-buster/3.31.2/g' syslog.yaml > syslog.yaml
  sed -i -e 's/3.31.2-buster/3.31.2/g' syslog2.yaml > syslog2.yaml

  # 以下、Diffコマンドを参考に、対象となるTagが変更されていることを確認
  diff -u syslog.yaml-bak syslog.yaml
  diff -u syslog2.yaml-bak syslog2.yaml

Syslogイメージのデプロイ

.. code-block:: cmdin

  cd ~/kubernetes-ingress/examples/custom-resources/dos
  kubectl apply -f syslog.yaml
  kubectl apply -f syslog2.yaml

デプロイされた内容の確認

.. code-block:: cmdin

  kubectl get deployment

.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル

  NAME       READY   UP-TO-DATE   AVAILABLE   AGE
  syslog     1/1     1            1           9m
  syslog-2   1/1     1            1           9m

.. code-block:: cmdin

  kubectl get svc| grep syslog-svc

.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル

  syslog-svc     ClusterIP   10.96.250.209    <none>        514/TCP   9m
  syslog-svc-2   ClusterIP   10.103.224.109   <none>        514/UDP   9m


Ingress Controller で WAF機能(NGINX App Protect WAF) のデプロイ
====

https://github.com/nginxinc/kubernetes-ingress/tree/v2.1.0/examples/custom-resources/waf


サンプルアプリケーションをデプロイ
----

アプリケーションをデプロイします。

.. code-block:: cmdin

  kubectl apply -f webapp.yaml
  kubectl apply -f ap-apple-uds.yaml
  kubectl apply -f ap-dataguard-alarm-policy.yaml
  kubectl apply -f ap-logconf.yaml
  kubectl apply -f waf.yaml
  kubectl apply -f virtual-server.yaml


リソースを確認
----

ポイントとなるファイルの内容を確認します。

NAP WAFのPolicyでは様々なセキュリティ機能を用いて外部からの攻撃をブロックします。
外部からの様々な攻撃を通信の特徴や、リクエストに含まれる文字列などから検知・ブロックするためのルールとしてSignatureがあります。
NAP WAFではお客様アプリケーションに合わせた制御や、特定の通信を制御するため、ユーザ定義シグネチャ(User-Defined Signature)の定義が可能です

| こちらで設定する ``ユーザ定義シグネチャ`` の詳細は、以下の内容を参照してください。
| `NGINX App Protect WAF Configuration Guide/User-Defined Signatures <https://docs.nginx.com/nginx-app-protect/configuration-guide/configuration/#user-defined-signatures>`__

| NGINX Ingress Controller での NAP WAF の詳細は、以下のページを参照してください。
| `NGINX Ingress Controller Configuration/App Protect User Defined Signatures <https://docs.nginx.com/nginx-ingress-controller/app-protect/configuration/#app-protect-user-defined-signatures>`__




| ``ap-apple-uds.yaml`` は、ユーザ独自のシグネチャの定義となります。
| 条件は ``rule`` に指定された内容となります。また、tagとして ``Fruits`` を指定します。

.. code-block:: yaml
  :linenos:
  :caption: ap-apple-uds.yaml
  :emphasize-lines: 13,18

  apiVersion: appprotect.f5.com/v1beta1
  kind: APUserSig
  metadata:
    name: apple
  spec:
    signatures:
    - accuracy: medium
      attackType:
        name: Brute Force Attack
      description: Medium accuracy user defined signature with tag (Fruits)
      name: Apple_medium_acc
      risk: medium
      rule: content:"apple"; nocase;
      signatureType: request
      systems:
      - name: Microsoft Windows
      - name: Unix/Linux
    tag: Fruits

``ap-dataguard-alarm-policy.yaml`` は、App ProtectのPolicy設定となります。tagとして ``Fruits`` を持つシグネチャを参照・有効にしています

.. code-block:: yaml
  :linenos:
  :caption: ap-dataguard-alarm-policy.yaml
  :emphasize-lines: 4,8,14

  apiVersion: appprotect.f5.com/v1beta1
  kind: APPolicy
  metadata:
    name: dataguard-alarm
  spec:
    policy:
      signature-requirements:
      - tag: Fruits
      signature-sets:
      - name: apple_sigs
        block: true
        signatureSet:
          filter:
            tagValue: Fruits
            tagFilter: eq
      applicationLanguage: utf-8
      blocking-settings:
        violations:
        - alarm: true
          block: false
          name: VIOL_DATA_GUARD
      data-guard:
        creditCardNumbers: true
        enabled: true
        enforcementMode: ignore-urls-in-list
        enforcementUrls: []
        lastCcnDigitsToExpose: 4
        lastSsnDigitsToExpose: 4
        maskData: true
        usSocialSecurityNumbers: true
      enforcementMode: blocking
      name: dataguard-alarm
      template:
        name: POLICY_TEMPLATE_NGINX_BASE

``ap-logconf.yaml`` は、Logの定義に関する設定となります。

.. code-block:: yaml
  :linenos:
  :caption: ap-logconf.yaml
  :emphasize-lines: 4

  apiVersion: appprotect.f5.com/v1beta1
  kind: APLogConf
  metadata:
    name: logconf
  spec:
    content:
      format: default
      max_message_size: 64k
      max_request_size: any
    filter:
      request_type: all

``waf.yaml`` は、VirtualServerが参照するPolicy設定となります。利用するApp ProtectのPolicyとして ``dataguard-alarm`` を指定し、Log 設定として ``logconf`` を指定します。

.. code-block:: yaml
  :linenos:
  :caption: waf.yaml
  :emphasize-lines: 4,8,11

  apiVersion: k8s.nginx.org/v1
  kind: Policy
  metadata:
    name: waf-policy
  spec:
    waf:
      enable: true
      apPolicy: "default/dataguard-alarm"
      securityLog:
        enable: true
        apLogConf: "default/logconf"
        logDest: "syslog:server=syslog-svc.default:514"

``virtual-server.yaml`` で、作成した ``waf-poicy`` を割り当てます 

.. code-block:: yaml
  :linenos:
  :caption: virtual-server.yaml
  :emphasize-lines: 7,8

  apiVersion: k8s.nginx.org/v1
  kind: VirtualServer
  metadata:
    name: webapp
  spec:
    host: webapp.example.com
    policies:
    - name: waf-policy
    upstreams:
    - name: webapp
      service: webapp-svc
      port: 80
    routes:
    - path: /
      action:
        pass: webapp

以下の通り、各リソースを適切に作成されていることを確認します。

.. code-block:: cmdin

  kubectl get APUserSig

.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル

  NAME    AGE
  apple   38m

.. code-block:: cmdin

  kubectl get aplogconf


.. code-block:: cmdin

  kubectl get appolicy

.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル

  NAME              AGE
  dataguard-alarm   39m

.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル

  NAME      AGE
  logconf   39m


.. code-block:: cmdin

  kubectl get policy

.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル
  
  NAME         STATE   AGE
  waf-policy   Valid   41m
  

動作確認
----

curlコマンドでリクエストを送信します。

.. code-block:: cmdin

  curl -v --resolve webapp.example.com:80:127.0.0.1 "http://webapp.example.com/"

.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル
  :emphasize-lines: 12,22

  * Added webapp.example.com:80:127.0.0.1 to DNS cache
  * Hostname webapp.example.com was found in DNS cache
  *   Trying 127.0.0.1:80...
  * TCP_NODELAY set
  * Connected to webapp.example.com (127.0.0.1) port 80 (#0)
  > GET / HTTP/1.1
  > Host: webapp.example.com
  > User-Agent: curl/7.68.0
  > Accept: */*
  >
  * Mark bundle as not supporting multiuse
  < HTTP/1.1 200 OK
  < Content-Type: text/plain
  < Content-Length: 157
  < Connection: keep-alive
  < Expires: Thu, 20 Jan 2022 03:07:27 GMT
  < Cache-Control: no-cache
  <
  Server address: 192.168.127.42:8080
  Server name: webapp-64d444885-jg6hf
  Date: 20/Jan/2022:03:07:28 +0000
  URI: /
  Request ID: e0b6f00106a11885f85300ffcaf5b912
  * Connection #0 to host webapp.example.com left intact

ログメッセージを見ると、通信をブロックせず転送(PASSED)していることが確認できます。NGINX App ProtectはBot Signatureの機能をもっておりますので、curlコマンドであることを“人によるブラウザの通信ではなくBot Clientである”という形で検知をしておりますが、即座に驚異であると判断される設定となっておりませんので適切な通信としてWebアプリケーションへ転送が行われております。

.. code-block:: json
  :linenos:
  :caption: 該当するSyslogのサンプル
  :emphasize-lines: 23,24,27,28,29,31,32,35
  
  Jan 20 03:07:28 nginx-ingress-5ddc7f4f-zjlt2 ASM:
  attack_type="Non-browser Client",
  blocking_exception_reason="N/A",
  date_time="2022-01-20 03:07:28",
  dest_port="80",
  ip_client="10.1.1.9",
  is_truncated="false",
  method="GET",
  policy_name="dataguard-alarm",
  protocol="HTTP",
  request_status="alerted",
  response_code="200",
  severity="Critical",
  sig_cves="N/A",
  sig_ids="N/A",
  sig_names="N/A",
  sig_set_names="N/A",
  src_port="49443",
  sub_violations="N/A",
  support_id="16242938385820378173",
  threat_campaign_names="N/A",
  unit_hostname="nginx-ingress-5ddc7f4f-zjlt2",
  uri="/",
  violation_rating="0",
  vs_name="32-webapp.example.com:8-/",
  x_forwarded_for_header_value="N/A",
  outcome="PASSED",
  outcome_reason="SECURITY_WAF_VIOLATION_TRANSPARENT_MODE",
  violations="Bot Client Detected",
  violation_details="N/A",
  bot_signature_name="curl",
  bot_category="HTTP Library",
  bot_anomalies="N/A",
  enforced_bot_anomalies="N/A",
  client_class="Untrusted Bot",
  client_application="N/A",
  client_application_version="N/A",
  request="GET / HTTP/1.1\r\nHost: webapp.example.com\r\nUser-Agent: curl/7.68.0\r\nAccept: */*\r\n\r\n",
  transport_protocol="HTTP/1.1"


次にNAP WAFで攻撃として検知するリクエストを、curlコマンドで送信します。クロスサイトスクリプティング(XSS)を想定した接続をします。

.. code-block:: cmdin

  curl -v --resolve webapp.example.com:80:127.0.0.1 "http://webapp.example.com/<script>"

.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル (区切り位置で改行して表示)
  :emphasize-lines: 12,20

  * Added webapp.example.com:80:127.0.0.1 to DNS cache
  * Hostname webapp.example.com was found in DNS cache
  *   Trying 127.0.0.1:80...
  * TCP_NODELAY set
  * Connected to webapp.example.com (127.0.0.1) port 80 (#0)
  > GET /<script> HTTP/1.1
  > Host: webapp.example.com
  > User-Agent: curl/7.68.0
  > Accept: */*
  >
  * Mark bundle as not supporting multiuse
  < HTTP/1.1 200 OK
  < Content-Type: text/html; charset=utf-8
  < Connection: close
  < Cache-Control: no-cache
  < Pragma: no-cache
  < Content-Length: 247
  <
  * Closing connection 0
  <html><head><title>Request Rejected</title></head><body>The requested URL was rejected. Please consult with your administrator.<br><br>Your support ID is: 16242938385820378683<br><br><a href='javascript:history.back();'>[Go Back]</a></body></html>

通信が ``拒否`` され、エラーページが応答されています。 ``support ID`` に表示される値を確認してください。

| ログメッセージを見ると、URLに不正な文字列が含まれており、XSS script tag(URI)などのSignatureで検知、通信をブロック(REJECTED)していることが確認できます。また、 ``violation_rating="5"`` となっています。
Violation Rating はNAP WAFが通信の内容を元にリクエストのリスクを判定します。デフォルトテンプレートはこちらの値を元にブロックする挙動となります。詳細は以下のページを参照してください。
| `NGINX App Protect WAF Configuration Guide/Basic Configuration and the Default Policy <https://docs.nginx.com/nginx-app-protect/configuration-guide/configuration/#basic-configuration-and-the-default-policy>`__

.. code-block:: json
  :linenos:
  :caption: 該当するSyslogのサンプル
  :emphasize-lines: 15,16,17,20,23,24,27,28,29,31,32,35

  Jan 20 03:07:39 nginx-ingress-5ddc7f4f-zjlt2 ASM:
  attack_type="Non-browser Client,Abuse of Functionality,Cross Site Scripting (XSS)",
  blocking_exception_reason="N/A",
  date_time="2022-01-20 03:07:39",
  dest_port="80",
  ip_client="10.1.1.9",
  is_truncated="false",
  method="GET",
  policy_name="dataguard-alarm",
  protocol="HTTP",
  request_status="blocked",
  response_code="0",
  severity="Critical",
  sig_cves="N/A",
  sig_ids="200000099,200000093",
  sig_names="XSS script tag (URI),XSS script tag end (URI)",
  sig_set_names="{Cross Site Scripting Signatures;High Accuracy Signatures},{Cross Site Scripting Signatures;High Accuracy Signatures}",
  src_port="61276",
  sub_violations="N/A",
  support_id="16242938385820378683",
  threat_campaign_names="N/A",
  unit_hostname="nginx-ingress-5ddc7f4f-zjlt2",
  uri="/<script>",
  violation_rating="5",
  vs_name="32-webapp.example.com:8-/",
  x_forwarded_for_header_value="N/A",
  outcome="REJECTED",
  outcome_reason="SECURITY_WAF_VIOLATION",
  violations="Illegal meta character in URL,Attack signature detected,Violation Rating Threat detected,Bot Client Detected",
  violation_details="<?xml version='1.0' encoding='UTF-8'?><BAD_MSG><violation_masks><block>410000000200c00-3a03030c30000072-8000000000000000-0</block><alarm>2477f0ffcbbd0fea-befbf35cb000007e-8000000000000000-0</alarm><learn>0-20-0-0</learn><staging>0-0-0-0</staging></violation_masks><request-violations><violation><viol_index>42</viol_index><viol_name>VIOL_ATTACK_SIGNATURE</viol_name><context>url</context><sig_data><sig_id>200000099</sig_id><blocking_mask>3</blocking_mask><kw_data><buffer>LzxzY3JpcHQ+</buffer><offset>1</offset><length>7</length></kw_data></sig_data><sig_data><sig_id>200000093</sig_id><blocking_mask>3</blocking_mask><kw_data><buffer>LzxzY3JpcHQ+</buffer><offset>2</offset><length>7</length></kw_data></sig_data></violation><violation><viol_index>26</viol_index><viol_name>VIOL_URL_METACHAR</viol_name><uri>LzxzY3JpcHQ+</uri><metachar_index>60</metachar_index><wildcard_entity>*</wildcard_entity><staging>0</staging></violation><violation><viol_index>26</viol_index><viol_name>VIOL_URL_METACHAR</viol_name><uri>LzxzY3JpcHQ+</uri><metachar_index>62</metachar_index><wildcard_entity>*</wildcard_entity><staging>0</staging></violation></request-violations></BAD_MSG>",
  bot_signature_name="curl",
  bot_category="HTTP Library",
  bot_anomalies="N/A",
  enforced_bot_anomalies="N/A",
  client_class="Untrusted Bot",
  client_application="N/A",
  client_application_version="N/A",
  request="GET /<script> HTTP/1.1\r\nHost: webapp.example.com\r\nUser-Agent: curl/7.68.0\r\nAccept: */*\r\n\r\n",
  transport_protocol="HTTP/1.1"

参考の情報ですが、curlコマンドの **<script>** を **?a=a?%27+OR+1=1--** などの文字列に入れ替えると、SQL Injectionのブロックを見ることができますのでご確認ください。

User Defined Signatureで指定した内容が正しく動作しているか確認します。Webアプリケーションに”apple”という文字を送信します。

.. code-block:: cmdin

  curl -v --resolve webapp.example.com:80:127.0.0.1 "http://webapp.example.com/" -X POST -d "apple"

.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル (区切り位置で改行して表示)
  :emphasize-lines: 12,20

  Note: Unnecessary use of -X or --request, POST is already inferred.
  * Added webapp.example.com:80:127.0.0.1 to DNS cache
  * Hostname webapp.example.com was found in DNS cache
  *   Trying 127.0.0.1:80...
  * TCP_NODELAY set
  * Connected to webapp.example.com (127.0.0.1) port 80 (#0)
  > POST / HTTP/1.1
  > Host: webapp.example.com
  > User-Agent: curl/7.68.0
  > Accept: */*
  > Content-Length: 5
  > Content-Type: application/x-www-form-urlencoded
  >
  * upload completely sent off: 5 out of 5 bytes
  * Mark bundle as not supporting multiuse
  < HTTP/1.1 200 OK
  < Content-Type: text/html; charset=utf-8
  < Connection: close
  < Cache-Control: no-cache
  < Pragma: no-cache
  < Content-Length: 247
  <
  * Closing connection 0
  <html><head><title>Request Rejected</title></head><body>The requested URL was rejected. Please consult with your administrator.<br><br>Your support ID is: 16242938385820379193<br><br><a href='javascript:history.back();'>[Go Back]</a></body></html>

ログメッセージを見ると、該当のログメッセージが、User Defined Signatureの ``Apple_medium_acc`` というSignature Nameで検知されブロック(REJECTED)されていることが確認できます。

.. code-block:: json
  :linenos:
  :caption: 該当するSyslogのサンプル (区切り位置で改行して表示)
  :emphasize-lines: 15,16,17,20,23,24,27,28,29,31,32,35

  Jan 20 03:07:51 nginx-ingress-5ddc7f4f-zjlt2 ASM:
  attack_type="Non-browser Client,Brute Force Attack",
  blocking_exception_reason="N/A",
  date_time="2022-01-20 03:07:51",
  dest_port="80",
  ip_client="10.1.1.9",
  is_truncated="false",
  method="POST",
  policy_name="dataguard-alarm",
  protocol="HTTP",
  request_status="blocked",
  response_code="0",
  severity="Critical",
  sig_cves="N/A",
  sig_ids="300000000",
  sig_names="Apple_medium_acc [Fruits]",
  sig_set_names="{apple_sigs}",
  src_port="63409",
  sub_violations="N/A",
  support_id="16242938385820379193",
  threat_campaign_names="N/A",
  unit_hostname="nginx-ingress-5ddc7f4f-zjlt2",
  uri="/",
  violation_rating="2",
  vs_name="32-webapp.example.com:8-/",
  x_forwarded_for_header_value="N/A",
  outcome="REJECTED",
  outcome_reason="SECURITY_WAF_VIOLATION",
  violations="Attack signature detected,Bot Client Detected",
  violation_details="<?xml version='1.0' encoding='UTF-8'?><BAD_MSG><violation_masks><block>410000000200c00-3a03030c30000072-8000000000000000-0</block><alarm>2477f0ffcbbd0fea-befbf35cb000007e-8000000000000000-0</alarm><learn>0-20-0-0</learn><staging>0-0-0-0</staging></violation_masks><request-violations><violation><viol_index>42</viol_index><viol_name>VIOL_ATTACK_SIGNATURE</viol_name><context>request</context><sig_data><sig_id>300000000</sig_id><blocking_mask>3</blocking_mask><kw_data><buffer>YXBwbGU=</buffer><offset>0</offset><length>5</length></kw_data></sig_data></violation></request-violations></BAD_MSG>",
  bot_signature_name="curl",
  bot_category="HTTP Library",
  bot_anomalies="N/A",
  enforced_bot_anomalies="N/A",
  client_class="Untrusted Bot",
  client_application="N/A",
  client_application_version="N/A",
  request="POST / HTTP/1.1\r\nHost: webapp.example.com\r\nUser-Agent: curl/7.68.0\r\nAccept: */*\r\nContent-Length: 5\r\nContent-Type: application/x-www-form-urlencoded\r\n\r\napple",
  transport_protocol="HTTP/1.1"


リソースの削除
----

.. code-block:: cmdin

  kubectl delete -f webapp.yaml
  kubectl delete -f ap-apple-uds.yaml
  kubectl delete -f ap-dataguard-alarm-policy.yaml
  kubectl delete -f ap-logconf.yaml
  kubectl delete -f waf.yaml
  kubectl delete -f virtual-server.yaml


Ingress Controller で 高度なDoS対策機能(NGINX App Protect DoS) のデプロイ
====

https://github.com/nginxinc/kubernetes-ingress/tree/v2.1.0/examples/custom-resources/dos


サンプルアプリケーションをデプロイ
----

アプリケーションをデプロイします。

.. code-block:: cmdin

  cd ~/kubernetes-ingress/examples/custom-resources/dos
  kubectl apply -f webapp.yaml
  kubectl apply -f apdos-protected.yaml
  kubectl apply -f apdos-policy.yaml
  kubectl apply -f apdos-logconf.yaml
  kubectl apply -f virtual-server.yaml


Syslogサーバのログの出力状況を確認します。新たに同ホストへ接続するターミナルを2つ用意し、それぞれのターミナルでログを表示してください

SyslogサーバのPod名を確認します

.. code-block:: cmdin
  kubectl get pod

.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル

  NAME                       READY   STATUS    RESTARTS       AGE
  syslog-2-96dfdf5c6-7t8d4   1/1     Running   0              1h
  syslog-cccc648c6-2n9v4     1/1     Running   0              1h
  webapp-64d444885-bgrj7     1/1     Running   0              6m


syslog、syslog-2 それぞれのPOD名を参考に、追加するターミナルでログを表示してください。

.. code-block:: cmdin

  # 追加するターミナル1 で 'syslog' の情報を表示する
  kubectl exec -it <syslog POD名> --  tail -f /var/log/messages
  # 追加するターミナル2 で 'syslog-2' の情報を表示する
  kubectl exec -it <syslog-2 POD名> -- tail -f /var/log/messages


リソースを確認
----

ポイントとなるファイルの内容を確認します。


``apdos-policy.yaml`` は、DosProtectResourceが参照する NAP DoS の Policy 設定となります。

.. code-block:: yaml
  :linenos:
  :caption: apdos-policy.yaml

  apiVersion: appprotectdos.f5.com/v1beta1
  kind: APDosPolicy
  metadata:
    name: dospolicy
  spec:
    mitigation_mode: "standard"
    signatures: "on"
    bad_actors: "on"
    automation_tools_detection: "on"
    tls_fingerprint: "on"


``apdos-logconf.yaml`` は、DosProtectResourceが参照する Security Log の設定となります。

.. code-block:: yaml
  :linenos:
  :caption: apdos-logconf.yaml
  :emphasize-lines: 3

  apiVersion: appprotectdos.f5.com/v1beta1
  kind: APDosLogConf
  metadata:
    name: doslogconf
  spec:
    content:
      format: splunk
      max_message_size: 64k
    filter:
      traffic-mitigation-stats: all
      bad-actors: top 10
      attack-signatures: top 10

``apdos-protected.yaml`` は、VirtualServerが参照する DosProtectResource の設定となります。
利用するNAP DoSのリソースとして ``dospolicy`` を指定し、Dos Access Log、Dos Security Logを指定します。

.. code-block:: yaml
  :linenos:
  :caption: apdos-protected.yaml
  :emphasize-lines: 3,8,13,17

  apiVersion: appprotectdos.f5.com/v1beta1
  kind: DosProtectedResource
  metadata:
    name: dos-protected
  spec:
    enable: true
    name: "webapp.example.com"
    apDosPolicy: "dospolicy"
    apDosMonitor:
      uri: "webapp.example.com"
      protocol: "http1"
      timeout: 5
    dosAccessLogDest: "syslog-svc-2.default.svc.cluster.local:514"
    dosSecurityLog:
      enable: true
      apDosLogConf: "doslogconf"
      dosLogDest: "syslog-svc.default.svc.cluster.local:514"


``virtual-server.yaml`` で、作成した ``dos-protected`` を割り当てます 

.. code-block:: yaml
  :linenos:
  :caption: virtual-server.yaml
  :emphasize-lines: 13

  apiVersion: k8s.nginx.org/v1
  kind: VirtualServer
  metadata:
    name: webapp
  spec:
    host: webapp.example.com
    upstreams:
      - name: webapp
        service: webapp-svc
        port: 80
    routes:
      - path: /
        dos: dos-protected
        action:
          pass: webapp

以下の通り、各リソースを適切に作成されていることを確認します。

.. code-block:: cmdin
  
  kubectl get apdoslogconf

.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル
  :emphasize-lines: 1

  NAME         AGE
  doslogconf   10m

.. code-block:: cmdin
  
  kubectl get apdospolicy

.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル
  :emphasize-lines: 1

  NAME        AGE
  dospolicy   10m

.. code-block:: cmdin
  
  kubectl get DosProtectedResource

.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル
  :emphasize-lines: 1

  NAME            AGE
  dos-protected   11m

.. code-block:: cmdin
  
  kubectl get vs

.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル
  :emphasize-lines: 1

  NAME     STATE   HOST                 IP    PORTS   AGE
  webapp   Valid   webapp.example.com                 12m

.. code-block:: cmdin
  
  kubectl get deployment

.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル
  :emphasize-lines: 1

  NAME       READY   UP-TO-DATE   AVAILABLE   AGE
  syslog     1/1     1            1           1h
  syslog-2   1/1     1            1           1h
  webapp     1/1     1            1           13m


動作確認
----


.. code-block:: cmdin

  curl -H "Host:webapp.example.com" http://localhost/

.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル
  :emphasize-lines: 1

  Server address: 192.168.127.38:8080
  Server name: webapp-64d444885-bgrj7
  Date: 20/Jan/2022:09:30:55 +0000
  URI: /
  Request ID: 8b6810ab8c5a8eabacb9d7da9d775094

.. code-block:: bash
  :linenos:
  :caption: Terminal1 Log (区切り位置で改行して表示)
  :emphasize-lines: 1
  
  # Terminal1 log : 上記アクセスをした際に、以下のログが出力されます
  Jan 20 09:30:55 nginx-ingress-5ddc7f4f-zjlt2 nginx: ,
  vs_name_al=default/dos-protected/webapp.example.com,
  ip=10.1.1.9,
  tls_fp=-,
  outcome=Allow,
  reason=Allow,
  ip_tls=10.1.1.9:-,


.. code-block:: bash
  :linenos:
  :caption: Terminal2 Log (区切り位置で改行して表示)
  :emphasize-lines: 1

  # Terminal2 log : 定期的にログが出力されます
  Jan 20 09:30:57 syslog-cccc648c6-2n9v4 syslog-ng[1]: Syslog connection accepted; fd='20', client='AF_INET(192.168.127.46:34588)', local='AF_INET(0.0.0.0:514)'
  Jan 20 09:30:57 192-168-127-46 date_time="Jan 20 2022 09:30:57",
  product="app-protect-dos",
  product_version="25+2.1.8-1~buster",
  unit_hostname="nginx-ingress-5ddc7f4f-zjlt2",
  instance_id=".scope",
  vs_name="default/dos-protected/webapp.example.com",
  dos_attack_id="0",
  attack_event="No Attack",
  stress_level="0.50",
  learning_confidence="Not ready",
  baseline_dps="0",
  incoming_dps="0",
  incoming_rps="0",
  successful_tps="0",
  unsuccessful_rps="0",
  incoming_datagrams="11",
  incoming_requests="11",
  successful_responses="5",
  unsuccessful_requests="6",
  active_connections="0",
  threshold_dps="2121.60",
  threshold_conns="2121.60",
  mitigated_bad_actors="0",
  mitigated_by_signatures="0",
  mitigated_by_global_rate="0",
  mitigated_slow="0",
  redirect_global="0",
  redirect_bad_actor="0",
  redirect_signature="0",
  redirect_slow="0",
  challenge_global="0",
  challenge_bad_actor="0",
  challenge_signature="0",
  challenge_slow="0",
  block_global="0",
  block_bad_actor="0",
  block_signature="0",
  block_slow="0",
  mitigated_connections="0",
  mitigated_bad_actors_rps="0",
  mitigated_by_signatures_rps="0",
  mitigated_by_global_rate_rps="0",
  mitigated_slow_rps="0",
  redirect_global_rps="0",
  redirect_bad_actor_rps="0",
  redirect_signature_rps="0",
  redirect_slow_rps="0",
  challenge_global_rps="0",
  challenge_bad_actor_rps="0",
  challenge_signature_rps="0",
  challenge_slow_rps="0",
  block_global_rps="0",
  block_bad_actor_rps="0",
  block_signature_rps="0",
  block_slow_rps="0",
  mitigated_connections_rps="0",
  Jan 20 09:30:57 syslog-cccc648c6-2n9v4 syslog-ng[1]: Syslog connection closed; fd='20', client='AF_INET(192.168.127.46:34588)', local='AF_INET(0.0.0.0:514)'


リソースの削除
----

.. code-block:: cmdin

  ## cd ~/kubernetes-ingress/examples/custom-resources/dos
  kubectl delete -f webapp.yaml
  kubectl delete -f apdos-protected.yaml
  kubectl delete -f apdos-policy.yaml
  kubectl delete -f apdos-logconf.yaml
  kubectl delete -f virtual-server.yaml

