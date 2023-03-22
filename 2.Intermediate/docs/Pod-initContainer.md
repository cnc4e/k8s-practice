前： [Pod-Probe](Pod-Probe.md)  

---

# Pod設定 initContainer

Pod内のメインコンテナを起動する前に一時的なコンテナをPod内で起動できます。
例えばメインコンテナ起動前の準備を実装したりできます。ポイントとしてはinitContainerで起動するコンテナはあくまでもメインコンテナ起動前の一時的なものであることです。
そのため、initContainerで指定するcommandはexit0で終わるものを指定しましょう。

# 演習

1. 以下を満たすDeployment, Serviceをデプロイしてください。

   - 要件
     - Deployment
       - 名前は`nginx`
       - replicas: `1`
       - labelはすべて`app: nginx`
       - Pod
         - 名前は`nginx`
         - イメージは`nginx:1.12`
     - Service
       - 名前は`nginx-svc`
       - 対象のlabelは`app: nginx`
       - プロトコルは`TCP`
       - Portは`80`
       - targetPortは`80`

1. 接続確認に使用するcurlコマンドが実行可能なtestpodを作成し、testpodから`Service:nginx-svc`に対して接続確認をしてください。  
   （`Nginxデフォルトのindex.htmlが表示されること`）

1. マニフェストを以下の内容に修正し、デプロイしてください。なお、initContainerについては[公式ドキュメント][1]を参考にしてください。

   - 要件
     - Deployment
       - Pod
         - nginx
           - volume:index-htmlを`/usr/share/nginx/html`にマウント
         - initContainer
           - 名前`init`
           - イメージは`busybox`
           - commandは`['/bin/sh','-c','echo initContainer de settei sita noda > /tmp/   index.html']`
           - volume:index-htmlを`/tmp`にマウント
         - volume
           - 名前は`index-html`
           - volumeプラグインは`emptyDir`を指定

1. Podの詳細を確認し、init Containers の箇所を確認してください。また、eventを確認しコンテナinitがメインコンテナの前に起動したことを確認してください。

1. testpodから`Service:nginx-service/index.html`に対して接続確認をしてください。（`initContainerで修正したindex.htmlが表示されること`）

1. 作成したリソースを削除してください。

以上で本演習は終了です。

具体的な操作およびその結果に関する回答例は[こちら](../ans/Pod-initContainer_answer.md)にあります。
具体的な操作方法がわからなかった場合や、想定した結果にならなかった場合などに参照してください。

[1]:https://kubernetes.io/docs/concepts/workloads/pods/init-containers/#init-containers-in-use

---

次： [Pod-nodeSelector](Pod-nodeSelector.md)  
