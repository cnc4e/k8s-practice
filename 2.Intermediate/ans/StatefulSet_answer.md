# 回答例

1. 以下を満たすマニフェストを作成しデプロイしてください。なお、StatefulSetについては[公式ドキュメント][1]を参考にしてください。

   - 要件
     - StatefulSet
       - 名前は`nginx-sts`
       - labelはすべて`app: nginx`
       - replicasは `3`
       - serviceNameは`nginx-svc`
       - Pod
         - 名前は`nginx`
         - イメージは`nginx:1.12`
     - Service
       - 名前は`nginx-svc`
       - typeは`ClusterIP`（`明示的に指定する`）
       - clusterIPに`None`
       - Protocolは`TCP`
       - 待ち受けポートは`80`
       - ターゲットポートは`80`
       - selectorは`app: nginx-sts`

   【回答例】

   ```yml
   apiVersion: apps/v1
   kind: StatefulSet
   metadata:
     name:
       nginx-sts
   spec:
     serviceName: nginx-sts
     replicas: 3
     selector:
       matchLabels:
         app: nginx-sts
     template:
       metadata:
         labels:
           app: nginx-sts
       spec:
         containers:
          - name: nginx
            image: nginx:1.12
            ports:
             - containerPort: 80
   ---
   apiVersion: v1
   kind: Service
   metadata:
     name: nginx-svc
   spec:
     type: ClusterIP
     clusterIP: None
     ports:
      - name: "http-port"
        protocol: "TCP"
        port: 80
        targetPort: 80
     selector:
       app: nginx-sts
   ```

   ```bash
   $ kubectl apply -f sts.yaml
   statefulset.apps/nginx-sts created
   service/nginx-svc created
   ```

1. StatefulSetリソースのオブジェクト一覧を表示し、`stateful`があることを確認してください。

   【回答例】

   ```bash
   $ kubectl get statefulset
   NAME                  READY   AGE
   nginx-sts   3/3     76s
   ```

1. Podリソースのオブジェクト一覧を表示し、`stateful-0,1,2`の３つのPodがあることを確認してください。また起動時間から起動が順番に行われたことを確認してください。

   【回答例】

   ```bash
   $ kubectl get pod
   NAME                     READY   STATUS    RESTARTS   AGE
   nginx-sts-0   1/1     Running   0          102s
   nginx-sts-1   1/1     Running   0          100s
   nginx-sts-2   1/1     Running   0          98s
   ```

1. Serviceリソースのオブジェクト一覧を表示し、`nginx-svc`があること。また、`Headless Service`であることを確認してください。  
  （Headless ServiceはCluster-IPがNoneなServiceです。）

   【回答例】

   ```bash
   $ kubectl get svc
   NAME               TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)    AGE
   nginx-service   ClusterIP   None         <none>        5432/TCP   3m25s
   ```

1. 接続確認に使用するcurlコマンドが実行可能なtestpodを作成し、testpodから次のコマンドで接続確認をしてください。

   ```bash
   nslookup nginx-svc
   curl -s nginx-sts-0.nginx-svc.default.svc.cluster.local
   curl -s nginx-sts-1.nginx-svc.default.svc.cluster.local
   curl -s nginx-sts-2.nginx-svc.default.svc.cluster.local
   ```

   【回答例】

   ```bash
   # 実行結果
   ## testpod作成
   $ kubectl run --image=appropriate/curl --restart=Never testpod sleep 3600
   pod/testpod created
   ## コマンド１つ目
   $ kubectl exec testpod -- nslookup nginx-svc

   nslookup: can't resolve '(null)': Name does not resolve
   Name:      nginx-svc
   Address 1: 192.168.54.123 nginx-sts-2.nginx-svc.default.svc.cluster.local
   Address 2: 192.168.42.244 nginx-sts-1.nginx-svc.default.svc.cluster.local
   Address 3: 192.168.27.110 nginx-sts-0.nginx-svc.default.svc.cluster.local

   ## コマンド2つ目
   $ kubectl exec testpod -- curl -s nginx-sts-0.nginx-svc.default.svc.cluster.local
   <!DOCTYPE html>
   <html>
   <head>
   <title>Welcome to nginx!</title>
     (略)
   </body>
   </html>

   ## コマンド3つ目
   $ kubectl exec testpod -- curl -s nginx-sts-1.nginx-svc.default.svc.cluster.local
   <!DOCTYPE html>
   <html>
   <head>
   <title>Welcome to nginx!</title>
     (略)
   </body>
   </html>

   ## コマンド4つ目
   $ kubectl exec testpod -- curl -s nginx-sts-2.nginx-svc.default.svc.cluster.local
   <!DOCTYPE html>
   <html>
   <head>
   <title>Welcome to nginx!</title>
     (略)
   </body>
   </html>
   ```

