NGINX Ingress Controller 環境のセットアップ
####

NGINX Plus / NGINX App Protect Ingress ControllerのDockerイメージ作成
====


| 以下の手順に従ってNGINX Ingress Controllerのイメージを作成します  
| 参考： `Installation with Manifests <https://docs.nginx.com/nginx-ingress-controller/installation/installation-with-manifests/>`__

1. 環境セットアップ
----

ファイルの取得します

::

   cd ~/
   git clone https://github.com/nginxinc/kubernetes-ingress/
   cd ~/kubernetes-ingress
   git checkout v2.1.0


ライセンスファイルをコピーしてください
ファイルが配置されていない場合、トライアルを申請し証明書と鍵を取得してください

::

   cp ~/nginx-repo* .
   ls nginx-repo.*

2. コンテナイメージの作成
----

| NGINX Plus ＋ NGINX App Protectのコンテナイメージを作成します
| 参考： `Building the Ingress Controller Image <https://docs.nginx.com/nginx-ingress-controller/installation/building-ingress-controller-image>`__

::
   
   make debian-image-nap-plus PREFIX=registry.example.com/root/nic/nginxplus-ingress-nap-dos TARGET=container TAG=2.0.3
   # Image の Build は数分(約5分)必要となります
   docker images | grep nginxplus-ingress-nap-dos

   ** 実行結果サンプル **
   registry.example.com/root/nic/nginxplus-ingress-nap-dos   2.1.0     5b5cdc61cf76   31 seconds ago   611MB


Container Image のPushのためにレジストリへログイン

::

   # registry.example.com にログイン
   docker login registry.example.com
   Username: root       << 左の文字列を入力
   Password: password   << 左の文字列を入力

   ** 実行結果サンプル **
   WARNING! Your password will be stored unencrypted in /home/ubuntu/.docker/config.json.
   Configure a credential helper to remove this warning. See
   https://docs.docker.com/engine/reference/commandline/login/#credentials-store

   Login Succeeded

Container Image のPush

::

   docker push registry.example.com/root/nic/nginxplus-ingress-nap-dos:2.1.0


3. NGINX Ingress Controller環境のセットアップ
----

先程の手順で取得したGitHubのフォルダへ移動し、必要となるリソースをデプロイします。

::
   
	cd ~/kubernetes-ingress/deployments
	kubectl apply -f common/ns-and-sa.yaml
	kubectl apply -f rbac/rbac.yaml
	kubectl apply -f rbac/ap-rbac.yaml
   kubectl apply -f rbac/apdos-rbac.yaml
	kubectl apply -f common/default-server-secret.yaml
	kubectl apply -f common/nginx-config.yaml
	kubectl apply -f common/ingress-class.yaml
	kubectl apply -f common/crds/k8s.nginx.org_virtualservers.yaml
	kubectl apply -f common/crds/k8s.nginx.org_virtualserverroutes.yaml
	kubectl apply -f common/crds/k8s.nginx.org_transportservers.yaml
	kubectl apply -f common/crds/k8s.nginx.org_policies.yaml
	kubectl apply -f common/crds/k8s.nginx.org_globalconfigurations.yaml
	kubectl apply -f common/crds/appprotect.f5.com_aplogconfs.yaml
	kubectl apply -f common/crds/appprotect.f5.com_appolicies.yaml
	kubectl apply -f common/crds/appprotect.f5.com_apusersigs.yaml
   kubectl apply -f common/crds/appprotectdos.f5.com_apdoslogconfs.yaml
   kubectl apply -f common/crds/appprotectdos.f5.com_apdospolicies.yaml
   kubectl apply -f common/crds/appprotectdos.f5.com_dosprotectedresources.yaml


4. NGINX Ingress Controllerの実行
----

NGINX Ingress Controllerのpodを実行します。DeploymentとDaemonSetによる実行が可能ですが、のこの記事ではDeploymentで実行します。DaemonSetで実行したい場合にはマニュアルを参照して適切に読み替えて進めてください。

