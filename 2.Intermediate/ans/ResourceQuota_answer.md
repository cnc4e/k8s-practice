# 回答例

## HWリソース制限

1. 以下を満たすマニフェストを作成しデプロイしてください。ResourceQuotaリソースについては[公式ドキュメント][1]を参考にしてください。

   - 要件
     - ResourceQuota
       - cpuの制限を1core
       - memoryの制限を1Gi
     - Deployment
       - イメージは何でもよい
       - replicas: 1
       - resources.limitsに以下を設定
         - cpu: 100m
         - memory: 100Mi
       - reqources.requestsに以下を設定
         - cpu: 100m
         - memory: 50Mi

   【回答例】

   ```yml
   # manifest
   apiVersion: v1
   kind: ResourceQuota
   metadata:
     name: resource-quota
   spec:
     hard:
      limits.cpu: "1.0"
      limits.memory: 1Gi
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
                 cpu: 100m
                 memory: 100Mi
               requests:
                 cpu: 100m
                 memory: 50Mi
   ```

   ```bash
   # 実行結果
   $ kubectl apply -f resource-quota.yaml
   resourcequota/resource-quota created
   deployment.apps/nginx created
   ```

1. デプロイしたResourceQuotaオブジェクトの詳細を表示してください。現在の利用量が表示されます。（利用量はrequestsで計算されます）

   【回答例】

   ```bash
   # 実行結果
   $ kubectl get quota
   NAME             AGE   REQUEST   LIMIT
   resource-quota   79s             limits.cpu: 100m/1, limits.memory: 100Mi/1Gi

   $ kubectl describe quota resource-quota
   Name:          resource-quota
   Namespace:     default
   Resource       Used       Hard
   --------       ----       ----
   limits.cpu     100m       1
   limits.memory  104857600  1Gi
   ```

1. Deploymentのreplica数を`10`に拡張してください。

   【回答例】

   ```yml
   # manifest
   apiVersion: v1
   kind: ResourceQuota
   metadata:
     name: resource-quota
   spec:
     hard:
      limits.cpu: "1.0"
      limits.memory: 1Gi
   ---
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
             ports:
               - containerPort: 80
             resources:
               limits:
                 cpu: 100m
                 memory: 100Mi
               requests:
                 cpu: 100m
                 memory: 50Mi
   ```

   ```bash
   # 実行結果
   $ kubectl apply -f resource-quota.yaml
   resourcequota/resource-quota configured
   deployment.apps/nginx configured
   ```

1. デプロイしたResourceQuotaオブジェクトの詳細を表示してください。現在の利用量が表示されます。（利用量はrequestsで計算されます）

   【回答例】

   ```bash
   # 実行結果
   $ kubectl get quota
   NAME             AGE     REQUEST   LIMIT
   resource-quota   3m44s             limits.cpu: 1/1, limits.memory: 500Mi/1Gi

   $ kubectl describe quota resource-quota
   Name:          resource-quota
   Namespace:     default
   Resource       Used     Hard
   --------       ----     ----
   limits.cpu     1        1
   limits.memory  1000Mi   1Gi
   ```

1. Deploymentのreplica数を`11`に拡張してください。

   【回答例】

   ```yml
   # manifest
   apiVersion: v1
   kind: ResourceQuota
   metadata:
     name: resource-quota
   spec:
     hard:
      limits.cpu: "1.0"
      limits.memory: 1Gi
   ---
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: nginx
   spec:
     replicas: 11
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
                 cpu: 100m
                 memory: 100Mi
               requests:
                 cpu: 100m
                 memory: 50Mi
   ```

   ```bash
   # 実行結果
   $ kubectl apply -f resource-quota.yaml
   resourcequota/resource-quota configured
   deployment.apps/nginx configured
   ```

