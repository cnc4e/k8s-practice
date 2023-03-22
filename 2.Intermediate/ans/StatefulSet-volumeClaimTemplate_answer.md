# 回答例

## PVC指定の場合

1. 以下を満たすマニフェストを作成しデプロイしてください。

   - 要件
     - Deployment
       - 名前は`nginx-dp`
       - labelはすべて`app: nginx-dp`
       - replicasは `3`
       - Pod
         - 名前は`nginx`
         - イメージは`nginx:1.12`
         - volumeプラグインでPVC:nginx-dp-pvcを指定
         - 上記で定義したボリュームをコンテナの`/usr/share/nginx/html`にマウント
         - lifecycleの`postStart`で`["/bin/sh","-c","echo $HOSTNAME >> /usr/share/nginx/html/index.html"]`を実行
     - PVC
       - 名前は`nginx-dp-pvc`
       - storageClassNameは`指定しない`（デフォルトのStorageClassを使用する）
       - accessModesは`ReadWriteOnce`
       - ストレージ容量は`1Gi`

   【回答例】

   ```yml
   # manifest
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name:
       nginx-dp
   spec:
     replicas: 3
     selector:
       matchLabels:
         app: nginx-dp
     template:
       metadata:
         labels:
           app: nginx-dp
       spec:
         containers:
          - name: nginx
            image: nginx:1.12
            ports:
             - containerPort: 80
            lifecycle:
              postStart:
                exec:
                  command:
                    ["/bin/sh", "-c", "echo $HOSTNAME >> /usr/share/nginx/html/index.html"]
            volumeMounts:
            - name: nginx-dp-pvc
              mountPath: /usr/share/nginx/html
         volumes:
           - name: nginx-dp-pvc
             persistentVolumeClaim:
               claimName: nginx-dp-pvc
   ---
   apiVersion: v1
   kind: PersistentVolumeClaim
   metadata:
     name: nginx-dp-pvc
   spec:
     accessModes:
     - ReadWriteOnce
     resources:
       requests:
         storage: 1Gi
   ```

   ```bash
   # 実行結果
   $ kubectl apply -f nginx-dp.yaml
   deployment.apps/nginx-dp created
   persistentvolumeclaim/nginx-dp-pvc created
   ```

1. PVCおよびPVリソースのオブジェクト一覧を表示し、`nginx-dp-pvc`を含むものがそれぞれ`1つ`作成されていることを確認してください。

   【回答例】

   ```bash
   # 実行結果
   ## PV
   $ kubectl get pv
   NAME                                       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                  STORAGECLASS   REASON   AGE
   pvc-18ca8f20-622d-4b71-a4b9-b0302372b609   1Gi        RWO            Delete           Bound    default/nginx-dp-pvc   gp2                     49s

   ## PVC
   $ kubectl get pvc
   NAME           STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
   nginx-dp-pvc   Bound    pvc-18ca8f20-622d-4b71-a4b9-b0302372b609   1Gi        RWO            gp2            2m14s
   ```

1. Podリソースのオブジェクト一覧を表示し、3つのPodの名前および稼働するワーカーノードを確認してください。

   【回答例】

   ```bash
   # 実行結果
   $ kubectl get pod -o wide
   NAME                        READY   STATUS    RESTARTS   AGE   IP               NODE                                          NOMINATED NODE   READINESS    GATES
   nginx-dp-6984587896-7m8nn   1/1     Running   0          45s   192.168.27.110   ip-192-168-5-135.us-west-1.compute.internal   <none>           <none>
   nginx-dp-6984587896-svqlg   1/1     Running   0          45s   192.168.4.255    ip-192-168-5-135.us-west-1.compute.internal   <none>           <none>
   nginx-dp-6984587896-xd7jb   1/1     Running   0          45s   192.168.8.207    ip-192-168-5-135.us-west-1.compute.internal   <none>           <none>
    ```

1. いずれか1つのPodにログインし、次のコマンドを発行してください。

   ```bash
   echo $HOSTNAME > /usr/share/nginx/html/index.html
   cat /usr/share/nginx/html/index.html
   ```

   【回答例】

   ```bash
   $ kubectl exec -it nginx-dp-6984587896-7m8nn -- /bin/bash
   root@nginx-dp-6984587896-7m8nn:/#
   root@nginx-dp-6984587896-7m8nn:/# echo $HOSTNAME > /usr/share/nginx/html/index.html
   root@nginx-dp-6984587896-7m8nn:/#
   root@nginx-dp-6984587896-7m8nn:/# cat /usr/share/nginx/html/index.html
   nginx-dp-6984587896-7m8nn
   root@nginx-dp-6984587896-7m8nn:/#
   root@nginx-dp-6984587896-7m8nn:/# exit
   exit
   ```

