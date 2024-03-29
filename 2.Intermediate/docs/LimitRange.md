前： [CronJob](CronJob.md)  

---

# LimitRange

LimitRangeはNamespace内のPodやPVCに対して、使用できるHWリソース（CPU,メモリなど）の制限やデフォルト値を設けるリソースです。
Namespaceに属します。ResourceQuotaとは違い、Podやコンテナに対して制限を設定します。
Pod(コンテナ)のHWリソース量はlimitsとrequestsで指定できますが、LimitRangeはそのlimitsとrequestsで指定できる値の範囲に制限を設けます。
制限はコンテナ単位やコンテナを合算したPod単位で設けることができます。
また、PVCで指定するボリュームサイズに対しても制限を設けることができます。
さらに、LimitRangeはHWリソース量のデフォルト値を設定することもできます。後述する[HPA](../../3.Advanced/docs/HorizontalPodAutoscaler.md)を使う環境では基本的にすべてのPod（コンテナ）に対してresourcesを設定することが望ましいです。
とはいえ、設定し忘れることもあるのでデフォルト値を指定しておけばより安全です。

# 演習

## コンテナの上限設定

まずはコンテナのHWリソース量の指定上限について確認します。

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

1. 上記マニフェストをデプロイし、Podリソースの一覧を表示してください。Podが作成されていないことを確認してください。

1. ReplicaSetリソースの一覧を表示し、上記マニフェストでデプロイしたReplicaSetの詳細情報を表示してください。以下のようにLimitRangeで指定した範囲に違反したためPodの作成に失敗した旨のメッセージを確認する。

   ```bash
   Error creating: pods "resources-6bf4ffd649-b9hb2" is forbidden: maximum cpu usage per Container is 300m, but limit is 400m.
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

1. Podリソースの一覧を表示してください。Podが作成されていないことを確認してください。

1. ReplicaSetリソースの一覧を表示し、上記マニフェストでデプロイしたReplicaSetの詳細情報を表示してください。以下のようにLimitRangeで指定した範囲に違反したためPodの作成に失敗した旨のメッセージを確認する。

   ```bash
   Error creating: pods "resources-787f48c74f-2t5rd" is forbidden: minimum memory usage per Container is 100Mi, but request is 10Mi.
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

1. Podリソースの一覧を表示してください。Podが`作成されている`ことを確認してください。

1. 作成したリソースを削除してください。

## Podの上限設定

次に、Podに対するHWリソース量の範囲制限について確認する。

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

1. 上記マニフェストをデプロイし、Podリソースの一覧を表示してください。Podが作成されていないことを確認してください。

1. ReplicaSetリソースの一覧を表示し、上記マニフェストでデプロイしたReplicaSetの詳細情報を表示してください。以下のようにLimitRangeで指定した範囲に違反したためPodの作成に失敗した旨のメッセージを確認する。

   ```bash
   Error creating: pods "resources-65fb6d7459-kkg7c" is forbidden: minimum memory usage per Pod is 200Mi, but request is 115343360.
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

1. Podリソースの一覧を表示してください。Podが`作成されている`ことを確認してください。

1. 作成したリソースを削除してください。

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

1. Podの詳細を確認し、limitsとrequestsが設定されていることを確認してください。

1. 作成したリソースを削除してください。

このようにLimitRangeでコンテナのデフォルトresourcesを設定できます。
デフォルトlimitsのみでrequestsを指定しない場合はlimitsと同じ値が設定されます。（resourceの時と同じです。)

以上で本演習は終了です。

具体的な操作およびその結果に関する回答例は[こちら](../ans/LimitRange_answer.md)にあります。
具体的な操作方法がわからなかった場合や、想定した結果にならなかった場合などに参照してください。

[1]:https://kubernetes.io/docs/concepts/policy/limit-range/

---

次： [ResourceQuota](ResourceQuota.md)  
