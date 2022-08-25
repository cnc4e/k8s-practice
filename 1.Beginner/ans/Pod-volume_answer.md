# 回答例

## チュートリアル1: Pod再起動時にデータが失われることを確認する

1. クラスタの参加ノードを確認しワーカーノードが`1台だけ`の状態にする。

   ```bash
   $ kubectl get node
   NAME                                         STATUS   ROLES    AGE     VERSION
   ip-192-168-33-4.us-west-1.compute.internal   Ready    <none>   2d19h   v1.22.12-eks-ba74326
   ```

1. 次のmanifestを使用して、DeploymentとServiceを作成する。

   ```yml
   kind: Service
   apiVersion: v1
   metadata:
     name: volume-svc
   spec:
     selector:
       app: volume
     ports:
     - protocol: TCP
       port: 80
       targetPort: 80
   ```

   ```yml
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: nginx
     labels:
       app: volume
   spec:
     replicas: 1
     selector:
       matchLabels:
         app: volume
     template:
       metadata:
         labels:
           app: volume
       spec:
         containers:
         - name: nginx
           image: nginx:1.22
   ```

   【回答例】

   ```bash
   # manifestを作成する
   cat <<EOF > ./Service.yaml
   kind: Service
   apiVersion: v1
   metadata:
     name: volume-svc
   spec:
     selector:
       app: volume
     ports:
     - protocol: TCP
       port: 80
       targetPort: 80
   EOF

   cat <<EOF > ./Deployment.yaml
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: nginx
     labels:
       app: volume
   spec:
     replicas: 1
     selector:
       matchLabels:
         app: volume
     template:
       metadata:
         labels:
           app: volume
       spec:
         containers:
         - name: nginx
           image: nginx:1.22
   EOF

   # DeplymentとServiceを作成する
   $ kubectl apply -f Deployment.yaml
   deployment.apps/nginx created
   $
   $ kubectl apply -f Service.yaml
   service/volume-svc created
   ```

1. デプロイしたPod内のコンテナに対して追加コマンドを発行し、/usr/share/nginx/html/index.htmlを以下内容に修正する。

   ```text
   zettai ni nakunatte ha ikenai data ga kokoni aru
   ```

   【回答例】

   ```bash
   # Pod名の確認
   $ kubectl get pod
   NAME                    READY   STATUS    RESTARTS   AGE
   nginx-7f7c98667-khks5   1/1     Running   0          103s

   # index.html修正
   $ kubectl exec nginx-7f7c98667-khks5 -- bash -c "echo 'zettai ni nakunatte ha ikenai data ga kokoni aru' > /usr/share/nginx/html/index.html"
   ```

1. curlを実行できるPodを展開し、Service:volume-svcを経由してDeploymentにアクセスする。
  （さきほど修正したindex.htmlが表示`される`ことを確認する）

   【回答例】

   ```bash
   # curl の起動
   $ kubectl run curl --image=appropriate/curl -- /bin/sh -c "sleep 3600"
   pod/curl created

   # Deploymentへアクセス
   $ kubectl exec curl -- curl -s volume-svc
   zettai ni nakunatte ha ikenai data ga kokoni aru
   ```

1. Podを削除（Deploymentは消さない！）し、Podが再作成されたことを確認する。

   【回答例】

   ```bash
   # Pod名の確認
   $ kubectl get pod
   NAME                    READY   STATUS    RESTARTS   AGE
   curl                    1/1     Running   0          2m10s
   nginx-7f7c98667-khks5   1/1     Running   0          103s

   # Podの削除
   $ kubectl delete pod nginx-7f7c98667-khks5
   pod "nginx-7f7c98667-khks5" deleted

   # Pod名の確認
   $ kubectl get pod
   NAME                    READY   STATUS    RESTARTS   AGE
   curl                    1/1     Running   0          2m10s
   nginx-7f7c98667-p9d7h   1/1     Running   0          7s
   ```