1. Podのオブジェクト一覧を表示してください。Podの数が10のままであること。

   【回答例】

   ```bash
   # 実行結果
   $ kubectl get pod
   NAME                     READY   STATUS    RESTARTS   AGE
   nginx-6b677f56cf-27mjh   1/1     Running   0          35s
   nginx-6b677f56cf-2c2pc   1/1     Running   0          35s
   nginx-6b677f56cf-9jjsq   1/1     Running   0          35s
   nginx-6b677f56cf-b99dh   1/1     Running   0          35s
   nginx-6b677f56cf-b9bsj   1/1     Running   0          35s
   nginx-6b677f56cf-bk8r5   1/1     Running   0          35s
   nginx-6b677f56cf-hzzqs   1/1     Running   0          35s
   nginx-6b677f56cf-mp9hz   1/1     Running   0          35s
   nginx-6b677f56cf-t9rhp   1/1     Running   0          35s
   nginx-6b677f56cf-vp4qx   1/1     Running   0          35s
   ```

1. ReplicaSetの詳細を表示し、以下のようにCPUの利用量がResourceQuotaで指定した範囲に違反したメッセージが表示されること。

   ```bash
   Error creating: pods "quota-5c4f499fb8-nxfqb" is forbidden: exceeded quota: quota, requested: cpu=100m, used: cpu=1, limited: cpu=1
   ```

   【回答例】

   ```bash
   # 実行結果
   $ kubectl get rs
   NAME               DESIRED   CURRENT   READY   AGE
   nginx-6b677f56cf   11        10        10      2m35s

   $ kubectl describe rs nginx-6b677f56cf
   Name:           nginx-6b677f56cf
   Namespace:      default
        (略)
     Containers:
      nginx:
       Image:      nginx:1.12
       Port:       80/TCP
       Host Port:  0/TCP
       Limits:
         cpu:     100m
         memory:  100Mi
       Requests:
         cpu:        100m
         memory:     50Mi
       Environment:  <none>
       Mounts:       <none>
     Volumes:        <none>
   Conditions:
     Type             Status  Reason
     ----             ------  ------
     ReplicaFailure   True    FailedCreate
   Events:
     Type     Reason            Age                    From                   Message
     ----     ------            ----                   ----                   -------
     Normal   SuccessfulCreate  2m50s                  replicaset-controller  Created pod: nginx-6b677f56cf-bk8r5
     Normal   SuccessfulCreate  2m50s                  replicaset-controller  Created pod: nginx-6b677f56cf-mp9hz
     Warning  FailedCreate      2m49s                  replicaset-controller  Error creating: pods "nginx-6b677f56cf-l9h2c" is forbidden: exceeded quota: resource-quota, requested: limits.cpu=100m,limits.memory=100Mi, used: limits.cpu=1,limits.memory=1000Mi, limited: limits.cpu=1,limits.memory=1Gi
     Warning  FailedCreate      2m46s (x6 over 2m48s)  replicaset-controller  (combined from similar events): Error creating: pods "nginx-6b677f56cf-5p6mz" is forbidden: exceeded quota: resource-quota, requested: limits.cpu=100m,limits.memory=100Mi, used: limits.cpu=1,limits.memory=1000Mi, limited: limits.cpu=1,limits.memory=1Gi
   ```

1. 作成したリソースを削除してください。

   【回答例】

   ```bash
   # 実行結果
   $ kubectl delete -f resource-quota.yaml
   resourcequota "resource-quota" deleted
   deployment.apps "nginx" deleted
   ```

## オブジェクト数制限

1. 以下を満たすマニフェストを作成しデプロイしてください。ResourceQuotaリソースについては[公式ドキュメント][1]を参考にしてください。

   - 要件
     - ResourceQuota
       - Podの制限を5
     - Deployment
       - イメージは何でもよい
       - replicas: 1

   【回答例】

   ```yml
   # manifest
   apiVersion: v1
   kind: ResourceQuota
   metadata:
     name: resource-quota-pod
   spec:
     hard:
       pods: "5"
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
   $ kubectl apply -f resource-quota-pod.yaml
   resourcequota/resource-quota-pod created
   deployment.apps/nginx created
   ```

1. デプロイしたResourceQuotaオブジェクトの詳細を表示してください。現在の利用量が表示されます。

   【回答例】

```bash
# 実行結果
$ kubectl get quota
NAME                 AGE   REQUEST     LIMIT
resource-quota-pod   25s   pods: 1/5

$ kubectl describe quota resource-quota-pod
Name:       resource-quota-pod
Namespace:  default
Resource    Used  Hard
--------    ----  ----
pods        1     5
```

