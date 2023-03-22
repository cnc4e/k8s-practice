# 回答例

1. 以下を満たすDeploymentをデプロイしてください。なお、lifecycleについては[公式ドキュメント][1]を参考にしてください。

   - 要件
     - Deployment
       - 名前は`nginx`
       - replicas: `1`
       - labelはすべて`app: nginx`
       - Pod
         - 名前は`nginx`
         - lifecycleの`postStart`で`['/bin/sh','-c','date > /usr/share/nginx/html/index.html']`を実行
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
             lifecycle:
               postStart:
                 exec:
                   command:
                     ["/bin/sh", "-c", "date > /usr/share/nginx/html/index.html"]
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
  （`時刻が１行表示されていること`）

   【回答例】

```bash
# testpod作成
$ kubectl run --image=appropriate/curl --restart=Never testpod sleep 3600
pod/testpod created

# 接続確認
$ kubectl exec testpod -- curl -s http://nginx-svc
Mon Sep  26 06:33:13 UTC 2022
```

1. Deployment:`nginx`のマニフェストを以下にように修正して再デプロイしてください。

   - 要件
     - Deployment
       - Pod
         - lifecycleの`preStop`で`['/bin/sh','-c','rm /usr/share/nginx/html/index.html']`を追加

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
             lifecycle:
               postStart:
                 exec:
                   command:
                     ["/bin/sh", "-c", "date > /usr/share/nginx/html/index.html"]
               preStop:
                 exec:
                   command:
                     ['/bin/sh','-c','rm /usr/share/nginx/html/index.html']
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

1. Pod:`lifecycle`を何度か手動で削除しセルフ・ヒーリングさせる。（Deploymentは消さない）

   【回答例】

   ```bash
   # 実行結果
   $ kubectl get pod
   NAME                     READY   STATUS    RESTARTS   AGE
   nginx-6d98b5c6d5-wnw6r   1/1     Running   0          17s
   $ kubectl delete pod nginx-6d98b5c6d5-wnw6r
   pod "nginx-6d98b5c6d5-wnw6r" deleted
   $ kubectl get pod
   NAME                     READY   STATUS    RESTARTS   AGE
   nginx-6d98b5c6d5-79rld   1/1     Running   0          7s
   $ kubectl delete pod nginx-6d98b5c6d5-79rld
   pod "nginx-6d98b5c6d5-79rld" deleted
   $ kubectl get pod
   NAME                     READY   STATUS    RESTARTS   AGE
   nginx-6d98b5c6d5-xl42w   1/1     Running   0          14s
   ```

1. Pod:`curl`から`lifecycle-svc`に対しcurlする。Podを作成した時刻が`一行のみ`出力されることを確認する。

   【回答例】

```bash
# 実行結果
$ kubectl exec testpod -- curl -s http://nginx-svc
Mon Sep  26 07:31:14 UTC 2022
```

1. 作成したリソースを削除してください。

   【回答例】

   ```bash
   # 実行結果
   $ kubectl delete -f nginx.yaml
   deployment.apps/nginx deleted
   service/nginx-svc deleted
   $ kubectl delete pod testpod
   pod "testpod" deleted
   ```

[1]:https://kubernetes.io/ja/docs/tasks/configure-pod-container/attach-handler-lifecycle-event/
