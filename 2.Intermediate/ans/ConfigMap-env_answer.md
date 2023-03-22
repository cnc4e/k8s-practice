# 回答例

1. 以下を満たすマニフェストを作成しデプロイしてください。ConfigMapについては[公式ドキュメント][1]を参考にしてください。PodでConfigMapを環境変数として読み込む方法は[公式ドキュメント][2]を参考にしてください。

   - 要件
     - ConfigMap
       - 以下のKey-Valueをdataとして持つ
         - EP1: The Python Menace
         - EP2: Attack of the Clones
         - EP3: Revenge of the Sith
         - EP4: A New Hoge
         - EF5: The Empire Strikes Back
         - EP6: Return of the Jedi
         - EP7: The Force Awakens
         - EP8: The Last Jedi
         - EP9: The Rise of Kubernetes
     - Deployment(1つ目)
       - メインコンテナはshが使用できれば何でも良い
       - 上記ConfigMapのKey-Valueをすべて環境変数として読み込む
       - メインコンテナのコマンドは以下を指定
         - command: `['/bin/sh','-c','echo "Star Wars no title ichiran \n $EP1 \n $EP2 \n $EP3 \n $EP4 \n $EP5 \n $EP6 \n $EP7 \n $EP8 \n $EP9"; sleep   3600']`
     - Deployment(2つ目)
       - メインコンテナはshが使用できれば何でも良い
       - 上記ConfigMapのKey-Valueをすべて環境変数として読み込む
       - メインコンテナのコマンドは以下を指定
         - command: `['/bin/sh','-c','echo "Star Wars no koukai jyun \n $EP4 \n $EP5 \n $EP6 \n $EP1 \n $EP2 \n $EP3 \n $EP7 \n $EP8 \n $EP9; sleep 3600']`

   【回答例】

   ```yml
   # manifest
   apiVersion: v1
   kind: ConfigMap
   metadata:
     name: starwars-ep
   data:
      EP1: The Python Menace
      EP2: Attack of the Clones
      EP3: Revenge of the Sith
      EP4: A New Hoge
      EF5: The Empire Strikes Back
      EP6: Return of the Jedi
      EP7: The Force Awakens
      EP8: The Last Jedi
      EP9: The Rise of Kubernetes
   ---
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name:
       starwars-title
   spec:
     replicas: 1
     selector:
       matchLabels:
         app: starwars-title
     template:
       metadata:
         labels:
           app: starwars-title
       spec:
         containers:
          - name: busybox
            image: busybox
            envFrom:
              - configMapRef:
                  name: starwars-ep
            command: ['/bin/sh','-c','echo "Star Wars no title ichiran \n $EP1 \n $EP2 \n $EP3 \n $EP4 \n $EP5 \n $EP6 \n $EP7 \n $EP8 \n $EP9"; sleep   3600']
   ---
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name:
       starwars-koukai
   spec:
     replicas: 1
     selector:
       matchLabels:
         app: starwars-koukai
     template:
       metadata:
         labels:
           app: starwars-koukai
       spec:
         containers:
          - name: busybox
            image: busybox
            envFrom:
              - configMapRef:
                  name: starwars-ep
            command: ['/bin/sh','-c','echo "Star Wars no koukai jyun \n $EP4 \n $EP5 \n $EP6 \n $EP1 \n $EP2 \n $EP3 \n $EP7 \n $EP8 \n $EP9"; sleep 3600']
   ```

   ```bash
   # 実行結果
   kubectl apply -f configmap-env.yaml
   configmap/starwars-ep created
   eployment.apps/starwars-title created
   eployment.apps/starwars-koukai created
   ```

1. 各Deploymentで展開したPodのログを表示してください。

   【回答例】

   ```bash
   # 実行結果
   $ kubectl logs starwars-title-8749d55ff-78ffs
   Star Wars no title ichiran \n The Python Menace \n Attack of the Clones \n Revenge of the Sith \n A New Hoge \n  \n Return of the Jedi \n The Force Awakens \n The Last Jedi \n The Rise of Kubernetes

   $ kubectl logs starwars-koukai-f6dff5457-zjfkt
   Star Wars no koukai jyun \n A New Hoge \n  \n Return of the Jedi \n The Python Menace \n Attack of the Clones \n Revenge of the Sith \n The Force Awakens \n The Last Jedi \n The Rise of Kubernetes
   ```

