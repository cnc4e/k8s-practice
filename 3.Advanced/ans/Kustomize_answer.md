# 回答例

1. 以下構造のKustomizeディレクトリを作成してください。

   ```bash
   Kustomize
   ├── base
   └── overlay
       ├── prod
       └── dev
   ```

   【回答例】

   ```bash
   # 実行結果
   $ mkdir -p Kustomize/base

   $ mkdir -p Kustomize/overlay/prod

   $ mkdir -p Kustomize/overlay/dev

   $ tree Kustommize/
   Kustommize/
   |-- base
   `-- overlay
       |-- dev
       `-- prod
   ```

1. kustomize/base配下に以下を満たすマニフェストを作成してください。（applyは不要です。）

   - 要件
     - ConfigMap
       - 名前はenv-config
       - dataはENV: base
     - Deployment
       - replica:1
       - コンテナイメージはnginx:1.12
       - ConfigMap:env-configのENVを環境変数ENVに格納
     - Service
       - 上記DeploymentをClusterIP:80で公開

   【回答例】

   ```yml
   # manifest
   apiVersion: v1
   kind: ConfigMap
   metadata:
     name: env-configmap
   data:
      ENV: base
   ---
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: kustomize-nginx
   spec:
     replicas: 1
     selector:
       matchLabels:
         app: kustomize-nginx
     template:
       metadata:
         labels:
           app: kustomize-nginx
       spec:
         containers:
          - name: kustomize-nginx
            image: nginx:1.12
            envFrom:
              - configMapRef:
                  name: env-configmap
   ---
   apiVersion: v1
   kind: Service
   metadata:
     name: kustomize-nginx-svc
   spec:
     ports:
      - name: "http-port"
        protocol: "TCP"
        port: 80
        targetPort: 80
     selector:
       app: kustomize-nginx
   ```

1. 上記マニフェストをresourcesとして指定したkustomize/base/kustomization.yamlを作成してください。kustomizeについては[kustomize公式ドキュメント][1]や[k8s公式ドキュメント][2]を参考にしてください。

   > :information_source:  
   > 公式ドキュメントでは正直理解しにくいため、インターネットで検索して、調べたほうが早いかもしれません。

   【回答例】

   ```yml
   # manifest
   resources:
     - kustomize-nginx.yaml
   ```

1. 以下のコマンドでkustomizeでapplyしてください。（以下コマンドはkustomizeディレクトリで実行した場合）

   ```bash
   kubectl apply -k base/
   ```

   【回答例】

   ```bash
   # 実行結果
   $ kubectl apply -k base/
   configmap/env-configmap created
   service/kustomize-nginx-svc created
   deployment.apps/kustomize-nginx created
   ```

1. 作成したオブジェクトを確認し、以下コマンドで削除してください。（以下コマンドはkustomizeディレクトリで実行した場合）

   ```bash
   kubectl delete -k base/
   ```

   【回答例】

   ```bash
   # 実行結果
   $ kubectl get cm,pod,svc
   NAME                         DATA   AGE
   configmap/env-configmap      1      10m
   configmap/kube-root-ca.crt   1      3h25m

   NAME                                   READY   STATUS    RESTARTS   AGE
   pod/kustomize-nginx-7984bfcc47-2k2mq   1/1     Running   0          10m

   NAME                          TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)   AGE
   service/kubernetes            ClusterIP   10.100.0.1       <none>        443/TCP   3h25m
   service/kustomize-nginx-svc   ClusterIP   10.100.186.130   <none>        80/TCP    10m

   $ kubectl delete -k base/
   configmap "env-configmap" deleted
   service "kustomize-nginx-svc" deleted
   deployment.apps "kustomize-nginx" deleted
   ```

