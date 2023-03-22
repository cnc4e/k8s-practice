前： [Job](Job.md)  

---

# CronJob

CronJobはJobを定期的に自動実行するリソースです。時間はCronと同じ形式で指定します。

# 演習

1. 以下を満たすマニフェストを作成しデプロイしてください。CronJobリソースについては[公式ドキュメント][1]を参考にしてください。

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
     - CronJob
       - スケジュールは1分間隔
       - コンテナイメージはcurlとshが使用可能なものなら何でも良い
       - 上記ConfigMapを適当な場所にマウント
       - 環境変数DESTに上記Serviceの名前を設定
       - commandでマウントしたbackup.shを実行
       - 成功したPodの保持数を2にしてください。

1. デプロイしてから2分くらい待ってからCronJob、Job、Podリソースのオブジェクト一覧を表示してください。STATUS:CompletedのPodが2つあり、起動時間が約1分ずれていること。

1. さらにもう1分以上してからPodリソースのオブジェクト一覧を表示してください。先程とは違うPod名だがSTATUS:CompletedのPodが2つあり、起動時間が約1分ずれていることを確認してください。（タイミングによっては3つ表示されるかもしれません。その場合は再度確認すれば2つになっているはずです。）

1. 作成したリソースを削除してください。

以上で本演習は終了です。

具体的な操作およびその結果に関する回答例は[こちら](../ans/CronJob_answer.md)にあります。
具体的な操作方法がわからなかった場合や、想定した結果にならなかった場合などに参照してください。

[1]:https://kubernetes.io/docs/tasks/job/automated-tasks-with-cron-jobs/

---

次： [LimitRange](LimitRange.md)  
