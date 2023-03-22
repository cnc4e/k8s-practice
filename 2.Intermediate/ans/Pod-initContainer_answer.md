# 回答例

1. 以下を満たすDeployment, Serviceをデプロイしてください。

   - 要件
     - Deployment
       - 名前は`nginx`
       - replicas: `1`
       - labelはすべて`app: nginx`
       - Pod
         - 名前は`nginx`
     - Service
       - 名前は`nginx-svc`
       - 対象のlabelは`app: nginx`
       - プロトコルは`TCP`
       - Portは`80`
       - targetPortは`80`

   【回答例】

   ```yml
   # manifest
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: nginx
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
             ports:
               - containerPort: 80
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
   deployment.apps/nginx created
   service/nginx-svc created
   ```

1. 接続確認に使用するcurlコマンドが実行可能なtestpodを作成し、testpodから`Service:nginx-svc`に対して接続確認をしてください。  
   （`Nginxデフォルトのindex.htmlが表示されること`）

   【回答例】

   ```bash
   # testpod作成
   $ kubectl run --image=appropriate/curl --restart=Never --namespace=web    testpod sleep 3600
   pod/testpod created

   # 接続確認
   $ kubectl exec testpod -- curl -sL http://nginx-service/index.html
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

1. マニフェストを以下の内容に修正し、デプロイしてください。なお、initContainerについては[公式ドキュメント][1]を参考にしてください。

   - 要件
     - Deployment
       - Pod
         - nginx
           - volume:index-htmlを`/usr/share/nginx/html`にマウント
         - initContainer
           - 名前`init`
           - イメージは`busybox`
           - commandは`['/bin/sh','-c','echo initContainer de settei sita noda > /tmp/   index.html']`
           - volume:index-htmlを`/tmp`にマウント
         - volume
           - 名前は`index-html`
           - volumeプラグインは`emptyDir`を指定

   【回答例】

   ```yml
   # manifest
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: nginx
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
             ports:
               - containerPort: 80
             volumeMounts:
               - name: index-html
                 mountPath: /usr/share/nginx/html
         initContainers:
           - name: init
             image: busybox
             command:
               ["/bin/sh","-c","echo initContainer de settei sita noda > /tmp/index.html"]
             volumeMounts:
               - name: index-html
                 mountPath: /tmp
         volumes:
           - name: index-html
             emptyDir:
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
   deployment.apps/nginx configured
   service/nginx-svc unchanged
   ```

1. Podの詳細を確認し、init Containers の箇所を確認してください。また、eventを確認しコンテナinitがメインコンテナの前に起動したことを確認してください。

   【回答例】

   ```bash
   # 実行結果
   ## Pod一覧の確認
   $ kubectl get pod
   NAME                            READY   STATUS    RESTARTS   AGE
   nginx-859988b86d-kr2b7   1/1     Running   0          19s

   ## Podの詳細確認
   $ kubectl describe pod nginx-859988b86d-kr2b7
   Name:         nginx-859988b86d-kr2b7
   Namespace:    default
   Priority:     0
   Node:         ip-192-168-13-204.us-west-1.compute.internal/192.168.13.204
   Start Time:   Thu, 29 Sep 2022 16:50:53 +0900
   Labels:       app=nginx
                 pod-template-hash=859988b86d
   Annotations:  kubectl.kubernetes.io/restartedAt: 2022-09-29T16:39:20+09:00
                 kubernetes.io/psp: eks.privileged
   Status:       Running
   IP:           192.168.25.50
   IPs:
     IP:           192.168.25.50
   Controlled By:  ReplicaSet/nginx-859988b86d
   Init Containers: ★
     init:
       Container ID:  docker://   e57cfb8b4bb98be60b2023709e59112fb3a3997ffc12e88b64e5d3309725aef7
       Image:         busybox:1.28
       Image ID:      docker-pullable://   busybox@sha256:141c253bc4c3fd0a201d32dc1f493bcf3fff003b6df416dea4f41046e0f37d47
       Port:          <none>
       Host Port:     <none>
       Command:
         /bin/sh
         -c
         echo initContainer de settei sita noda > /tmp/index.html
       State:          Terminated
         Reason:       Completed
         Exit Code:    0
         Started:      Thu, 29 Sep 2022 16:50:54 +0900
         Finished:     Thu, 29 Sep 2022 16:50:54 +0900
       Ready:          True
       Restart Count:  0
       Environment:    <none>
       Mounts:
         /tmp from index-html (rw)
         /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-s2cmg (ro)
     (中略)
   Events: ★
     Type    Reason     Age   From               Message
     ----    ------     ----  ----               -------
     Normal  Scheduled  14m   default-scheduler  Successfully assigned web/   nginx-859988b86d-kr2b7 to ip-192-168-13-204.us-west-1.compute.internal
     Normal  Pulled     13m   kubelet            Container image "busybox:1.28"    already present on machine
     Normal  Created    13m   kubelet            Created container init
     Normal  Started    13m   kubelet            Started container init
     Normal  Pulled     13m   kubelet            Container image "dayan888/   springdemo:nginx" already present on machine
     Normal  Created    13m   kubelet            Created container nginx
     Normal  Started    13m   kubelet            Started container nginx
   ```

1. testpodから`Service:nginx-service/index.html`に対して接続確認をしてください。（`initContainerで修正したindex.htmlが表示されること`）

   【回答例】

   ```bash
   $ kubectl exec testpod -- curl -s http://nginx-svc
   initContainer de settei sita noda
   ```

1. 作成したリソースを削除してください。

   【回答例】

   ```bash
   # 実行結果
   $ kubectl delete -f nginx.yaml
   deployment.apps/nginx deleted
   service/nginx-svc deleted
   ```

[1]:https://kubernetes.io/docs/concepts/workloads/pods/init-containers/#init-containers-in-use
