# 回答例

## コンテナの上限設定

1. 以下を満たすマニフェストを作成してください。（デプロイは次の手順でやる）　LimitRangeリソースについては[公式ドキュメント][1]を参考にしてください。

   - 要件
     - LimitRange
       - 制限対象はコンテナ
       - max
         - cpuは300m
         - memoryは300Mi
       - min
         - cpuは100m
         - memoryは100Mi
     - Deployment
       - メインコンテナ1
         - イメージは何でも良い
         - resourcesで以下のHWリソース量を指定
           - limits
             - cpu: 400m
             - memory: 300Mi
           - requests
             - cpu: 100m
             - memory: 100Mi

   【回答例】

   ```yml
   # manifest
   apiVersion: v1
   kind: LimitRange
   metadata:
     name: limit-range
   spec:
     limits:
       - max:
           cpu: 300m
           memory: 300Mi
         min:
           cpu: 100m
           memory: 100Mi
         type: Container
   ---
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
             resources:
               limits:
                 cpu: 400m
                 memory: 300Mi
               requests:
                 cpu: 100m
                 memory: 100Mi
   ```

   ```bash
   # 実行結果
   $ kubectl apply -f limitrange.yaml
   limitrange/limit-range created
   deployment.apps/nginx created
   ```

1. 上記マニフェストをデプロイし、Podリソースの一覧を表示してください。Podが作成されていないことを確認してください。

   【回答例】

   ```bash
   # 実行結果
   $ kubectl get pod
   No resources found in default namespace.
   ```

1. ReplicaSetリソースの一覧を表示し、上記マニフェストでデプロイしたReplicaSetの詳細情報を表示してください。以下のようにLimitRangeで指定した範囲に違反したためPodの作成に失敗した旨のメッセージを確認する。

   ```bash
   Error creating: pods "resources-6bf4ffd649-b9hb2" is forbidden: maximum cpu usage per Container is 300m, but limit is 400m.
   ```

   【回答例】

   ```bash
   # 実行結果
   $ kubectl get rs
   NAME               DESIRED   CURRENT   READY   AGE
   nginx-55db84869b   1         0         0       2m26s

   $ kubectl describe rs nginx-55db84869b
   Name:           nginx-55db84869b
   Namespace:      default
        (略)
     Containers:
      nginx:
       Image:      nginx:1.12
       Port:       80/TCP
       Host Port:  0/TCP
       Limits:
         cpu:     400m
         memory:  300Mi
       Requests:
         cpu:        100m
         memory:     100Mi
       Environment:  <none>
       Mounts:       <none>
     Volumes:        <none>
   Conditions:
     Type             Status  Reason
     ----             ------  ------
     ReplicaFailure   True    FailedCreate
   Events:
     Type     Reason        Age                 From                   Message
     ----     ------        ----                ----                   -------
     Warning  FailedCreate  2m52s               replicaset-controller  Error creating: pods "nginx-55db84869b-ms46t" is forbidden: maximum cpu usage per Container is 300m, but limit is 400m
     Warning  FailedCreate  8s (x7 over 2m49s)  replicaset-controller  (combined from similar events): Error creating: pods "nginx-55db84869b-fbwp8" is forbidden: maximum cpu usage per Container is 300m, but limit is 400m
   ```

1. Deploymentを`削除し`、以下のようにマニフェストを修正しデプロイしてください。（変更箇所を`ハイライト`にする）

   - 要件
     - Deployment
       - メインコンテナ1
         - イメージは何でも良い
         - resourcesで以下のHWリソース量を指定
           - limits
             - cpu: `300m`
             - memory: 300Mi
           - requests
             - cpu: 100m
             - memory: `10Mi`

   【回答例】

   ```yml
   # manifest
   apiVersion: v1
   kind: LimitRange
   metadata:
     name: limit-range
   spec:
     limits:
       - max:
           cpu: 300m
           memory: 300Mi
         min:
           cpu: 100m
           memory: 100Mi
         type: Container
   ---
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
             resources:
               limits:
                 cpu: 300m
                 memory: 300Mi
               requests:
                 cpu: 100m
                 memory: 10Mi
   ```

   ```bash
   # 実行結果
   $ kubectl apply -f limitrange.yaml
   limitrange/limit-range created
   deployment.apps/nginx created
   ```

