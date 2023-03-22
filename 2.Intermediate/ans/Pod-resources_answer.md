# 回答例

1. 以下を満たすDeploymentをデプロイしてください。なお、resourcesについては[公式ドキュメント][1]を参考にしてください。

   - 要件
     - Deployment
       - 名前は`nginx`
       - replicas: `1`
       - labelはすべて`app: nginx`
       - Pod
         - 名前は`nginx`
         - イメージは`nginx:1.12`
         - 以下のHWリソース容量を指定
           - CPU要求`500m`
           - CPU上限`600m`

   【回答例】

   ```yml
   # manifest
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: nginx
   spec:
     replicas: 3
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
             resources:
               requests:
                 cpu: 500m
               limits:
                 cpu: 600m
   ```

   ```bash
   # 実行結果
   $ kubectl apply -f nginx.yaml
   deployment.apps/nginx created
   ```

1. デプロイしたPodの詳細を表示し、コンテナにrequestsとlimitsが設定されていることを確認してください。

   【回答例】

   ```bash
   # 実行結果
   $ kubectl get pod
   NAME                          READY   STATUS    RESTARTS   AGE
   nginx-777576b7-dthkw   1/1     Running   0          5m8s
   nginx-777576b7-hfks2   1/1     Running   0          5m10s
   nginx-777576b7-qz9gb   1/1     Running   0          5m6s

   $ kubectl describe pod nginx-777576b7-dthkw
   Name:         nginx-777576b7-dthkw
   Namespace:    default
   Priority:     0
      (中略)
       Restart Count:  0
       Limits:
         cpu:  600m ★
       Requests:
         cpu:        500m ★
       Liveness:     http-get http://:80/ delay=120s timeout=1s period=10s #success=1 #failure=3
       Readiness:    http-get http://:80/ delay=0s timeout=1s period=10s #success=1 #failure=3
       Environment:  <none>
     (後略)
   ```

1. マニフェストを以下の内容に修正し、再デプロイしてください。

   - 要件
     - Deployment
       - Pod
         - 以下のHWリソース容量を指定
           - memory上限`100Mi`

   【回答例】

   ```yml
   # manifest
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: nginx
   spec:
     replicas: 3
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
             resources:
               requests:
                 cpu: 500m
               limits:
                 cpu: 600m
                 memory: 100Mi
   ```

   ```bash
   # 実行結果
   $ kubectl apply -f nginx.yaml
   deployment.apps/nginx configured
   ```

1. デプロイしたPodの詳細を表示し、各コンテナにresourcesが設定さていることを確認してください。またmemoryの`requetsとlimitsが同じ値`で設定されていることを確認してください。

   【回答例】

   ```bash
   # 実行結果
   $ kubectl  get pod
   NAME                            READY   STATUS    RESTARTS   AGE
   nginx-75b5d76495-fsmjr   1/1     Running   0          101s
   nginx-75b5d76495-vkxn7   1/1     Running   0          102s
   nginx-75b5d76495-xmsfx   1/1     Running   0          100s

   $ kubectl describe pod nginx-75b5d76495-fsmjr
   Name:         nginx-75b5d76495-fsmjr
   Namespace:    default
   Priority:     0
     (中略)
       Restart Count:  0
       Limits:
         cpu:     600m
         memory:  100Mi ★
       Requests:
         cpu:        500m
         memory:     100Mi ★
       Liveness:     http-get http://:80/ delay=120s timeout=1s period=10s #success=1 #failure=3
       Readiness:    http-get http://:80/ delay=0s timeout=1s period=10s #success=1 #failure=3
       Environment:  <none>
     (後略)
   ```

1. Deploymentのreplica数を10に拡張してください。

   【回答例】

   ```yml
   # manifest
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: nginx
   spec:
     replicas: 10
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
             resources:
               requests:
                 cpu: 500m
               limits:
                 cpu: 600m
                 memory: 100Mi
   ```

   ```bash
   # 実行結果
   $ kubectl apply -f nginx.yaml
   deployment.apps/nginx configured
   ```

1. Podリソースのオブジェクト一覧を表示し、STATUS:PendingのPodがあることを確認してください。（もしなかった場合、Deploymentのreplica数をより大きな値に修正し、再度確認してください。）

   【回答例】

   ```bash
   $ kubectl get pod
   NAME                            READY   STATUS    RESTARTS   AGE
   nginx-75b5d76495-fsmjr   1/1     Running   0          3m40s
   nginx-75b5d76495-gfxd7   1/1     Running   0          11s
   nginx-75b5d76495-jkcd4   1/1     Running   0          11s
   nginx-75b5d76495-jwhk2   1/1     Running   0          11s
   nginx-75b5d76495-mlp7s   1/1     Running   0          12s
   nginx-75b5d76495-nd8ws   1/1     Running   0          11s
   nginx-75b5d76495-pwhsc   0/1     Pending   0          11s ★
   nginx-75b5d76495-vkxn7   1/1     Running   0          3m41s
   nginx-75b5d76495-xmsfx   1/1     Running   0          3m39s
   nginx-75b5d76495-zh6xm   1/1     Running   0          11s
   ```

1. STATUS:PendingのPodの詳細を表示しeventを確認してください。以下の様なメッセージが出力されていることを確認してください。（これはワーカー3台の環境で要求した量のCPUを確保できるノードが見つからなかった場合のメッセージです。）

   ```bash
   0/3 nodes are available: 3 Insufficient cpu.
   ```

   【回答例】

   ```bash
   $ kubectl describe pod nginx-75b5d76495-pwhsc
   Name:           nginx-75b5d76495-pwhsc
   Namespace:      default
   Priority:       0
     (中略)
     Type     Reason            Age                 From               Message
     ----     ------            ----                ----               -------
     Warning  FailedScheduling  59s (x3 over 2m6s)  default-scheduler  0/3 nodes are available: 3 Insufficient cpu.
   ```

1. 作成したリソースを削除してください。

   【回答例】

   ```bash
   # 実行結果
   $ kubectl delete -f nginx.yaml
   deployment.apps/nginx deleted
   ```

[1]:https://kubernetes.io/docs/concepts/configuration/manage-compute-resources-container/
