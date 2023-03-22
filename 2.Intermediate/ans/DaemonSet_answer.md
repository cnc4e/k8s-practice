# 回答例

1. クラスタの参加ノードを確認し、ワーカーノードが2台以上の状態にしてください。

   【回答例】

   ```bash
   $ kubectl get node
   NAME                                           STATUS   ROLES    AGE   VERSION
   ip-192-168-40-85.us-west-1.compute.internal    Ready    <none>   18d   v1.22.17-eks-48e63af
   ip-192-168-5-135.us-west-1.compute.internal    Ready    <none>   18d   v1.22.17-eks-48e63af
   ip-192-168-60-176.us-west-1.compute.internal   Ready    <none>   18d   v1.22.17-eks-48e63af
   ```

1. 以下を満たすマニフェストを作成しデプロイしてください。なお、DaemonSetについては[公式ドキュメント][1]を参考にしてください。

   - 要件
     - DaemonSet
       - 名前は`daemon`
       - labelはすべて`app: nginx`
       - Pod
         - メインコンテナ
           - 名前は`nginx`
           - イメージは`nginx:1.12`

   【回答例】

   ```yml
   # manifest
   apiVersion: apps/v1
   kind: DaemonSet
   metadata:
     name: deamon
   spec:
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
   ```

   ```bash
   # 実行結果
   $ kubectl apply -f nginx.yaml
   daemonset.apps/deamon created
   ```

1. DaemonSetリソースのオブジェクト一覧を表示し、`daemon`があることを確認してください。

   【回答例】

   ```bash
   # 実行結果
   $ kubectl get daemonset
   NAME     DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR   AGE
   deamon   3         3         3       3            3           <none>          57s
   ```

1. Podリソースのオブジェクト一覧を起動ノード名も一緒に表示し、すべてのワーカーノードで`daemon`Podが起動していることを確認してください。

   【回答例】

   ```bash
   # 実行結果
   $ kubectl get pod -o wide
   NAME           READY   STATUS    RESTARTS   AGE   IP               NODE                                           NOMINATED NODE   READINESS GATES
   deamon-7f5pk   1/1     Running   0          88s   192.168.11.137   ip-192-168-5-135.us-west-1.compute.internal    <none>           <none>
   deamon-fptq4   1/1     Running   0          88s   192.168.43.138   ip-192-168-60-176.us-west-1.compute.internal   <none>           <none>
   deamon-mhb7q   1/1     Running   0          88s   192.168.55.180   ip-192-168-40-85.us-west-1.compute.internal    <none>           <none>
   ```

1. `daemon`Podを1つ削除してからPodリソースのオブジェクト一覧を表示し、セルフヒーリングでPodが再作成されていることを確認してください。

   【回答例】

   ```bash
   # 実行結果
   $ kubectl delete pod deamon-fptq4
   pod "deamon-fptq4" deleted

   $ kubectl get pod -o wide
   NAME           READY   STATUS    RESTARTS   AGE     IP               NODE                                           NOMINATED NODE   READINESS GATES
   deamon-7f5pk   1/1     Running   0          2m34s   192.168.11.137   ip-192-168-5-135.us-west-1.compute.internal    <none>           <none>
   deamon-mhb7q   1/1     Running   0          2m34s   192.168.55.180   ip-192-168-40-85.us-west-1.compute.internal    <none>           <none>
   deamon-q6tzz   1/1     Running   0          12s     192.168.54.123   ip-192-168-60-176.us-west-1.compute.internal   <none>           <none>
   ```

1. 作成したリソースを削除してください。

   【回答例】

   ```bash
   # 実行結果
   $ kubectl delete -f daemonset.yaml
   daemonset.apps "deamon" deleted
   ```

[1]:https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/