1. Podリソースの一覧を表示してください。Podが作成されていないことを確認してください。

   【回答例】

   ```bash
   # 実行結果
   $ kubectl get pod
   No resources found in default namespace.
   ```

1. ReplicaSetリソースの一覧を表示し、上記マニフェストでデプロイしたReplicaSetの詳細情報を表示してください。以下のようにLimitRangeで指定した範囲に違反したためPodの作成に失敗した旨のメッセージを確認する。

   ```bash
   Error creating: pods "resources-787f48c74f-2t5rd" is forbidden: minimum memory usage per Container is 100Mi, but request is 10Mi.
   ```

   【回答例】

   ```bash
   # 実行結果
   $ kubectl get rs
   NAME               DESIRED   CURRENT   READY   AGE
   nginx-684bdc8c7b   1         0         0       8m43s

   $ kubectl describe rs nginx-684bdc8c7b
   Name:           nginx-684bdc8c7b
   Namespace:      default
        (略)
     Containers:
      nginx:
       Image:      nginx:1.12
       Port:       80/TCP
       Host Port:  0/TCP
       Limits:
         cpu:     300m
         memory:  300Mi
       Requests:
         cpu:        100m
         memory:     10Mi
       Environment:  <none>
       Mounts:       <none>
     Volumes:        <none>
   Conditions:
     Type             Status  Reason
     ----             ------  ------
     ReplicaFailure   True    FailedCreate
   Events:
     Type     Reason        Age               From                   Message
     ----     ------        ----              ----                   -------
     Warning  FailedCreate  20s               replicaset-controller  Error creating: pods "nginx-684bdc8c7b-7ts8c" is forbidden: minimum memory usage per Container is 100Mi, but request is 10Mi
     Warning  FailedCreate  1s (x4 over 19s)  replicaset-controller  (combined from similar events): Error creating: pods "nginx-684bdc8c7b-plh75" is forbidden: minimum memory usage per Container is 100Mi, but request is 10Mi
   ```

1. Deploymentを`削除し`、以下のようにマニフェストを修正しデプロイしてください。（変更箇所を`ハイライト`にする）

   - 要件
     - Deployment
       - メインコンテナ1
         - イメージは何でも良い
         - resourcesで以下のHWリソース量を指定
           - limits
             - cpu: 300m
             - memory: 300Mi
           - requests
             - cpu: 100m
             - memory: `100Mi`

   【回答例】

   ```yml
   # manifest
   apiVersion: v1
   kind: LimitRange
   metadata:
     name: limit-range
   spec:
     limits:
       - max:
           cpu: 300m
           memory: 300Mi
         min:
           cpu: 100m
           memory: 100Mi
         type: Container
   ---
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
             resources:
               limits:
                 cpu: 300m
                 memory: 300Mi
               requests:
                 cpu: 100m
                 memory: 100Mi
   ```

   ```bash
   # 実行結果
   $ kubectl apply -f limitrange.yaml
   limitrange/limit-range created
   deployment.apps/nginx created
   ```

1. Podリソースの一覧を表示してください。Podが`作成されている`ことを確認してください。

   【回答例】

   ```bash
   # 実行結果
   $ kubectl get pod
   NAME                   READY   STATUS    RESTARTS   AGE
   nginx-58db6cc8-gq5t2   1/1     Running   0          20s
   ```

1. 作成したリソースを削除してください。

   【回答例】

   ```bash
   # 実行結果
   $ kubectl delete -f limitrange.yaml
   limitrange "limit-range" deleted
   deployment.apps "nginx" deleted
   ```

## Podの上限設定

