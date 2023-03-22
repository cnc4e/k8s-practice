# 回答例

## 問題1: NFSサーバ

次の要件を満たすmanifestを作成し、デプロイしてください。

- 要件
  - Namespace
    - 名前は`sbdemo-nfs`
  - PVC
    - 名前は`sbdemo-nfs-server-pvc`
    - Namespaceは`sbdemo-nfs`
    - storageClassNameは`指定しない`（デフォルトのStorageClassを使用する）
    - accessModesは`ReadWriteOnce`
    - ストレージ容量は`1Gi`
  - Deployment
    - 名前は`sbdemo-nfs-server`
    - Namespaceは`sbdemo-nfs`
    - replicas: `1`
    - labelはすべて`role: nfs-server`
    - Pod
      - イメージは`k8s.gcr.io/volume-nfs:0.8`
      - volumeプラグインで上記PVC:`nfs-server-pvc`を指定
      - 上記で定義したボリュームをコンテナの`/exports`にマウント
      - 待ち受けポートは次の通り
        - nfs: 2049
        - mountd: 20048
        - rpcbind: 111
      - securityContextに`privileged: true`を指定
  - Service
    - 名前は`sbdemo-nfs-server-service`
    - Namespaceは`sbdemo-nfs`
    - タイプは指定なし（ClusterIP）
    - 待ち受けポートは次の通り
      - nfs: 2049
      - mountd: 20048
      - rpcbind: 111
    - selectorは`role: nfs-server`
  - PersistentVolume
    - 名前は`sbdemo-nfs-pv`
    - labelはすべて`role: nfs-pv`
    - accessModesは`ReadWriteOnce`
    - ストレージ容量は`1Gi`
    - nfsの設定は次の通り
      - server: <sbdemo-nfs-server-serviceのClusterIP>
      - path: "/"

【回答例】

```yml
# manifest
apiVersion: v1
kind: Namespace
metadata:
  name: sbdemo-nfs
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: sbdemo-nfs-server-pvc
  namespace: sbdemo-nfs
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: sbdemo-nfs-server
  namespace: sbdemo-nfs
spec:
  replicas: 1
  selector:
    matchLabels:
      role: nfs-server
  template:
    metadata:
      labels:
        role: nfs-server
    spec:
      containers:
      - name: nfs-server
        image: k8s.gcr.io/volume-nfs:0.8
        ports:
          - name: nfs
            containerPort: 2049
          - name: mountd
            containerPort: 20048
          - name: rpcbind
            containerPort: 111
        securityContext:
          privileged: true
        volumeMounts:
        - mountPath: /exports
          name: sbdemo-nfs-server-pvc
      volumes:
        - name: sbdemo-nfs-server-pvc
          persistentVolumeClaim:
            claimName: sbdemo-nfs-server-pvc
---
apiVersion: v1
kind: Service
metadata:
  name: sbdemo-nfs-server-service
  namespace: sbdemo-nfs
spec:
  ports:
  - name: nfs
    port: 2049
  - name: mountd
    port: 20048
  - name: rpcbind
    port: 111
  selector:
    role: nfs-server
```

```bash
# 実行結果
$ kubectl apply -f nfs-server.yaml
namespace/sbdemo-nfs created
persistentvolumeclaim/sbdemo-nfs-server-pvc created
deployment.apps/sbdemo-nfs-server created
service/sbdemo-nfs-server-service created
```

```bash
# 実行結果
$ kubectl get svc -n sbdemo-nfs
NAME                        TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)                      AGE
sbdemo-nfs-server-service   ClusterIP   10.100.80.196   <none>        2049/TCP,20048/TCP,111/TCP   8m10s
```

```yml
# manifest
apiVersion: v1
kind: PersistentVolume
metadata:
  name: sbdemo-nfs-pv
  labels:
    role: nfs-pv
spec:
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteMany
  nfs:
    server: "10.100.80.196"
    path: "/"
```

```bash
# 実行結果
$ kubectl apply -f nfs-pv.yaml
persistentvolume/sbdemo-nfs-pv created
```

## 問題2: Redisサーバ

