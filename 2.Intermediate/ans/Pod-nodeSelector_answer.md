# 回答例

1. クラスタの参加ノードを確認し、ワーカーノードが2台以上存在していることを確認してください。

   【回答例】

   ```bash
   # 実行結果
   $ kubectl get node
   NAME                                           STATUS   ROLES    AGE   VERSION
   ip-192-168-40-85.us-west-1.compute.internal    Ready    <none>   18d   v1.22.17-eks-48e63af
   ip-192-168-5-135.us-west-1.compute.internal    Ready    <none>   18d   v1.22.17-eks-48e63af
   ip-192-168-60-176.us-west-1.compute.internal   Ready    <none>   18d   v1.22.17-eks-48e63af
   ```

1. ノードに付与されたラベルを表示してください。

   【回答例】

   ```bash
   # 実行結果
   $ kubectl get node --show-labels
   NAME                                           STATUS   ROLES    AGE   VERSION                LABELS
   ip-192-168-40-85.us-west-1.compute.internal    Ready    <none>   18d   v1.22.17-eks-48e63af   alpha.eksctl.io/cluster-name=k8s-practice,alpha.eksctl.io/nodegroup-name=ng-9c2e75c1,beta.kubernetes.io/arch=amd64,beta.kubernetes.io/instance-type=t3.medium,beta.kubernetes.io/os=linux,eks.mazonaws.com/capacityType=ON_DEMAND,eks.amazonaws.com/nodegroup-image=ami-031dbcbd53df1bd92,eks.amazonaws.com/nodegroup=ng-9c2e75c1,eks.amazonaws.com/sourceLaunchTemplateId=lt-01641ac2d4fc5edc7,eks.amazonaws.com/sourceLaunchTemplateVersion=1,failure-domain.beta.kubernetes.io/region=us-west-1,failure-domain.beta.kubernetes.io/zone=us-west-1c,k8s.io/cloud-provider-aws=332fe4c6de0222fd19fe146d29242c3a,kubernetes.io/arch=amd64,kubernetes.io/hostname=ip-192-168-40-85.us-west-1.compute.internal,kubernetes.io/os=linux,node-spec=monster,node.kubernetes.io/instance-type=t3.medium,topology.kubernetes.io/region=us-west-1,topology.kubernetes.io/zone=us-west-1c
   ip-192-168-5-135.us-west-1.compute.internal    Ready    <none>   18d   v1.22.17-eks-48e63af   alpha.eksctl.io/cluster-name=k8s-practice,alpha.eksctl.io/nodegroup-name=ng-9c2e75c1,beta.kubernetes.io/arch=amd64,beta.kubernetes.io/instance-type=t3.medium,beta.kubernetes.io/os=linux,eks.amazonaws.com/capacityType=ON_DEMAND,eks.amazonaws.com/nodegroup-image=ami-031dbcbd53df1bd92,eks.amazonaws.com/nodegroup=ng-9c2e75c1,eks.amazonaws.com/sourceLaunchTemplateId=lt-01641ac2d4fc5edc7,eks.amazonaws.com/sourceLaunchTemplateVersion=1,failure-domain.beta.kubernetes.io/region=us-west-1,failure-domain.beta.kubernetes.io/zone=us-west-1a,k8s.io/cloud-provider-aws=332fe4c6de0222fd19fe146d29242c3a,kubernetes.io/arch=amd64,kubernetes.io/hostname=ip-192-168-5-135.us-west-1.compute.internal,kubernetes.io/os=linux,node.kubernetes.io/instance-type=t3.medium,topology.kubernetes.io/region=us-west-1,topology.kubernetes.io/zone=us-west-1a
   ip-192-168-60-176.us-west-1.compute.internal   Ready    <none>   18d   v1.22.17-eks-48e63af   alpha.eksctl.io/cluster-name=k8s-practice,alpha.eksctl.io/nodegroup-name=ng-9c2e75c1,beta.kubernetes.io/arch=amd64,beta.kubernetes.io/instance-type=t3.medium,beta.kubernetes.io/os=linux,eks.amazonaws.com/capacityType=ON_DEMAND,eks.amazonaws.com/nodegroup-image=ami-031dbcbd53df1bd92,eks.amazonaws.com/nodegroup=ng-9c2e75c1,eks.amazonaws.com/sourceLaunchTemplateId=lt-01641ac2d4fc5edc7,eks.amazonaws.com/sourceLaunchTemplateVersion=1,failure-domain.beta.kubernetes.io/region=us-west-1,failure-domain.beta.kubernetes.io/zone=us-west-1c,k8s.io/cloud-provider-aws=332fe4c6de0222fd19fe146d29242c3a,kubernetes.io/arch=amd64,kubernetes.io/hostname=ip-192-168-60-176.us-west-1.compute.internal,kubernetes.io/os=linux,node.kubernetes.io/instance-type=t3.medium,topology.kubernetes.io/region=us-west-1,topology.kubernetes.io/zone=us-west-1c
   ```

