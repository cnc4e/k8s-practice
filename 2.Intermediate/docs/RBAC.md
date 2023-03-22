前： [ResourceQuota](ResourceQuota.md)  

---

# K8sのアクセス制御

## K8sのアカウント

K8sにはアカウント（ユーザ）が2種類あります。

- ServiceAccount
  - K8sリソースの1つ。
  - Podが他のリソースを操作するための認証に使用する
- UserAccount
  - K8sの管理対象ではない。
  - 外部の仕組みで認証される（AWSのIAM、ActiveDirectoryなど）

## RBAC

K8sにはRBAC(Role Based Access Control)という仕組みがあります。
RBACは権限を表すルールを指定する`Role`とRoleとアカウントを紐付ける`RoleBinging`を作成し、アクセス制御を行います。
RBACはクラスタ操作権限を制限したい場合（例えば、admin権限と参照権限だけ持つアカウントを分ける、操作できるNamespaceの範囲を絞るなど）やK8sクラスタ内のPodからkube-apiserverへ通信したい場合（例えば、とあるPodからkubectlコマンドを打鍵して、Podの一覧を取得するなど）などに用います。

RBACに関するリソースは4つが存在します。

- Role
- ClusterRole
- RoleBinding
- ClusterRoleBinding

### Role/ClusterRole

Role/ClusterRoleは`どのリソースに対して`どのような`操作`を許可するのかを規定したルールです。

RoleとClusterRoleの違いはNamespaceで分離されるかどうかです。
Roleは特定のNamespace内でのみ使用可能なルールです。
ClusterRoleは、Roleと異なり、クラスタ全体で使用可能なルールです。

### RoleBinding/ClusterRoleBinding

RoleBinding/ClusterRoleBindingは`Role/ClusterRoleとアカウントの紐づけ`です。
Roleとの紐づけをRoleBinding、ClusterRoleとの紐づけをClusterRoleBindingと呼びます。

# 演習

1. 以下を満たすマニフェストを作成しデプロイしてください。

   - 要件
     - Deployment
       - イメージは`bitnami/kubectl`
       - replicas: 1
       - commandは["/bin/sh","-c","sleep 3600"]

1. デプロイしたPodに「kubectl get pod」の追加コマンドを発行し以下のようなメッセージで失敗することを確認してください。

   ```bash
   Error from server (Forbidden): pods is forbidden: User "system:serviceaccount:default:default" cannot list resource "pods" in API group "" in the namespace "default"
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

1. 作成したServiceAccount,Role,RoleBindingを確認してください。

1. Deploymentのマニフェストを修正しServiceAccount:get-podでPodを起動するように修正しデプロイしなおしてください。

1. デプロイしたPodに「kubectl get pod」の追加コマンドを発行しコマンドが`実行できる`ことを確認してください。

1. デプロイしたPodに「kubectl get deployment」の追加コマンドを発行し以下のようなメッセージで失敗することを確認してください。

   ```bash
   Error from server (Forbidden): deployments.extensions is forbidden: User "system:serviceaccount:default:get-pod" cannot list resource "deployments" in API group "extensions" in the namespace "default"
   ```

1. Deploymentのマニフェストを修正しServiceAccount:get-deployでPodを起動するように修正しデプロイしなおしてください。

1. デプロイしたPodに「kubectl get pod」の追加コマンドを発行しコマンドが`実行できない`ことを確認してください。

1. デプロイしたPodに「kubectl get deployment」の追加コマンドを発行しコマンドが`実行できる`ことを確認してください。

1. 1セット目のRoleBindingのマニフェストを修正し、紐付けるServiceAccountをget-podからget-deployに変更しデプロイしなおしてください。

1. デプロイしたPodに「kubectl get pod」の追加コマンドを発行しコマンドが`実行できる`ことを確認してください。

1. デプロイしたPodに「kubectl get deployment」の追加コマンドを発行しコマンドが`実行できる`ことを確認してください。

1. 作成したリソースを削除してください。

以上で本演習は終了です。

具体的な操作およびその結果に関する回答例は[こちら](../ans/RBAC_answer.md)にあります。
具体的な操作方法がわからなかった場合や、想定した結果にならなかった場合などに参照してください。

[1]:https://kubernetes.io/docs/tasks/configure-pod-container/configure-service-account/
[2]:https://kubernetes.io/docs/reference/access-authn-authz/rbac/#role-and-clusterrole
[3]:https://kubernetes.io/docs/reference/access-authn-authz/rbac/#rolebinding-and-clusterrolebinding

---

次： [章末問題](Practice.md)  
