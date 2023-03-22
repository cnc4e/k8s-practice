前： [ConfigMap(mount)](ConfigMap-mount.md)  

---

# Secret

SecretはConfigMapとほぼ一緒の使い方ができるリソースです。
値を環境変数として読み込んだり、ファイルをボリュームマウントすることが可能です。
違いはdata部分がbase64エンコードされるか否かです。

ConfigMapはbase64エンコードされないため、dataの内容がそのまま見えます。
Secretはdata部分がbase64でエンコードされ、暗号化されます。（ただし、base64デコードすることで簡単に復号できます。）
ログイン情報など、Pod外で管理したいが、値を隠したい情報を格納したい場合に使用します。（ただし繰り返しになりますが、単純にbase64エンコードしたのみなので、base64デコードで簡単に復号できます。）

# 演習

1. 以下を満たすマニフェストを作成しデプロイしてください。Secretリソースについては[公式ドキュメント][1]を参考にしてください。

   - 要件
     - Secret
       - 以下のKey-Valueをdataとして持つ（以下はbase64デコードされた状態）
         - password: "zettai ni sirarete ha ikenai jyouhou"
     - Deployment
       - 上記Secretのpasswordの値を環境変数PASSWORDに格納

1. 上記Deploymentで展開したPodに「echo $PASSWORD」の追加コマンドを発行してください。Secretの内容が表示されることを確認してください。

1. Secretの詳細を表示してください。dataのvalue部分が見えないことを確認してください。

1. Secretの内容をYAML形式で表示してください。passwordのvalueが表示されるのでコピーしてください。

1. 以下のコマンドを実行してください。valueの中身が復号されることを確認してください。

1. 作成したリソースを削除してください。

このように、SecretではK8sにアクセス可能な人ならだれでも値が知れてしまうので注意が必要です。
また、Gitなどにマニフェストを置く時はSecretをさらに暗号化する仕組み(kubesec,sealedsecret,vault)を併用しましょう。

以上で本演習は終了です。

具体的な操作およびその結果に関する回答例は[こちら](../ans/Secret_answer.md)にあります。
具体的な操作方法がわからなかった場合や、想定した結果にならなかった場合などに参照してください。

[1]:https://kubernetes.io/docs/concepts/configuration/secret/

---

次： [Job](Job.md)  