1. overlay/dev/およびoverlay/prod/に以下を満たすマニフェストおよびkustomization.yamlを作成してください。

   - 要件
     - prod
       - baseは../../base
       - 作成するオブジェクトのプレフィックスに`prod-`をつける
       - 作成するオブジェクトに`env: prod`のラベルを追加
       - ConfigMap:env-configを上書き（patches）し`ENV: prod`に変更してください。
     - dev
       - baseは../../base
       - 作成するオブジェクトのプレフィックスに`dev-`をつける
       - 作成するオブジェクトに`env: dev`のラベルを追加
       - ConfigMap:env-configを上書き（patches）し`ENV: dev`に変更してください。

   【回答例】

   ```yml
   # manifest (prod)
   ## configmap.yaml
   apiVersion: v1
   kind: ConfigMap
   metadata:
     name: env-configmap
   data:
     ENV: prod
   ```

   ```yml
   # manifest (prod)
   ## kustomization.yaml
   bases:
     - ../../base
   patches:
     - configmap.yaml

   namePrefix: prod-
   commonLabels:
     env: prod
   ```

   ```yml
   # manifest (dev)
   ## configmap.yaml
   apiVersion: v1
   kind: ConfigMap
   metadata:
     name: env-configmap
   data:
     ENV: dev
   ```

   ```yml
   # manifest (prod)
   ## kustomization.yaml
   bases:
     - ../../base
   patches:
     - configmap.yaml

   namePrefix: dev-
   commonLabels:
     env: dev
   ```

1. 以下のコマンドでprodとdevをデプロイしてください。（以下コマンドはkustomizeディレクトリで実行した場合）

   ``` sh
   kubectl apply -k overlay/prod
   kubectl apply -k overlay/dev
   ```

   【回答例】

   ```bash
   $ kubectl apply -k overlay/prod
   configmap/prod-env-configmap created
   service/prod-kustomize-nginx-svc created
   deployment.apps/prod-kustomize-nginx created

   $ kubectl apply -k overlay/dev
   configmap/dev-env-configmap created
   service/dev-kustomize-nginx-svc created
   deployment.apps/dev-kustomize-nginx created
   ```

1. 作成したオブジェクトを確認してください。prodおよびdevのPodに対して以下の追加コマンドを発行し、環境変数がオーバーライドできていることを確認してください。

   ```bash
   echo $ENV
   ```

   【回答例】

   ```bash
   # 実行結果
   $ kubectl get cm,pod,svc
   NAME                           DATA   AGE
   configmap/dev-env-configmap    1      7m37s
   configmap/kube-root-ca.crt     1      4h9m
   configmap/prod-env-configmap   1      7m45s

   NAME                                        READY   STATUS    RESTARTS   AGE
   pod/dev-kustomize-nginx-688f759667-c9xzk    1/1     Running   0          7m36s
   pod/prod-kustomize-nginx-67df57964c-dn8mt   1/1     Running   0          7m44s

   NAME                               TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)   AGE
   service/dev-kustomize-nginx-svc    ClusterIP   10.100.131.202   <none>        80/TCP    7m38s
   service/kubernetes                 ClusterIP   10.100.0.1       <none>        443/TCP   4h10m
   service/prod-kustomize-nginx-svc   ClusterIP   10.100.230.45    <none>        80/TCP    7m45s

   $ kubectl exec -it pod/prod-kustomize-nginx-67df57964c-dn8mt -- sh
   # echo $ENV
   prod

   $ kubectl exec -it pod/dev-kustomize-nginx-688f759667-c9xzk -- sh
   # echo $ENV
   dev
   ```

1. deploymentを以下の様にオーバーライドして再デプロイしてください。なお、オーバーライドする時のマニフェストは必要最小限の記載にすることに注意してください。

   - 要件
     - prod
       - replicas: 3
       - resources.requests.memory: 100Mi
     - dev
       - replicas: 2
       - resources.requests.memory: 50Mi

   【回答例】

   ```yml
   # manifest (prod)
   ## deployment.yaml
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: kustomize-nginx
   spec:
     replicas: 3
     template:
       spec:
         containers:
           - name: kustomize-nginx
             resources:
               requests:
                 memory: 100Mi
   ```

   ```yml
   # manifest (prod)
   ## kustomization.yaml
   bases:
     - ../../base
   patches:
     - configmap.yaml
     - deployment.yaml

   namePrefix: prod-
   commonLabels:
     env: prod
   ```

   ```yml
   # manifest (dev)
   ## deployment.yaml
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: kustomize-nginx
   spec:
     replicas: 2
     template:
       spec:
         containers:
           - name: kustomize-nginx
             resources:
               requests:
                 memory: 50Mi
   ```

   ```yml
   # manifest (dev)
   ## kustomization.yaml
   bases:
     - ../../base
   patches:
     - configmap.yaml
     - deployment.yaml

   namePrefix: dev-
   commonLabels:
     env: dev
   ```

   ```bash
   # 実行結果
   $ kubectl apply -k overlay/prod
   configmap/prod-env-configmap unchanged
   service/prod-kustomize-nginx-svc unchanged
   deployment.apps/prod-kustomize-nginx configured

   $ kubectl apply -k overlay/dev
   configmap/dev-env-configmap unchanged
   service/dev-kustomize-nginx-svc unchanged
   deployment.apps/dev-kustomize-nginx configured
   ```

