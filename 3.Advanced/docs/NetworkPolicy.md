前： [Ingress](Ingress.md)  

---

# NetworkPolicy

NetworkPolicyはK8sクラスタ内で使えるセキュリティグループの様なものです。
指定したラベルが付与されたPodやNamespaceに対するインバウンドおよびアウトバウンドの通信制限を設けることができます。
デフォルトの状態では同一クラスタ内のPodはNamespaceが違っても制限なしに相互アクセス可能です。
例えばNamespaceで顧客ごとに分割をしていても、ネットワークアクセス可能だとあまり意味がありません。
その様な場合にNetworkPolicyでアクセスを制限すればよりセキュアな構成にできます。

なお、NetworkPolicyはK8sの標準リソースですが、使用するにはクラスタのネットワークが対応したCNIプラグインで構成されている必要があります。
例えばEKSの場合、デフォルトでは「Amazon VPC CNI plugin for Kubernetes」を使用していますが、2020/03時点で「Amazon VPC CNI plugin for Kubernetes」はNetworkPolicyに対応していません。
（たしかGKEとAKSは標準のCNIで対応しているはず。）
NetworkPolicyに対応したCNIプラグインとしてはCalicoなどがあります。

## Calicoのデプロイ

> :information_source:  
> 本手順は、EKS等、NetworkPolicyに対応していないCNIを使用している場合に実施してください。

Calicoをクラスタにデプロイします。[公式の手順][1]を参考に作業します。

1. 以下のコマンドを実行してください。

   ``` sh
   kubectl delete daemonset -n kube-system aws-node
   ```

1. 以下のコマンドを実行してください。

   ``` sh
   kubectl apply -f https://raw.githubusercontent.com/projectcalico/calico/v3.25.0/manifests/calico-vxlan.yaml
   ```

1. 以下のコマンドを実行してください。

   ``` sh
   kubectl -n kube-system set env daemonset/calico-node FELIX_AWSSRCDSTCHECK=Disable
   ```

1. ワーカーノードを再作成してください。

## NetworkPolicy(Namespace単位)

続いてNamespace単位のNetworkPolicyについて確認する。

1. 以下を満たすマニフェストを作成しデプロイしてください。

   - 要件
     - 1セット目
       - Namespace
         - 1セット目とわかる名前にする。
         - 1セット目とわかるラベルをつける
       - Deployment
         - １つ目
           - namespaceは1セット目
           - 1つ目のDeployment固有のラベルをつける
           - nginx:1.12のコンテナ
           - /usr/share/nginx/html/index.htmlの内容を1つ目のアプリケーションであることがわかる内容に書き換える
         - 2つ目
           - namespaceは1セット目
           - 2つ目のDeployment固有のラベルをつける
           - curlが実行できるコンテナ
       - Service
         - namespaceは1セット目
         - 上記1つ目のDeploymentをClusterIPのPort:80で公開
     - 2セット目
       - 1セット目と基本同じでNamespaceの名前、ラベルとindex.htmlの内容を変える

1. 各NamespaceのcurlできるPodから各NamespaceのService:nginxに`通信可能`であることを確認してください。（計4回curlします。他Namespaceへcurlする場合、<Service名>.<Namespace名>でアクセス可能です。）

1. 以下を満たすマニフェストを作成しデプロイしてください。NetworkPolicyリソースについては[公式ドキュメント][2]を参考にしてください。

   - 要件
      - 1セット目
        - NetworkPolicy
          - 対象は同じnamespaceのPodすべて
          - 1セット目のNamespaceからのインバウンド通信を許可する。（言い方を変えると1セット目のNamespace以外からの通信を許可しない）
      - 2セット目
        - 1セット目と基本同じで2セット目のNamespaceからのインバウンド通信のみを許可する。

1. 各NamespaceのPod:curlから各NamespaceのService:nginxに`curl -s -m 10`で通信してください。-mはタイムアウト値(秒)を指定するオプションです。  
   Namespaceを跨いだ通信は失敗することを確認してください。

1. 作成した`NetworkPolicy`を削除してください。

## NetworkPolicy(Pod単位)

上記のようにNetworkPolicyで通信制御をNamespaceにかけることができる。さらに、同じNamespace内でもPodごとに通信制御をかけることもできます。

1. 以下を満たすマニフェストを作成しデプロイしてください。

   - 要件
      - Deployment
        - 3つ目
          - 3つ目のDeployment固有のラベルをつける
          - curlが実行できるコンテナ
      - NetworkPolicy
        - 対象は1つ目のDeployment
        - 3つ目のDeploymentのみ通信を許可する。

1. 2つ目と3つ目のDeploymentでデプロイしたPodからService:nginxに対して通信してください。3つ目でデプロイしたPodからのみアクセスできることを確認してください。（もしできてしまう場合、同一Namespaceからの通信を許可するNetworkPolicyが残っていないか確認し、残っていれば削除する）

1. 作成したリソースを削除してください。

具体的な操作およびその結果に関する回答例は[こちら](../ans/NetworkPolicy_answer.md)にあります。
具体的な操作方法がわからなかった場合や、想定した結果にならなかった場合などに参照してください。

[1]:https://docs.tigera.io/calico/latest/getting-started/kubernetes/managed-public-cloud/eks
[2]:https://kubernetes.io/docs/concepts/services-networking/network-policies/

---

次： [StorageClass](StorageClass.md)  
