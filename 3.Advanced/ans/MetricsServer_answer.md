# 回答例

1. metrics serverをデプロイしてください。[公式ドキュメント][1]や[GitHubリポジトリ][2]を参考にしてください。

【回答例】

   ```bash
   # 実行結果
   $ kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
   serviceaccount/metrics-server created
   clusterrole.rbac.authorization.k8s.io/system:aggregated-metrics-reader created
   clusterrole.rbac.authorization.k8s.io/system:metrics-server created
   rolebinding.rbac.authorization.k8s.io/metrics-server-auth-reader created
   clusterrolebinding.rbac.authorization.k8s.io/metrics-server:system:auth-delegator created
   clusterrolebinding.rbac.authorization.k8s.io/system:metrics-server created
   service/metrics-server created
   deployment.apps/metrics-server created
   apiservice.apiregistration.k8s.io/v1beta1.metrics.k8s.io created
   ```

1. 以下コマンドを実行しmetrics serverでメトリクスの取得ができていることを確認してください。

   ```bash
   kubectl top node
   kubectl top pod -n kube-system
   ```

> :information_source:  
> metrics serverが起動してから情報収集まで1分ほど時間がかかります。  
> 時間が経ってもkubectl topで結果がうまく出力されず、metrics serverのログに`unable to fetch node metrics for node ~`や`"unable to fetch node metrics for pod ~"などが出力されている場合はデバッグが必要になります。
> インターネットを調べてデバッグしてください。

【回答例】

   ```bash
   # 実行結果
   $ kubectl top node
   NAME                                           CPU(cores)   CPU%   MEMORY(bytes)   MEMORY%
   ip-192-168-16-0.us-west-1.compute.internal     42m          2%     519Mi           15%
   ip-192-168-32-153.us-west-1.compute.internal   48m          2%     536Mi           16%
   ip-192-168-49-60.us-west-1.compute.internal    44m          2%     496Mi           14%

   $ kubectl top pod -n kube-system
   NAME                              CPU(cores)   MEMORY(bytes)
   aws-node-nlxbc                    4m           40Mi
   aws-node-rkpwt                    3m           40Mi
   aws-node-scqlf                    4m           40Mi
   coredns-66747cfb9-7jr9t           1m           12Mi
   coredns-66747cfb9-t8wdh           1m           12Mi
   kube-proxy-drzhx                  1m           10Mi
   kube-proxy-qhfwm                  1m           10Mi
   kube-proxy-r95mr                  1m           10Mi
   metrics-server-68c56b9d8c-jgm68   2m           16Mi
   ```

1. 作成したリソースを削除してください。

【回答例】

   ```bash
   # 実行結果
   $ kubectl delete -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
   serviceaccount "metrics-server" deleted
   clusterrole.rbac.authorization.k8s.io "system:aggregated-metrics-reader" deleted
   clusterrole.rbac.authorization.k8s.io "system:metrics-server" deleted
   rolebinding.rbac.authorization.k8s.io "metrics-server-auth-reader" deleted
   clusterrolebinding.rbac.authorization.k8s.io "metrics-server:system:auth-delegator" deleted
   clusterrolebinding.rbac.authorization.k8s.io "system:metrics-server" deleted
   service "metrics-server" deleted
   deployment.apps "metrics-server" deleted
   apiservice.apiregistration.k8s.io "v1beta1.metrics.k8s.io" deleted
   ```

[1]:https://kubernetes.io/docs/tasks/debug-application-cluster/resource-metrics-pipeline/#metrics-server
[2]:https://github.com/kubernetes-sigs/metrics-server
