[トップ](../README.md)  
前： [コンポーネントバイナリの検証](binary.md)  

---

# etcdの暗号化

etcdの中にはSecretの値など機密情報が含まれます。etcdに直接アクセスできるとそういった機密情報を盗み見ることができます。etcdを暗号化すれば直接アクセスしてもetcd内の情報を見ることができなくなります。

## etcd内のデータ参照

1. genericタイプのSecretを作成してください。値は任意で良いです。

2. masterノードからetcdctlを使ってetcdにアクセスし、上記作成したSecretの中身を表示してください。Secretの内容が見えてしまうことを確認してください。（ヒント①：[etcdの操作](https://kubernetes.io/docs/tasks/administer-cluster/configure-upgrade-etcd/#securing-communication)、ヒント②：etcdへ接続するための証明書情報はkube-apiserverがマウントしています。ヒント③：etcd内のSecretリソースのパスは`/registry/secrets/<ns名>/<secret名>`です。）

ここで作成したsecretは消さないでおいてください、後ほどまた使います。

## etcdの暗号化

暗号化にはEncryptionConfigurationを使います。これはマニフェストで定義します。暗号化方式をprovidersで定義できます。providersの順番には注意が必要です。一番上にあるproviderが暗号化に使用されます。一番上に暗号化しないidentityを定義するとEncryptionConfigurationを定義しても暗号化されません。二番目以降のproviderは読み込み時に使用されます。[ネタ元](https://kubernetes.io/docs/tasks/administer-cluster/encrypt-data/#understanding-the-encryption-at-rest-configuration)

また、暗号化はetcdに書き込まれる時に行われるため、すでにetcd内にあるデータは暗号化されません。

1. masterノードの/etc/kubernetes配下にetcdディレクトリを作成してください。

2. masterノードで以下を満たすマニフェストを作成してください。マニフェストは上記作成したetcdディレクトリ以下に配置してください。（ヒント：[こちら](https://kubernetes.io/docs/tasks/administer-cluster/encrypt-data/#encrypting-your-data)）

- kind: EncryptionConfiguration
- 暗号化対象はSecret
- 暗号化の方式はaescbc,identity。必ずaescbcを上に書くこと
- 暗号化のキー名とシークレットキーの値は任意で設定。ただし、シークレットキーはbase64エンコードすること。エンコード前の文字数は16、24、32文字のいずれかであること。

3. masterノードにあるkube-apiserverのマニフェストをバックアップとして/etc/kubernetes/manifests/以外にコピーしてください。
   
4. kube-apiserverに`encryption-provider-config`のオプションを追加してください。対象として上記作成したマニフェストファイルを指定してください。また、masterノードの/etc/kubernetes/etcdをマウントしてください。（kube-apiserverはスタティックPodのためマニフェストを更新すると自動で再作成されます。再作成中はapiserverに接続できません。しばらくたってもapiserverと通信できない場合はマニフェストの記述に誤りがあるかもしれません。）

5. ふたたびgenericタイプのSecretを作成してください。値は任意で良いです。

6. 上記作成したSecretをetcdctlで表示してください。今度はSecretの内容が暗号化されていることを確認してください。

7. etcdctlでetcd暗号化前に作成したsecretの情報を表示してください。暗号化されていないままであることを確認してください。

8. etcd内のsecretをすべて暗号化してください。その後ふたたびetcdctlでetcd暗号化前に作成したsecretの情報を表示し、内容が暗号化されていることを確認してください。（ヒント：[こちら](https://kubernetes.io/docs/tasks/administer-cluster/encrypt-data/#ensure-all-secrets-are-encrypted)）

9.  kubectlを使いSecretの情報をyaml形式で出力してください。Secret内の情報が見えることを確認してください。（このように、etcdを暗号化してもkubectl経由だと値を見ることができます。）

## etcdの復号化

復号にするにはEncryptionConfigurationの一番上のproviderにidentityを設定し、etcd

1. EncryptionConfigurationのマニフェストを修正しaescbcとidentityの順番を逆にしてください。（identityが一番上）

2. kube-apiserverに`encryption-provider-config`のオプションをコメントアウトしてください。しばらくするとkube-apiserverが再起動します。再起動した再度`encryption-provider-config`のオプションをつけてkube-apiserverを起動してください。

3. etcd内のsecretをすべて復号してください。その後作成したSecret内の情報をetcdctlで表示してください。復号されていることを確認してください。（ヒント:[こちら](https://kubernetes.io/docs/tasks/administer-cluster/encrypt-data/#decrypting-all-data)）

4. apiserverのマニフェストをバックアップから復元してもとに戻してください。

5. 作成したSecret2つを削除してください。

[*解答例*](../ans/etcd.md)  

---

次： [kube-benchによる構成の検証](kube-bench.md)  
[トップ](../README.md)  