1. 先ほどとは別のPodにログインし、次のコマンドを発行してください。表示されるホスト名が`最初にログインしたPod名が表示される`ことを確認してください。

   ```bash
   cat /usr/share/nginx/html/index.html
   ```

   【回答例】

   ```bash
   $ kubectl exec -it nginx-dp-6984587896-svqlg -- /bin/bash
   root@nginx-dp-6984587896-svqlg:/#
   root@nginx-dp-6984587896-svqlg:/# cat /var/lib/nginxql/host.txt
   nginx-dp-6984587896-7m8nn
   root@nginx-dp-6984587896-svqlg:/#
   root@nginx-dp-6984587896-svqlg:/# exit
   exit
   ```

1. Deploymet:nginx-dp、PVC:nginx-dp-pvcを削除してください。

   【回答例】

   ```bash
   $ kubectl delete -f nginx-dp.yaml
   deployment.apps "nginx-dp" deleted
   persistentvolumeclaim "nginx-dp-pvc" deleted
   ```

## volumeClaimTemplateの場合

1. 以下を満たすマニフェストを作成しデプロイしてください。volumeClaimTemplateについては[公式ドキュメント][1]を参考にしてください。

   - 要件
     - StatefulSet
       - 名前は`nginx-vct-sts`
       - labelはすべて`app: nginx`
       - replicasは `3`
       - serviceNameは`nginx-vct-svc`
       - Pod
         - 名前は`nginx-vct`
         - イメージは`nginx:1.12`
         - volumeプラグインでPVC:nginx-vct-pvcを指定
         - 上記で定義したボリュームをコンテナの`/usr/share/nginx/html`にマウント
         - lifecycleの`postStart`で`["/bin/sh","-c","echo $HOSTNAME >> /usr/share/nginx/html/index.html"]`を実行
       - volumeClaimTemplates
         - 名前はindex
         - storageClassNameは`指定しない`（デフォルトのStorageClassを使用する）
         - accessModesは`ReadWriteOnce`
         - ストレージ容量は`1Gi`

   【回答例】

   ```yml
   # manifest
   apiVersion: apps/v1
   kind: StatefulSet
   metadata:
     name: nginx-vct-sts
   spec:
     serviceName: nginx-vct-svc
     replicas: 3
     selector:
       matchLabels:
         app: nginx-vct-sts
     template:
       metadata:
         labels:
           app: nginx-vct-sts
       spec:
         containers:
           - name: nginx-vct
             image: nginx:1.12
             ports:
               - containerPort: 80
             lifecycle:
               postStart:
                 exec:
                   command:
                     [
                       "/bin/sh",
                       "-c",
                       "echo $HOSTNAME >> /usr/share/nginx/html/index.html",
                     ]
             volumeMounts:
               - name: index
                 mountPath: /usr/share/nginx/html
     volumeClaimTemplates:
       - metadata:
           name: index
         spec:
           accessModes: ["ReadWriteOnce"]
           storageClassName:
           resources:
             requests:
               storage: 1Gi
   ```

   ```bash
   # 実行結果
   $ kubectl apply -f nginx-vct.yaml
   statefulset.apps/nginx-vct-sts created
   ```

1. PVCおよびPVリソースのオブジェクト一覧を表示し、それぞれ`3つ`作成されていることを確認してください。

   【回答例】

   ```bash
   # 実行結果
   ## PV
   $ kubectl get pv
   NAME                                       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                           STORAGECLASS   REASON   AGE
   pvc-66d2bb4d-6e94-42b7-8ca5-2424e055603d   1Gi        RWO            Delete           Bound    default/index-nginx-vct-sts-1   gp2                     3m5s
   pvc-c7d544c7-5884-45e5-9d7a-396ad0314410   1Gi        RWO            Delete           Bound    default/index-nginx-vct-sts-0   gp2                     3m19s
   pvc-e856af9a-98f1-4070-86b3-5e6862ae4677   1Gi        RWO            Delete           Bound    default/index-nginx-vct-sts-2   gp2                     2m47s

   # PVC
   $ kubectl get pvc
   NAME                STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
   index-nginx-vct-sts-0   Bound    pvc-c7d544c7-5884-45e5-9d7a-396ad0314410   1Gi        RWO            gp2            4m14s
   index-nginx-vct-sts-1   Bound    pvc-66d2bb4d-6e94-42b7-8ca5-2424e055603d   1Gi        RWO            gp2            3m59s
   index-nginx-vct-sts-2   Bound    pvc-e856af9a-98f1-4070-86b3-5e6862ae4677   1Gi        RWO            gp2            3m42s
   ```

