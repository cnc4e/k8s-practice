# 回答例

## 問題1

次の要件を満たすmanifestを作成する。

- 要件
  - Deployment
    - Deploymentの名前は`practice`
    - replicas: `2`
    - Pod
      - labelはすべて`app: practice`
      - containerのイメージは`nginx:1.22`
      - manifestで指定する環境変数は`ENV1=Kono Container ha`、`ENV2= To Iimasu`
    - volumes
      - `hostPath`のボリュームプラグインを使用
      - ホスト (worker node) の/mntなど任意のディレクトリを対象とする
    - volumeMount
      - 上記で定義したhostPathのボリュームを指定する
      - コンテナのマウントパスは/usr/share/nginx/html/を指定する
  - Service
    - Serviceの名前は`practice-svc`
    - typeは`指定なし`（type: ClusterIPでも可）
    - Portは80
    - 上記Deployment:practiceで展開したPodを対象とする

   【回答例】

   ```yml
   # Deployment
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: practice
     labels:
       app: practice
   spec:
     replicas: 2
     selector:
       matchLabels:
         app: practice
     template:
       metadata:
         labels:
           app: practice
       spec:
         containers:
         - name: nginx
           image: nginx:1.22
           env:
           - name: ENV1
             value: "Kono Container ha "
           - name: ENV2
             value: "Nginx desu"
           volumeMounts:
           - mountPath: /usr/share/nginx/html
             name: index-html
         volumes:
         - name: index-html
           hostPath:
             path: /mnt
             type: Directory
   # Service
   kind: Service
   apiVersion: v1
   metadata:
     name: practice-svc
   spec:
     selector:
       app: practice
     ports:
     - protocol: TCP
       port: 80
       targetPort: 80
   ```

## 問題2

1. 問題1で作成したmanifestをK8sクラスタ上にデプロイする。

   【回答例】

   ```bash
   # Deployment
   cat <<EOF > ./Deployment.yaml
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: practice
     labels:
       app: practice
   spec:
     replicas: 2
     selector:
       matchLabels:
         app: practice
     template:
       metadata:
         labels:
           app: practice
       spec:
         containers:
         - name: nginx
           image: nginx:1.22
           env:
           - name: ENV1
             value: "Kono Container ha "
           - name: ENV2
             value: "Nginx desu"
           volumeMounts:
           - mountPath: /usr/share/nginx/html
             name: index-html
         volumes:
         - name: index-html
           hostPath:
             path: /mnt
             type: Directory
   EOF
   $
   $ kubectl apply -f Deployment.yaml
   deployment.apps/practice created

   # Service
   cat <<EOF > ./Service.yaml
   kind: Service
   apiVersion: v1
   metadata:
     name: practice-svc
   spec:
     selector:
       app: practice
     ports:
     - protocol: TCP
       port: 80
       targetPort: 80
   EOF

   $ kubectl apply -f Service.yaml
   service/practice-svc created
   ```

1. デプロイしたPod内のコンテナに対して追加コマンドを発行し、/usr/share/nginx/html/index.htmlを以下内容に修正する。

   ```text
   zettai ni ushiatte ha ikenai data ga kochira dasu
   \$ENV1 \$ENV2
   ```

   【回答例】

   ```bash
   # Pod名の確認
   $ kubectl get pod
   NAME                    READY   STATUS    RESTARTS   AGE
   practice-788b88f664-2ncmv   1/1     Running   0          35s
   practice-788b88f664-v5xzr   1/1     Running   0          35s

   # index.html修正
   $ kubectl exec practice-788b88f664-2ncmv -- bash -c "echo zettai ni ushiatte ha ikenai data ga kochira dasu> /usr/share/nginx/html/index.html"
   $ kubectl exec practice-788b88f664-2ncmv -- bash -c "echo \$ENV1 \$ENV2 >> /usr/share/nginx/html/index.html"
   $
   $ kubectl exec practice-788b88f664-v5xzr -- bash -c "echo zettai ni ushiatte ha ikenai data ga kochira dasu> /usr/share/nginx/html/index.html"
   $ kubectl exec practice-788b88f664-v5xzr -- bash -c "echo \$ENV1 \$ENV2 >> /usr/share/nginx/html/index.html"
   ```

1. curlを実行できるPod:curlを作成する。

   【回答例】

   ```bash
   $ kubectl run curl --image=appropriate/curl -- /bin/sh -c "sleep 3600"
   pod/curl created
   ```

1. Service:practice-svcを経由してDeployment:practiceにアクセスする。
  （さきほど修正したindex.htmlが表示`される`ことを確認する）

   【回答例】

   ```bash
   $ kubectl exec curl -- curl -s practice-svc
   zettai ni ushiatte ha ikenai data ga kochira dasu
   Kono Container ha Nginx desu
   ```

1. Deployment:practiceのアクセスログを確認する。

   【回答例】

   ```bash
   # Pod名の確認
   $ kubectl get pod
   NAME                   READY   STATUS    RESTARTS   AGE
   practice-84c98c8576-h8wx5   1/1     Running       0          7s

   $ kubectl logs env-84c98c8576-h8wx5
   192.168.20.141 - - [22/Aug/2022:12:32:48 +0000] "GET / HTTP/1.1" 200 117 "-" "curl/7.59.0" "-"
   192.168.20.141 - - [22/Aug/2022:12:33:43 +0000] "GET / HTTP/1.1" 200 79 "-" "curl/7.59.0" "-"
   ```

## 問題3

1. Deployment:practiceのNginxのバージョンを`1.23`に`無停止で`アップデートする。

   【回答例】

   ```bash
   # manifest修正
   $ cat Deployment.yaml | sed 's/nginx:1.22/nginx:1.23/' > Deployment.yaml
   # 修正を適用
   $ kubectl apply -f Deployment.yaml
   deployment.apps/practice configured
   ```

1. Pod:curlからService:practice-svcを経由してDeploymentにアクセスし続ける。
   - アップデート中にタイムアウトが発生しないことを確認する
   - 問題2で修正したindex.htmlが表示`される`ことを確認する。

   【回答例】

   ```bash
   $ kubectl exec curl -- curl -s practice-svc
   zettai ni ushiatte ha ikenai data ga kochira dasu
   Kono Container ha Nginx desu
   ```

1. Deployment:practiceとService:practice-svcを削除する。

   【回答例】

   ```bash
   $ kubectl delete -f Deployment.yaml
   deployment.apps "practice" deleted

   $ kubectl delete -f Service.yaml
   service "practice-svc" deleted
   ```
