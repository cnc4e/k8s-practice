前： [StorageClass](StorageClass.md)  

---

# Kustomize

Kustomizeはマニフェスト管理に役立つコマンドラインツールです。
v1.14のkubectlからは標準で使用できます。K8sを複数の環境（開発/ステージング/本番等）で使用する場合、環境間では差異が生じると思います。
（環境変数のドメイン名が違う。HWリソース量が違う。など）
差異がある場合、各環境ごとにマニフェストを用意することになりますが、共通箇所に修正が発生した場合にすべての環境のマニフェストをいちいち修正するのは大変です。
そこでKustomizeを使用すると、ベースとなるマニフェストを作成しておき、環境差異が生じる部分だけ各環境で上書きしてマニフェストを生成するということができます。

# 演習

1. 以下構造のKustomizeディレクトリを作成してください。

   ```bash
   Kustomize
   ├── base
   └── overlay
       ├── prod
       └── dev
   ```

1. kustomize/base配下に以下を満たすマニフェストを作成してください。（applyは不要です。）

   - 要件
     - ConfigMap
       - 名前はenv-config
       - dataはENV: base
     - Deployment
       - replica:1
       - コンテナイメージはnginx:1.12
       - ConfigMap:env-configのENVを環境変数ENVに格納
     - Service
       - 上記DeploymentをClusterIP:80で公開

1. 上記マニフェストをresourcesとして指定したkustomize/base/kustomization.yamlを作成してください。kustomizeについては[kustomize公式ドキュメント][1]や[k8s公式ドキュメント][2]を参考にしてください。

   > :information_source:  
   > 公式ドキュメントでは正直理解しにくいため、インターネットで検索して、調べたほうが早いかもしれません。

1. 以下のコマンドでkustomizeでapplyしてください。（以下コマンドはkustomizeディレクトリで実行した場合）

   ```bash
   kubectl apply -k base/
   ```

1. 作成したオブジェクトを確認し、以下コマンドで削除してください。（以下コマンドはkustomizeディレクトリで実行した場合）

   ```bash
   kubectl delete -k base/
   ```

1. overlay/dev/およびoverlay/prod/に以下を満たすマニフェストおよびkustomization.yamlを作成してください。

   - 要件
     - prod
       - baseは../../base
       - 作成するオブジェクトのプレフィックスに`prod-`をつける
       - 作成するオブジェクトに`env: prod`のラベルを追加
       - ConfigMap:env-configを上書き（patches）し`ENV: prod`に変更してください。
     - dev
       - baseは../../base
       - 作成するオブジェクトのプレフィックスに`dev-`をつける
       - 作成するオブジェクトに`env: dev`のラベルを追加
       - ConfigMap:env-configを上書き（patches）し`ENV: dev`に変更してください。

1. 以下のコマンドでprodとdevをデプロイしてください。（以下コマンドはkustomizeディレクトリで実行した場合）

   ``` sh
   kubectl apply -k overlay/prod
   kubectl apply -k overlay/dev
   ```

1. 作成したオブジェクトを確認してください。prodおよびdevのPodに対して以下の追加コマンドを発行し、環境変数がオーバーライドできていることを確認してください。

   ```bash
   echo $ENV
   ```

1. deploymentを以下の様にオーバーライドして再デプロイしてください。なお、オーバーライドする時のマニフェストは必要最小限の記載にすることに注意してください。

   - 要件
     - prod
       - replicas: 3
       - resources.requests.memory: 100Mi
     - dev
       - replicas: 2
       - resources.requests.memory: 50Mi

1. 上記オーバーライドした内容が反映されていることを確認してください。

1. Podの/usr/share/nginx/html/index.htmlの内容を環境変数ENVの値とし、各環境で表示を変えます。kustomize/base配下のみを修正し、実装してください。

1. curlが実行可能なPodを展開し、prodとdevそれぞれにcurlしてください。各環境名が表示されること

1. 作成したリソースを削除してください。

以上で本演習は終了です。
具体的な操作およびその結果に関する回答例は[こちら](../ans/Kustomize_answer.md)にあります。
具体的な操作方法がわからなかった場合や、想定した結果にならなかった場合などに参照してください。

[1]:https://github.com/kubernetes-sigs/kustomize
[2]:https://kubernetes.io/docs/tasks/manage-kubernetes-objects/kustomization/

---

次： -  
