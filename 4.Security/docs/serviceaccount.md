[トップ](../README.md)  
前： [特定の操作しかできないユーザを追加する](user.md)  

---

# ServiceAccountのトークン自動マウントを無効化、デフォルトの権限

## トークンの自動マウントを無効化

ServiceAccountのトークンはデフォルトでは自動でPodにマウントされる。不要なPodにはServiceAccountのトークンをマウントさせないこともできる。

1. ns:defaultにsa:non-mount-tokenを作成してください。このSAは自動でトークンをマウントしない設定をしてください。(ヒント：[こちら](https://kubernetes.io/docs/tasks/configure-pod-container/configure-service-account/#use-the-default-service-account-to-access-the-api-server))

2. sa:non-mount-tokenでPodを起動してください。起動したPodにログインし、/run/secrets/kubernetes.io/serviceaccount配下にtokenが無いことを確認してください。（/run以下にsecretsがないはずです。）

3. 作成したPodをSAを削除してください。

## 広い範囲のServiceAccountに紐付ける

あるNamespaceに属するするべてのServiceAccountに共通する権限を設定することもできます。また、すべてのNamespaceのServiceAccountに対しても設定できます。

1. ns:defaultおよびns:defaultに適当なsecretおよびconfigmapを作成してください。

2. secretをgetできるclusterrole:get-secretを作成してください。また、configmapをgetできるclusterrole:get-configを作成してください。

3. ns:defaultのすべてのSAにclusterrole:get-secretを紐付けてください。また、すべてのNSのSAにclusterrole:get-configを紐付けてください。（ヒント：Namespace内の複数のSAを紐付けるにはグループで指定します。[これ](https://kubernetes.io/docs/reference/access-authn-authz/rbac/#referring-to-subjects)）

4. ns:defaultにsa:testでnginxのPodを作成してください。Podにログインし、apiserverにcurlし、configmapとsecretがgetできることを確認してください。（ヒント①：kubectlを使わずにapiserverと問い合わせるにはcurlします。[これ](https://kubernetes.io/docs/tasks/administer-cluster/access-cluster-api/#without-kubectl-proxy)　ヒント②：httpsの場合、--insecureではなく-kを指定します。）

5. ns:kube-configにsa:testでnginxのPodを作成してください。Podにログインし、apiserverにcurlし、configmapがgetできることを確認してください。secretはgetできない（403になってしまう）ことを確認してください。

6. 作成したPod、SA、RBAC関連リソースを削除してください。

[*解答例*](../ans/serviceaccount.md)  

---

次： [apparmorによるファイルアクセスの制限](apparmor.md)  
[トップ](../README.md)  
