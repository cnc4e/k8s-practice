# 回答例

1. 以下を満たすマニフェストを作成しデプロイしてください。Secretリソースについては[公式ドキュメント][1]を参考にしてください。

   - 要件
     - Secret
       - 以下のKey-Valueをdataとして持つ（以下はbase64デコードされた状態）
         - password: "zettai ni sirarete ha ikenai jyouhou"
     - Deployment
       - 上記Secretのpasswordの値を環境変数PASSWORDに格納

   【回答例】

   ```bash
   # base64エンコード
   $ echo -n zettai ni sirarete ha ikenai jyouhou |base64
   emV0dGFpIG5pIHNpcmFyZXRlIGhhIGlrZW5haSBqeW91aG91
   ```

   ```yml
   # manifest
   apiVersion: v1
   kind: Secret
   metadata:
     name: password
   data:
     PASSWORD: emV0dGFpIG5pIHNpcmFyZXRlIGhhIGlrZW5haSBqeW91aG91
   ---
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name:
       secret
   spec:
     replicas: 1
     selector:
       matchLabels:
         app: secret
     template:
       metadata:
         labels:
           app: secret
       spec:
         containers:
          - name: busybox
            image: busybox
            envFrom:
              - secretRef:
                  name: password
            command: ['/bin/sh','-c','sleep 3600']
   ```

   ```bash
   # 実行結果
   $ kubectl apply -f secret.yaml
   secret/password created
   deployment.apps/secret created
   ```

1. 上記Deploymentで展開したPodに「echo $PASSWORD」の追加コマンドを発行してください。Secretの内容が表示されることを確認してください。

   【回答例】

   ```bash
   # 実行結果
   $ kubectl get pod
   NAME                      READY   STATUS    RESTARTS   AGE
   secret-867945d467-4wq48   1/1     Running   0          7s

   $ kubectl exec secret-867945d467-4wq48 -it -- /bin/sh
   / #
   / # echo $PASSWORD
   zettai ni sirarete ha ikenai jyouhou
   / # exit
   ```

1. Secretの詳細を表示してください。dataのvalue部分が見えないことを確認してください。

   【回答例】

   ```bash
   # 実行結果
   $ kubectl describe secret password
   Name:         password
   Namespace:    default
   Labels:       <none>
   Annotations:  <none>

   Type:  Opaque

   Data
   ====
   PASSWORD:  36 bytes ★
   ```

1. Secretの内容をYAML形式で表示してください。passwordのvalueが表示されるのでコピーしてください。

   【回答例】

   ```bash
   # 実行結果
   $ kubectl get secret password -o yaml
   apiVersion: v1
   data:
     PASSWORD: emV0dGFpIG5pIHNpcmFyZXRlIGhhIGlrZW5haSBqeW91aG91
   kind: Secret
   metadata:
     annotations:
       kubectl.kubernetes.io/last-applied-configuration: |
         {"apiVersion":"v1","data":{"PASSWORD":"emV0dGFpIG5pIHNpcmFyZXRlIGhhIGlrZW5haSBqeW91aG91"},"kind":"Secret","metadata":{"annotations":{},"name":"password","namespace":"default"}}
     creationTimestamp: "2023-03-21T18:08:10Z"
     name: password
     namespace: default
     resourceVersion: "5683241"
     uid: 72aa0177-f7c1-4bf9-a678-e95bbdc65b00
   type: Opaque
   ```

1. 以下のコマンドを実行してください。valueの中身が復号されることを確認してください。

   ```bash
   echo -n <前の手順でコピーしたvalue> | base64 --decode
   ```

   【回答例】

   ```bash
   # 実行結果
   echo -n emV0dGFpIG5pIHNpcmFyZXRlIGhhIGlrZW5haSBqeW91aG91 | base64 --decode
   zettai ni sirarete ha ikenai jyouhou
   ```

1. 作成したリソースを削除してください。

   【回答例】

   ```bash
   # 実行結果
   $ kubectl delete -f secret.yaml
   secret "password" deleted
   deployment.apps "secret" deleted
   ```

[1]:https://kubernetes.io/docs/concepts/configuration/secret/
