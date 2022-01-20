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

取得するSyslogイメージのタグを変更

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
  :emphasize-lines: 1

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

.. code-block:: yaml
  :linenos:
  :caption: ap-apple-uds.yaml
  :emphasize-lines: 1

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
  
.. code-block:: yaml
  :linenos:
  :caption: ap-logconf.yaml
  :emphasize-lines: 1

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
    cat waf.yaml
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

.. code-block:: yaml
  :linenos:
  :caption: ap-dataguard-alarm-policy.yaml
  :emphasize-lines: 1

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

.. code-block:: yaml
  :linenos:
  :caption: virtual-server.yaml
  :emphasize-lines: 1

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


.. code-block:: cmdin

  kubectl apply -f webapp.yaml
  kubectl apply -f ap-apple-uds.yaml
  kubectl apply -f ap-dataguard-alarm-policy.yaml
  kubectl apply -f ap-logconf.yaml
  kubectl apply -f waf.yaml
  kubectl apply -f virtual-server.yaml


リソースを確認
----

.. code-block:: cmdin

  kubectl get APUserSig

.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル

  NAME    AGE
  apple   38m

.. code-block:: cmdin

  kubectl get aplogconf

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

  NAME                  STATE   AGE
  ingress-mtls-policy   Valid   11h
  waf-policy            Valid   39m

.. code-block:: cmdin

  kubectl get appolicy

.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル

  NAME              AGE
  dataguard-alarm   39m

.. code-block:: cmdin

  kubectl get policy

.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル

  NAME                  STATE   AGE
  ingress-mtls-policy   Valid   11h
  waf-policy            Valid   39m

.. code-block:: cmdin

  kubectl get policy

.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル
  
  NAME         STATE   AGE
  waf-policy   Valid   41m
  

動作確認
----



.. code-block:: cmdin

  curl -v --resolve webapp.example.com:80:127.0.0.1 "http://webapp.example.com/"

.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル

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

.. code-block:: xml
  :linenos:
  :caption: 該当するSyslogのサンプル

  Jan 20 03:07:28 nginx-ingress-5ddc7f4f-zjlt2 ASM:attack_type="Non-browser Client",blocking_exception_reason="N/A",date_time="2022-01-20 03:07:28",dest_port="80",ip_client="10.1.1.9",is_truncated="false",method="GET",policy_name="dataguard-alarm",protocol="HTTP",request_status="alerted",response_code="200",severity="Critical",sig_cves="N/A",sig_ids="N/A",sig_names="N/A",sig_set_names="N/A",src_port="49443",sub_violations="N/A",support_id="16242938385820378173",threat_campaign_names="N/A",unit_hostname="nginx-ingress-5ddc7f4f-zjlt2",uri="/",violation_rating="0",vs_name="32-webapp.example.com:8-/",x_forwarded_for_header_value="N/A",outcome="PASSED",outcome_reason="SECURITY_WAF_VIOLATION_TRANSPARENT_MODE",violations="Bot Client Detected",violation_details="N/A",bot_signature_name="curl",bot_category="HTTP Library",bot_anomalies="N/A",enforced_bot_anomalies="N/A",client_class="Untrusted Bot",client_application="N/A",client_application_version="N/A",request="GET / HTTP/1.1\r\nHost: webapp.example.com\r\nUser-Agent: curl/7.68.0\r\nAccept: */*\r\n\r\n",transport_protocol="HTTP/1.1"

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


.. code-block:: cmdin

  curl -v --resolve webapp.example.com:80:127.0.0.1 "http://webapp.example.com/<script>"

.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル

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
  <html><head><title>Request Rejected</title></head><body>The requested URL was rejected. Please consult with your administrator.<br><br>Your support ID is: 16242938385820378683<br><br><a href='javascript:history.back();'>[Go Back]</a></body></html>ubuntu@ip-10-1-1-8:~/kubernetes-ingress/examples/custom-resources/waf$