1. どれか一台のノードに`node-spec=monster`のラベルを付与してください。

   【回答例】

   ```bash
   # 実行結果
   $ kubectl label nodes ip-192-168-40-85.us-west-1.compute.internal node-spec=monster
   node/ip-192-168-40-85.us-west-1.compute.internal labeled
   ```

1. 他のノードには`node-spec=normal`のラベルを付与してください。

   【回答例】

   ```bash
   # 実行結果
   $ kubectl label nodes ip-192-168-5-135.us-west-1.compute.internal node-spec=normal
   node/ip-192-168-5-135.us-west-1.compute.internal labeled
   $ kubectl label nodes ip-192-168-60-176.us-west-1.compute.internal node-spec=normal
   node/ip-192-168-60-176.us-west-1.compute.internal labeled
   ```

1. key:node-specをカラムに追加してノードの一覧で表示してください。一台のノードにmonsterのvalueがあることを確認してください。

   【回答例】

   ```bash
   # 実行結果
   $ kubectl get node -L node-spec
   NAME                                           STATUS   ROLES    AGE   VERSION                NODE-SPEC
   ip-192-168-40-85.us-west-1.compute.internal    Ready    <none>   18d   v1.22.17-eks-48e63af   monster
   ip-192-168-5-135.us-west-1.compute.internal    Ready    <none>   18d   v1.22.17-eks-48e63af   normal
   ip-192-168-60-176.us-west-1.compute.internal   Ready    <none>   18d   v1.22.17-eks-48e63af   normal
   ```

1. 以下を満たすマニフェストを作成しデプロイしてください。なお、nodeSelectorについては[公式ドキュメント][1]を参考にしてください。

   - 要件
     - Deployment
       - 名前は`nginx`
       - replicas: `1`
       - labelはすべて`app: nginx`
       - Pod
         - 名前は`nginx`
         - イメージは`nginx:1.12`
       - nodeSelector
         - ラベル`node-spec`にvalue`monster`が付与されたノードを指定

   【回答例】

   ```yml
   # manifest
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
         nodeSelector:
           node-spec: monster
   ```

   ```bash
   # 実行結果
   $ kubectl apply -f nginx.yaml
   deployment.apps/nginx created
   ```

1. Podリソースの一覧をwideで表示し、Podが`node-spec:monster`のラベルを付与したノードで起動していることを確認してください。

   【回答例】

   ```bash
   # 実行結果
   $ kubectl get pod -o wide
   NAME                     READY   STATUS    RESTARTS   AGE   IP               NODE                                          NOMINATED NODE   READINESS GATES
   nginx-7c56bf788c-wvgmd   1/1     Running   0          49s   192.168.37.167   ip-192-168-40-85.us-west-1.compute.internal   <none>           <none>
   ```

