前： [Secret](Secret.md)  

---

# Job

Deployment、DaemonSet、StatefulSetのPodコントローラは`Podの状態を維持すること`を目的としたPodコントローラです。
そのため、Pod（コンテナ）も動き続けるようにしたものを使います。
JobもPodコントローラの一種ですが、少し異なり、`実行させること`を目的としたPodコントローラです。
そのため、Pod（コンテナ）はexit 0で終了するようにしたものを使います。
例えばPod内の永続データのバックアップやアクセストークンの更新など、ある時にだけ必要となる処理を実行するのに使います。

# 演習

1. 以下を満たすマニフェストを作成しデプロイしてください。Jobリソースについては[公式ドキュメント][1]を参考にしてください。

   - 要件
     - ConfigMap
       - 以下の内容が記述されたbackup.shをdataとして持つ

         ```sh
         #!/bin/sh
         echo "tensai teki na Backup Script."
         echo "imakara $DEST no Backup wo jikkou suru!"

         # Backup command no tumori desu
         curl $DEST >& /dev/null

         if [ $? -eq 0 ];then
           echo "Backup seikou!"
           exit 0
         else
           echo "Backup sippai..."
           exit 1
         fi
         ```

     - Deployment
       - nginxのPodを実行
     - Service
       - 上記DeploymentをClusterIPのPort80で公開
     - Job
       - コンテナイメージはcurlとshが使用可能なものなら何でも良い
       - 上記ConfigMapを適当な場所にマウント
       - 環境変数DESTに上記Serviceの名前を設定
       - commandでマウントしたbackup.shを実行

1. Podリソースのオブジェクト一覧を表示し、STATUS:CompletedになっているPodがあることを確認してください。

1. STATUS:CompletedになっているPodのログを表示する。"Backup seikou!"が出力されていることを確認してください。

1. Jobリソースのオブジェクト一覧を表示し、COMPLETIONSが1/1になっていることを確認してください。

1. Jobを削除してから以下の修正を加え、デプロイしてください。

   - 要件
     - Job
       - 環境変数DESTの値をService名でない値にする
       - job失敗時の再試行回数を3にする

1. 30秒ほどしてからPodリソースのオブジェクト一覧を表示し、STATUS:ErrorになっているPodが3つあることを確認してください。

1. STATUS:ErrorになっているPod3つのログを表示する。3つとも"Backup sippai..."が表示されることを確認してください。

1. Jobリソースのオブジェクト一覧を表示し、COMPLETIONSが0/1になっていることを確認してください。

1. 作成したリソースを削除してください。

以上で本演習は終了です。

具体的な操作およびその結果に関する回答例は[こちら](../ans/Job_answer.md)にあります。
具体的な操作方法がわからなかった場合や、想定した結果にならなかった場合などに参照してください。

[1]:https://kubernetes.io/docs/concepts/workloads/controllers/jobs-run-to-completion/

---

次： [CronJob](CronJob.md)  
