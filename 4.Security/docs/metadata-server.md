[トップ](../README.md)  
前： [NetworkPolicyによるアクセス制御](networkpolicy.md)  

---

# metadataサーバへのアクセス制御

メタデータはパブリッククラウドにおいてインスタンス等の情報です。このデータにアクセスすることでクラウド上のサーバ情報を知ることができます。Podが乗っ取られた場合を考えPodからメタデータへのアクセスを制限します。

1. nginxのPodを起動しそのPodから以下curlを実行しメタデータからホストの情報を取得できることを確認してください。（AWSの場合です。）

``` sh
curl -s http://169.254.169.254/latest/meta-data/
```

2. ネットワークポリシーを作成し、**メタデータへのアウトバウンドのみ**を制限してください。（ヒント：特定のIPのみ除外する場合は[except](https://kubernetes.io/docs/reference/kubernetes-api/policy-resources/network-policy-v1/)を使います。）

3. 再度nginxのPodからcurlを実行し、メタデータが取得できないことを確認してください。また、Podから他インターネット上のサービスにアクセスできることを確認してください。

4. PodおよびNetworkPolicyを削除してください。

[*解答例*](../ans/metadata-server.md)  

---

次： [特定の操作しかできないユーザを追加する](user.md)  
[トップ](../README.md)  
