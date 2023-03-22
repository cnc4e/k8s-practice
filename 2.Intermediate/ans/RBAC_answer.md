# 回答例

1. 以下を満たすマニフェストを作成しデプロイしてください。

   - 要件
     - Deployment
       - イメージは`bitnami/kubectl`
       - replicas: 1
       - commandは["/bin/sh","-c","sleep 3600"]

   【回答例】

   ```yml
   # manifest
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: kubectl
   spec:
     replicas: 1
     selector:
       matchLabels:
         app: kubectl
     template:
       metadata:
         labels:
           app: kubectl
       spec:
         containers:
           - name: kubectl
             image: bitnami/kubectl
             command: ["/bin/sh","-c","sleep 3600"]
   ```

   ```bash
   # 実行結果
   $ kubectl apply -f kubectl.yaml
   deployment.apps/kubectl created
   ```

1. デプロイしたPodに「kubectl get pod」の追加コマンドを発行し以下のようなメッセージで失敗することを確認してください。

   ```bash
   Error from server (Forbidden): pods is forbidden: User "system:serviceaccount:default:default" cannot list resource "pods" in API group "" in the namespace "default"
   ```

   【回答例】

   ```bash
   # 実行結果
   $ kubectl get pod
   NAME                       READY   STATUS    RESTARTS   AGE
   kubectl-85bfb44556-m2qvt   1/1     Running   0          68s

   $ kubectl exec kubectl-85bfb44556-m2qvt -- kubectl get pod
   Error from server (Forbidden): pods is forbidden: User "system:serviceaccount:default:default" cannot list resource "pods" in API group "" in the namespace    "default"
   command terminated with exit code 1
   ```

1. 以下を満たすマニフェストを作成しデプロイしてください。ServiceAccountリソースについては[公式ドキュメント][1]を参考にしてください。Roleリソースについては[公式ドキュメント][2]を参考にしてください。RoleBindingリソースについては[公式ドキュメント][3]を参考にしてください。

   - 要件
     - 1セット目
       - ServiceAccount
         - 名前はget-pod
       - Role
         - `CoreAPI`グループ内の`pods`リソースに対する`get`と`list`を許可する
       - RoleBinding
         - 上記、get-podとRoleを紐付ける
     - 2セット目
       - ServiceAccount
         - 名前はget-deploy
       - Role
         - `extensions`および`apps`グループ内の`deployments`リソースに対する`get`と`list`を許可する
       - RoleBinding
         - 上記、get-deployとRoleを紐付ける

   【回答例】

   ```yml
   # manifest(1セット目)
   apiVersion: v1
   kind: ServiceAccount
   metadata:
     name: get-pod
     namespace: default
   ---
   apiVersion: rbac.authorization.k8s.io/v1
   kind: Role
   metadata:
     name: get-pod-role
   rules:
   - apiGroups: [""]
     resources: ["pods"]
     verbs: ["get", "list"]
   ---
   apiVersion: rbac.authorization.k8s.io/v1
   kind: RoleBinding
   metadata:
     name: get-pod-rolebinding
   roleRef:
     apiGroup: rbac.authorization.k8s.io
     kind: Role
     name: get-pod-role
   subjects:
   - kind: ServiceAccount
     name: get-pod
     namespace: default
   ---
   # manifest(2セット目)
   apiVersion: v1
   kind: ServiceAccount
   metadata:
     name: get-deploy
     namespace: default
   ---
   apiVersion: rbac.authorization.k8s.io/v1
   kind: Role
   metadata:
     name: get-deploy-role
   rules:
   - apiGroups: ["extensions", "apps"]
     resources: ["deployments"]
     verbs: ["get", "list"]
   ---
   apiVersion: rbac.authorization.k8s.io/v1
   kind: RoleBinding
   metadata:
     name: get-deploy-rolebinding
   roleRef:
     apiGroup: rbac.authorization.k8s.io
     kind: Role
     name: get-deploy-role
   subjects:
   - kind: ServiceAccount
     name: get-deploy
     namespace: default
   ```

   ```bash
   # 実行結果
   $ kubectl apply -f rbac.yaml
   serviceaccount/get-pod created
   role.rbac.authorization.k8s.io/get-pod-role created
   rolebinding.rbac.authorization.k8s.io/get-pod-rolebinding created
   serviceaccount/get-deploy created
   role.rbac.authorization.k8s.io/get-deploy-role created
   rolebinding.rbac.authorization.k8s.io/get-deploy-rolebinding created
   ```

