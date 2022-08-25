# 回答例

## チュートリアル1: Podを作成・操作する

1. 次のmanifestを使用して、pod:testを作成する。

   ``` yml
   apiVersion: v1
   kind: Pod
   metadata:
     name: test
     labels:
       app: test
   spec:
     containers:
       - image: nginx:1.12
         name: nginx
   ```

   【回答例】

   ```bash
   # manifestを作成する
   cat <<EOF > ./Pod.yaml
   apiVersion: v1
   kind: Pod
   metadata:
     name: test
     labels:
       app: test
   spec:
     containers:
       - image: nginx:1.12
         name: nginx
   EOF

   # Podを起動する
   $ kubectl apply -f Pod.yaml
   pod/test created
   ```

1. Podの一覧を参照し、pod:testを確認する。

   【回答例】

   ```bash
   $ kubectl get pod
   NAME   READY   STATUS    RESTARTS   AGE
   test   1/1     Running   0          91s
   ```

1. pod:testの詳細情報を参照する。

   【回答例】

   ```bash
   $ kubectl describe pod test
   Name:         test
   Namespace:    default
   Priority:     0
   Node:         ip-192-168-38-218.us-west-1.compute.internal/192.168.38.218
   Start Time:   Thu, 16 Aug 2022 11:23:46 +0900
   Labels:       app=test
   Annotations:  kubernetes.io/psp: eks.privileged
   Status:       Running
   IP:           192.168.46.104
   IPs:
     IP:  192.168.46.104
   Containers:
     nginx:
       Container ID:   docker://   6dc69e410aeeaedaa1dd1fd0d76413a997329f45cfd9015af1d215a7af4705af
       Image:          nginx:1.12
       Image ID:       docker-pullable://   nginx@sha256:72daaf46f11cc753c4eab981cbf869919bd1fee3d2170a2adeac12400f494728
       Port:           <none>
       Host Port:      <none>
       State:          Running
         Started:      Thu, 25 Aug 2022 01:23:47 +0900
       Ready:          True
       Restart Count:  0
       Environment:    <none>
       Mounts:
         /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-w75ff (ro)
   Conditions:
     Type              Status
     Initialized       True
     Ready             True
     ContainersReady   True
     PodScheduled      True
   Volumes:
     kube-api-access-w75ff:
       Type:                    Projected (a volume that contains injected data from    multiple sources)
       TokenExpirationSeconds:  3607
       ConfigMapName:           kube-root-ca.crt
       ConfigMapOptional:       <nil>
       DownwardAPI:             true
   QoS Class:                   BestEffort
   Node-Selectors:              <none>
   Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for    300s
                                node.kubernetes.io/unreachable:NoExecute op=Exists    for 300s
   Events:
     Type    Reason     Age    From               Message
     ----    ------     ----   ----               -------
     Normal  Scheduled  2m20s  default-scheduler  Successfully assigned default/test    to ip-192-168-38-218.us-west-1.compute.internal
     Normal  Pulled     2m19s  kubelet            Container image "nginx:1.12"    already present on machine
     Normal  Created    2m19s  kubelet            Created container nginx
     Normal  Started    2m19s  kubelet            Started container nginx
   ```

1. pod:testの中に入って、`/usr/share/nginx/html/index.html`を参照する。

   【回答例】

   ```bash
   $ kubectl exec -it test -- /bin/bash
   root@test:/# 
   root@test:/# cat /usr/share/nginx/html/index.html
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
   root@test:/# exit
   exit
   ```

1. pod:testを削除する。

   【回答例】

   ```bash
   $ kubectl delete -f Pod.yaml
   pod "test" deleted
   $
   $ kubectl get pod
   No resources found in default namespace.
   ```

## チュートリアル2: 同じPod内で複数のコンテナを作成する

1. 次のmanifestを使用して、pod:test2を作成する。

   ``` yml
   apiVersion: v1
   kind: Pod
   metadata:
     name: test2
     labels:
       app: test2
   spec:
     containers:
       - image: nginx
         name: nginx
       - image: appropriate/curl
         name: curl
         command: ["sleep", "3600"]
   ```

   【回答例】

   ```bash
   # manifestを作成する
   cat <<EOF > ./Pod-test2.yaml
   apiVersion: v1
   kind: Pod
   metadata:
     name: test2
     labels:
       app: test2
   spec:
     containers:
       - image: nginx
         name: nginx
       - image: appropriate/curl
         name: curl
         command: ["sleep", "3600"]
   EOF

   # Podを起動する
   $ kubectl apply -f Pod-test2.yaml
   pod/test2 created
   ```

1. Podの一覧を参照し、作成したPodを確認する。

   【回答例】
   `READY`のカラムが`2/2`となっており、コンテナが2つ起動していることがわかります。

   ```bash
   $ kubectl get pod
   NAME    READY   STATUS    RESTARTS   AGE
   test2   2/2     Running   0          42s
   ```

