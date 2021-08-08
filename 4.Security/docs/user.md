[トップ](../README.md)  
前： [metadataサーバへのアクセス制御](metadata-server.md)  

---

# 特定の操作しかできないユーザを追加する

K8sにはrole/rolebindingを使ったRBACの仕組みがあります。これを使うことで特定の操作のみを許可したユーザを作ることもできます。なお、K8sにはユーザというリソースはありません。認証は外部の仕組みを使用します。

1. `add-user`という名前で認証されたユーザにclusterrole:viewを割り当ててください

2. kubeconfigを`add-user`でアクセスする様に修正し、add-userでリソースの一覧を取得してください。また、リソースの作成に失敗することを確認してください。（ヒント：`add-user`用の証明書と秘密鍵を作成します。証明書を署名する認証局はapiserverが認証に使用する認証局で行います。apiserverでは--client-ca-fileで認証局の証明書を指定しています。）

3. kubeconfigを`kubernetes-admin`に戻してください。

4. ns:defaultにsecretリソースを作成、参照できるroleを作成してください。

5. 作成したroleを紐付け、`add-user`がns:defaultにsecretを作成できるようにしてください。

6. kubeconfigを`add-user`に変更し、ns:defaultにsecretを作成してください。

7. kubeconfigを`kubernetes-admin`に戻し、作成したsecret、RBAC関連リソースを削除してください。


[*解答例*](../ans/user.md)  

---

次： [ServiceAccountのトークン自動マウントを無効化、デフォルトの権限](serviceaccount.md)  
[トップ](../README.md)  
