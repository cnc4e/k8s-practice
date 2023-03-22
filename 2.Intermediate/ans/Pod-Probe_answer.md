# 回答例

1. 以下を満たすDeployment, Serviceをデプロイしてください。なお、probeについては[公式ドキュメント][1]を参考にしてください。

   - 要件
     - Deployment
       - 名前は`nginx`
       - replicas: `1`
       - labelはすべて`app: nginx`
       - Pod
         - 名前は`nginx`
         - イメージは`nginx:1.12`
         - readinessProbe
           - `httpGet`のヘルスチェックを行う
           - pathは`/`
           - portは`80`
           - ヘルスチェックの詳細設定はとくに行わない
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
             readinessProbe:
               httpGet:
                 path: /
                 port: 80
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
   $ kubectl apply -f nginx-prove.yaml
   deployment.apps/nginx created
   service/nginx-svc created
   ```

1. Podリソースのオブジェクト一覧を表示し、上記マニフェストでデプロイしたPodの`READYが1/1`になっていることを確認してください。

   【回答例】

   ```bash
   $ kubectl get pod
   NAME                     READY   STATUS    RESTARTS   AGE
   nginx-7756577f9f-fj7xz   1/1     Running   0          4m34s
   ```

1. Podの詳細を表示し、readinessProbeが設定されていることを確認してください。

   【回答例】

   ```bash
   $ kubectl describe pod nginx-7756577f9f-fj7xz
   Name:         nginx-7756577f9f-fj7xz
   Namespace:    default
   Priority:     0
   Node:         ip-192-168-13-204.us-west-1.compute.internal/192.168.13.204
   Start Time:   Thu, 22 Sep 2022 16:19:05 +0900
   Labels:       app=nginx
                 pod-template-hash=7756577f9f
   Annotations:  kubernetes.io/psp: eks.privileged
   Status:       Running
   IP:           192.168.31.14
   IPs:
     IP:           192.168.31.14
   Controlled By:  ReplicaSet/nginx-7756577f9f
   Containers:
     nginx:
       Container ID:   docker://13df49334a3ffa262ff2759e26186dd963f0a62590b3460d7819c1b137db0da6
       Image:          nginx:1.12
       Image ID:       docker-pullable://nginx@sha256:9992b46482fff13a4c1e13b88e64bd14d46201480e624519982294a0579885d2
       Port:           80/TCP
       Host Port:      0/TCP
       State:          Running
         Started:      Thu, 22 Sep 2022 16:19:05 +0900
       Ready:          True
       Restart Count:  0
       Readiness:      http-get http://:80/ delay=0s timeout=1s period=10s #success=1 #failure=3  ★
       Environment:    <none>
       (後略)
   ```

1. 接続確認に使用するcurlコマンドが実行可能なtestpodを作成し、testpodから`Service:nginx-svc`に対して接続確認をしてください。（`応答があること`）

   【回答例】

   ```bash
   # testpod作成
   $ kubectl run --image=appropriate/curl --restart=Never testpod sleep 3600
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

1. Service:nginx-svcの詳細を表示し、EndpointsにPodのIPアドレスが`設定されている`ことを確認してください。

   【回答例】

   ```bash
   $ kubectl describe svc nginx-svc
   Name:              nginx-svc
   Namespace:         default
   Labels:            <none>
   Annotations:       <none>
   Selector:          app=nginx
   Type:              ClusterIP
   IP Family Policy:  SingleStack
   IP Families:       IPv4
   IP:                10.100.81.83
   IPs:               10.100.81.83
   Port:              http-port  80/TCP
   TargetPort:        80/TCP
   Endpoints:         192.168.31.14:80 ★
   Session Affinity:  None
   Events:            <none>
   ```

1. Pod:nginxに以下の追加コマンドを発行してください。

   ```bash
   kill -SIGSTOP 7
   ```

   【回答例】

   ```bash
   $ kubectl exec nginx-7756577f9f-fj7xz -it -- /bin/bash
   root@nginx-7756577f9f-fj7xz:/#
   root@nginx-7756577f9f-fj7xz:/# kill -SIGSTOP 7
   root@nginx-7756577f9f-fj7xz:/#
   root@nginx-7756577f9f-fj7xz:/# exit
   exit
   ```

1. しばらく（30秒ほど）してからPodリソースのオブジェクト一覧を確認し、Podの`READYが0/1`になっていることを確認してください。

   【回答例】

   ```bash
   $ kubectl get pod
   NAME                            READY   STATUS    RESTARTS   AGE
   nginx-7756577f9f-fj7xz   0/1     Running   0          36m
   testpod                         1/1     Running   0          19m
   ```

1. testpodから`Service:nginx-svc`に対して接続確認をしてください。（`応答がないこと`）

   【回答例】

   ```bash
   $ kubectl exec testpod -- curl -s http://nginx-svc
   command terminated with exit code 7
   ```