1. Deploymentのreplica数を5に拡張してください。

   【回答例】

   ```yml
   # manifest
   apiVersion: v1
   kind: ResourceQuota
   metadata:
     name: resource-quota-pod
   spec:
     hard:
       pods: "5"
   ---
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: nginx
   spec:
     replicas: 5
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
   $ kubectl apply -f resource-quota-pod.yaml
   resourcequota/resource-quota-pod unchanged
   deployment.apps/nginx configured
   ```

1. デプロイしたResourceQuotaオブジェクトの詳細を表示してください。現在の利用量が表示されます。

   【回答例】

   ```bash
   # 実行結果
   $ kubectl get quota
   NAME                 AGE   REQUEST     LIMIT
   resource-quota-pod   2m24s   pods: 5/5

   $ kubectl describe quota resource-quota-pod
   Name:       resource-quota-pod
   Namespace:  default
   Resource    Used  Hard
   --------    ----  ----
   pods        5     5
   ```

1. Deploymentのreplica数を6に拡張してください。

   【回答例】

   ```yml
   # manifest
   apiVersion: v1
   kind: ResourceQuota
   metadata:
     name: resource-quota-pod
   spec:
     hard:
       pods: "5"
   ---
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: nginx
   spec:
     replicas: 6
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
   $ kubectl apply -f resource-quota-pod.yaml
   resourcequota/resource-quota-pod unchanged
   deployment.apps/nginx configured
   ```

1. Podのオブジェクト一覧を表示してください。Podの数が5のままであること。

   【回答例】

   ```bash
   # 実行結果
   $ kubectl get pod
   NAME                    READY   STATUS    RESTARTS   AGE
   nginx-f77774fc5-9dtnp   1/1     Running   0          2m32s
   nginx-f77774fc5-9tg9h   1/1     Running   0          2m32s
   nginx-f77774fc5-dfbnc   1/1     Running   0          3m52s
   nginx-f77774fc5-f42kx   1/1     Running   0          2m32s
   nginx-f77774fc5-m6tk5   1/1     Running   0          2m32s
   ```

1. ReplicaSetの詳細を表示し、以下のようにCPUの利用量がResourceQuotaで指定した範囲に違反したメッセージが表示されること。

   ```bash
   replicaset-controller  Error creating: pods "quota-68847df66-bplgc" is forbidden: exceeded quota: quota, requested: count/pods=1, used: count/pods=5, limited: count/pods=5
   ```

   【回答例】

   ```bash
   # 実行結果
   $ kubectl get rs
   NAME              DESIRED   CURRENT   READY   AGE
   nginx-f77774fc5   6         5         5       4m31s

   $ kubectl describe rs nginx-f77774fc5
   Name:           nginx-f77774fc5
   Namespace:      default
        (略)
     Containers:
      nginx:
       Image:        nginx:1.12
       Port:         80/TCP
       Host Port:    0/TCP
       Environment:  <none>
       Mounts:       <none>
     Volumes:        <none>
   Conditions:
     Type             Status  Reason
     ----             ------  ------
     ReplicaFailure   True    FailedCreate
   Events:
     Type     Reason            Age                From                   Message
     ----     ------            ----               ----                   -------
     Normal   SuccessfulCreate  4m44s              replicaset-controller  Created pod: nginx-f77774fc5-dfbnc
     Normal   SuccessfulCreate  3m24s              replicaset-controller  Created pod: nginx-f77774fc5-f42kx
     Warning  FailedCreate      76s                replicaset-controller  Error creating: pods "nginx-f77774fc5-xwjvc" is forbidden: exceeded quota: resource-quota-pod, requested: pods=1, used: pods=5, limited: pods=5
     Warning  FailedCreate      36s (x5 over 75s)  replicaset-controller  (combined from similar events): Error creating: pods "nginx-f77774fc5-2pq8z" is forbidden: exceeded quota: resource-quota-pod, requested: pods=1, used: pods=5, limited: pods=5
   ```

1. 作成したリソースを削除してください。

   【回答例】

   ```bash
   # 実行結果
   $ kubectl delete -f resource-quota-pod.yaml
   resourcequota "resource-quota-pod" deleted
   deployment.apps "nginx" deleted
   ```

[1]:https://kubernetes.io/docs/concepts/policy/resource-quotas/
