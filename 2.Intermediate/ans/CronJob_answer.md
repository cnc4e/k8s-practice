# 回答例

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

   【回答例】

   ```yml
   # manifest
   apiVersion: v1
   kind: ConfigMap
   metadata:
     name: backup.sh
   data:
     backup.sh: | 
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
   ---
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: nginx
   spec:
     replicas: 1
     selector:
       matchLabels:
         app: nginx
     template:
       metadata:
         labels:
           app: nginx
       spec:
         containers:
           - name: nginx
             image: nginx:1.12
             ports:
               - containerPort: 80
   ---
   apiVersion: v1
   kind: Service
   metadata:
     name: nginx-svc
   spec:
     ports:
      - name: "http-port"
        protocol: "TCP"
        port: 80
        targetPort: 80
     selector:
       app: nginx
   ---
   apiVersion: batch/v1
   kind: CronJob
   metadata:
     name: backup
   spec:
     schedule: "* * * * *"
     jobTemplate:
       spec:
         template:
           spec:
             containers:
             - name: backup
               image: appropriate/curl
               command: ["/bin/sh",  "-c", "sh /tmp/backup.sh"]
               volumeMounts:
                 - name: backup-sh
                   mountPath: /tmp
               env:
               - name: DEST
                 value: "http://nginx-svc"
             restartPolicy: Never
             volumes:
               - name: backup-sh
                 configMap:
                   name: backup.sh
                   items:
                     - key: backup.sh
                       path: backup.sh
     successfulJobsHistoryLimit: 2
   ```

   ```bash
   # 実行結果
   $ kubectl apply -f cronjob.yaml
   configmap/backup.sh created
   deployment.apps/nginx created
   service/nginx-svc created
   cronjob.batch/backup created
   ```

1. デプロイしてから2分くらい待ってからCronJob、Job、Podリソースのオブジェクト一覧を表示してください。STATUS:CompletedのPodが2つあり、起動時間が約1分ずれていること。

   【回答例】

   ```bash
   # 実行結果
   $ kubectl get cronjob,job,pod
   NAME                   SCHEDULE    SUSPEND   ACTIVE   LAST SCHEDULE   AGE
   cronjob.batch/backup   * * * * *   False     0        11s             92s

   NAME                        COMPLETIONS   DURATION   AGE
   job.batch/backup-27990474   1/1           4s         72s
   job.batch/backup-27990475   1/1           4s         12s

   NAME                        READY   STATUS      RESTARTS   AGE
   pod/backup-27990474-sq5s6   0/1     Completed   0          72s
   pod/backup-27990475-7rrss   0/1     Completed   0          12s
   pod/nginx-f77774fc5-6xb9p   1/1     Running     0          94s
   ```

1. さらにもう1分以上してからPodリソースのオブジェクト一覧を表示してください。先程とは違うPod名だがSTATUS:CompletedのPodが2つあり、起動時間が約1分ずれていることを確認してください。（タイミングによっては3つ表示されるかもしれません。その場合は再度確認すれば2つになっているはずです。）

   【回答例】

   ```bash
   # 実行結果
   $ kubectl get cronjob,job,pod
   NAME                   SCHEDULE    SUSPEND   ACTIVE   LAST SCHEDULE   AGE
   cronjob.batch/backup   * * * * *   False     0        12s             2m33s

   NAME                        COMPLETIONS   DURATION   AGE
   job.batch/backup-27990475   1/1           4s         73s
   job.batch/backup-27990476   1/1           4s         13s

   NAME                        READY   STATUS      RESTARTS   AGE
   pod/backup-27990475-7rrss   0/1     Completed   0          73s
   pod/backup-27990476-qlfj7   0/1     Completed   0          13s
   pod/nginx-f77774fc5-6xb9p   1/1     Running     0          2m35s
   ```

1. 作成したリソースを削除してください。

   【回答例】

   ```bash
   # 実行結果
   $ kubectl delete -f cronjob.yaml
   configmap "backup.sh" deleted
   deployment.apps "nginx" deleted
   service "nginx-svc" deleted
   cronjob.batch "backup" deleted
   ```

[1]:https://kubernetes.io/docs/tasks/job/automated-tasks-with-cron-jobs/
