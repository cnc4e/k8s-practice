前： [LimitRange](LimitRange.md)  

---

# ResourceQuota

ResourcesQuotaはNamespaceに対しHWリソース上限や作成できるK8sのオブジェクト数に制限を設けるリソースです。Namespaceに属します。
LimitRangeとは違い、Namespaceに対する制限です。

# 演習

## HWリソース制限

まずはHWリソース量の制限について確認します。

※ ワーカーノードのHWリソースを十分な量(CPU:1コア、メモリ:1G)確保しておいてください。

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

1. デプロイしたResourceQuotaオブジェクトの詳細を表示してください。現在の利用量が表示されます。（利用量はrequestsで計算されます）

1. Deploymentのreplica数を`10`に拡張してください。

1. デプロイしたResourceQuotaオブジェクトの詳細を表示してください。現在の利用量が表示されます。（利用量はrequestsで計算されます）

1. Deploymentのreplica数を`11`に拡張してください。

1. Podのオブジェクト一覧を表示してください。Podの数が10のままであること。

1. ReplicaSetの詳細を表示し、以下のようにCPUの利用量がResourceQuotaで指定した範囲に違反したメッセージが表示されること。

   ```bash
   Error creating: pods "quota-5c4f499fb8-nxfqb" is forbidden: exceeded quota: quota, requested: cpu=100m, used: cpu=1, limited: cpu=1
   ```

1. 作成したリソースを削除してください。

以上のようにNamespaceに対してHWリソース量の制限を設けることができます。ちなみに、ResourceQuotaで制限を設けている状態でrequestsを指定しないPodを起動しようとしても起動できません。

## オブジェクト数制限

次にK8sリソースのオブジェクト数の制限について確認します。

1. 以下を満たすマニフェストを作成しデプロイしてください。ResourceQuotaリソースについては[公式ドキュメント][1]を参考にしてください。

   - 要件
     - ResourceQuota
       - Podの制限を5
     - Deployment
       - イメージは何でもよい
       - replicas: 1

1. デプロイしたResourceQuotaオブジェクトの詳細を表示してください。現在の利用量が表示されます。

1. Deploymentのreplica数を5に拡張してください。

1. デプロイしたResourceQuotaオブジェクトの詳細を表示してください。現在の利用量が表示されます。

1. Deploymentのreplica数を6に拡張してください。

1. Podのオブジェクト一覧を表示してください。Podの数が5のままであること。

1. ReplicaSetの詳細を表示し、以下のようにCPUの利用量がResourceQuotaで指定した範囲に違反したメッセージが表示されること。

   ```bash
   replicaset-controller  Error creating: pods "quota-68847df66-bplgc" is forbidden: exceeded quota: quota, requested: count/pods=1, used: count/pods=5, limited: count/pods=5
   ```

1. 作成したリソースを削除してください。

以上のようにNamespace内のK8sオブジェクト数に制限を設けることができます。

Namespace単位で制限をかけられるのでLimitRangeよりも手っ取り早くリソース制限をかけることができます。
しかし、各オブジェクト単位のリソース量制限はできため、その場合はLimitRangeを使います。
このように、ResourceQuotaはたとえば1つのK8sクラスタを複数のチームで共有利用する場合などに使える機能です。

以上で本演習は終了です。

具体的な操作およびその結果に関する回答例は[こちら](../ans/ResourceQuota_answer.md)にあります。
具体的な操作方法がわからなかった場合や、想定した結果にならなかった場合などに参照してください。

[1]:https://kubernetes.io/docs/concepts/policy/resource-quotas/

---

次： [RBAC](RBAC.md)  
