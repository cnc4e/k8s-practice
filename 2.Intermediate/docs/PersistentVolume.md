
前： -

---

# K8sでのデータ永続化

初級の[volume](../../1.Beginner/docs/Pod-volume.md)の章において

- Pod内のデータは一時的なものである
- Podが消えると同時にデータも消える
- データを保持し続ける（＝データを永続化する）の仕組みがK8sには用意されている
と解説しました。

K8sのデータ永続化について、本章でより詳細に説明します。

## データ永続化する方法

K8sにおいて、データを永続化する方法は大きく分類して次の2つになります。

- Volume
  - Podに永続領域を直接割り当てる
    - 初級編で操作したhostPathはこちら
  - Podの機能の一部
- PersistentVolume (PV)
  - Podとは独立した、永続領域を定義するAPIリソース
  - PVC(PersistentVolumeClaim)を経由して、使用する

### Volume

APIリソース：Podに用意されている機能です。
PodにVolumeを設定することで、Podのデータを永続化することができます。

設定可能なVolumeには様々な種類があります。
代表的なものは次の通りです。

1. Volume設定なし (Volumeに関する設定を記載しない)
   - Podが起動するホストが持つが領域を一時的に間借りする
   - Podの消滅とともに消える
     - 意図的な削除(Terminate)、障害(Crash)のいずれでも消える
1. emptyDir
   - Podが起動するホストが持つが領域を一時的に間借りする
   - Podの消滅とともに消える
     - 意図的な削除(Terminate)時に消える
1. [hostPath](./../../1.Beginner/docs/Pod-volume.md)
   - ホストが持つ領域を割り当てる
   - Podが消滅してもデータは消えない
1. NFS
   - NW上にNFSボリュームをPodに直接マウントする
   - Podが消滅してもデータは消えない

### PersistentVolume

PersistentVolumeはK8sが管理する永続化領域です。
Podに直接パスを書くのではなく、PersistentVolumeを使用することで、より可搬性が向上する他、Podに対して永続化領域を動的に割り当てることも可能になります。

一言でPersistentVolumeと表現していますが、実際にこの仕組みを実現するために、

- 「このK8sクラスタでは、どのような種類のストレージを使用可能か」を定義した`StorageClass (SC)`
- 「実際に切り出され、クラスタで使用可能なディスク」を示す`PersistentVolume (PV)`
- 「このPodはどんなディスクを必要とするか」を宣言する`PersistentVolumeClaim (PVC)`

というAPIリソースが使用されています。

### DynamicVolumeProvisioning

最後にDynamicVolumeProvisioningの解説をします。
DynamicVolumeProvisioning(動的プロビジョニング/dvp)はK8sの機能の１つです。

簡単に言うと、`PVCでSCにストレージを要求すると、条件を満たすPVを自動的に作成、Podに割り当ててくれる`という機能です。
この機能により利用者はPVを事前に用意する必要がなくなります。

例えば、Podの負荷に応じてPodを動的に増加（いわゆるAutoScaling）させる場合、
このdvpがなければ、想定される最大数のPVを事前に用意、プールしておく必要があります。

しかしこの機能を使用すると、Pod起動時に合わせてPVも自動作成してくれるため、
事前の利用予測のような手間から利用者は解放されます。

# 演習

## 概要

NFSサーバを構築します。
具体的な流れは次の通りです。

> :information_source:  
> この演習はAWS EKSでの実施を前提としています。
> AWS以外のクラウドを使用する際は、内容を適宜読み替えてください。

1. NFSサーバ(コンテナ)にマウントするPVCを作成する
1. NFSサーバを作成する(Deployment/Service)

## 問題

1. 以下を満たすマニフェストを作成しデプロイしてください。PVCは[公式ドキュメント][1]を参考にしてください。

   - 要件
     - PersistentVolumeClaim
       - 名前は`nfs-pvc`
       - storageClassNameは`指定しない`（デフォルトのStorageClassを使用する）
       - accessModesは`ReadWriteOnce`
       - ストレージ容量は`1Gi`

1. PVCリソースのオブジェクト一覧を確認してください。

1. 以下を満たすマニフェストを作成しデプロイしてください。

   - 要件
     - Deployment
       - 名前は`nfs-server`
       - replicas: `1`
       - labelはすべて`role: nfs-server`
       - Pod
         - イメージは`k8s.gcr.io/volume-nfs:0.8`
         - volumeプラグインで上記PVC:`nfs-server-pvc`を指定
         - 上記で定義したボリュームをコンテナの`/exports`にマウント

1. PVCリソースのオブジェクト一覧を確認してください。

1. PVリソースのオブジェクト一覧を確認してください。

1. AWSのマネジメントコンソールなどでEBSボリュームを確認し「kubernetes-dynamic-pvc-」から始まる名前のボリュームが作成されていることを確認してください。  
   (dvpによってEBSボリュームが自動作成されることを確認します)

1. Deployment, Podのオブジェクト一覧を表示し、`nfs-server`を含んだDeployment, Podが作成されていることを確認してください。

1. 作成したリソースを削除してください。

以上で本演習は終了です。

具体的な操作およびその結果に関する回答例は[こちら](../ans/PersistentVolume_answer.md)にあります。
具体的な操作方法がわからなかった場合や、想定した結果にならなかった場合などに参照してください。

[1]:https://kubernetes.io/docs/concepts/storage/persistent-volumes/#persistentvolumeclaims

---

次： [Service-LB](Service-LB.md)  
