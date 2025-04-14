[トップ](../README.md)  
前： [metadataサーバへのアクセス制御](metadata-server.md)  

---

# 特定の操作しかできないユーザーを追加する

K8sにはRole/RoleBindingを使ったRBACの仕組みがあります。これを使うことで特定の操作のみを許可したユーザーを作ることもできます。なお、K8sにはユーザーというリソースはありません。認証は外部の仕組みを使用します。

1. `add-user`という名前で認証されたユーザーにclusterrole:viewを割り当ててください

2. kubeconfigを`add-user`でアクセスするように修正し、add-userでリソースの一覧を取得してください。また、リソースの作成に失敗することを確認してください。（ヒント：`add-user`用の証明書と秘密鍵を作成します。証明書を署名する認証局はapiserverが認証に使用する認証局で行います。証明書の作成は以下コマンドを参考にしてください。apiserverでは--client-ca-fileで認証局の証明書を指定しています。）

``` sh
openssl genrsa -out <秘密鍵> 4096
openssl req -new -key <秘密鍵> -out <証明書要求>
openssl x509 -req -days 9999 -in <証明書要求> -CA <CA証明書> -CAkey <CA秘密鍵> -CAcreateserial -out <証明書>
```

3. kubeconfigを`kubernetes-admin`に戻してください。

4. Namespace:defaultにSecretリソースを作成、参照できるRoleを作成してください。

5. 作成したRoleを紐付け、`add-user`がNamespace:defaultにSecretを作成できるようにしてください。

6. kubeconfigを`add-user`に変更し、Namespace:defaultにSecretを作成してください。

7. kubeconfigを`kubernetes-admin`に戻し、作成したSecret、RBAC関連リソースを削除してください。


[*解答例*](../ans/user.md)  

---

次： [ServiceAccountのトークン自動マウントを無効化、デフォルトの権限](serviceaccount.md)  
[トップ](../README.md)  