次の要件を満たすmanifestを作成し、デプロイしてください。

- 要件
  - Namespace
    - 名前は`sbdemo-redis`
  - Deployment
    - 名前は`sbdemo-redis`
    - Namespaceは`sbdemo-redis`
    - replicas: `1`
    - labelはすべて`app: sbdemo-redis`
    - Pod
      - 名前は`sbdemo-redis`
      - イメージは`redis:alpine`
      - 待ち受けポートは`6379`
  - Service
    - 名前は`sbdemo-redis-service`
    - Namespaceは`sbdemo-redis`
    - タイプは指定なし（ClusterIP）
    - Protocolは`TCP`
    - 待ち受けポートは`6379`
    - ターゲットポートは`6379`
    - selectorは`app: sbdemo-redis`

【回答例】

```yml
# manifest
apiVersion: v1
kind: Namespace
metadata:
  name: sbdemo-redis
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: sbdemo-redis
  namespace: sbdemo-redis
spec:
  replicas: 1
  selector:
    matchLabels:
      app: sbdemo-redis
  template:
    metadata:
      labels:
        app: sbdemo-redis
    spec:
      containers:
        - name: redis
          image: redis:alpine
          ports:
            - containerPort: 6379
---
apiVersion: v1
kind: Service
metadata:
  name: sbdemo-redis-service
  namespace: sbdemo-redis
spec:
  type: ClusterIP
  ports:
    - name: "redis-port"
      protocol: "TCP"
      port: 6379
      targetPort: 6379
  selector:
    app: sbdemo-redis
```

```bash
# 実行結果
$ kubectl apply -f redis.yaml
namespace/sbdemo-redis created
deployment.apps/sbdemo-redis created
service/sbdemo-redis-service created
```

## 問題3: DBサーバ

次の要件を満たすmanifestを作成し、デプロイしてください。

- 要件
  - Namespace
    - 名前は`sbdemo-db`
  - StatefulSet
    - 名前は`sbdemo-postgres-sts`
    - Namespaceは`sbdemo-db`
    - labelはすべて`app: sbdemo-postgres-sts`
    - replicasは `3`
    - Pod
      - 名前は`postgres`
      - イメージは`dayan888/springdemo:postgres9.6`
      - 待ち受けポートは`5432`
  - Service
    - 名前は`sbdemo-postgres-service`
    - Namespaceは`sbdemo-db`
    - typeは`ClusterIP`（`明示的に指定する`）
    - clusterIPに`None`
    - Protocolは`TCP`
    - 待ち受けポートは`5432`
    - ターゲットポートは`5432`
    - selectorは`app: sbdemo-postgres-sts`

【回答例】

```yml
# manifest
apiVersion: v1
kind: Namespace
metadata:
  name: sbdemo-db
---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name:
    sbdemo-postgres-sts
  namespace: sbdemo-db
spec:
  serviceName: sbdemo-postgres-service
  replicas: 1
  selector:
    matchLabels:
      app: sbdemo-postgres-sts
  template:
    metadata:
      labels:
        app: sbdemo-postgres-sts
    spec:
      containers:
       - name: postgres
         image: dayan888/springdemo:postgres9.6
         ports:
          - containerPort: 5432
         volumeMounts:
         - name: pvc-db-volume
           mountPath: /var/lib/postgresql
  volumeClaimTemplates:
  - metadata:
      name: pvc-db-volume
    spec:
      accessModes:
      - ReadWriteOnce
      resources:
        requests:
          storage: 1G
---
apiVersion: v1
kind: Service
metadata:
  name: sbdemo-postgres-service
  namespace: sbdemo-db
spec:
  type: ClusterIP
  clusterIP: None
  ports:
   - name: "db-port"
     protocol: "TCP"
     port: 5432
     targetPort: 5432
  selector:
    app: sbdemo-postgres-sts
```

```bash
# 実行結果
$ kubectl apply -f db.yaml
namespace/sbdemo-db created
statefulset.apps/sbdemo-postgres-sts created
service/sbdemo-postgres-service created
```

## 問題4: APサーバ

