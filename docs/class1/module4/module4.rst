NICによるWebアプリのセキュリティ
####

この章では、実際にデプロイしたNGINX Ingress Controllerを使い、Webアプリケーションに対するWAF・DoS対策の実施方法を確認します。
設定例は `NGINX Inc GitHubの examples/custom-resources/ <https://github.com/nginxinc/kubernetes-ingress/tree/v2.1.0/examples/custom-resources>`__ に管理されております

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