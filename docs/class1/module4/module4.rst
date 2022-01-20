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
  :emphasize-lines: 1

  syslog-svc     ClusterIP   10.96.250.209    <none>        514/TCP   9m
  syslog-svc-2   ClusterIP   10.103.224.109   <none>        514/UDP   9m


Ingress Controller で WAF機能(NGINX App Protect WAF) のデプロイ
====

https://github.com/nginxinc/kubernetes-ingress/tree/v2.1.0/examples/custom-resources/waf


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