次の要件を満たすmanifestを作成し、デプロイしてください。

- 要件
  - Namespace
    - 名前は`sbdemo-ap`
  - PersistentVolumeClaim
    - 名前は`sbdemo-nfs-pvc`
    - Namespaceは`spdemo-ap`
    - storageClassNameは`指定しない`
    - selectorは`role: nfs-pv`
    - accessModesは`ReadWriteOnce`
    - ストレージ容量は`1Gi`
  - ConfigMap
    - 名前は`ap-config`
    - Namespaceは`sbdemo-ap`
    - 以下のKey-Valueをdataとして持つ
      - SPRING_PROFILES_ACTIVE: "prd"
      - DB_URL: "jdbc:postgresql://sbdemo-postgres-service.sbodemo-db:5432/demodb?   user=postgres&password=postgres"
      - PIC_DIR: "/opt/picDir"
      - REDIS_HOST: "sbdemo-redis-service.sbdemo-redis"
      - REDIS_PORT: "6379"
  - Deployment
    - Deployment
      - 名前は`sbdemo-apserver`
      - Namespaceは`sbdemo-ap`
      - labelはすべて`app: sbdemo-apserver`
      - replicasは `3`
      - Pod
        - 名前は`apserver`
        - イメージは`dayan888/springdemo:apserver`
        - 待ち受けポートは`8080`
        - env
          - ConfigMap:ap-config
        - volumeプラグインでPVC:`sbdemo-nfs-pvc`を指定
        - 上記で定義したボリュームをコンテナの`/opt/picDir`にマウント
    - Service
      - 名前は`sbdemo-postgres-service`
      - Namespaceは`sbdemo-ap`
      - typeは`ClusterIP`
      - Protocolは`TCP`
      - 待ち受けポートは`8080`
      - ターゲットポートは`8080`
      - selectorは`app: sbdemo-apserver`

【回答例】

```yml
# manifest
apiVersion: v1
kind: Namespace
metadata:
  name: sbdemo-ap
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: sbdemo-nfs-pvc
  namespace: sbdemo-ap
spec:
  selector:
    matchLabels:
      role: nfs-pv
  accessModes:
    - ReadWriteMany
  storageClassName: ""
  resources:
    requests:
      storage: 1Gi
---
apiVersion: v1
kind: ConfigMap
metadata:
  name: ap-config
  namespace: sbdemo-ap
data:
  SPRING_PROFILES_ACTIVE: "prd"
  DB_URL: "jdbc:postgresql://sbdemo-postgres-service.sbdemo-db:5432/demodb?user=postgres&password=postgres"
  PIC_DIR: "/opt/picDir"
  REDIS_HOST: "sbdemo-redis-service.sbdemo-redis"
  REDIS_PORT: "6379"
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: sbdemo-apserver
  namespace: sbdemo-ap
spec:
  replicas: 3
  selector:
    matchLabels:
      app: sbdemo-apserver
  template:
    metadata:
      labels:
        app: sbdemo-apserver
    spec:
      containers:
        - name: apserver
          image: dayan888/springdemo:apserver
          imagePullPolicy: Always
          ports:
            - containerPort: 8080
          readinessProbe:
            httpGet:
              path: /login
              port: 8080
          envFrom:
            - configMapRef:
                name: ap-config
          volumeMounts:
            - mountPath: "/opt/picDir"
              name: apserver-pvc
      volumes:
        - name: apserver-pvc
          persistentVolumeClaim:
            claimName: sbdemo-nfs-pvc
---
apiVersion: v1
kind: Service
metadata:
  name: sbdemo-apserver-service
  namespace: sbdemo-ap
spec:
  type: ClusterIP
  ports:
    - name: "http-port"
      protocol: "TCP"
      port: 8080
      targetPort: 8080
  selector:
    app: sbdemo-apserver
```

```bash
# 実行結果
$ kubectl apply -f ap.yaml
namespace/sbdemo-ap created
persistentvolumeclaim/sbdemo-nfs-pvc created
configmap/ap-config created
deployment.apps/sbdemo-apserver created
service/sbdemo-apserver-service created
```

