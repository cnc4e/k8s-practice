前： [postStart/preStop](Pod-lifecycle.md)  

---

# Namespaceとは

NamespaceはK8sクラスタ内に`仮想的なクラスタ分離機能`を提供します。

K8sクラスタの初期状態では、クラスタ上の全てのリソースは`誰でも`、`どのAPIリソース同士でも`参照・操作可能です。
Namespaceを分けることで、参照できるAPIリソースや操作できるAPIリソースを`ある程度`制御することが可能になります。
また別のK8sリソース（[ResourceQuota](ResourceQuota.md)、[NetworkPolicy](../../3.Advanced/docs/NetworkPolicy.md)）と合わせて使用することで、`Namespace単位で使用できるHWリソースの上限を設定`したり、`Namespace間の通信制御`を行うこともできます。

複数の顧客向けサービスの提供（マルチテナント）や複数チームでK8sクラスタを共有する時などに使用します。

初期状態では次のようなNamespaceが用意されています。
（K8sサービスによっては他のNamespaceがあるかもしれません）

- default
  - 実際にユーザが作成したPodやDeploymentなどが所属するデフォルトのNamespace
    - 作成時に明示的にNamespaceを指定しなかった場合、このNamespaceに所属する
  - 初期状態では何も配置されていない
- kube-system
  - 主にクラスタを構成するコンポーネントやK8s自身が作成したAPIリソースが配置される
- kube-public
  - ユーザ共通のConfigMapなどが配置される

全てのAPIリソースがNamespaceに所属するわけではなく、PersistentVolume(PV)など、Namespaceに所属しないAPIリソースも存在します。

# 演習

1. Namespaceリソースのオブジェクト一覧を表示してください。

1. Namespace:kube-systemに属するPodリソースのオブジェクト一覧を表示してください。  
   (-n <Namespace名> で参照するNamespaceを指定できます)

1. 以下を満たすマニフェストを作成しデプロイしてください。なお、Namespaceについては[公式ドキュメント][1]を参考にしてください。

   - 要件
     - Namespace
       - 名前は`web`
     - Deployment
       - 名前は`nginx`
       - Namespaceは`web`
       - replicas: `1`
       - labelはすべて`app: nginx`
       - Pod
         - 名前は`nginx`
     - Service
       - 名前は`nginx-svc`
       - Namespaceは`web`
       - 対象のlabelは`app: nginx`
       - プロトコルは`TCP`
       - Portは`80`
       - targetPortは`80`

1. Namespaceリソースのオブジェクト一覧を表示し、`web`が作成されていることを確認してください。

1. Namespace:`default`のDeployment, Pod, Serviceのオブジェクト一覧を表示し、上記手順で作成したAPIリソースが`表示されない`ことを確認してください。

1. Namespace:`web`のDeployment, Pod, Serviceのオブジェクト一覧を表示し、`nginx`を含んだDeployment, Pod, Serviceがあることを確認してください。

1. 接続確認に使用するcurlコマンドが実行可能なtestpodを`Namespace:default`に作成し、testpodから`Service:nginx-svc`に対して接続確認をしてください。  

1. testpodから`Namespace:webに所属するService:nginx-svc`に対して接続確認をしてください。  
   （別NamespaceのServiceを指定する場合は`<service名>.<namespace名>`と表記します。）

1. 接続確認に使用するcurlコマンドが実行可能なtestpodを`Namespace:web`に作成し、testpodから`Service:nginx-svc`に対して接続確認をしてください。

1. 作成したリソースを削除してください。

以上で本演習は終了です。

具体的な操作およびその結果に関する回答例は[こちら](../ans/Namespace_answer.md)にあります。
具体的な操作方法がわからなかった場合や、想定した結果にならなかった場合などに参照してください。

[1]:https://kubernetes.io/docs/tasks/administer-cluster/namespaces/#creating-a-new-namespace

---

次： [DaemonSet](DaemonSet.md)  