1. Podリソースのオブジェクト一覧を表示し、3つのPodの名前および稼働するワーカーノードを確認してください

   【回答例】

   ```bash
   # 実行結果
   $ kubectl get pod -o wide
   AME          READY   STATUS    RESTARTS   AGE     IP               NODE                                           NOMINATED NODE   READINESS GATES

   nginx-vct-sts-0   1/1     Running   0          4m30s   192.168.11.137   ip-192-168-5-135.us-west-1.compute.internal    <none>           <none>
   nginx-vct-sts-1   1/1     Running   0          4m16s   192.168.50.88    ip-192-168-40-85.us-west-1.compute.internal    <none>           <none>
   nginx-vct-sts-2   1/1     Running   0          3m59s   192.168.45.253   ip-192-168-60-176.us-west-1.compute.internal   <none>           <none>
   ```

1. 1つのPodにログインし、次のコマンドを発行してください。

   ```bash
   echo $HOSTNAME > /usr/share/nginx/html/index.html
   cat /usr/share/nginx/html/index.html
   ```

   【回答例】

   ```bash
   # 実行結果
   $ kubectl exec -it nginx-vct-sts-0 -- /bin/bash
   root@nginx-vct-sts-0:/#
   root@nginx-vct-sts-0:/# echo $HOSTNAME > /usr/share/nginx/html/index.html
   root@nginx-vct-sts-0:/#
   root@nginx-vct-sts-0:/# cat /usr/share/nginx/html/index.html
   nginx-vct-sts-0
   root@nginx-vct-sts-0:/#
   root@nginx-vct-sts-0:/# exit
   exit
   ```

1. 先ほどとは別のPodにログインし、次のコマンドを発行してください。index.htmlの内容が`異なる`ことを確認してください。

   ```bash
   cat /usr/share/nginx/html/index.html
   ```

   【回答例】

   ```bash
   # 実行結果
   $ kubectl exec -it nginx-vct-sts-1 -- /bin/bash
   root@nginx-vct-sts-1:/#
   root@nginx-vct-sts-1:/# cat /usr/share/nginx/html/index.html
   nginx-vct-sts-1
   root@nginx-vct-sts-1:/#
   root@nginx-vct-sts-1:/# exit
   exit
   ```

1. StatefulSet:nginx-vct-stsを削除してください。

   【回答例】

   ```bash
   $ kubectl delete statefulset nginx-vct-sts
   statefulset.apps "nginx-vct-sts" deleted
   ```

1. PVCおよびPVリソースのオブジェクト一覧を表示し、`削除されていない`ことを確認してください。

   【回答例】

   ```bash
   # 実行結果
   ## PV
   $ kubectl get pv
   NAME                                       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                           STORAGECLASS   REASON   AGE
   pvc-66d2bb4d-6e94-42b7-8ca5-2424e055603d   1Gi        RWO            Delete           Bound    default/index-nginx-vct-sts-1   gp2                     11m
   pvc-c7d544c7-5884-45e5-9d7a-396ad0314410   1Gi        RWO            Delete           Bound    default/index-nginx-vct-sts-0   gp2                     11m
   pvc-e856af9a-98f1-4070-86b3-5e6862ae4677   1Gi        RWO            Delete           Bound    default/index-nginx-vct-sts-2   gp2                     11m

   ## PVC
   $ kubectl get pvc
   NAME                    STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
   index-nginx-vct-sts-0   Bound    pvc-c7d544c7-5884-45e5-9d7a-396ad0314410   1Gi        RWO            gp2            12m
   index-nginx-vct-sts-1   Bound    pvc-66d2bb4d-6e94-42b7-8ca5-2424e055603d   1Gi        RWO            gp2            12m
   index-nginx-vct-sts-2   Bound    pvc-e856af9a-98f1-4070-86b3-5e6862ae4677   1Gi        RWO            gp2            11m
   ```

1. PVCおよびPVリソースを削除してください。

   【回答例】

   ```bash
   # 実行結果
   # kubectl delete pv,pvc --all
   persistentvolume "pvc-66d2bb4d-6e94-42b7-8ca5-2424e055603d" deleted
   persistentvolume "pvc-c7d544c7-5884-45e5-9d7a-396ad0314410" deleted
   persistentvolume "pvc-e856af9a-98f1-4070-86b3-5e6862ae4677" deleted
   persistentvolumeclaim "index-nginx-vct-sts-0" deleted
   persistentvolumeclaim "index-nginx-vct-sts-1" deleted
   persistentvolumeclaim "index-nginx-vct-sts-2" deleted
   ```

[1]:https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/#components

---

次： [ConfigMap(env)](ConfigMap-env.md)  
