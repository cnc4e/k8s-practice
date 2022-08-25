# 回答例

## チュートリアル: Serviceを作成する

1. 次のmanifestを使用して、Service:nginx-svcを作成する。

   ```yml
   kind: Service
   apiVersion: v1
   metadata:
     name: nginx-svc
   spec:
     selector:
       app: test
     ports:
     - protocol: TCP
       port: 80
       targetPort: 80
   ```

   【回答例】

   ```bash
   # manifestを作成する
   cat <<EOF > ./Service.yaml
   kind: Service
   apiVersion: v1
   metadata:
     name: nginx-svc
   spec:
     selector:
       app: test
     ports:
     - protocol: TCP
       port: 80
       targetPort: 80
   EOF

   # Serviceを作成する
   $ kubectl apply -f Service.yaml
   service/nginx-svc created
   ```

1. 作成したService:nginx-svcに設定されたIPアドレスを確認する。

   【回答例】

   ```bash
   $ kubectl get service
   NAME         TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)   AGE
   kubernetes   ClusterIP   10.100.0.1       <none>        443/TCP   2d18h
   nginx-svc    ClusterIP   10.100.204.175   <none>        80/TCP    41s
   ```

1. Serviceを経由してDeploymentにアクセスする。
   （宛先は先ほど確認した`Service:nginx-svcのIPアドレス`を指定する）

   【回答例】

   ``` sh
   # curl の起動
   kubectl run curl --image=appropriate/curl -- /bin/sh -c "sleep 3600"

   # Deploymentへアクセス
   $ kubectl exec curl -- curl -s 10.100.204.175
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

1. Serviceを経由してDeploymentにアクセスする。
   （宛先は`nginx-svc`を指定する）

   【回答例】

   ``` sh
   # Deploymentへアクセス
   $ kubectl exec curl -- curl -s nginx-svc
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

1. 2つあるPod:nginx-XXXXXそれぞれに含まれるtestコンテナに対して追加コマンドを発行し、コンテナ内の/usr/share/nginx/html/index.htmlを以下内容に修正する。

    - どちらかのコンテナ

     ```text
     watashi no sentouryoku ha
     ```

   - もう一方のコンテナ

     ```text
     530000 desu
     ```

   【回答例】

   ```bash
   # Pod名の確認
   $ kubectl get pod
   NAME                     READY   STATUS    RESTARTS   AGE
   curl                     1/1     Running   0          5m44s
   nginx-77b58b966f-7z2vq   1/1     Running   0          58s
   nginx-77b58b966f-nvqt6   1/1     Running   0          9m25s

   # どちらかのコンテナ
   $ kubectl exec nginx-77b58b966f-7z2vq -- bash -c "echo 'watashi no sentouryoku ha' > /usr/share/nginx/html/index.html"

   # もう一方のコンテナ
   $ kubectl exec nginx-77b58b966f-nvqt6 -- bash -c "echo '530000 desu' > /usr/share/nginx/html/index.html"
   ```

1. curlコマンドの宛先を`nginx-svc`とし、Serviceを経由してPodに複数回アクセスする。
   (表示される結果が`ランダムで変わる`ことを確認する）

   【回答例】
   次の1つの応答がランダムに表示される。

   ```bash
   $ kubectl exec curl -- curl -s nginx-svc
   watashi no sentouryoku ha
   $
   $ kubectl exec curl -- curl -s nginx-svc
   530000 desu
   ```

1. Service:nginx-svcのlabelSelectorを「app: test」から「`app: test2`」に修正し、修正を適用する。

   【回答例】

   ```bash
   # manifest修正
   $ cat Service.yaml | sed 's/app: test/app: test2/' > Service.yaml
   $
   $ cat Service.yaml
   kind: Service
   apiVersion: v1
   metadata:
     name: nginx-svc
   spec:
     selector:
       app: test2
     ports:
     - protocol: TCP
       port: 80
       targetPort: 80

   # 修正を適用
   $ kubectl apply -f Service.yaml
   service/nginx-svc configured
   ```

1. curlコマンドの宛先を`nginx-svc`とし、Serviceを経由してPodに複数回アクセスする。
   (`アクセスできない`ことを確認する）

   【回答例】
   Service:nginx-svcは通信を`app: test2`というLabelを持つPodに転送しようとしているが、該当Labelを持ったPodが存在しないため、エラーとなる。

   ```bash
   $ kubectl exec curl -- curl nginx-svc
   % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                  Dload  Upload   Total   Spent    Left  Speed
   0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0
   curl: (7) Failed to connect to nginx-svc port 80: Connection refused
   command terminated with exit code 7
   ```

1. Deployment:nginx、Deployment:curlおよびService:nginx-svcを削除する。

   【回答例】

   ```bash
   $ kubectl delete -f Deployment.yaml
   deployment.apps "nginx" deleted

   $ kubectl delete -f Service.yaml
   service "nginx-svc" deleted
   ```
