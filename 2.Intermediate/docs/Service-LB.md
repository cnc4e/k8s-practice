前： [PersistentVolume](PersistentVolume.md)

---

# Service LoadBalancer

今までの内容ではPodへのアクセスをK8sクラスタ内部で行っていました。
次はK8sクラスタ外部からアクセスしたいと思います。
K8sクラスタ外部からアクセスを受けられるようにするにはLoadBalancerタイプのServiceを作成するのがもっとも手っ取り早いです。
ただし、LoadBalancerタイプのServiceはAWSなどの対応したクラウドプロバイダで動いている場合のみ利用可能です。

また、LoadBalancerタイプのServiceは1つのServiceに1つのLoadBalancer(AWSだとELB)を作成するため、たくさん外部公開したいときは[Ingress](../../3.Advanced/3-06.Ingress.md)などの利用を検討しましょう。

# 演習

1. 以下を満たすDeployment, Serviceをデプロイしてください。Service Type:LoadBalancerについては[公式ドキュメント][1]を参考にしてください。

   - 要件
     - Deployment
       - 名前は`nginx`
       - replicas: `1`
       - labelはすべて`app: nginx`
       - Pod
         - 名前は`nginx`
         - イメージは`nginx:1.12`
     - Service
       - 名前は`nginx-lb`
       - Namespaceは`web`
       - 対象のlabelは`app: nginx`
       - プロトコルは`TCP`
       - Portは`80`
       - clusterIPは`指定なし`で良い
       - typeは`LoadBalancer`

1. Serviceリソースの一覧を表示しデプロイしたnginx-lbの`EXTERNAL-IP`を確認してください。（あとで使うのでメモしておく）

1. インターネット接続可能な端末のwebブラウザからさきほど確認したnginx-lbのEXTERNAL-IPにアクセスしてください。  
  （AWSの場合、ELBが使用可能になるまで2分くらいかかる。最初はエラーになるので何度かアクセスしてみる。最長でも5分くらいすればアクセス可能になる。）

1. AWSマネジメントコンソールなどでELBを確認し、EXTERNAL-IPと同じDNS名を持つclassicのLBがデプロイされていることを確認してください。また、LBにアタッチされたsecurity group名を確認してください。

1. AWSマネジメントコンソールなどでK8sのワーカーにアタッチされているsecurity groupを確認し、LBにアタッチされたsecurity groupからのインバウンドが許可されていることを確認してください。

1. curlを実行できるPodを展開し、Service:lb-svcに`ClusterIP`に対してcurlを実行し、アクセスできることを確認してください。  

1. 作成したリソースを削除してください。

1. AWSマネジメントコンソールなどでELBおよびワーカーノードのsecurity groupを確認し、Type:LoadBalancerの設定が消えていることを確認してください。

以上で本演習は終了です。

具体的な操作およびその結果に関する回答例は[こちら](../ans/Service-LB_answer.md)にあります。
具体的な操作方法がわからなかった場合や、想定した結果にならなかった場合などに参照してください。

[1]:https://kubernetes.io/docs/concepts/services-networking/service/#loadbalancer

---

次： [Pod-resources](Pod-resources.md)  
