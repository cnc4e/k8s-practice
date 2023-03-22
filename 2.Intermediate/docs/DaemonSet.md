前： [Namespace](Namespace.md)  

---

# DaemonSet

DaemonSetは常にすべてのワーカーノードで同じPodを起動するPod Controllerです。クラスタ管理系機能（ログやメトリクスの収集）によく使われます。

1. クラスタの参加ノードを確認し、ワーカーノードが2台以上の状態にしてください。

   【回答例】

1. 以下を満たすマニフェストを作成しデプロイしてください。なお、DaemonSetについては[公式ドキュメント][1]を参考にしてください。

   - 要件
     - DaemonSet
       - 名前は`daemon`
       - labelはすべて`app: nginx`
       - Pod
         - メインコンテナ
           - 名前は`nginx`
           - イメージは`nginx:1.12`

1. DaemonSetリソースのオブジェクト一覧を表示し、`daemon`があることを確認してください。

1. Podリソースのオブジェクト一覧を起動ノード名も一緒に表示し、すべてのワーカーノードで`daemon`Podが起動していることを確認してください。

1. `daemon`Podを1つ削除してからPodリソースのオブジェクト一覧を表示し、セルフヒーリングでPodが再作成されていることを確認してください。

1. 作成したリソースを削除してください。

以上で本演習は終了です。

具体的な操作およびその結果に関する回答例は[こちら](../ans/DaemonSet_answer.md)にあります。
具体的な操作方法がわからなかった場合や、想定した結果にならなかった場合などに参照してください。

[1]:https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/

---

次： [StatefulSet](StatefulSet.md)  
