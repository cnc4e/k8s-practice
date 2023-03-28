# 回答例

1. 以下を満たすマニフェストを作成しデプロイしてください。HPAについては[公式ドキュメント][1]を参考にしてください。

   - 要件
     - Deployment
       - replica: 1
       - image:nginx: 1.12
       - resources.limits.cpu: 100m
     - Service
       - 上記Deploymentをtype:ClusterIPで公開
       - Portは80
     - HPA
       - 上記Deploymentを対象にする
       - Pod数を最小1 ~ 最大5の範囲とする
       - CPU使用率80%を目標値とする

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
             resources:
               limits:
                 cpu: 100m
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
   ---
   apiVersion: autoscaling/v1
   kind: HorizontalPodAutoscaler
   metadata:
     name: nginx
   spec:
     scaleTargetRef:
       apiVersion: apps/v1
       kind: Deployment
       name: nginx
     minReplicas: 1
     maxReplicas: 5
     targetCPUUtilizationPercentage: 80
   ```

   ```bash
   # 実行結果
   $ kubectl apply -f hpa-nginx.yaml
   deployment.apps/nginx created
   service/nginx-svc created
   horizontalpodautoscaler.autoscaling/nginx created
   ```

1. 作成したオブジェクトを確認してください。Podが1つであることを確認してください。

   【回答例】

   ```bash
   # 実行結果
   $ kubectl get pod
   NAME                     READY   STATUS    RESTARTS   AGE
   nginx-74c9bf5488-tdnsq   1/1     Running   0          39m
   ```

1. 以下を満たすマニフェストを作成しデプロイしてください。

   - 要件
     - Deployment
       - image: httpd
       - resources.limits.cpu: 100m
     - Service
       - 上記Deploymentをtype:ClusterIPで公開
       - Portは80
     - HPA
       - 上記Deploymentを対象にする
       - Pod数を最小1 ~ 最大5の範囲とする
       - CPU使用率80%を目標値とする

   【回答例】

   ```yml
   # manifest
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: httpd
   spec:
     replicas: 1
     selector:
       matchLabels:
         app: httpd
     template:
       metadata:
         labels:
           app: httpd
       spec:
         containers:
           - name: httpd
             image: httpd:latest
             ports:
               - containerPort: 80
             resources:
               limits:
                 cpu: 100m
   ---
   apiVersion: v1
   kind: Service
   metadata:
     name: httpd-svc
   spec:
     ports:
       - name: "http-port"
         protocol: "TCP"
         port: 80
         targetPort: 80
     selector:
       app: httpd
   ---
   apiVersion: autoscaling/v1
   kind: HorizontalPodAutoscaler
   metadata:
     name: httpd
   spec:
     scaleTargetRef:
       apiVersion: apps/v1
       kind: Deployment
       name: httpd
     minReplicas: 1
     maxReplicas: 5
     targetCPUUtilizationPercentage: 80
   ```

   ```bash
   # 実行結果
   $ kubectl apply -f hpa-httpd.yaml
   deployment.apps/httpd created
   service/httpd-svc created
   horizontalpodautoscaler.autoscaling/httpd created
   ```

1. 上記作成したPodに以下の様な追加コマンドを発行してください。(宛先は作成したService名にする。最後の/を忘れずに。-nと-cの値はお好みで)

   ```bash
   ab -n 1000000 -c 100 http://httpd-svc/
   ```

   【回答例】

   ```bash
   # 実行結果
   ## testpod作成
   $ kubectl run --image=httpd --restart=Never testpod sleep 3600
   pod/testpod created

   $ ベンチマークコマンド実行
   kubectl exec testpod -- ab -n 1000000 -c 100 http://httpd-svc/
   ```

1. 別ターミナルを開き、「kubectl top pod」と「kubectl get pod」をwatch等で監視し、以下を確認してください。
   - しばらくするとPodの数が増えること
   - Podが5つまで自動拡張されること
   - すべてのPodの負荷が80%(80m)を超えていてもPodが5つ以上増えないこと

   【回答例】

   ```bash
   # 実行結果
   ## ベンチマーク開始直後
   $ kubectl get pod
   NAME                     READY   STATUS    RESTARTS   AGE
   httpd-5d77646999-9h8kr   1/1     Running   0          19m
   nginx-74c9bf5488-tdnsq   1/1     Running   0          64m

   $ kubectl top pod
   NAME                     CPU(cores)   MEMORY(bytes)
   httpd-5d77646999-9h8kr   19m          39Mi
   nginx-74c9bf5488-tdnsq   0m           2Mi

   ## HPAによるAutoScaling開始
   $ kubectl get pod
   NAME                     READY   STATUS    RESTARTS   AGE
   httpd-5d77646999-9h8kr   1/1     Running   0          31m
   httpd-5d77646999-w8r5k   1/1     Running   0          8m11s
   nginx-74c9bf5488-tdnsq   1/1     Running   0          77m

   $ kubectl top pod
   NAME                     CPU(cores)   MEMORY(bytes)
   httpd-5d77646999-9h8kr   98m          33Mi
   httpd-5d77646999-w8r5k   95m          13Mi
   nginx-74c9bf5488-tdnsq   0m           2Mi

   ## 一定時間経過後
   $ kubectl get pod
   NAME                     READY   STATUS    RESTARTS   AGE
   httpd-5d77646999-8mj9z   1/1     Running   0          5m41s
   httpd-5d77646999-9h8kr   1/1     Running   0          31m
   httpd-5d77646999-d9gb4   1/1     Running   0          6m41s
   httpd-5d77646999-gbr6z   1/1     Running   0          7m26s
   httpd-5d77646999-w8r5k   1/1     Running   0          8m11s
   nginx-74c9bf5488-tdnsq   1/1     Running   0          77m

   $ kubectl top pod
   NAME                     CPU(cores)   MEMORY(bytes)
   httpd-5d77646999-8mj9z   97m          13Mi
   httpd-5d77646999-9h8kr   97m          31Mi
   httpd-5d77646999-d9gb4   99m          23Mi
   httpd-5d77646999-gbr6z   96m          22Mi
   httpd-5d77646999-w8r5k   100m         16Mi
   nginx-74c9bf5488-tdnsq   0m           2Mi
   ```

1. 作成したリソースを削除してください。

   【回答例】

   ```bash
   # 実行結果
   $ kubectl delete -f hpa-nginx.yaml
   deployment.apps "nginx" deleted
   service "nginx-svc" deleted
   horizontalpodautoscaler.autoscaling "nginx" deleted

   $ kubectl delete -f hpa-httpd.yaml
   deployment.apps "httpd" deleted
   service "httpd-svc" deleted
   horizontalpodautoscaler.autoscaling "httpd" deleted

   $ kubectl delete pod testpod
   pod "testpod" deleted
   ```

[1]:https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale-walkthrough/
