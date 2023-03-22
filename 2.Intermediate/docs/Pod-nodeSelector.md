前： [Pod-initContainer](Pod-initContainer.md)  

---

# Pod設定 nodeSelector

Podはマスターコンポーネントのkube-schedulerによって起動するワーカーノードが決められます。
このPodをワーカーノードに割り当てることをK8sでは`スケジュール`といいます。
Podにスケジュール関連のパラメータが設定されていない場合、kube-schedulerはノードの負荷状況などを考慮してスケジュールします。
しかし、どうしても起動先ノードを指定したい場合もあります。（例えば、GPUを使用するPodをGPU搭載のワーカーノードで起動したい場合など）
その様な場合にPodのスケジュール先を指定するもっとも簡単な方法がnodeSelectorです。

# 演習

1. クラスタの参加ノードを確認し、ワーカーノードが2台以上存在していることを確認してください。

1. ノードに付与されたラベルを表示してください。

1. どれか一台のノードに`node-spec=monster`のラベルを付与してください。

1. 他のノードには`node-spec=normal`のラベルを付与してください。

1. key:node-specをカラムに追加してノードの一覧で表示してください。一台のノードにmonsterのvalueがあることを確認してください。

1. 以下を満たすマニフェストを作成しデプロイしてください。なお、nodeSelectorについては[公式ドキュメント][1]を参考にしてください。

   - 要件
     - Deployment
       - 名前は`nginx`
       - replicas: `1`
       - labelはすべて`app: nginx`
       - Pod
         - 名前は`nginx`
         - イメージは`nginx:1.12`
       - nodeSelector
         - ラベル`node-spec`にvalue`monster`が付与されたノードを指定

1. Podリソースの一覧をwideで表示し、Podが`node-spec:monster`のラベルを付与したノードで起動していることを確認してください。

1. Deployment:node-selectorのPodレプリカ数を10に変更してください。新たに作成されたPodもすべて`node-spec:monster`のラベルを付与したノードで起動していることを確認してください。

1. Deployment:nginxのnodeSelectorをkey`node-spec`にvalue`normal`が付与されたノード指定に変更してください。

1. Podのローリングアップデートが実行され、すべてのPodが`node-spec:normal`のラベルを付与したノードで起動していることを確認してください。

1. Deployment:nginxをいったん削除してください。

1. Deployment:nginxのnodeSelectorをkey`node-spec`にvalue`super`が付与されたノード指定に変更しデプロイしてください。

1. すべてのPodのSTATUSが`Pending`となることを確認してください。

1. Deployment:nginxを削除してください。

1. ノードに付与されたkey`node-spec`のラベルを削除してください。

以上で本演習は終了です。

具体的な操作およびその結果に関する回答例は[こちら](../ans/Pod-nodeSelector_answer.md)にあります。
具体的な操作方法がわからなかった場合や、想定した結果にならなかった場合などに参照してください。

[1]:https://kubernetes.io/docs/concepts/configuration/assign-pod-node/#nodeselector

---

次： [postStart/preStop](Pod-lifecycle.md)
