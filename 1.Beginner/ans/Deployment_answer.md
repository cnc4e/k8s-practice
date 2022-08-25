# 回答例

## チュートリアル1: Deploymentを作ってみる

1. 次のmanifestを使用して、Deployment:nginxを作成する。

   ``` yml
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: nginx
     labels:
       app: nginx
   spec:
     replicas: 1
     selector:
       matchLabels:
         app: test
     template:
       metadata:
         labels:
           app: test
       spec:
         containers:
         - name: nginx
           image: nginx:1.22
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
       app: nginx
   spec:
     replicas: 1
     selector:
       matchLabels:
         app: test
     template:
       metadata:
         labels:
           app: test
       spec:
         containers:
         - name: nginx
           image: nginx:1.22
   EOF

   # Deploymentを起動する
   $ kubectl apply -f Deployment.yaml
   deployment.apps/nginx created
   ```

1. Deployment、ReplicaSet、Podそれぞれのオブジェクト一覧を表示する。

   【回答例】

   ```bash
   # Deployment
   $ kubectl get deployment
   NAME    READY   UP-TO-DATE   AVAILABLE   AGE
   nginx   1/1     1            1           83s

   # ReplicaSet
   $ kubectl get replicaset
   NAME               DESIRED   CURRENT   READY   AGE
   nginx-77b58b966f   1         1         1       2m

   # Pod
   $ kubectl get pod
   NAME                     READY   STATUS    RESTARTS   AGE
   nginx-77b58b966f-shcxp   1/1     Running   0          2m16s
   ```

1. 作成したnginx-XXXXXXという名前の`Pod`を削除する。(Deploymentは消してはダメ！)

   【回答例】

   ```bash
   $ kubectl delete pod nginx-77b58b966f-shcxp
   pod "nginx-77b58b966f-shcxp" deleted
   ```

1. Podのオブジェクト一覧を表示する。(さきほどとは違う名前のPodが作成されていることを確認してください)

   【回答例】
   先ほどとはPodの名称が変わっています。これはReplicaSetの自動復旧によりPodが自動的に再作成されたためです。

   ```bash
   $ kubectl get pod
   NAME                     READY   STATUS    RESTARTS   AGE
   nginx-77b58b966f-dfhlb   1/1     Running   0          35s
   ```

1. manifestを修正し、Deploymentのreplicasを`2`に修正し、修正を適用する。

   【回答例】

   ```bash
   # manifest修正
   $ cat Deployment.yaml | sed 's/replicas: 1/replicas: 2/' > Deployment.yaml
   $
   $ cat Deployment.yaml
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: nginx
     labels:
       app: nginx
   spec:
     replicas: 2
     selector:
       matchLabels:
         app: test
     template:
       metadata:
         labels:
           app: test
       spec:
         containers:
         - name: nginx
           image: nginx:1.22

   # 修正を適用
   $ kubectl apply -f Deployment.yaml
   deployment.apps/nginx configured
   ```

1. Podのオブジェクト一覧を表示する。(Podの数が増えていることを確認してください)

   【回答例】

   ```bash
   $ kubectl get pod
   NAME                     READY   STATUS    RESTARTS   AGE
   nginx-77b58b966f-dfhlb   1/1     Running   0          8m54s
   nginx-77b58b966f-z6vmq   1/1     Running   0          39s
   ```

## チュートリアル2: ローリングアップデートする

1. 次のmanifestを使用して、Deployment:nginx-rollupを作成する。

   ``` yml
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: nginx-rollup
     labels:
       app: nginx-rollup
   spec:
     replicas: 2
     selector:
       matchLabels:
         app: test
     template:
       metadata:
         labels:
           app: test
       spec:
         containers:
         - name: nginx
           image: nginx:1.22
           lifecycle:
             preStop:
               exec:
                 command: ['sh', '-c', 'sleep 30']
         initContainers:
         - name: busybox
           image: busybox:1.28
           command: ['sh', '-c', 'sleep 30']
    ```

   【回答例】

   ```bash
   # manifestを作成する
   cat <<EOF > ./Deployment-rollup.yaml
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: nginx-rollup
     labels:
       app: nginx-rollup
   spec:
     replicas: 2
     selector:
       matchLabels:
         app: test
     template:
       metadata:
         labels:
           app: test
       spec:
         containers:
         - name: nginx
           image: nginx:1.22
           lifecycle:
             preStop:
               exec:
                 command: ['sh', '-c', 'sleep 30']
         initContainers:
         - name: busybox
           image: busybox:1.28
           command: ['sh', '-c', 'sleep 30']
   EOF

   # Deploymentを起動する
   $ kubectl apply -f Deployment-rollup.yaml
   deployment.apps/nginx-rollup created
   ```

1. Deployment、ReplicaSet、Podそれぞれのオブジェクト一覧を表示する。

   【回答例】

   ```bash
   # Deployment
   $ kubectl get deployment
   NAME           READY   UP-TO-DATE   AVAILABLE   AGE
   nginx-rollup   2/2     2            2           2m21s

   # ReplicaSet
   $ kubectl get replicaset
   NAME                      DESIRED   CURRENT   READY   AGE
   nginx-rollup-85c9c58db7   2         2         2       3m7s

   # Pod
   $ kubectl get pod
   NAME                            READY   STATUS    RESTARTS   AGE
   nginx-rollup-85c9c58db7-bz6ff   1/1     Running   0          2m40s
   nginx-rollup-85c9c58db7-wwsrv   1/1     Running   0          2m40s
   ```