1. Deployment:node-selectorのPodレプリカ数を10に変更してください。新たに作成されたPodもすべて`node-spec:monster`のラベルを付与したノードで起動していることを確認してください。

   【回答例】

   ```yml
   # manifest
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: nginx
   spec:
     replicas: 10
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
         nodeSelector:
           node-spec: monster
   ```

   ```bash
   # 実行結果
   ## 設定修正
   $ kubectl apply -f nginx.yaml
   deployment.apps/nginx configured

   ## Pod確認
   $ kubectl get pod -o wide
   NAME                     READY   STATUS    RESTARTS   AGE     IP               NODE                                          NOMINATED NODE   READINESS GATES
   nginx-7c56bf788c-2dv6l   1/1     Running   0          38s     192.168.33.58    ip-192-168-40-85.us-west-1.compute.internal   <none>           <none>
   nginx-7c56bf788c-8slh2   1/1     Running   0          38s     192.168.55.180   ip-192-168-40-85.us-west-1.compute.internal   <none>           <none>
   nginx-7c56bf788c-cp767   1/1     Running   0          38s     192.168.61.111   ip-192-168-40-85.us-west-1.compute.internal   <none>           <none>
   nginx-7c56bf788c-khtns   1/1     Running   0          38s     192.168.49.129   ip-192-168-40-85.us-west-1.compute.internal   <none>           <none>
   nginx-7c56bf788c-kvrg4   1/1     Running   0          38s     192.168.42.244   ip-192-168-40-85.us-west-1.compute.internal   <none>           <none>
   nginx-7c56bf788c-qqsqt   1/1     Running   0          38s     192.168.50.88    ip-192-168-40-85.us-west-1.compute.internal   <none>           <none>
   nginx-7c56bf788c-r24jp   1/1     Running   0          38s     192.168.61.27    ip-192-168-40-85.us-west-1.compute.internal   <none>           <none>
   nginx-7c56bf788c-v5wv2   1/1     Running   0          38s     192.168.32.39    ip-192-168-40-85.us-west-1.compute.internal   <none>           <none>
   nginx-7c56bf788c-wvgmd   1/1     Running   0          2m39s   192.168.37.167   ip-192-168-40-85.us-west-1.compute.internal   <none>           <none>
   nginx-7c56bf788c-wwdw4   1/1     Running   0          38s     192.168.35.188   ip-192-168-40-85.us-west-1.compute.internal   <none>           <none>
   ```

1. Deployment:nginxのnodeSelectorをkey`node-spec`にvalue`normal`が付与されたノード指定に変更してください。

   【回答例】

   ```yml
   # manifest
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: nginx
   spec:
     replicas: 10
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
         nodeSelector:
           node-spec: normal
   ```

   ```bash
   # 実行結果
   $ kubectl apply -f nginx.yaml
   deployment.apps/nginx configured
   ```

1. Podのローリングアップデートが実行され、すべてのPodが`node-spec:normal`のラベルを付与したノードで起動していることを確認してください。

   【回答例】

   ```bash
   # 実行結果
   $ kubectl get pod -o wide
   NAME                     READY   STATUS    RESTARTS   AGE   IP               NODE                                           NOMINATED NODE   READINESS GATES
   nginx-655b864556-4gf6r   1/1     Running   0          35s   192.168.11.137   ip-192-168-5-135.us-west-1.compute.internal    <none>           <none>
   nginx-655b864556-74755   1/1     Running   0          35s   192.168.27.110   ip-192-168-5-135.us-west-1.compute.internal    <none>           <none>
   nginx-655b864556-d4nsx   1/1     Running   0          35s   192.168.43.138   ip-192-168-60-176.us-west-1.compute.internal   <none>           <none>
   nginx-655b864556-kd76d   1/1     Running   0          35s   192.168.20.188   ip-192-168-5-135.us-west-1.compute.internal    <none>           <none>
   nginx-655b864556-kh5w7   1/1     Running   0          30s   192.168.4.255    ip-192-168-5-135.us-west-1.compute.internal    <none>           <none>
   nginx-655b864556-m8k65   1/1     Running   0          32s   192.168.54.123   ip-192-168-60-176.us-west-1.compute.internal   <none>           <none>
   nginx-655b864556-mdx7k   1/1     Running   0          28s   192.168.50.254   ip-192-168-60-176.us-west-1.compute.internal   <none>           <none>
   nginx-655b864556-mnflp   1/1     Running   0          35s   192.168.45.253   ip-192-168-60-176.us-west-1.compute.internal   <none>           <none>
   nginx-655b864556-mvsv9   1/1     Running   0          32s   192.168.42.61    ip-192-168-60-176.us-west-1.compute.internal   <none>           <none>
   nginx-655b864556-z45h8   1/1     Running   0          32s   192.168.8.207    ip-192-168-5-135.us-west-1.compute.internal    <none>           <none>
   ```