1. Service:nginx-svcの詳細を表示し、EndpointsにPodのIPアドレスが`設定されていない`ことを確認してください。

   【回答例】

   ```bash
   $ kubectl describe svc nginx-svc
   Name:              nginx-svc
   Namespace:         default
   Labels:            <none>
   Annotations:       <none>
   Selector:          app=nginx
   Type:              ClusterIP
   IP Family Policy:  SingleStack
   IP Families:       IPv4
   IP:                10.100.81.83
   IPs:               10.100.81.83
   Port:              http-port  80/TCP
   TargetPort:        80/TCP
   Endpoints:                ★
   Session Affinity:  None
   Events:            <none>
   ```

1. Pod:nginxに以下の追加コマンドを発行してください。

   ``sh
   kill -CONT 7
``

   【回答例】

   ```bash
   $ kubectl exec nginx-7756577f9f-fj7xz -it -- /bin/bash
   root@nginx-7756577f9f-fj7xz:/#
   root@nginx-7756577f9f-fj7xz:/# kill -CONT 7
   root@nginx-7756577f9f-fj7xz:/#
   root@nginx-7756577f9f-fj7xz:/# exit
   exit
   ```

1. しばらく（10秒ほど）してからPodリソースのオブジェクト一覧を確認し、Podの`READYが1/1`になっていることを確認してください。。

   【回答例】

   ```bash
   $ kubectl get pod
   NAME                            READY   STATUS    RESTARTS   AGE
   nginx-7756577f9f-fj7xz   1/1     Running   0          41m
   testpod                         1/1     Running   0          24m
   ```

1. testpodから`Service:nginx-svc`に対して接続確認してください。（`応答があること`）

   【回答例】

   ```bash
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

1. Service:nginx-svcの詳細を表示し、EndpointsにPodのIPアドレスが`設定されている`ことを確認してください。

   【回答例】

   ```bash
   $ kubectl describe svc nginx-svc
   Name:              nginx-svc
   Namespace:         default
   Labels:            <none>
   Annotations:       <none>
   Selector:          app=nginx
   Type:              ClusterIP
   IP Family Policy:  SingleStack
   IP Families:       IPv4
   IP:                10.100.81.83
   IPs:               10.100.81.83
   Port:              http-port  80/TCP
   TargetPort:        80/TCP
   Endpoints:         192.168.31.14:80 ★
   Session Affinity:  None
   Events:            <none>
   ```

1. マニフェストを以下の内容に修正し、デプロイしてください。
    - readinessProbeのブロックを丸コピペし、readinessProbeを`livenessProbe`に書き換える
    - readinessProbeのブロックを`コメントアウト`する

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
    #         readinessProbe:
    #           httpGet:
    #             path: /
    #             port: 80
             livenessProbe:
               httpGet:
                 path: /
                 port: 80
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
   $ kubectl apply -f nginx-prove.yaml
   deployment.apps/nginx configured
   service/nginx-svc unchanged
   ```

1. Podリソースのオブジェクト一覧を確認し、Podの`RESTARTS`の値を確認してください。（たぶん0）

   【回答例】

   ```bash
   $ kubectl get pod
   NAME                           READY   STATUS    RESTARTS   AGE
   nginx-5b9f4b647-ztwkb   1/1     Running   0          81s
   testpod                        1/1     Running   0          54m
   ```

1. Podの詳細を表示し、livenessProbeが設定されていることを確認してください。

   【回答例】

   ```bash
   $ kubectl describe pod nginx-5b9f4b647-ztwkb
   Name:         nginx-5b9f4b647-ztwkb
   Namespace:    default
   Priority:     0
   Node:         ip-192-168-13-204.us-west-1.compute.internal/192.168.13.204
   Start Time:   Thu, 29 Sep 2022 07:28:40 +0900
   Labels:       app=nginx
                 pod-template-hash=5b9f4b647
   Annotations:  kubernetes.io/psp: eks.privileged
   Status:       Running
   IP:           192.168.25.50
   IPs:
     IP:           192.168.25.50
   Controlled By:  ReplicaSet/nginx-5b9f4b647
   Containers:
     nginx:
       Container ID:   docker://7ef2c604673f441e680a8129355814dd2ecae3beb48123b085f4e5a0953917b5
       Image:          nginx:1.12
       Image ID:       docker-pullable://nginx@sha256:9992b46482fff13a4c1e13b88e64bd14d46201480e624519982294a0579885d2
       Port:           80/TCP
       Host Port:      0/TCP
       State:          Running
         Started:      Thu, 29 Sep 2022 07:28:41 +0900
       Ready:          True
       Restart Count:  0
       Liveness:       http-get http://:80/ delay=0s timeout=1s period=10s #success=1 #failure=3 ★
       Environment:    <none>
       (略)
   ```