1. manifestの`nginx:1.22`を`nginx:1.23`へと修正し、修正を適用する。

   【回答例】

   ```bash
   # manifest修正
   $ cat Deployment-rollup.yaml | sed 's/nginx:1.22/nginx:1.23/' > Deployment-rollup.yaml
   $
   $ cat Deployment-rollup.yaml
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: nginx-rollup
     labels:
       app: nginx-rollup
   spec:
     replicas: 2
     selector:
       matchLabels:
         app: test
     template:
       metadata:
         labels:
           app: test
       spec:
         containers:
         - name: nginx
           image: nginx:1.23
           lifecycle:
             preStop:
               exec:
                 command: ['sh', '-c', 'sleep 30']
         initContainers:
         - name: busybox
           image: busybox:1.28
           command: ['sh', '-c', 'sleep 30']

   # 修正を反映
   $ kubectl apply -f Deployment-rollup.yaml
   deployment.apps/nginx-rollup configured
   ```

1. Deployment、ReplicaSet、Podそれぞれのオブジェクト一覧を繰り返し表示する。（順番にPodが再作成されることを確認してください。）

       【回答例】

   ```bash
   # 1秒間隔でDeployment、ReplicaSet、Podの各オブジェクト一覧を表示させる
   while true; do echo '# Deployment'; kubectl get deployment; echo '# ReplicaSet'; kubectl get replicaset; echo '# Pod'; kubectl get pod; sleep 1;done

   # Deployment
   NAME           READY   UP-TO-DATE   AVAILABLE   AGE
   nginx-rollup   2/2     1            2           14m
   # ReplicaSet
   NAME                      DESIRED   CURRENT   READY   AGE
   nginx-rollup-785d697996   1         1         0       7m38s
   nginx-rollup-85c9c58db7   2         2         2       14m
   # Pod
   NAME                            READY   STATUS     RESTARTS   AGE
   nginx-rollup-785d697996-wlvd6   0/1     Init:0/1   0          7s
   nginx-rollup-85c9c58db7-v4457   1/1     Running    0          5m41s
   nginx-rollup-85c9c58db7-w45zk   1/1     Running    0          5m9s

     ※ 新バージョンのnginxコンテナ(wlvd6)が起動を開始する

   # Deployment
   NAME           READY   UP-TO-DATE   AVAILABLE   AGE
   nginx-rollup   2/2     1            2           14m
   # ReplicaSet
   NAME                      DESIRED   CURRENT   READY   AGE
   nginx-rollup-785d697996   1         1         0       8m5s
   nginx-rollup-85c9c58db7   2         2         2       14m
   # Pod
   NAME                            READY   STATUS        RESTARTS   AGE
   nginx-rollup-785d697996-p29qw   0/1     Init:0/1      0          1s
   nginx-rollup-785d697996-wlvd6   1/1     Running       0          34s
   nginx-rollup-85c9c58db7-v4457   1/1     Terminating   0          6m8s
   nginx-rollup-85c9c58db7-w45zk   1/1     Running       0          5m36s

     ※ 新バージョンのnginxコンテナ(wlvd6)が起動が完了する
        新バージョンのnginxコンテナ2つ目(p29qw)が起動が開始する
        旧バージョンのnginxコンテナ(v4457)が終了する

   # Deployment
   NAME           READY   UP-TO-DATE   AVAILABLE   AGE
   nginx-rollup   2/2     2            2           15m
   # ReplicaSet
   NAME                      DESIRED   CURRENT   READY   AGE
   nginx-rollup-785d697996   2         2         2       8m39s
   nginx-rollup-85c9c58db7   0         0         0       15m
   # Pod
   NAME                            READY   STATUS        RESTARTS   AGE
   nginx-rollup-785d697996-p29qw   1/1     Running       0          35s
   nginx-rollup-785d697996-wlvd6   1/1     Running       0          68s
   nginx-rollup-85c9c58db7-w45zk   1/1     Terminating   0          6m10s

     ※ 新バージョンのnginxコンテナ2つ目(p29qw)が起動が完了する
        旧バージョンのnginxコンテナ(w45zk)が終了する

   # Deployment
   NAME           READY   UP-TO-DATE   AVAILABLE   AGE
   nginx-rollup   2/2     2            2           15m
   # ReplicaSet
   NAME                      DESIRED   CURRENT   READY   AGE
   nginx-rollup-785d697996   2         2         2       9m29s
   nginx-rollup-85c9c58db7   0         0         0       15m
   # Pod
   NAME                            READY   STATUS    RESTARTS   AGE
   nginx-rollup-785d697996-p29qw   1/1     Running   0          85s
   nginx-rollup-785d697996-wlvd6   1/1     Running   0          118s

     ※ ローリングアップデートが完了する。
   ```

1. Deployment:nginx-rollupを削除する。

   【回答例】

   ```bash
   $ kubectl delete -f Deployment-rollup.yaml
   deployment.apps "nginx-rollup" deleted
   ```