argsで指定するパラメータの詳細は [Command-line Arguments](https://docs.nginx.com/nginx-ingress-controller/configuration/global-configuration/command-line-arguments)を参照してください

::

   ## cd ~/kubernetes-ingress/deployments
   vi deployment/nginx-plus-ingress.yaml

コメントを付与した行を適切な内容に修正してください

::

   ** 省略 **
      spec:
         serviceAccountName: nginx-ingress
         containers:
         - image: registry.example.com/root/nic/nginxplus-ingress-nap-dos:2.1.0  # 対象のレジストリを指定してください
         imagePullPolicy: IfNotPresent
         name: nginx-plus-ingress
   ** 省略 **
         args:
            - -nginx-plus
            - -nginx-configmaps=$(POD_NAMESPACE)/nginx-config
            - -default-server-tls-secret=$(POD_NAMESPACE)/default-server-secret
            - -enable-app-protect                            # App Protect WAFを有効にします
            #- -enable-app-protect-dos                        # App Protect DoSを利用する場合、有効にします
            #- -v=3 # Enables extensive logging. Useful for troubleshooting.
            #- -report-ingress-status
            #- -external-service=nginx-ingress
            #- -enable-prometheus-metrics
            #- -global-configuration=$(POD_NAMESPACE)/nginx-configuration
            - -enable-preview-policies                       # OIDCに必要となるArgsを有効にします
            - -enable-snippets                               # OIDCで一部設定を追加するためsnippetsを有効にします


修正したマニフェストを指定しPodを作成します。

::
   
   ## cd ~/kubernetes-ingress/deployments
   kubectl apply -f deployment/nginx-plus-ingress.yaml
   
   ** 実行結果サンプル **
   deployment.apps/nginx-ingress created

   kubectl get pods --namespace=nginx-ingress
   
   ** 実行結果サンプル **
   NAME                             READY   STATUS             RESTARTS   AGE
   nginx-ingress-7f67968b56-d8gf5       1/1     Running   0          3s

   kubectl get deployment -n nginx-ingress

   ** 実行結果サンプル **
   NAME            READY   UP-TO-DATE   AVAILABLE   AGE
   nginx-ingress   1/1     1            1           2m52s


5. NGINX Ingress Controller を外部へ NodePort で公開する
----

本ラボの環境ではKubernetesへのアクセスを受けるため、NGINX Ingress Controllerを外部へNodePortで公開します。
以下コマンドで設定の内容を確認します。type NodePortでHTTP、HTTPSで待ち受ける設定であることを確認します。

::
   
   ## cd ~/kubernetes-ingress/deployments
   cat service/nodeport.yaml

   ** 実行結果サンプル **
	apiVersion: v1
	kind: Service
	metadata:
	  name: nginx-ingress
	  namespace: nginx-ingress
	spec:
	  type: NodePort
	  ports:
	  - port: 80
	    targetPort: 80
	    protocol: TCP
	    name: http
	  - port: 443
	    targetPort: 443
	    protocol: TCP
	    name: https
	  selector:
	    app: nginx-ingress


NodePortをデプロイします。

::
   
   ## cd ~/kubernetes-ingress/deployments
   kubectl apply -f service/nodeport.yaml

   ** 実行結果サンプル **
	service/nginx-ingress created

	kubectl get svc -n nginx-ingress

   ** 実行結果サンプル **
   NAME            TYPE       CLUSTER-IP       EXTERNAL-IP   PORT(S)                      AGE
   nginx-ingress   NodePort   10.108.250.160   <none>        80:32692/TCP,443:31957/TCP   5s



このコマンドを実行した結果、Kubernetes の Worker Nodeでそれぞれのサービスに対しポートが割り当てられています。
図の内容を確認してください。

   .. image:: ./media/nodeport.jpg
       :width: 400

| クライアントからアクセスするため、HTTP(TCP/80)、HTTPS(TCP/443)を待ち受け、それぞれNodePortで公開するポート番号へ転送するLBを用意します。
| 今回のラボ環境では同Linux Host上にNGINX Plusをインストールし以下nginx.confとしました。NGINX OSSでも同様の設定で問題ありません

.. NOTE::
   NGINX Plusをインストールする場合、こちらの手順「 `NGINX Plusのインストール (15min) <https://f5j-nginx-plus-lab1.readthedocs.io/en/latest/class1/module2/module2.html#nginx-plus-15min>`__」を参考に、NGINX Plusをインストールしてください。

先程確認したNoder Portで割り当てられたポート番号宛に通信を転送するように、NGINXを設定します。

::

   sudo cp /etc/nginx/nginx.conf /etc/nginx/nginx.conf-
   sudo cat << EOF > nginx.conf
   user  nginx;
   worker_processes  auto;

   error_log  /var/log/nginx/error.log notice;
   pid        /var/run/nginx.pid;


   events {
      worker_connections  1024;
   }


   # TCP/UDP load balancing
   #
   stream {
      upstream tcp80_backend {
         server localhost:32692；    # HTTP(TCP/80)に割り当てられたポート番号
      }
      upstream tcp443_backend {
         server localhost:31957;     # HTTPS(TCP/443)に割り当てられたポート番号
      }

      server {
         listen 80;
         proxy_pass tcp80_backend;
      }
      server {
         listen 443;
         proxy_pass tcp443_backend;
      }
   }
   EOF
   sudo cp nginx.conf /etc/nginx/nginx.conf
   sudo nginx -s reload

   
現在の状態は以下となり、サービスを外部に公開する準備が完了しました。

   .. image:: ./media/set_external_nginx.jpg
       :width: 400