1. testpodから`Service:nginx-svc`に対して接続確認してください。（`応答があること`）

   【回答例】

   ```bash
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

1. Pod:nginxに以下の追加コマンドを発行してください。

   ```bash
   kill -SIGSTOP 6
   ```

   【回答例】

   ```bash
   $ kubectl exec nginx-5b9f4b647-ztwkb -it -- /bin/bash
   root@nginx-5b9f4b647-ztwkb:/#
   root@nginx-5b9f4b647-ztwkb:/# kill -SIGSTOP 6
   root@nginx-5b9f4b647-ztwkb:/#
   root@nginx-5b9f4b647-ztwkb:/# exit
   exit
   ```

1. しばらく（30秒ほど）してからPodリソースのオブジェクト一覧を確認し、Podの`RESTARTSが増えている`ことを確認してください。

   【回答例】

   ```bash
   $ kubectl get pod
   NAME                           READY   STATUS      RESTARTS     AGE
   nginx-5b9f4b647-ztwkb   1/1     Running     1 (9s ago)   11m
   testpod                        1/1     Running     0            54m
   ```

1. testpodから`Service:nginx-svc`に対して接続確認してください。（`応答があること`）

   【回答例】

   ```bash
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

1. マニフェストを以下の内容に修正し、デプロイしてください。
    - nginxのcommandを`['/bin/sh','-c','sleep 60;nginx -g "daemon off;"']`にする（nginxのプロセスがPod起動してから60秒後に起動する）
    - コメントアウトしていたreadinessProbeのブロックの`コメントアウトを解除`する
    - livenessProbeのヘルスチェックの詳細設定に`initialDelaySeconds: 120`を設定

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
             command: ["/bin/sh", "-c", 'sleep 60;nginx -g "daemon off;"']
             readinessProbe:
               httpGet:
                 path: /
                 port: 80
             livenessProbe:
               httpGet:
                 path: /
                 port: 80
               initialDelaySeconds: 120
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
   $ kubectl apply -f nginx-prove.yaml
   deployment.apps/nginx configured
   service/nginx-svc unchanged
   ```

1. （デプロイしてから60秒以内に実施）Podリソースのオブジェクト一覧を確認し、上記マニフェストでデプロイしたPodの`READYが0/1`になっていることを確認してください。

   【回答例】

   ```bash
   $ kubectl get pod
   NAME                            READY   STATUS    RESTARTS        AGE
   nginx-65676d644b-xqgvd   0/1     Running   0               31s
   testpod                         1/1     Running   0               4m22s
   ```

1. （デプロイしてから60秒以内に実施）testpodから`Service:nginx-svc`に対して接続確認をしてください。（`応答がないこと`）

   【回答例】

   ```bash
   $ kubectl exec testpod -- curl -s http://nginx-svc
   command terminated with exit code 7
   ```

1. （デプロイしてから60秒以降に実施）Podリソースのオブジェクト一覧を確認し、上記マニフェストでデプロイしたPodの`READYが1/1`になっていることを確認してください。

   【回答例】

   ```bash
   $ kubectl get pod
   NAME                            READY   STATUS    RESTARTS   AGE
   nginx-65676d644b-5r2j4   1/1     Running   0          5m38s
   testpod                         1/1     Running   0          12m
   ```

1. （デプロイしてから60秒以降に実施）testpodから`Service:nginx-svc`に対して接続確認をしてください。（`応答があること`）

   【回答例】

   ```bash
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

1. Pod:nginxに以下の追加コマンドを発行してください。

   ```bash
   kill -SIGSTOP 9
   ```

   【回答例】

   ```bash
   $ kubectl exec nginx-65676d644b-5r2j4 -it -- /bin/bash
   root@nginx-65676d644b-5r2j4:/#
   root@nginx-65676d644b-5r2j4:/# kill -SIGSTOP 9
   root@nginx-65676d644b-5r2j4:/#
   root@nginx-65676d644b-5r2j4:/# exit
   exit
   ```

1. しばらく（30秒ほど）してからPodリソースのオブジェクト一覧を確認し、Podの`RESTARTSが増えている`ことを確認してください。

   【回答例】

   ```bash
   $ kubectl get pod
   NAME                            READY   STATUS    RESTARTS     AGE
   nginx-65676d644b-5r2j4   0/1     Running   1 (5s ago)   9m17s
   testpod                         1/1     Running   0            15m
   ```

1. しばらく（60秒ほど）してからPodリソースのオブジェクト一覧を確認し、Podのステータスが`READY 1/1`となっていることを確認してください。

   【回答例】

   ```bash
   $ kubectl get pod
   NAME                            READY   STATUS    RESTARTS        AGE
   nginx-65676d644b-5r2j4   1/1     Running   1 (2m10s ago)   11m
   testpod                         1/1     Running   0               17m
   ```

1. testpodから`Service:nginx-svc`に対して接続確認をしてください。（`応答があること`）

   【回答例】

   ```bash
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
   $ kubectl delete -f nginx-prove.yaml
   deployment.apps/nginx deleted
   service/nginx-svc deleted
   ```

[1]:https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/