## 問題5: WEBサーバ

次の要件を満たすmanifestを作成し、デプロイしてください。

- 要件
  - Namespace
    - 名前は`sbdemo-web`
  - ConfigMap(1つめ)
    - 名前は`nginx-conf`
    - Namespaceは`sbdemo-web`
    - 以下の内容が記述されたnginx.confをdataとして持つ

      ```nginx
      user  nginx;
      worker_processes 1;
      pid        /var/run/nginx.pid;
      events {
          worker_connections 1024;
      }
      http {
          include       /etc/nginx/mime.types;
          default_type  application/octet-stream;
          log_format  main  '$remote_addr - $remote_user [$time_local]  request" '
                            '$status $body_bytes_sent "$http_referer" '
                            '"$http_user_agent" "$http_x_forwarded_for"';
          access_log /dev/stdout;
          error_log  /dev/stderr debug;
          sendfile        on;
          keepalive_timeout  65;
          include /etc/nginx/conf.d/server.conf;
      }
      ```

  - ConfigMap(2つめ)
    - 名前は`server-conf`
    - Namespaceは`sbdemo-web`
    - 以下の内容が記述されたserver.confをdataとして持つ

      ```nginx
      server {
          server_name  springdemo.example.com;
          listen 80;
          location ~* \.(js|jpg|png|css)$ {
              expires 168h;
              root   /usr/share/nginx/html;
          }
          location / {
              proxy_set_header X-Forwarded-Host $host;
              proxy_set_header X-Forwarded-Server $host;
              proxy_set_header X-Forwarded-For $remote_addr;
              proxy_set_header X-Forwarded-Proto $scheme;
              proxy_set_header X-Real-IP $remote_addr;
              proxy_set_header Host $http_host;
              proxy_pass http://sbdemo-apserver-service.sbdemo-ap:8080;
              proxy_cookie_path / /;
          }
          error_page   500 502 503 504  /50x.html;
          location = /50x.html {
              root   /usr/share/nginx/html;
          }
      }
      ```

  - Deployment
    - Deployment
      - 名前は`sbdemo-nginx`
      - Namespaceは`sbdemo-web`
      - labelはすべて`app: sbdemo-nginx`
      - replicasは `3`
      - Pod
        - 名前は`nginx`
        - イメージは`dayan888/springdemo:nginx`
        - 待ち受けポートは`80`
        - ConfigMap:nginx-confを`/etc/nginx/nginx.conf`にマウントする
          - subPath:nginx.conf を設定する
        - ConfigMap:server-confを`/etc/nginx/conf.d/server.conf`にマウントする
          - subPath:server.conf を設定する
    - Service
      - 名前は`sbdemo-nginx-service`
      - Namespaceは`sbdemo-web`
      - typeは`ClusterIP`
      - Protocolは`TCP`
      - 待ち受けポートは`80`
      - ターゲットポートは`80`
      - selectorは`app: sbdemo-nginx`

【回答例】

