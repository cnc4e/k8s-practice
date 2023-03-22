# 回答例

1. 以下を満たすマニフェストを作成しデプロイしてください。

   - 要件
     - ConfigMap
       - 以下の内容が記述されたindex.htmlをdataとして持つ
         - ConfigMap ni kaita naiyou dayo
     - Deployment
       - イメージはnginx:1.12
       - 上記ConfigMapをボリュームとしてマウント
       - マウント先は/usr/share/nginx/html
     - Service
       - 上記DeploymentをClusterIPのPort:80で公開

   【回答例】

   ```yml
   # manifest
   apiVersion: v1
   kind: ConfigMap
   metadata:
     name: index.html
   data:
      index.html: ConfigMap ni kaita naiyou dayo
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
             volumeMounts:
               - name: index-html
                 mountPath: /usr/share/nginx/html
         volumes:
           - name: index-html
             configMap:
               name: index.html
               items:
                 - key: index.html
                   path: index.html
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
   ```

   ```bash
   # 実行結果
   $ kc apply -f configmap-mount.yaml
   configmap/index.html created
   deployment.apps/nginx created
   service/nginx-svc created
   ```

1. 接続確認に使用するcurlコマンドが実行可能なtestpodを作成し、testpodから`Service:nginx-svc`に対して接続確認をしてください。ConfigMapの内容が表示されることを確認してください。

   【回答例】

   ```bash
   # 実行結果
   ## testpod作成
   $ kubectl run --image=appropriate/curl --restart=Never testpod sleep 3600
   ## 接続確認
   $ kubectl exec testpod -- curl -s http://nginx-svc
   ConfigMap ni kaita naiyou dayo
   ```

1. ConfigMapのindex.htmlの内容を以下に修正し、ConfigMapを再デプロイしてください。
   - ConfigMap wo henkou suruto 60byou kuraide Pod nimo hanei sareruzo

   【回答例】

   ```yml
   # manifest
   apiVersion: v1
   kind: ConfigMap
   metadata:
     name: index.html
   data:
      index.html: ConfigMap wo henkou suruto 60byou kuraide Pod nimo hanei sareruzo
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
             volumeMounts:
               - name: index-html
                 mountPath: /usr/share/nginx/html
         volumes:
           - name: index-html
             configMap:
               name: index.html
               items:
                 - key: index.html
                   path: index.html
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
   ```

   ```bash
   # 実行結果
   $ kubectl apply -f configmap-mount.yaml
   configmap/index.html configured
   deployment.apps/nginx unchanged
   service/nginx-svc unchanged
   ```

1. （configmapをデプロイして60秒以内）curlが実行可能なPodからServiceを指定してcurlを実行してください。ConfigMap修正前の内容が表示されることを確認してください。

   【回答例】

   ```bash
   # 実行結果
   $ kubectl exec testpod -- curl -s http://nginx-svc
   ConfigMap ni kaita naiyou dayo
   ```

1. （configmapをデプロイして60秒以降）curlが実行可能なPodからServiceを指定してcurlを実行してください。ConfigMap修正後の内容が表示されることを確認してください。

   【回答例】

   ```bash
   # 実行結果
   $ kubectl exec testpod -- curl -s http://nginx-svc
   ConfigMap wo henkou suruto 60byou kuraide Pod nimo hanei sareruzo
   ```

1. 作成したリソースを削除してください。

   【回答例】

   ```bash
   # 実行結果
   $ kubectl delete -f configmap-mount.yaml
   configmap "index.html" deleted
   deployment.apps "nginx" deleted
   service "nginx-svc" deleted
   ```