1. 再度Service:volume-svcを経由してDeploymentにアクセスする。
  （さきほど修正したindex.htmlが表示`されない`ことを確認する）

   【回答例】

   ```bash
   # Deploymentへアクセス
   $ kubectl exec curl -- curl -s volume-svc
   <!DOCTYPE html>
   <html>
   <head>
   <title>Welcome to nginx!</title>
   <style>
   html { color-scheme: light dark; }
   body { width: 35em; margin: 0 auto;
   font-family: Tahoma, Verdana, Arial, sans-serif; }
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

1. Deploymentを削除してください。（Serviceはそのままで良い）

   【回答例】

   ```bash
   $ kubectl delete -f Deployment.yaml
   deployment.apps "nginx" deleted
   ```

## チュートリアル2: hostPathを使用することで、データが保持され続けることを確認する

1. 次のmanifestを使用して、Deploymentを作成する。

   ```yml
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: nginx
      labels:
        app: volume
    spec:
      replicas: 1
      selector:
        matchLabels:
          app: volume
      template:
        metadata:
          labels:
            app: volume
        spec:
          containers:
          - name: nginx
            image: nginx:1.22
            volumeMounts:
            - mountPath: /usr/share/nginx/html
              name: index-html
          volumes:
          - name: index-html
            hostPath:
              path: /mnt
              type: Directory
   ```

   【回答例】

   ```bash
   # manifestを作成する
   cat <<EOF > ./Deployment.yaml
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: nginx
     labels:
       app: volume
   spec:
     replicas: 1
     selector:
       matchLabels:
         app: volume
     template:
       metadata:
         labels:
           app: volume
       spec:
         containers:
         - name: nginx
           image: nginx:1.22
           volumeMounts:
           - mountPath: /usr/share/nginx/html
             name: index-html
         volumes:
         - name: index-html
           hostPath:
             path: /mnt
             type: Directory
   EOF

   # Deplymentを作成する
   $ kubectl apply -f Deployment.yaml
   deployment.apps/nginx created
   ```

1. デプロイしたPod内のコンテナに対して追加コマンドを発行し、/usr/share/nginx/html/index.htmlを以下内容に修正する。

   ```text
   zettai ni nakunatte ha ikenai data ga kokoni aru
   ```

   【回答例】

   ```bash
   # Pod名の確認
   $ kubectl get pod
   NAME                     READY   STATUS    RESTARTS   AGE
   curl                     1/1     Running   0          30m
   nginx-65ddbc7b46-8wmmx   1/1     Running   0          43s

   # index.html修正
   $ kubectl exec nginx-65ddbc7b46-8wmmx -- bash -c "echo 'zettai ni nakunatte ha ikenai data ga kokoni aru' > /usr/share/nginx/html/index.html"
   ```

1. Service:volume-svcを経由してDeploymentにアクセスする。
  （さきほど修正したindex.htmlが表示`される`ことを確認する）

   【回答例】

   ```bash
   # Deploymentへアクセス
   $ kubectl exec curl -- curl -s volume-svc
   zettai ni nakunatte ha ikenai data ga kokoni aru
   ```

1. Podを削除（Deploymentは消さない！）し、Podが再作成されたことを確認する。

   【回答例】

   ```bash
   # Pod名の確認
   $ kubectl get pod
   NAME                    READY   STATUS    RESTARTS   AGE
   curl                    1/1     Running   0          2m10s
   nginx-7f7c98667-khks5   1/1     Running   0          103s

   # Podの削除
   $ kubectl delete pod nginx-7f7c98667-khks5
   pod "nginx-65ddbc7b46-8wmmx" deleted

   # Pod名の確認
   $ kubectl get pod
   NAME                     READY   STATUS    RESTARTS   AGE
   curl                     1/1     Running   0          33m
   nginx-65ddbc7b46-6ngvq   1/1     Running   0          40s
   ```

1. 再度1. Service:volume-svcを経由してDeploymentにアクセスする。
  （さきほど修正したindex.htmlが表示`される`ことを確認する）

   【回答例】

   ```bash
   # Deploymentへアクセス
   $ kubectl exec curl -- curl -s volume-svc
   zettai ni nakunatte ha ikenai data ga kokoni aru
   ```

1. Deployment:volume、Pod:curl、Service:volume-svcを削除してください。

  【回答例】

   ```bash
   $ kubectl delete -f Deployment.yaml
   deployment.apps "nginx" deleted

   $ kubectl delete -f Service.yaml
   service "volume-svc" deleted

   $ kubectl delete pod curl
   pod "curl" deleted
   ```