1. 作成したPodの詳細情報を参照し、2つのコンテナが起動していることを確認する。

   【回答例】
   `Containers:`の情報から、`nginx`と`curl`の2つのコンテナが起動していることがわかります。

   ```bash
   $ kubectl describe pod test2
   Name:         test2
   Namespace:    default
   Priority:     0
   Node:         ip-192-168-33-4.us-west-1.compute.internal/192.168.33.4
   Start Time:   Thu, 16 Aug 2022 11:35:52 +0900
   Labels:       app=test2
   Annotations:  kubernetes.io/psp: eks.privileged
   Status:       Running
   IP:           192.168.49.65
   IPs:
     IP:  192.168.49.65
   Containers:
     nginx:
       Container ID:   docker://159a7792f1b9be8dec6941e1fbd78404b2bfcdadb2fc1ae16fe082b219ddf120
       Image:          nginx
       Image ID:       docker-pullable://nginx@sha256:b95a99feebf7797479e0c5eb5ec0bdfa5d9f504bc94da550c2f58e839ea6914f
       Port:           <none>
       Host Port:      <none>
       State:          Running
         Started:      Thu, 16 Aug 2022 11:35:56 +0900
       Ready:          True
       Restart Count:  0
       Environment:    <none>
       Mounts:
         /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-xjqml (ro)
     curl:
       Container ID:  docker://19911d15aae88647d2613163251b8e07372ed25e01e81bdb0b49434bcae43ea6
       Image:         appropriate/curl
       Image ID:      docker-pullable://appropriate/curl@sha256:c8bf5bbec6397465a247c2bb3e589bb77e4f62ff88a027175ecb2d9e4f12c9d7
       Port:          <none>
       Host Port:     <none>
       Command:
         sleep
         3600
       State:          Running
         Started:      Thu, 16 Aug 2022 11:35:57 +0900
       Ready:          True
       Restart Count:  0
       Environment:    <none>
       Mounts:
         /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-xjqml (ro)
   Conditions:
     Type              Status
     Initialized       True
     Ready             True
     ContainersReady   True
     PodScheduled      True
   Volumes:
     kube-api-access-xjqml:
       Type:                    Projected (a volume that contains injected data from multiple sources)
       TokenExpirationSeconds:  3607
       ConfigMapName:           kube-root-ca.crt
       ConfigMapOptional:       <nil>
       DownwardAPI:             true
   QoS Class:                   BestEffort
   Node-Selectors:              <none>
   Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for    300s
                                node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
   Events:
     Type    Reason     Age   From               Message
     ----    ------     ----  ----               -------
     Normal  Scheduled  2m5s  default-scheduler  Successfully assigned default/test2 to ip-192-168-33-4.us-west-1.compute.internal
     Normal  Pulling    2m4s  kubelet            Pulling image "nginx"
     Normal  Pulled     2m1s  kubelet            Successfully pulled image "nginx" in 3.261875603s
     Normal  Created    2m1s  kubelet            Created container nginx
     Normal  Started    2m1s  kubelet            Started container nginx
     Normal  Pulling    2m1s  kubelet            Pulling image "appropriate/curl"
     Normal  Pulled     2m    kubelet            Successfully pulled image "appropriate/curl" in 833.747035ms
     Normal  Created    2m    kubelet            Created container curl
     Normal  Started    2m    kubelet            Started container curl
   ```

1. 各Podの中に入る。
   `-c <コンテナ名>`というオプションを追加することで、Pod内のどのコンテナに入るかを明示的にを指定することができる。

   ```bash
   kubectl exec -it test2 -c nginx -- /bin/sh
   kubectl exec -it test2 -c curl -- /bin/sh
   ```

   【回答例】

   ```bash
   # nginxコンテナ

   $ kubectl exec -it test2 -c nginx -- /bin/sh
   #
   # exit

   # curlコンテナ
   $ kubectl exec -it test2 -c curl -- /bin/sh
   / #
   / # exit
   ```

1. 次のコマンドを打鍵して、各コンテナの`ホスト名`と`IPアドレス`が`一致していること`を確認する。

   ```bash
   # ホスト名の確認
   hostname
   # IPアドレスの確認
   hostname -i 
   ```

   【回答例】
   戻り値から`ホスト名`と`IPアドレス`が一致していることがわかります。

   ```bash
   # nginxコンテナ

   $ kubectl exec -it test2 -c nginx -- /bin/sh
   #
   # hostname
   test2
   #
   # hostname -i
   192.168.49.65
   #
   # exit

   # curlコンテナ
   $ kubectl exec -it test2 -c curl -- /bin/sh
   / #
   / # hostname
   test2
   / #
   / #
   / # hostname -i
   192.168.49.65
   / #
   / # exit
   ```

1. コンテナ`curl`で次のコマンドを打鍵する。

   ```bash
   curl localhost
   ```

   【回答例】
   同じPodに所属するコンテナはNICを共有し、同一のIPアドレスを持つことから、自身と同じように隣接コンテナとも疎通できます。

   ```bash
   $ kubectl exec -it test2 -c curl -- /bin/sh
   / #
   / # curl localhost
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
   / #
   / # exit
   exit
   ```

1. pod:test2を削除する。

   【回答例】

   ```bash
   $ kubectl delete -f Pod-test2.yaml
   pod "test2" deleted
   ```
