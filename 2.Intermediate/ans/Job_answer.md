# 回答例

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
   kind: Job
   metadata:
     name: backup
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
   ```

   ```bash
   # 実行結果
   $ kubectl apply -f job.yaml
   configmap/backup.sh created
   deployment.apps/nginx created
   service/nginx-svc created
   job.batch/backup created
   ```

1. Podリソースのオブジェクト一覧を表示し、STATUS:CompletedになっているPodがあることを確認してください。

   【回答例】

   ```bash
   # 実行結果
   $ kubectl get pod
   NAME                    READY   STATUS      RESTARTS   AGE
   backup-6tzqp            0/1     Completed   0          4s
   nginx-f77774fc5-99wq6   1/1     Running     0          5s
   ```

1. STATUS:CompletedになっているPodのログを表示する。"Backup seikou!"が出力されていることを確認してください。

   【回答例】

   ```bash
   # 実行結果
   $ kubectl logs backup-6tzqp
   tensai teki na Backup Script.
   imakara http://nginx-svc no Backup wo jikkou suru!
   Backup seikou!
   ```

1. Jobリソースのオブジェクト一覧を表示し、COMPLETIONSが1/1になっていることを確認してください。

   【回答例】

   ```bash
   # 実行結果
   $ kubectl get job
   NAME     COMPLETIONS   DURATION   AGE
   backup   1/1           4s         5m14s
   ```

1. Jobを削除してから以下の修正を加え、デプロイしてください。

   - 要件
     - Job
       - 環境変数DESTの値をService名でない値にする
       - job失敗時の再試行回数を3にする

   【回答例】

   ```bash
   # Jobを削除
   $ kubectl delete -f job.yaml
   configmap "backup.sh" deleted
   deployment.apps "nginx" deleted
   service "nginx-svc" deleted
   job.batch "backup" deleted
   ```

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
   kind: Job
   metadata:
     name: backup
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
             value: "http://localhost"
         restartPolicy: Never
         volumes:
           - name: backup-sh
             configMap:
               name: backup.sh
               items:
                 - key: backup.sh
                   path: backup.sh
     backoffLimit: 3
   ```

   ```bash
   # 実行結果
   $ kubectl apply -f job.yaml
   configmap/backup.sh created
   deployment.apps/nginx created
   service/nginx-svc created
   job.batch/backup created
   ```

1. 30秒ほどしてからPodリソースのオブジェクト一覧を表示し、STATUS:ErrorになっているPodが3つあることを確認してください。

   【回答例】

   ```bash
   # 実行結果
   $ kubectl get pod
   NAME                    READY   STATUS    RESTARTS   AGE
   backup-kc7lc            0/1     Error     0          54s
   backup-n8q77            0/1     Error     0          50s
   backup-x5znc            0/1     Error     0          40s
   nginx-f77774fc5-wfdtk   1/1     Running   0          55s
   ```

1. STATUS:ErrorになっているPod3つのログを表示する。3つとも"Backup sippai..."が表示されることを確認してください。

   【回答例】

   ```bash
   # 実行結果
   $ kubectl logs backup-kc7lc
   tensai teki na Backup Script.
   imakara http://localhost no Backup wo jikkou suru!
   Backup sippai...

   $ kubectl logs backup-n8q77
   tensai teki na Backup Script.
   imakara http://localhost no Backup wo jikkou suru!
   Backup sippai...

   $ kubectl logs backup-x5znc
   tensai teki na Backup Script.
   imakara http://localhost no Backup wo jikkou suru!
   Backup sippai...

   ```

1. Jobリソースのオブジェクト一覧を表示し、COMPLETIONSが0/1になっていることを確認してください。

   【回答例】

   ```bash
   # 実行結果
   $ kubectl get job
   NAME     COMPLETIONS   DURATION   AGE
   backup   0/1           2m41s      2m41s
   ```

1. 作成したリソースを削除してください。

   【回答例】

   ```bash
   # 実行結果
   $ kubectl delete -f job.yaml
   configmap "backup.sh" deleted
   deployment.apps "nginx" deleted
   service "nginx-svc" deleted
   job.batch "backup" deleted
   ```

[1]:https://kubernetes.io/docs/concepts/workloads/controllers/jobs-run-to-completion/
