前： [NetworkPolicy](NetworkPolicy.md)  

---

# StorageClass

以前、[PersistentVolume](../../2.Intermediate/docs/PersistentVolume.md)の章では、デフォルトのStorageClassを使用してDVPを行いました。
StorageClassはストレージの種類を定義したリソースです。EKSの場合はEBS(gp2)のStorageClassがデフォルトで定義されています。
EBS(gp2)以外のディスクタイプを使用してDVPを行うには追加でStorageClassを定義し、PVCでそのStorageClassを指定することで、使用可能です。

なお、StorageClassとして定義可能なストレージサービスには制限があります。
Provisionerというストレージサービスと連携をとるための機能が提供されているもののみになります。
K8sにはデフォルトでAWS EBSやAzure Diskなど代表的なストレージサービスのProvisionerが組み込まれています。
デフォルトで組み込み済のProvisionerは[公式ドキュメント][1]で確認できます。
組み込まれていないストレージサービス（たとえばAWS EFSなど）を使用してDVPを行いたい場合は、利用者でProvisionerを導入し、StorageClassを定義します。

# 演習

1. 以下を満たすマニフェストを作成しデプロイしてください。StorageClassリソースについては[公式ドキュメント][2]を参考にしてください。なお、デプロイする際は必ずStorageClassを最初にデプロイするように注意してください。

   - 要件
     - StorageClass
       - 名前はio1
       - ebsのprovisionerを使用
       - reclaimPolicyはDelete
     - Deployment
       - イメージは何でもよい
       - volumeMountsで2つのボリュームを別々の適当なpathにマウントしてください。
       - volumeで以下2つのpvcをそれぞれ指定してください。
         - normal-disk-pvc
         - fast-disk-pvc
     - PVC
       - normal-disk-pvc
         - storageClassNameはデフォルト
         - readwriteonce
         - 容量: 1G
       - fast-disk-pvc
         - storageClassNameはio1
         - readwriteonce
         - 容量: 4G

1. 作成したオブジェクトを確認してください。なお、StorageClassのgp2はデフォルトで作成されているものです。

1. マネジメントコンソールからEBSを確認し、gp2とio1のボリュームがあることを確認してください。

1. 作成したリソースを削除してください。

具体的な操作およびその結果に関する回答例は[こちら](../ans/StorageClass_answer.md)にあります。
具体的な操作方法がわからなかった場合や、想定した結果にならなかった場合などに参照してください。

[1]:https://kubernetes.io/docs/concepts/storage/storage-classes/#provisioner
[2]:https://kubernetes.io/docs/concepts/storage/storage-classes/

---

次： [Kustomize](Kustomize.md)  
