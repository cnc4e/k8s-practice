前： [DaemonSet](DaemonSet.md)  

---

# StatefulSetとHeadless Service

## StatefulSet

StatefulSetは[Deployment（ReplicaSet）](../../1.Beginner/docs/Deployment.md)と同じく指定した数のPodを常に起動するようにするAPIリソースです。  
違いは以下の3つです。わかりやすい1つ目と2つ目の動作をまず確認します。3つ目は長くなるので次章で解説します。

- 起動するPod名に連番が付与される（Deploymentはランダム文字列）
- 名前の`連番通り`にPodを1つずつ起動し、前のPodが起動完了してから次のPodを起動する。（Deploymentは起動順を気にしない）
- 各PodごとにPVを割り当てるvolumeClaimTemplateが使える（Deploymentでは使えない）

StatefulSetはDBなどの`ステートフル`なワークロードを想定したリソースです。
たとえば3ノードのDBだとMaster:1,Slave:2の構成などにすると思います。
この際、Masterをまず起動し、次いでSlaveを起動します。
このような順番を意識した起動はDeploymentではできないため、StatefulSetを使います。
またHeadless Serviceを使った特定Podへのアクセスを行うこともあります。

# 演習

1. 以下を満たすマニフェストを作成しデプロイしてください。なお、StatefulSetについては[公式ドキュメント][1]を参考にしてください。

   - 要件
     - StatefulSet
       - 名前は`nginx-sts`
       - labelはすべて`app: nginx`
       - replicasは `3`
       - serviceNameは`nginx-svc`
       - Pod
         - 名前は`nginx`
         - イメージは`nginx:1.12`
     - Service
       - 名前は`nginx-svc`
       - typeは`ClusterIP`（`明示的に指定する`）
       - clusterIPに`None`
       - Protocolは`TCP`
       - 待ち受けポートは`80`
       - ターゲットポートは`80`
       - selectorは`app: nginx-sts`

1. StatefulSetリソースのオブジェクト一覧を表示し、`stateful`があることを確認してください。

1. Podリソースのオブジェクト一覧を表示し、`stateful-0,1,2`の３つのPodがあることを確認してください。また起動時間から起動が順番に行われたことを確認してください。

1. Serviceリソースのオブジェクト一覧を表示し、`nginx-svc`があること。また、`Headless Service`であることを確認してください。  
  （Headless ServiceはCluster-IPがNoneなServiceです。）

1. 接続確認に使用するcurlコマンドが実行可能なtestpodを作成し、testpodから次のコマンドで接続確認をしてください。

   ```bash
   nslookup nginx-svc
   curl -s nginx-sts-0.nginx-svc.default.svc.cluster.local
   curl -s nginx-sts-1.nginx-svc.default.svc.cluster.local
   curl -s nginx-sts-2.nginx-svc.default.svc.cluster.local
   ```

1. Pod:`nginx-sts-1`のIPアドレスを確認してからPod:`nginx-sts-1`を削除してください。

1. Pod:`nginx-sts-1`がセルフ・ヒーリングされていることを確認してください。また、IPアドレスが変わっていることを確認してください。

1. testpodから次のコマンドで接続確認をしてください。

   ```bash
   nslookup nginx-svc
   curl -s nginx-sts-1.nginx-svc.default.svc.cluster.local
   ```

1. 作成したリソースを削除してください。

上記の様に、Headless Serviceを使うことでPod IPアドレスが変わっても任意のPodにアクセスすることができます。単純に特定PodにアクセスするだけならPodのIPアドレスでも可能ですが、StatefulSetで展開しているPodでもセルフヒーリングで再作成されるとIPアドレスは変わってしまいます。Headless Serviceであればセルフヒーリング後も自動でPodを検出するため、宛先の指定を変える必要がありません。

とはいえ、Headless Serviceは少しイレギュラーなServiceです。replicas:2以上のStatefulSet以外で使うことは稀だと思います。replicas:1のStatefulSetやDeploymentの場合は通常のServiceを使うことがほとんどだと思います。

以上で本演習は終了です。

具体的な操作およびその結果に関する回答例は[こちら](../ans/StatefulSet_answer.md)にあります。
具体的な操作方法がわからなかった場合や、想定した結果にならなかった場合などに参照してください。

[1]:https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/

---

次： [StatefulSet-volumeClaimTemplate](StatefulSet-volumeClaimTemplate.md)  
