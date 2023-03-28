前： -

---

# metrics server

metrics serverはPodやワーカーノードのHWリソース量(CPU/メモリ)を取得するアドオン機能です。
metrics serverはK8s内にPodとして起動させます。
metrics serverを導入すると「kubectl top」コマンドが使用可能になります。
Pod数を自動で増減させるHorizontal Pod Autoscalerを使用するにはmetrics serverが必要となります。
（なお、EKSの場合は利用者で手動デプロイが必要となりますが、AKSやGKEではマネージド・サービスの中で自動的にデプロイされます。）

# 演習

1. metrics serverをデプロイしてください。[公式ドキュメント][1]や[GitHubリポジトリ][2]を参考にしてください。

1. 以下コマンドを実行しmetrics serverでメトリクスの取得ができていることを確認してください。

   ```bash
   kubectl top node
   kubectl top pod -n kube-system
   ```

> :information_source:  
> metrics serverが起動してから情報収集まで1分ほど時間がかかります。  
> 時間が経ってもkubectl topで結果がうまく出力されず、metrics serverのログに`unable to fetch node metrics for node ~`や`"unable to fetch node metrics for pod ~"などが出力されている場合はデバッグが必要になります。
> インターネットを調べてデバッグしてください。

1. 作成したリソースを削除してください。

以上で本演習は終了です。

具体的な操作およびその結果に関する回答例は[こちら](../ans/MetricServer_answer.md)にあります。
具体的な操作方法がわからなかった場合や、想定した結果にならなかった場合などに参照してください。

[1]:https://kubernetes.io/docs/tasks/debug-application-cluster/resource-metrics-pipeline/#metrics-server
[2]:https://github.com/kubernetes-sigs/metrics-server

---

次： [HorizontalPodAutoscaler](HorizontalPodAutoscaler.md)  
