前： [Service-LB](Service-LB.md)  

---

# Pod設定 resources

Podはとくに指定がない場合、ワーカーノードのHWリソース（CPU/メモリ）を限界まで使用します。
これだと、各Podが際限なくHWリソースを要求し、最終的にワーカーノードのHWリソースが足りなくなる場合もあります。
このような事態を防ぐため、PodにHWリソースの`要求容量（requests）`と`上限容量（limits）`を設定できます。この設定がresourcesです。
resourcesはPodに含まれる各コンテナ単位で設定できます。Podのresourcesは含まれるコンテナのresourcesを合算した値です。

要求容量（requests）はPod起動時に最低限確保が保証されるHWリソース容量です。
Podはこの要求容量が確保できるワーカーノードで起動します。
もし、どのワーカーノードでも要求容量が確保できなかった場合、Podは実行されず実行待ち状態（Pending）になります。

上限容量（limits）はPodが使用できるHWリソース容量の上限です。
上限までHWリソースを使用できるが確保は保証されません。
つまり、上限容量はワーカーノード上でオーバーコミットされる可能性があります。
オーバーコミットを許容しない場合は各Podの要求容量と上限容量を同じにすることで可能です。（もしくは上限容量だけ設定すると自動で要求容量も同じ値で設定されます。）

# 演習

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

1. デプロイしたPodの詳細を表示し、コンテナにrequestsとlimitsが設定されていることを確認してください。

1. マニフェストを以下の内容に修正し、再デプロイしてください。

   - 要件
     - Deployment
       - Pod
         - 以下のHWリソース容量を指定
           - memory上限`100Mi`

1. デプロイしたPodの詳細を表示し、各コンテナにresourcesが設定さていることを確認してください。またmemoryの`requetsとlimitsが同じ値`で設定されていることを確認してください。

1. Deploymentのreplica数を10に拡張してください。

1. Podリソースのオブジェクト一覧を表示し、STATUS:PendingのPodがあることを確認してください。（もしなかった場合、Deploymentのreplica数をより大きな値に修正し、再度確認してください。）

1. STATUS:PendingのPodの詳細を表示しeventを確認してください。以下の様なメッセージが出力されていることを確認してください。（これはワーカー3台の環境で要求した量のCPUを確保できるノードが見つからなかった場合のメッセージです。）

   ```bash
   0/3 nodes are available: 3 Insufficient cpu.
   ```

1. 作成したリソースを削除してください。

以上で本演習は終了です。

具体的な操作およびその結果に関する回答例は[こちら](../ans/Pod-resources_answer.md)にあります。
具体的な操作方法がわからなかった場合や、想定した結果にならなかった場合などに参照してください。

[1]:https://kubernetes.io/docs/concepts/configuration/manage-compute-resources-container/

---

次： [Pod-Probe](Pod-Probe.md)  