1. 以下を満たすマニフェストを作成してください。（デプロイは次の手順で実施します）LimitRangeリソースについては[公式ドキュメント][1]を参考にしてください。

   - 要件
     - LimitRange
       - 制限対象はPod
       - max
         - cpuは400m
         - memoryは300Mi
       - min
         - cpuは200m
         - memoryは200Mi
     - Deployment
       - メインコンテナ1
         - イメージは何でも良い
         - resourcesで以下のHWリソース量を指定
           - limits
             - cpu: 200m
             - memory: 200Mi
           - requests
             - cpu: 100m
             - memory: 100Mi
       - メインコンテナ2
         - イメージは何でも良い
         - resourcesで以下のHWリソース量を指定
           - limits
             - cpu: 100m
             - memory: 100Mi
           - requests
             - cpu: 100m
             - memory: `10Mi`

   【回答例】

   ```yml
   # manifest
   apiVersion: v1
   kind: LimitRange
   metadata:
     name: limit-range
   spec:
     limits:
       - max:
           cpu: 400m
           memory: 300Mi
         min:
           cpu: 200m
           memory: 200Mi
         type: Pod
   ---
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
             resources:
               limits:
                 cpu: 200m
                 memory: 200Mi
               requests:
                 cpu: 100m
                 memory: 100Mi
           - name: curl
             image: appropriate/curl
             command: ["/bin/sh", "-c", "sleep 3600"]
             resources:
               limits:
                 cpu: 100m
                 memory: 100Mi
               requests:
                 cpu: 100m
                 memory: 10Mi
   ```

1. 上記マニフェストをデプロイし、Podリソースの一覧を表示してください。Podが作成されていないことを確認してください。

   【回答例】

   ```bash
   # 実行結果
   $ kubectl apply -f limitrange-pod.yaml
   limitrange/limit-range created
   deployment.apps/nginx created

   $ kubectl get pod
   No resources found in default namespace.
   ```

1. ReplicaSetリソースの一覧を表示し、上記マニフェストでデプロイしたReplicaSetの詳細情報を表示してください。以下のようにLimitRangeで指定した範囲に違反したためPodの作成に失敗した旨のメッセージを確認する。

   ```bash
   Error creating: pods "resources-65fb6d7459-kkg7c" is forbidden: minimum memory usage per Pod is 200Mi, but request is 115343360.
   ```

   【回答例】

   ```bash
   # 実行結果
   $ kubectl get rs
   NAME               DESIRED   CURRENT   READY   AGE
   nginx-7df5dddcc9   1         0         0       54s

   $ kubectl describe rs nginx-7df5dddcc9
   Name:           nginx-7df5dddcc9
   Namespace:      default
        (略)
     Containers:
      nginx:
       Image:      nginx:1.12
       Port:       80/TCP
       Host Port:  0/TCP
       Limits:
         cpu:     200m
         memory:  200Mi
       Requests:
         cpu:        100m
         memory:     100Mi
       Environment:  <none>
       Mounts:       <none>
      curl:
       Image:      appropriate/curl
       Port:       <none>
       Host Port:  <none>
       Command:
         /bin/sh
         -c
         sleep 3600
       Limits:
         cpu:     100m
         memory:  100Mi
       Requests:
         cpu:        100m
         memory:     10Mi
       Environment:  <none>
       Mounts:       <none>
     Volumes:        <none>
   Conditions:
     Type             Status  Reason
     ----             ------  ------
     ReplicaFailure   True    FailedCreate
   Events:
     Type     Reason        Age                From                   Message
     ----     ------        ----               ----                   -------
     Warning  FailedCreate  92s                replicaset-controller  Error creating: pods "nginx-7df5dddcc9-t6pct" is forbidden: minimum memory usage per Pod is 200Mi, but request is 115343360
     Warning  FailedCreate  11s (x6 over 91s)  replicaset-controller  (combined from similar events): Error creating: pods "nginx-7df5dddcc9-ng6qx" is forbidden: minimum memory usage per Pod is 200Mi, but request is 115343360
   ```