1. 上記オーバーライドした内容が反映されていることを確認してください。

   【回答例】

   ```bash
   # 実行結果
   $ kubectl get pod
   NAME                                    READY   STATUS    RESTARTS   AGE
   dev-kustomize-nginx-6776c5cf6-6lgv5     1/1     Running   0          9m59s
   dev-kustomize-nginx-6776c5cf6-qtxst     1/1     Running   0          10m
   prod-kustomize-nginx-8559c46c56-nrftj   1/1     Running   0          11m
   prod-kustomize-nginx-8559c46c56-rcckn   1/1     Running   0          11m
   prod-kustomize-nginx-8559c46c56-zfbwd   1/1     Running   0          11m

   $ kubectl describe pod prod-kustomize-nginx-8559c46c56-nrftj
   Name:         prod-kustomize-nginx-8559c46c56-nrftj
   Namespace:    default
        (略)
   Containers:
        (略)
       Requests:
         memory:  100Mi
        (略)

   $ kubectl describe pod dev-kustomize-nginx-6776c5cf6-6lgv5
   Name:         dev-kustomize-nginx-6776c5cf6-6lgv5
   Namespace:    default
        (略)
   Containers:
        (略)
       Requests:
         memory:  50Mi
        (略)
   ```

1. Podの/usr/share/nginx/html/index.htmlの内容を環境変数ENVの値とし、各環境で表示を変えます。kustomize/base配下のみを修正し、実装してください。

   【回答例】

   ```yml
   # manifest (base)
   apiVersion: v1
   kind: ConfigMap
   metadata:
     name: env-configmap
   data:
     ENV: base
   ---
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: kustomize-nginx
   spec:
     replicas: 1
     selector:
       matchLabels:
         app: kustomize-nginx
     template:
       metadata:
         labels:
           app: kustomize-nginx
       spec:
         containers:
           - name: kustomize-nginx
             image: nginx:1.12
             envFrom:
               - configMapRef:
                   name: env-configmap
             lifecycle:
               postStart:
                 exec:
                   command:
                     ["/bin/sh", "-c", "echo $ENV > /usr/share/nginx/html/index.html"]
   ---
   apiVersion: v1
   kind: Service
   metadata:
     name: kustomize-nginx-svc
   spec:
     ports:
       - name: "http-port"
         protocol: "TCP"
         port: 80
         targetPort: 80
     selector:
       app: kustomize-nginx
   ```

   ```bash
   # 実行結果
   $ kubectl apply -k overlay/prod
   configmap/prod-env-configmap unchanged
   service/prod-kustomize-nginx-svc unchanged
   deployment.apps/prod-kustomize-nginx configured

   $ kubectl apply -k overlay/dev
   configmap/dev-env-configmap unchanged
   service/dev-kustomize-nginx-svc unchanged
   deployment.apps/dev-kustomize-nginx configured
   ```

1. curlが実行可能なPodを展開し、prodとdevそれぞれにcurlしてください。各環境名が表示されること

   【回答例】

   ```bash
   # 実行結果
   $ kubectl run --image=appropriate/curl --restart=Never testpod sleep 3600
   pod/testpod created

   $ kubectl exec testpod -- curl -s http://prod-kustomize-nginx-svc
   prod

   $ kubectl exec testpod -- curl -s http://dev-kustomize-nginx-svc
   dev
   ```

1. 作成したリソースを削除してください。

   【回答例】

   ```bash
   # 実行結果
   $ kubectl delete -k overlay/prod
   configmap "prod-env-configmap" deleted
   service "prod-kustomize-nginx-svc" deleted
   deployment.apps "prod-kustomize-nginx" deleted

   $ kubectl delete -k overlay/dev
   configmap "dev-env-configmap" deleted
   service "dev-kustomize-nginx-svc" deleted
   deployment.apps "dev-kustomize-nginx" deleted
   ```

[1]:https://github.com/kubernetes-sigs/kustomize
[2]:https://kubernetes.io/docs/tasks/manage-kubernetes-objects/kustomization/