1. Deployment:nginxをいったん削除してください。

   【回答例】

   ```bash
   # 実行結果
   $ kubectl delete -f nginx.yaml
   deployment.apps "nginx" deleted
   ```

1. Deployment:nginxのnodeSelectorをkey`node-spec`にvalue`super`が付与されたノード指定に変更しデプロイしてください。

   【回答例】

   ```yml
   # manifest
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: nginx
   spec:
     replicas: 10
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
         nodeSelector:
           node-spec: super
   ```

   ```bash
   # 実行結果
   $ kubectl apply -f nginx.yaml
   deployment.apps/nginx created
   ```

1. すべてのPodのSTATUSが`Pending`となることを確認してください。

   【回答例】

   ```bash
   # 実行結果
   $ kubectl get pod -o wide
   NAME                     READY   STATUS    RESTARTS   AGE   IP       NODE     NOMINATED NODE   READINESS GATES
   nginx-54db8bd786-7fjxc   0/1     Pending   0          28s   <none>   <none>   <none>           <none>
   nginx-54db8bd786-7tqbt   0/1     Pending   0          28s   <none>   <none>   <none>           <none>
   nginx-54db8bd786-b6sjh   0/1     Pending   0          28s   <none>   <none>   <none>           <none>
   nginx-54db8bd786-ll2nz   0/1     Pending   0          28s   <none>   <none>   <none>           <none>
   nginx-54db8bd786-ngqgg   0/1     Pending   0          28s   <none>   <none>   <none>           <none>
   nginx-54db8bd786-npsq8   0/1     Pending   0          28s   <none>   <none>   <none>           <none>
   nginx-54db8bd786-pk54f   0/1     Pending   0          28s   <none>   <none>   <none>           <none>
   nginx-54db8bd786-ptvfd   0/1     Pending   0          28s   <none>   <none>   <none>           <none>
   nginx-54db8bd786-w84tx   0/1     Pending   0          28s   <none>   <none>   <none>           <none>
   nginx-54db8bd786-z9qxx   0/1     Pending   0          28s   <none>   <none>   <none>           <none>
   ```

1. Deployment:nginxを削除してください。

   【回答例】

   ```bash
   # 実行結果
   $ kubectl delete -f nginx.yaml
   deployment.apps "nginx" deleted
   ```

1. ノードに付与されたkey`node-spec`のラベルを削除してください。

   【回答例】

   ```bash
   # 実行結果
   $ kubectl label nodes ip-192-168-40-85.us-west-1.compute.internal node-spec-
   node/ip-192-168-40-85.us-west-1.compute.internal unlabeled
   $ kubectl label nodes ip-192-168-5-135.us-west-1.compute.internal node-spec-
   node/ip-192-168-5-135.us-west-1.compute.internal unlabeled
   $ kubectl label nodes ip-192-168-60-176.us-west-1.compute.internal node-spec-
   node/ip-192-168-60-176.us-west-1.compute.internal unlabeled
   ```