```yml
# manifest
apiVersion: v1
kind: Namespace
metadata:
  name: sbdemo-web
---
apiVersion: v1
kind: ConfigMap
metadata:
  name: nginx-conf
  namespace: sbdemo-web
data:
  nginx.conf: |
    user  nginx;
    worker_processes 1;
    pid        /var/run/nginx.pid;
    events {
        worker_connections 1024;
    }
    http {
        include       /etc/nginx/mime.types;
        default_type  application/octet-stream;
        log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                          '$status $body_bytes_sent "$http_referer" '
                          '"$http_user_agent" "$http_x_forwarded_for"';
        access_log /dev/stdout;
        error_log  /dev/stderr debug;
        sendfile        on;
        keepalive_timeout  65;
        include /etc/nginx/conf.d/server.conf;
    }
---
apiVersion: v1
kind: ConfigMap
metadata:
  name: server-conf
  namespace: sbdemo-web
data:
  server.conf: |
    server {
        server_name  springdemo.example.com;
        listen 80;
        location ~* \.(js|jpg|png|css)$ {
            expires 168h;
            root   /usr/share/nginx/html;
        }
        location / {
            proxy_set_header X-Forwarded-Host $host;
            proxy_set_header X-Forwarded-Server $host;
            proxy_set_header X-Forwarded-For $remote_addr;
            proxy_set_header X-Forwarded-Proto $scheme;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header Host $http_host;
            proxy_pass http://sbdemo-apserver-service.sbdemo-ap:8080;
            proxy_cookie_path / /;
        }
        location = /index.html {
            root   /usr/share/nginx/html;
            index index.html;
        }
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   /usr/share/nginx/html;
        }
    }
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: sbdemo-nginx
  namespace: "sbdemo-web"
spec:
  replicas: 13
  selector:
    matchLabels:
      app: sbdemo-nginx
  template:
    metadata:
      labels:
        app: sbdemo-nginx
    spec:
      containers:
        - name: nginx
          image: dayan888/springdemo:nginx
          ports:
            - containerPort: 80
          volumeMounts:
            - name: nginx-conf
              mountPath: /etc/nginx/nginx.conf
              subPath: nginx.conf
            - name: server-conf
              mountPath: /etc/nginx/conf.d/server.conf
              subPath: server.conf
        - name: busybox
          image: busybox:latest
          command: ["/bin/sh", "-c", "sleep 3600"]
          resources:
            requests:
              cpu: 100m
              memory: 100Mi
            limits:
              cpu: 100m
              memory: 100Mi
      volumes:
        - name: nginx-conf
          configMap:
            name: nginx-conf
            items:
              - key: nginx.conf
                path: nginx.conf
        - name: server-conf
          configMap:
            name: server-conf
            items:
              - key: server.conf
                path: server.conf
---
apiVersion: v1
kind: Service
metadata:
  name: sbdemo-nginx-service
  namespace: sbdemo-web
spec:
  type: ClusterIP
  ports:
    - name: "http-port"
      protocol: "TCP"
      port: 80
      targetPort: 80
  selector:
    app: sbdemo-nginx
```

```bash
# 実行結果
$ kubectl apply -f web.yaml
namespace/sbdemo-web created
configmap/nginx-conf created
configmap/server-conf created
deployment.apps/sbdemo-nginx created
service/sbdemo-nginx-service created
```

## 問題6: フロントLoadBalancer

次の要件を満たすmanifestを作成し、デプロイしてください。

- 要件
  - Service
    - 名前は`sbdemo-nginx-lb`
    - Namespaceは`sbdemo-web`
    - 対象のlabelは`app: sbdemo-nginx`
    - プロトコルは`TCP`
    - Portは`80`
    - clusterIPは`指定なし`で良い
    - typeは`LoadBalancer`

【回答例】

```yml
# manifest
apiVersion: v1
kind: Service
metadata:
  name: sbdemo-nginx-lb
  namespace: sbdemo-web
spec:
  type: LoadBalancer
  ports:
    - name: "http-port"
      protocol: "TCP"
      port: 80
      targetPort: 80
  selector:
    app: sbdemo-nginx
```

```bash
# 実行結果
$ kubectl apply -f lb.yaml
service/sbdemo-nginx-lb created
```

## 問題7: 接続確認

インターネット接続可能な端末のwebブラウザからさきほど確認したsbdemo-nginx-lbのEXTERNAL-IPにアクセスしてください。

```bash
# 実行結果
$ kubectl get svc -n sbdemo-web
NAME                   TYPE           CLUSTER-IP       EXTERNAL-IP                                                             PORT(S)        AGE
sbdemo-nginx-lb        LoadBalancer   10.100.17.117    a33b7e7de3d2f463198b3b10bc8cd11f-24885768.us-west-1.elb.amazonaws.com   80:31520/TCP   54ssbdemo-nginx-service   ClusterIP      10.100.129.108   <none>                                                                  80/TCP         2m1s
```

※ WEBブラウザから`http://a33b7e7de3d2f463198b3b10bc8cd11f-24885768.us-west-1.elb.amazonaws.com`に接続し、ログイン画面が表示されれば、接続成功です。