1. ConfigMapの値の誤りを修正し再デプロイしてください。なお、正しいvalueは[参考サイト][3]などを参照してください。  
  (誤りは2か所)

   【回答例】

   ```yml
   # manifest
   apiVersion: v1
   kind: ConfigMap
   metadata:
     name: starwars-ep
   data:
      EP1: The Python Menace
      EP2: Attack of the Clones
      EP3: Revenge of the Sith
      EP4: A New Hope
      EF5: The Empire Strikes Back
      EP6: Return of the Jedi
      EP7: The Force Awakens
      EP8: The Last Jedi
      EP9: The Rise of Skywalker
   ---
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name:
       starwars-title
   spec:
     replicas: 1
     selector:
       matchLabels:
         app: starwars-title
     template:
       metadata:
         labels:
           app: starwars-title
       spec:
         containers:
          - name: busybox
            image: busybox
            envFrom:
              - configMapRef:
                  name: starwars-ep
            command: ['/bin/sh','-c','echo "Star Wars no title ichiran \n $EP1 \n $EP2 \n $EP3 \n $EP4 \n $EP5 \n $EP6 \n $EP7 \n $EP8 \n $EP9"; sleep   3600']
   ---
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name:
       starwars-koukai
   spec:
     replicas: 1
     selector:
       matchLabels:
         app: starwars-koukai
     template:
       metadata:
         labels:
           app: starwars-koukai
       spec:
         containers:
          - name: busybox
            image: busybox
            envFrom:
              - configMapRef:
                  name: starwars-ep
            command: ['/bin/sh','-c','echo "Star Wars no koukai jyun \n $EP4 \n $EP5 \n $EP6 \n $EP1 \n $EP2 \n $EP3 \n $EP7 \n $EP8 \n $EP9"; sleep 3600']
   ```

   ```bash
   # 実行結果
   kubectl apply -f configmap-env.yaml
   configmap/starwars-ep configured
   deployment.apps/starwars-title unchanged
   deployment.apps/starwars-koukai unchanged
   ```

1. 各Deploymentで展開したPodのログを表示してください。表示が`変わらないこと`を確認してください。

   【回答例】

   ```bash
   # 実行結果
   $ kubectl logs starwars-title-8749d55ff-78ffs
   Star Wars no title ichiran \n The Python Menace \n Attack of the Clones \n Revenge of the Sith \n A New Hoge \n  \n Return of the Jedi \n The Force Awakens \n The Last Jedi \n The Rise of Kubernetes

   $ kubectl logs starwars-koukai-f6dff5457-zjfkt
   Star Wars no koukai jyun \n A New Hoge \n  \n Return of the Jedi \n The Python Menace \n Attack of the Clones \n Revenge of the Sith \n The Force Awakens \n The Last Jedi \n The Rise of Kubernetes
   ```

1. 各Deploymentで展開したPodのログを削除し、セルフ・ヒーリングさせてください。

   【回答例】

   ```bash
   # 実行結果
   $ kubectl delete pod --all
   pod "starwars-koukai-f6dff5457-zjfkt" deleted
   pod "starwars-title-8749d55ff-78ff" deleted
   ```

1. 各Deploymentで展開したPodのログを表示してください。表示が変わることを確認してください。

   【回答例】

   ```bash
   # 実行結果
   $ kubectl logs starwars-title-8749d55ff-mplsz
   Star Wars no title ichiran \n The Python Menace \n Attack of the Clones \n Revenge of the Sith \n A New Hope \n  \n Return of the Jedi \n The Force Awakens \n The Last Jedi \n The Rise of Skywalker

   $ kubectl logs starwars-koukai-f6dff5457-rx575
   Star Wars no koukai jyun \n A New Hope \n  \n Return of the Jedi \n The Python Menace \n Attack of the Clones \n Revenge of the Sith \n The Force Awakens \n The Last Jedi \n The Rise of Skywalker
   ```

1. 作成したリソースを削除してください。

   【回答例】

   ```bash
   # 実行結果
   # kubectl delete -f configmap-env.yaml
   configmap "starwars-ep" deleted
   deployment.apps "starwars-title" deleted
   deployment.apps "starwars-koukai" deleted
   ```

[1]:https://kubernetes.io/docs/tasks/configure-pod-container/configure-pod-configmap/
[2]:https://kubernetes.io/docs/tasks/configure-pod-container/configure-pod-configmap/#configure-all-key-value-pairs-in-a-configmap-as-container-environment-variables
[3]:https://dic.nicovideo.jp/a/%E3%82%B9%E3%82%BF%E3%83%BC%E3%82%A6%E3%82%A9%E3%83%BC%E3%82%BA
