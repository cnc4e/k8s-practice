
# 回答例

1. Namespaceリソースのオブジェクト一覧を表示してください。

   【回答例】

   ```bash
   $ kubectl get namespaces
   NAME              STATUS   AGE
   default           Active   15h
   kube-node-lease   Active   15h
   kube-public       Active   15h
   kube-system       Active   15h
   ```

1. Namespace:kube-systemに属するPodリソースのオブジェクト一覧を表示してください。  
   (-n <Namespace名> で参照するNamespaceを指定できます)

   【回答例】

   ```bash
   $ kubectl get pod -n kube-system
   NAME                      READY   STATUS    RESTARTS   AGE
   aws-node-g9jh8            1/1     Running   0          15h
   aws-node-tdw42            1/1     Running   0          15h
   aws-node-zlcvp            1/1     Running   0          15h
   coredns-66747cfb9-mpxbq   1/1     Running   0          16h
   coredns-66747cfb9-nfzw7   1/1     Running   0          16h
   kube-proxy-72sng          1/1     Running   0          15h
   kube-proxy-g74wg          1/1     Running   0          15h
   kube-proxy-s9f5r          1/1     Running   0          15h
   ```

1. 以下を満たすマニフェストを作成しデプロイしてください。なお、Namespaceについては[公式ドキュメント][1]を参考にしてください。

   - 要件
     - Namespace
       - 名前は`web`
     - Deployment
       - 名前は`nginx`
       - Namespaceは`web`
       - replicas: `1`
       - labelはすべて`app: nginx`
       - Pod
         - 名前は`nginx`
     - Service
       - 名前は`nginx-svc`
       - Namespaceは`web`
       - 対象のlabelは`app: nginx`
       - プロトコルは`TCP`
       - Portは`80`
       - targetPortは`80`

   【回答例】

   ```yml
   # manifest
   apiVersion: v1
   kind: Namespace
   metadata:
     name: web
   ---
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: nginx
     namespace: web
   spec:
     replicas: 1
     selector:
       matchLabels:
         app: nginx
     template:
       metadata:
         labels:
           app: nginx
       spec:
         containers:
           - name: nginx
             image: nginx:1.12
   ---
   apiVersion: v1
   kind: Service
   metadata:
     name: nginx-svc
   spec:
     ports:
      - name: "http-port"
        protocol: "TCP"
        port: 80
        targetPort: 80
     selector:
       app: nginx
   ```

   ```bash
   # 実行結果
   $ kubectl apply -f nginx.yaml
   namespace/web created
   deployment.apps/nginx created
   service/nginx-svc created
   ```

1. Namespaceリソースのオブジェクト一覧を表示し、`web`が作成されていることを確認してください。

   【回答例】

   ```bash
   $ kubectl get namespaces
   NAME              STATUS   AGE
   default           Active   16h
   kube-node-lease   Active   16h
   kube-public       Active   16h
   kube-system       Active   16h
   web               Active   4m36s
   ```

1. Namespace:`default`のDeployment, Pod, Serviceのオブジェクト一覧を表示し、上記手順で作成したAPIリソースが`表示されない`ことを確認してください。

   【回答例】

   ```bash
   # Deployment
   $ kubectl get deployment
   No resources found in default namespace.

   # Pod
   $ kubectl get pod
   No resources found in default namespace.

   # Service
   $ kubectl get service
   NAME         TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
   kubernetes   ClusterIP   10.100.0.1   <none>        443/TCP   16h
   ```

1. Namespace:`web`のDeployment, Pod, Serviceのオブジェクト一覧を表示し、`nginx`を含んだDeployment, Pod, Serviceがあることを確認してください。

   【回答例】

   ```bash
   # Deployment
   $ kubectl get deployment -n web
   NAME           READY   UP-TO-DATE   AVAILABLE   AGE
   nginx          1/1     1            1           7m14s

   # Pod
   $ kubectl get pod -n web
   NAME                            READY   STATUS    RESTARTS   AGE
   nginx-55df9d9465-nwz8z          1/1     Running   0          7m34s

   # Service
   $ kubectl get service -n web
   NAME                   TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)    AGE
   nginx-svc              ClusterIP   10.100.105.107   <none>        80/TCP   7m48s
   ```

1. 接続確認に使用するcurlコマンドが実行可能なtestpodを`Namespace:default`に作成し、testpodから`Service:nginx-svc`に対して接続確認をしてください。  

   【回答例】

   ```bash
   # 実行結果
   ## testpod作成
   $ kubectl run --image=appropriate/curl --restart=Never testpod sleep 3600
   pod/testpod created

   ## 接続確認
   $ kubectl exec testpod -- curl -s http://nginx-svc
   command terminated with exit code 6
   ```

1. testpodから`Namespace:webに所属するService:nginx-svc`に対して接続確認をしてください。  
   （別NamespaceのServiceを指定する場合は`<service名>.<namespace名>`と表記します。）

   【回答例】

   ```bash
   # 接続確認
   $ kubectl exec testpod -- curl -s http://nginx-svc.web
   <!DOCTYPE html>
   <html>
   <head>
   <title>Welcome to nginx!</title>
   <style>
       body {
           width: 35em;
           margin: 0 auto;
           font-family: Tahoma, Verdana, Arial, sans-serif;
       }
   </style>
   </head>
   <body>
   <h1>Welcome to nginx!</h1>
   <p>If you see this page, the nginx web server is successfully installed and
   working. Further configuration is required.</p>

   <p>For online documentation and support please refer to
   <a href="http://nginx.org/">nginx.org</a>.<br/>
   Commercial support is available at
   <a href="http://nginx.com/">nginx.com</a>.</p>

   <p><em>Thank you for using nginx.</em></p>
   </body>
   </html>
   ```

1. 接続確認に使用するcurlコマンドが実行可能なtestpodを`Namespace:web`に作成し、testpodから`Service:nginx-svc`に対して接続確認をしてください。

   【回答例】

   ```bash
   # 実行結果
   ## testpod作成
   $ kubectl run --image=appropriate/curl --restart=Never --namespace=web testpod sleep 3600
   pod/testpod created

   # 接続確認
   $ kubectl exec testpod -- curl -s http://nginx-svc
   <!DOCTYPE html>
   <html>
   <head>
   <title>Welcome to nginx!</title>
   <style>
       body {
           width: 35em;
           margin: 0 auto;
           font-family: Tahoma, Verdana, Arial, sans-serif;
       }
   </style>
   </head>
   <body>
   <h1>Welcome to nginx!</h1>
   <p>If you see this page, the nginx web server is successfully installed and
   working. Further configuration is required.</p>

   <p>For online documentation and support please refer to
   <a href="http://nginx.org/">nginx.org</a>.<br/>
   Commercial support is available at
   <a href="http://nginx.com/">nginx.com</a>.</p>

   <p><em>Thank you for using nginx.</em></p>
   </body>
   </html>
   ```

1. 作成したリソースを削除してください。

   【回答例】

   ```bash
   # 実行結果
   $ kubectl delete -f nginx.yaml
   namespace "web" deleted
   deployment.apps/nginx deleted
   service/nginx-svc deleted
   $ kubectl delete pod testpod
   pod "testpod" deleted
   $ kubectl delete pod -n web testpod
   pod "testpod" deleted
   ```

[1]:https://kubernetes.io/docs/tasks/administer-cluster/namespaces/#creating-a-new-namespace
