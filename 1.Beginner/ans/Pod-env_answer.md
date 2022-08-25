# 回答例

## チュートリアル: 環境変数をmanifestに定義する

1. 次のmanifestを使用して、Deployment:envを作成する。

    ``` yaml
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: env
      labels:
        app: test
    spec:
      replicas: 1
      selector:
        matchLabels:
          app: test
      template:
        metadata:
          labels:
            app: test
        spec:
          containers:
          - name: busybox
            image: busybox:1.35.0
            command: ['sh', '-c', 'echo $ENV1 $HOSTNAME $ENV2 && sleep 3600']
    ```

   【回答例】

   ```bash
   # manifestを作成する
   cat <<EOF > ./Deployment.yaml
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: env
     labels:
       app: test
   spec:
     replicas: 1
     selector:
       matchLabels:
         app: test
     template:
       metadata:
         labels:
           app: test
       spec:
         containers:
         - name: busybox
           image: busybox:1.35.0
           command: ['sh', '-c', 'echo \$ENV1 \$HOSTNAME \$ENV2 && sleep 3600']
   EOF

   # Deplymentを作成する
   $ kubectl apply -f Deployment.yaml
   deployment.apps/nginx created
   ```

1. デプロイしたPod（コンテナ）のログを表示し、「<Pod名>」が表示されることを確認する。

   【回答例】

   ```bash
   # Pod名の確認
   $ kubectl get pod
   NAME                   READY   STATUS    RESTARTS   AGE
   env-d59f49b84-l7tb5    1/1     Running       0          10s

   $ kubectl logs env-d59f49b84-l7tb5
   env-d59f49b84-l7tb5
   ```

1. デプロイしたPod（コンテナ）に追加コマンドを発行し設定されている環境変数を確認する。
   （環境変数の一覧は`printenv`というコマンドで確認できる。）

   【回答例】

   ```bash
   $ kubectl exec env-d59f49b84-l7tb5 -- printenv
   PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
   HOSTNAME=env-d59f49b84-l7tb5
   KUBERNETES_SERVICE_PORT=443
   KUBERNETES_SERVICE_PORT_HTTPS=443
   KUBERNETES_PORT=tcp://10.100.0.1:443
   KUBERNETES_PORT_443_TCP=tcp://10.100.0.1:443
   KUBERNETES_PORT_443_TCP_PROTO=tcp
   KUBERNETES_PORT_443_TCP_PORT=443
   KUBERNETES_PORT_443_TCP_ADDR=10.100.0.1
   KUBERNETES_SERVICE_HOST=10.100.0.1
   HOME=/root
   ```

1. Deployment:envを削除する。

   【回答例】

   ```bash
   $ kubectl delete -f Deployment.yaml
   deployment.apps "env" deleted
   ```

1. 次のmanifestを使用して、Deployment:envをデプロイする。

    ``` yaml
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: env
      labels:
        app: test
    spec:
      replicas: 1
      selector:
        matchLabels:
          app: test
      template:
        metadata:
          labels:
            app: test
        spec:
          containers:
          - name: busybox
            image: busybox:1.35.0
            command: ['sh', '-c', 'echo \$ENV1 \$HOSTNAME \$ENV2 && sleep 3600']
            env:
            - name: ENV1
              value: "watashi no namae ha "
            - name: ENV2
              value: " desuYO"
    ```

   【回答例】

   ```bash
   # manifestを作成する
   cat <<EOF > ./Deployment.yaml
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: env
     labels:
       app: test
   spec:
     replicas: 1
     selector:
       matchLabels:
         app: test
     template:
       metadata:
         labels:
           app: test
       spec:
         containers:
         - name: busybox
           image: busybox:1.35.0
           command: ['sh', '-c', 'echo \$ENV1 \$HOSTNAME \$ENV2 && sleep 3600']
           env:
           - name: ENV1
             value: "watashi no namae ha "
           - name: ENV2
             value: " desuYO"
   EOF

   # Deplymentを作成する
   $ kubectl apply -f Deployment.yaml
   deployment.apps/env created
   ```

1. デプロイしたPod（コンテナ）のログを表示し「<Pod名>」が表示されることを確認する。

  【回答例】

   ```bash
   # Pod名の確認
   $ kubectl get pod
   NAME                   READY   STATUS    RESTARTS   AGE
   env-84c98c8576-h8wx5   1/1     Running       0          7s

   $ kubectl logs env-84c98c8576-h8wx5
   watashi no namae ha env-84c98c8576-h8wx5 desuYO
   ```

1. デプロイしたPod（コンテナ）に追加コマンドを発行し設定されている環境変数を確認する。

   【回答例】

   ```bash
   $ kubectl exec env-84c98c8576-h8wx5 -- printenv
   PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
   HOSTNAME=env-84c98c8576-h8wx5
   ENV1=watashi no namae ha
   ENV2= desuYO
   KUBERNETES_SERVICE_PORT_HTTPS=443
   KUBERNETES_PORT=tcp://10.100.0.1:443
   KUBERNETES_PORT_443_TCP=tcp://10.100.0.1:443
   KUBERNETES_PORT_443_TCP_PROTO=tcp
   KUBERNETES_PORT_443_TCP_PORT=443
   KUBERNETES_PORT_443_TCP_ADDR=10.100.0.1
   KUBERNETES_SERVICE_HOST=10.100.0.1
   KUBERNETES_SERVICE_PORT=443
   HOME=/root
   ```

1. Deployment:envを削除する。

  【回答例】

   ```bash
   $ kubectl delete -f Deployment.yaml
   deployment.apps "env" deleted
   ```