1. 作成したServiceAccount,Role,RoleBindingを確認してください。

   【回答例】

   ```bash
   # 実行結果
   $ kubectl get sa,role,rolebinding
   NAME                        SECRETS   AGE
   serviceaccount/default      1         1d
   serviceaccount/get-deploy   1         72s
   serviceaccount/get-pod      1         73s

   NAME                                             CREATED AT
   role.rbac.authorization.k8s.io/eks:az-poller     2023-01-01T16:21:24Z
   role.rbac.authorization.k8s.io/get-deploy-role   2023-01-21T22:56:17Z
   role.rbac.authorization.k8s.io/get-pod-role      2023-01-21T22:56:15Z

   NAME                                                           ROLE                   AGE
   rolebinding.rbac.authorization.k8s.io/eks:az-poller            Role/eks:az-poller     1d
   rolebinding.rbac.authorization.k8s.io/get-deploy-rolebinding   Role/get-deploy-role   72s
   rolebinding.rbac.authorization.k8s.io/get-pod-rolebinding      Role/get-pod-role      73s
   ```

1. Deploymentのマニフェストを修正しServiceAccount:get-podでPodを起動するように修正しデプロイしなおしてください。

   【回答例】

   ```yml
   # manifest
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: kubectl
   spec:
     replicas: 1
     selector:
       matchLabels:
         app: kubectl
     template:
       metadata:
         labels:
           app: kubectl
       spec:
         serviceAccountName: get-pod
         containers:
           - name: kubectl
             image: bitnami/kubectl
             command: ["/bin/sh","-c","sleep 3600"]
   ```

   ```bash
   # 実行結果
   $ kubectl apply -f kubectl.yaml
   deployment.apps/kubectl configured
   ```

1. デプロイしたPodに「kubectl get pod」の追加コマンドを発行しコマンドが`実行できる`ことを確認してください。

   【回答例】

   ```bash
   # 実行結果
   $ kubectl get pod
   NAME                       READY   STATUS    RESTARTS   AGE
   kubectl-86f8ddfcc8-nvzbg   1/1     Running   0          68s

   $ kubectl exec kubectl-86f8ddfcc8-nvzbg -- kubectl get pod
   NAME                       READY   STATUS    RESTARTS   AGE
   kubectl-86f8ddfcc8-nvzbg   1/1     Running   0          10m
   ```

1. デプロイしたPodに「kubectl get deployment」の追加コマンドを発行し以下のようなメッセージで失敗することを確認してください。

   ```bash
   Error from server (Forbidden): deployments.extensions is forbidden: User "system:serviceaccount:default:get-pod" cannot list resource "deployments" in API group "extensions" in the namespace "default"
   ```

   【回答例】

   ```bash
   # 実行結果
   $ kubectl exec kubectl-86f8ddfcc8-nvzbg -- kubectl get deploy
   Error from server (Forbidden): deployments.apps is forbidden: User "system:serviceaccount:default:get-pod" cannot list resource "deployments" in API group "apps" in the namespace "default"
   command terminated with exit code 1
   ```

1. Deploymentのマニフェストを修正しServiceAccount:get-deployでPodを起動するように修正しデプロイしなおしてください。

   【回答例】

   ```yml
   # manifest
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: kubectl
   spec:
     replicas: 1
     selector:
       matchLabels:
         app: kubectl
     template:
       metadata:
         labels:
           app: kubectl
       spec:
         serviceAccountName: get-deploy
         containers:
           - name: kubectl
             image: bitnami/kubectl
             command: ["/bin/sh","-c","sleep 3600"]
   ```

   ```bash
   # 実行結果
   $ kubectl apply -f kubectl.yaml
   deployment.apps/kubectl configured
   ```

1. デプロイしたPodに「kubectl get pod」の追加コマンドを発行しコマンドが`実行できない`ことを確認してください。

   【回答例】

   ```bash
   # 実行結果
   $ kubectl get pod
   NAME                       READY   STATUS    RESTARTS   AGE
   kubectl-58c594d468-rs7kf   1/1     Running   0          87s

   $ kubectl exec kubectl-58c594d468-rs7kf -- kubectl get pod
   Error from server (Forbidden): pods is forbidden: User "system:serviceaccount:default:get-deploy" cannot list resource "pods" in API group "" in the namespace "default"
   command terminated with exit code 1
   ```

