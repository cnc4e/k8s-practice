前： [Pod-nodeSelector](Pod-nodeSelector.md)  

---

# Pod設定 postStart/preStop

Pod内のコンテナ起動後とコンテナ停止前に任意のコマンドを実行できます。
その設定がlifecycleのpostStartとpreStopです。とくにpreStopはコンテナ(Pod)をいつでも安全な停止ができるようにするため重要です。
Podを手動で停止(kubectl delete)すると、SIGTARMが送られ30秒の猶予期間後にSIGKILLでプロセス強制終了となります。
SIGTARMやSIGKILLで終了したくないコンテナにはpreStopでプロセスの終了コマンドを実行しましょう。
また、postStartはコンテナ起動時に実行されるコマンドですが、プロセス起動前の実行は保証されません。
そのため、プロセス起動前に行いたい処理はinitContainerやコンテナ起動コマンドに埋め込むなどしてください。

1. 以下を満たすDeploymentをデプロイしてください。なお、lifecycleについては[公式ドキュメント][1]を参考にしてください。

   - 要件
     - Deployment
       - 名前は`nginx`
       - replicas: `1`
       - labelはすべて`app: nginx`
       - Pod
         - 名前は`nginx`
         - lifecycleの`postStart`で`['/bin/sh','-c','date > /usr/share/nginx/html/index.html']`を実行
     - Service
       - 名前は`nginx-svc`
       - 対象のlabelは`app: nginx`
       - プロトコルは`TCP`
       - Portは`80`
       - targetPortは`80`

1. 接続確認に使用するcurlコマンドが実行可能なtestpodを作成し、testpodから`Service:nginx-svc`に対して接続確認をしてください。  
  （`時刻が１行表示されていること`）

1. Deployment:`nginx`のマニフェストを以下にように修正して再デプロイしてください。

   - 要件
     - Deployment
       - Pod
         - lifecycleの`preStop`で`['/bin/sh','-c','rm /usr/share/nginx/html/index.html']`を追加

1. Pod:`lifecycle`を何度か手動で削除しセルフ・ヒーリングさせる。（Deploymentは消さない）

1. Pod:`curl`から`lifecycle-svc`に対しcurlする。Podを作成した時刻が`一行のみ`出力されることを確認する。

1. 作成したリソースを削除してください。

以上で本演習は終了です。

具体的な操作およびその結果に関する回答例は[こちら](../ans/Pod-lifecycle_answer.md)にあります。
具体的な操作方法がわからなかった場合や、想定した結果にならなかった場合などに参照してください。

[1]:https://kubernetes.io/ja/docs/tasks/configure-pod-container/attach-handler-lifecycle-event/

---

次： [Namespace](Namespace.md)  