1. Deploymentを`削除し`、以下のようにマニフェストを修正しデプロイしてください。（変更箇所を`ハイライト`にする）

   - 要件
     - Deployment
       - メインコンテナ1
         - イメージは何でも良い
         - resourcesで以下のHWリソース量を指定
           - limits
             - cpu: 200m
             - memory: 200Mi
           - requests
             - cpu: 100m
             - memory: 100Mi
       - メインコンテナ2
         - イメージは何でも良い
         - resourcesで以下のHWリソース量を指定
           - limits
             - cpu: 100m
             - memory: 100Mi
           - requests
             - cpu: 100m
             - memory: `100Mi`

   【回答例】

   ```yml
   # manifest
   apiVersion: v1
   kind: LimitRange
   metadata:
     name: limit-range
   spec:
     limits:
       - max:
           cpu: 400m
           memory: 300Mi
         min:
           cpu: 200m
           memory: 200Mi
         type: Pod
   ---
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
             resources:
               limits:
                 cpu: 200m
                 memory: 200Mi
               requests:
                 cpu: 100m
                 memory: 100Mi
           - name: curl
             image: appropriate/curl
             command: ["/bin/sh", "-c", "sleep 3600"]
             resources:
               limits:
                 cpu: 100m
                 memory: 100Mi
               requests:
                 cpu: 100m
                 memory: 100Mi
   ```

   ```bash
   # 実行結果
   $ kubectl apply -f limitrange-pod.yaml
   limitrange/limit-range created
   deployment.apps/nginx created
   ```

1. Podリソースの一覧を表示してください。Podが`作成されている`ことを確認してください。

   【回答例】

   ```bash
   # 実行結果
   $ kubectl get pod
   NAME                     READY   STATUS    RESTARTS   AGE
   nginx-559b64f657-8cxrm   2/2     Running   0          26s
   ```

1. 作成したリソースを削除してください。

   【回答例】

   ```bash
   # 実行結果
   $ kubectl delete -f limitrange-pod.yaml
   limitrange "limit-range" deleted
   deployment.apps "nginx" deleted
   ```

## デフォルトリソース量の設定

1. 以下を満たすマニフェストを作成しデプロイしてください。LimitRangeリソースについては[公式ドキュメント][1]を参考にしてください。

   - 要件
     - LimitRange
       - 制限対象はコンテナ
       - デフォルトlimits
         - cpuは300m
         - memoryは300Mi
       - デフォルトrequests
         - cpuは100m
     - Deployment
       - メインコンテナ1
         - イメージは何でも良い
         - resourcesでHWリソース量を`指定しない`

   【回答例】

   ```yml
   # manifest
   apiVersion: v1
   kind: LimitRange
   metadata:
     name: limit-range
   spec:
     limits:
       - default:
           cpu: 300m
           memory: 300Mi
         defaultRequest:
           cpu: 100m
         type: Container
   ---
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
   ```

   ```bash
   # 実行結果
   $ kubectl apply -f limitrange-default.yaml
   limitrange/limit-range created
   deployment.apps/nginx created
   ```

1. Podの詳細を確認し、limitsとrequestsが設定されていることを確認してください。

   【回答例】

   ```bash
   # 実行結果
   $ kubectl get pod
   NAME                    READY   STATUS    RESTARTS   AGE
   nginx-f77774fc5-dr9g8   1/1     Running   0          46s

   $ kubectl describe pod nginx-f77774fc5-dr9g8
   Name:         nginx-f77774fc5-dr9g8
   Namespace:    default
           (略)
       Limits:
         cpu:     300m
         memory:  300Mi
       Requests:
         cpu:        100m
         memory:     300Mi
       Environment:  <none>
       Mounts:
         /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-t52tr (ro)
   ```

1. 作成したリソースを削除してください。

   【回答例】

   ```bash
   # 実行結果
   $ kubectl delete -f limitrange-default.yaml
   limitrange "limit-range" deleted
   deployment.apps "nginx" deleted
   ```

[1]:https://kubernetes.io/docs/concepts/policy/limit-range/