1. デプロイしたPodに「kubectl get deployment」の追加コマンドを発行しコマンドが`実行できる`ことを確認してください。

   【回答例】

   ```bash
   $ kubectl exec kubectl-58c594d468-rs7kf -- kubectl get deploy
   NAME      READY   UP-TO-DATE   AVAILABLE   AGE
   kubectl   1/1     1            1           15m
   ```

1. 1セット目のRoleBindingのマニフェストを修正し、紐付けるServiceAccountをget-podからget-deployに変更しデプロイしなおしてください。

   【回答例】

   ```yml
   # manifest(1セット目)
   apiVersion: v1
   kind: ServiceAccount
   metadata:
     name: get-pod
     namespace: default
   ---
   apiVersion: rbac.authorization.k8s.io/v1
   kind: Role
   metadata:
     name: get-pod-role
   rules:
   - apiGroups: [""]
     resources: ["pods"]
     verbs: ["get", "list"]
   ---
   apiVersion: rbac.authorization.k8s.io/v1
   kind: RoleBinding
   metadata:
     name: get-pod-rolebinding
   roleRef:
     apiGroup: rbac.authorization.k8s.io
     kind: Role
     name: get-pod-role
   subjects:
   - kind: ServiceAccount
     name: get-deploy
     namespace: default
   ---
   # manifest(2セット目)
   apiVersion: v1
   kind: ServiceAccount
   metadata:
     name: get-deploy
     namespace: default
   ---
   apiVersion: rbac.authorization.k8s.io/v1
   kind: Role
   metadata:
     name: get-deploy-role
   rules:
   - apiGroups: ["extensions", "apps"]
     resources: ["deployments"]
     verbs: ["get", "list"]
   ---
   apiVersion: rbac.authorization.k8s.io/v1
   kind: RoleBinding
   metadata:
     name: get-deploy-rolebinding
   roleRef:
     apiGroup: rbac.authorization.k8s.io
     kind: Role
     name: get-deploy-role
   subjects:
   - kind: ServiceAccount
     name: get-deploy
     namespace: default
   ```

   ```bash
   # 実行結果
   $ kubectl apply -f rbac.yaml
   serviceaccount/get-pod unchanged
   role.rbac.authorization.k8s.io/get-pod-role unchanged
   rolebinding.rbac.authorization.k8s.io/get-pod-rolebinding configured
   serviceaccount/get-deploy unchanged
   role.rbac.authorization.k8s.io/get-deploy-role unchanged
   rolebinding.rbac.authorization.k8s.io/get-deploy-rolebinding unchanged
   ```

1. デプロイしたPodに「kubectl get pod」の追加コマンドを発行しコマンドが`実行できる`ことを確認してください。

   【回答例】

   ```bash
   $ kubectl exec kubectl-58c594d468-rs7kf -- kubectl get pod
   NAME                       READY   STATUS    RESTARTS   AGE
   kubectl-58c594d468-rs7kf   1/1     Running   0          4m27s
   ```

1. デプロイしたPodに「kubectl get deployment」の追加コマンドを発行しコマンドが`実行できる`ことを確認してください。

   【回答例】

   ```bash
   $ kubectl exec kubectl-58c594d468-rs7kf -- kubectl get deploy
   NAME      READY   UP-TO-DATE   AVAILABLE   AGE
   kubectl   1/1     1            1           17m
   ```

1. 作成したリソースを削除してください。

   【回答例】

   ```bash
   # 実行結果
   $ kubectl delete -f kubectl.yaml
   deployment.apps "kubectl" deleted

   $ kubectl delete -f rbac.yaml
   serviceaccount "get-pod" deleted
   role.rbac.authorization.k8s.io "get-pod-role" deleted
   rolebinding.rbac.authorization.k8s.io "get-pod-rolebinding" deleted
   serviceaccount "get-deploy" deleted
   role.rbac.authorization.k8s.io "get-deploy-role" deleted
   rolebinding.rbac.authorization.k8s.io "get-deploy-rolebinding" deleted
   ```

[1]:https://kubernetes.io/docs/tasks/configure-pod-container/configure-service-account/
[2]:https://kubernetes.io/docs/reference/access-authn-authz/rbac/#role-and-clusterrole
[3]:https://kubernetes.io/docs/reference/access-authn-authz/rbac/#rolebinding-and-clusterrolebinding