1. Pod:`nginx-sts-1`のIPアドレスを確認してからPod:`nginx-sts-1`を削除してください。

   【回答例】

   ```bash
   # 実行結果
   ## IPアドレス確認 (kubectl describe pod nginx-sts-1 などでも可)
   $ kubectl get pod -o wide
   NAME          READY   STATUS    RESTARTS   AGE   IP               NODE                                           NOMINATED NODE   READINESS GATES
   nginx-sts-0   1/1     Running   0          11s   192.168.11.137   ip-192-168-5-135.us-west-1.compute.internal    <none>           <none>
   nginx-sts-1   1/1     Running   0          8s    192.168.55.180   ip-192-168-40-85.us-west-1.compute.internal    <none>           <none>
   nginx-sts-2   1/1     Running   0          7s    192.168.43.138   ip-192-168-60-176.us-west-1.compute.internal   <none>           <none>

   # Pod:nginx-sts-1を削除
   $ kubectl delete pod nginx-sts-1
   pod "nginx-sts-1" deleted
   ```

1. Pod:`nginx-sts-1`がセルフ・ヒーリングされていることを確認してください。また、IPアドレスが変わっていることを確認してください。

   【回答例】

   ```bash
   # 実行結果
   $ kubectl get pod -o wide
   NAME                    READY   STATUS    RESTARTS   AGE   IP               NODE                                           NOMINATED NODE   READINESS GATES
   nginx-sts-0   1/1     Running   0          21m   192.168.34.170   ip-192-168-50-200.us-west-1.compute.internal   <none>           <none>
   nginx-sts-1   1/1     Running   0          11s   192.168.42.244   ip-192-168-40-85.us-west-1.compute.internal    <none>           <none>
   nginx-sts-2   1/1     Running   0          21m   192.168.20.119   ip-192-168-9-62.us-west-1.compute.internal     <none>           <none>
   ```

1. testpodから次のコマンドで接続確認をしてください。

   ```bash
   nslookup nginx-svc
   curl -s nginx-sts-1.nginx-svc.default.svc.cluster.local
   ```

   【回答例】

   ```bash
   # 実行結果
   ## コマンド１つ目
   $ kubectl exec testpod -- nslookup nginx-svc

   nslookup: can't resolve '(null)': Name does not resolve
   Name:      nginx-svc
   Address 1: 192.168.43.138 nginx-sts-2.nginx-svc.default.svc.cluster.local
   Address 2: 192.168.42.244 nginx-sts-1.nginx-svc.default.svc.cluster.local
   Address 3: 192.168.11.137 nginx-sts-0.nginx-svc.default.svc.cluster.local

   ## コマンド2つ目
   $ kubectl exec testpod -- curl -s nginx-sts-1.nginx-svc.default.svc.cluster.local
   <!DOCTYPE html>
   <html>
   <head>
   <title>Welcome to nginx!</title>
     (略)
   </body>
   </html>
   ```

1. 作成したリソースを削除してください。

   【回答例】

   ```bash
   # 実行結果
   $ kubectl delete -f sts.yaml
   statefulset.apps "nginx-sts" deleted
   service "nginx-svc" deleted
   ```

[1]:https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/