.. code-block:: xml
  :linenos:
  :caption: 該当するSyslogのサンプル

  Jan 20 03:07:39 nginx-ingress-5ddc7f4f-zjlt2 ASM:attack_type="Non-browser Client,Abuse of Functionality,Cross Site Scripting (XSS)",blocking_exception_reason="N/A",date_time="2022-01-20 03:07:39",dest_port="80",ip_client="10.1.1.9",is_truncated="false",method="GET",policy_name="dataguard-alarm",protocol="HTTP",request_status="blocked",response_code="0",severity="Critical",sig_cves="N/A",sig_ids="200000099,200000093",sig_names="XSS script tag (URI),XSS script tag end (URI)",sig_set_names="{Cross Site Scripting Signatures;High Accuracy Signatures},{Cross Site Scripting Signatures;High Accuracy Signatures}",src_port="61276",sub_violations="N/A",support_id="16242938385820378683",threat_campaign_names="N/A",unit_hostname="nginx-ingress-5ddc7f4f-zjlt2",uri="/<script>",violation_rating="5",vs_name="32-webapp.example.com:8-/",x_forwarded_for_header_value="N/A",outcome="REJECTED",outcome_reason="SECURITY_WAF_VIOLATION",violations="Illegal meta character in URL,Attack signature detected,Violation Rating Threat detected,Bot Client Detected",violation_details="<?xml version='1.0' encoding='UTF-8'?><BAD_MSG><violation_masks><block>410000000200c00-3a03030c30000072-8000000000000000-0</block><alarm>2477f0ffcbbd0fea-befbf35cb000007e-8000000000000000-0</alarm><learn>0-20-0-0</learn><staging>0-0-0-0</staging></violation_masks><request-violations><violation><viol_index>42</viol_index><viol_name>VIOL_ATTACK_SIGNATURE</viol_name><context>url</context><sig_data><sig_id>200000099</sig_id><blocking_mask>3</blocking_mask><kw_data><buffer>LzxzY3JpcHQ+</buffer><offset>1</offset><length>7</length></kw_data></sig_data><sig_data><sig_id>200000093</sig_id><blocking_mask>3</blocking_mask><kw_data><buffer>LzxzY3JpcHQ+</buffer><offset>2</offset><length>7</length></kw_data></sig_data></violation><violation><viol_index>26</viol_index><viol_name>VIOL_URL_METACHAR</viol_name><uri>LzxzY3JpcHQ+</uri><metachar_index>60</metachar_index><wildcard_entity>*</wildcard_entity><staging>0</staging></violation><violation><viol_index>26</viol_index><viol_name>VIOL_URL_METACHAR</viol_name><uri>LzxzY3JpcHQ+</uri><metachar_index>62</metachar_index><wildcard_entity>*</wildcard_entity><staging>0</staging></violation></request-violations></BAD_MSG>",bot_signature_name="curl",bot_category="HTTP Library",bot_anomalies="N/A",enforced_bot_anomalies="N/A",client_class="Untrusted Bot",client_application="N/A",client_application_version="N/A",request="GET /<script> HTTP/1.1\r\nHost: webapp.example.com\r\nUser-Agent: curl/7.68.0\r\nAccept: */*\r\n\r\n",transport_protocol="HTTP/1.1"

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



.. code-block:: cmdin

  curl -v --resolve webapp.example.com:80:127.0.0.1 "http://webapp.example.com/" -X POST -d "apple"

.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル

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
  <html><head><title>Request Rejected</title></head><body>The requested URL was rejected. Please consult with your administrator.<br><br>Your support ID is: 16242938385820379193<br><br><a href='javascript:history.back();'>[Go Back]</a></body></html>ubuntu@ip-10-1-1-8:~/kubernetes-ingress/examples/custom-resources/waf$

.. code-block:: xml
  :linenos:
  :caption: 該当するSyslogのサンプル

  Jan 20 03:07:51 nginx-ingress-5ddc7f4f-zjlt2 ASM:attack_type="Non-browser Client,Brute Force Attack",blocking_exception_reason="N/A",date_time="2022-01-20 03:07:51",dest_port="80",ip_client="10.1.1.9",is_truncated="false",method="POST",policy_name="dataguard-alarm",protocol="HTTP",request_status="blocked",response_code="0",severity="Critical",sig_cves="N/A",sig_ids="300000000",sig_names="Apple_medium_acc [Fruits]",sig_set_names="{apple_sigs}",src_port="63409",sub_violations="N/A",support_id="16242938385820379193",threat_campaign_names="N/A",unit_hostname="nginx-ingress-5ddc7f4f-zjlt2",uri="/",violation_rating="2",vs_name="32-webapp.example.com:8-/",x_forwarded_for_header_value="N/A",outcome="REJECTED",outcome_reason="SECURITY_WAF_VIOLATION",violations="Attack signature detected,Bot Client Detected",violation_details="<?xml version='1.0' encoding='UTF-8'?><BAD_MSG><violation_masks><block>410000000200c00-3a03030c30000072-8000000000000000-0</block><alarm>2477f0ffcbbd0fea-befbf35cb000007e-8000000000000000-0</alarm><learn>0-20-0-0</learn><staging>0-0-0-0</staging></violation_masks><request-violations><violation><viol_index>42</viol_index><viol_name>VIOL_ATTACK_SIGNATURE</viol_name><context>request</context><sig_data><sig_id>300000000</sig_id><blocking_mask>3</blocking_mask><kw_data><buffer>YXBwbGU=</buffer><offset>0</offset><length>5</length></kw_data></sig_data></violation></request-violations></BAD_MSG>",bot_signature_name="curl",bot_category="HTTP Library",bot_anomalies="N/A",enforced_bot_anomalies="N/A",client_class="Untrusted Bot",client_application="N/A",client_application_version="N/A",request="POST / HTTP/1.1\r\nHost: webapp.example.com\r\nUser-Agent: curl/7.68.0\r\nAccept: */*\r\nContent-Length: 5\r\nContent-Type: application/x-www-form-urlencoded\r\n\r\napple",transport_protocol="HTTP/1.1"

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
リソースを確認
----
動作確認
----
リソースの削除
----

.. code-block:: cmdin

.. code-block:: bash
  :linenos:
  :caption: 実行結果サンプル
  :emphasize-lines: 1
