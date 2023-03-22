前： [StatefulSet](StatefulSet.md)  

---

# StatefulSet volumeClaimTemplate

## PVC指定の場合

volumeClaimTemplateはPodごとにPVCおよびPVを作成する、StatefulSetで使える機能です。
この機能を理解するため、まずはDeploymentでのボリュームプロビジョニングの特徴をおさらいします。

> :information_source:  
> この演習はAWS EKSでの実施を前提としています。
> AWS以外のクラウドを使用する際は、内容を適宜読み替えてください。

> :information_source:  
> 本演習を実施するにあたり、ワーカーノードを2台以上にしてください。

1. 以下を満たすマニフェストを作成しデプロイしてください。

   - 要件
     - Deployment
       - 名前は`nginx-dp`
       - labelはすべて`app: nginx-dp`
       - replicasは `3`
       - Pod
         - 名前は`nginx`
         - イメージは`nginx:1.12`
         - volumeプラグインでPVC:nginx-dp-pvcを指定
         - 上記で定義したボリュームをコンテナの`/usr/share/nginx/html`にマウント
         - lifecycleの`postStart`で`["/bin/sh","-c","echo $HOSTNAME >> /usr/share/nginx/html/index.html"]`を実行
     - PVC
       - 名前は`nginx-dp-pvc`
       - storageClassNameは`指定しない`（デフォルトのStorageClassを使用する）
       - accessModesは`ReadWriteOnce`
       - ストレージ容量は`1Gi`

1. PVCおよびPVリソースのオブジェクト一覧を表示し、`nginx-dp-pvc`を含むものがそれぞれ`1つ`作成されていることを確認してください。

1. Podリソースのオブジェクト一覧を表示し、3つのPodの名前および稼働するワーカーノードを確認してください。

1. いずれか1つのPodにログインし、次のコマンドを発行してください。

   ```bash
   echo $HOSTNAME > /usr/share/nginx/html/index.html
   cat /usr/share/nginx/html/index.html
   ```

1. 先ほどとは別のPodにログインし、次のコマンドを発行してください。表示されるホスト名が`最初にログインしたPod名が表示される`ことを確認してください。

   ```bash
   cat /usr/share/nginx/html/index.html
   ```

1. Deploymet:nginx-dp、PVC:nginx-dp-pvcを削除してください。

   【回答例】

   ```bash
   $ kubectl delete -f nginx-dp.yaml
   deployment.apps "nginx-dp" deleted
   persistentvolumeclaim "nginx-dp-pvc" deleted
   ```

ここまでがDeploymentでのボリュームプロビジョニングのおさらいです。
注目するポイントとしては展開したPodすべてで同じボリュームを共有する点です。
そのため、PVCおよびPVは1つしか作られません。ボリュームの中身もすべてのPodで共有します。
ですので、例えばWEBサーバなどのワークロードでセッション情報をPod間で共有したいなどと言った場合にはこの構成が有効です。
一方で、DBなどのワークロードで各Podが専用のボリュームを確保したい場合には適しません。
また、今回はPVCでstorageClassNameを指定しなかったためデフォルトのStorageClassであるEBSのtype:gp2(EKSの場合)で実際のボリュームは作られています。
EBSは単一のEC2インスタンスにしかボリュームを提供できません。
そのため、replica数が増えても単一のワーカーノード（EC2）にしかPodがスケジュールできません。複数のワーカーノードに分散させるには、EBSではなくEFSなどRead/Write Anyできるボリュームが使えるStorageClassを使用する必要があります。

## volumeClaimTemplateの場合

つぎに、StatefulSetでvolumeClaimTemplateを使用した場合の挙動を確認します。

1. 以下を満たすマニフェストを作成しデプロイしてください。volumeClaimTemplateについては[公式ドキュメント][1]を参考にしてください。

   - 要件
     - StatefulSet
       - 名前は`nginx-vct-sts`
       - labelはすべて`app: nginx`
       - replicasは `3`
       - serviceNameは`nginx-vct-svc`
       - Pod
         - 名前は`nginx-vct`
         - イメージは`nginx:1.12`
         - volumeプラグインでPVC:nginx-vct-pvcを指定
         - 上記で定義したボリュームをコンテナの`/usr/share/nginx/html`にマウント
         - lifecycleの`postStart`で`["/bin/sh","-c","echo $HOSTNAME >> /usr/share/nginx/html/index.html"]`を実行
       - volumeClaimTemplates
         - 名前はindex
         - storageClassNameは`指定しない`（デフォルトのStorageClassを使用する）
         - accessModesは`ReadWriteOnce`
         - ストレージ容量は`1Gi`

1. PVCおよびPVリソースのオブジェクト一覧を表示し、それぞれ`3つ`作成されていることを確認してください。

1. Podリソースのオブジェクト一覧を表示し、3つのPodの名前および稼働するワーカーノードを確認してください

1. 1つのPodにログインし、次のコマンドを発行してください。

   ```bash
   echo $HOSTNAME > /usr/share/nginx/html/index.html
   cat /usr/share/nginx/html/index.html
   ```

1. 先ほどとは別のPodにログインし、次のコマンドを発行してください。index.htmlの内容が`異なる`ことを確認してください。

   ```bash
   cat /usr/share/nginx/html/index.html
   ```

1. StatefulSet:nginx-vct-stsを削除してください。

1. PVCおよびPVリソースのオブジェクト一覧を表示し、`削除されていない`ことを確認してください。

1. PVCおよびPVリソースを削除してください。

以上がvolumeClaimTemplateを使用したボリュームプロビジョニングです。
Deploymentの時とは違い、各PodそれぞれにPVCおよびPVが作成されました。
また、StorageClassはデフォルトのEBSですが、Podのスケジュール先も分散されました。
この様にPodごとに専用のボリュームを確保したい時にvolumeClaimTemplateは有効です。
また、Podがセルフ・ヒーリングで再作成された場合はそのPodに対応したボリュームが引き続き使用されるます。
なお、volumeClaimTemplateで作成されたPVCおよびPVはStatefulSetを削除しても`消えません`。（データを残すためにあえてこういう仕様になっていると思われます。）
この特性を利用してreplica1でもあえてStatefulSetを使用することもありますが、ボリュームが不要になった時は手動で消すのを忘れないようにしましょう。

なお、StatefulSetでもDeploymentと同じようにPVCを指定すれば1つのボリュームを複数Podで共有することもできます。

以上で本演習は終了です。

具体的な操作およびその結果に関する回答例は[こちら](../ans/StatefulSet-volumeClaimTemplate_answer.md)にあります。
具体的な操作方法がわからなかった場合や、想定した結果にならなかった場合などに参照してください。

[1]:https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/#components

---

次： [ConfigMap(env)](ConfigMap-env.md)  
