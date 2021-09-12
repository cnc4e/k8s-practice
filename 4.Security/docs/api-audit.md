[トップ](../README.md)  
前： [Falcoによる挙動の監視](falco.md)  

---

# APIサーバーの監査ログ設定

kube-apiserverに監査ログを設定できます。監査ログを設定することでクラスタに対して誰が何をしたか記録できます。

1. 以下イベントをそれぞれ指定した監査レベルで記録するルールファイルをmasterノード上に作成してください。（[ヒント](https://kubernetes.io/docs/tasks/debug-application-cluster/audit/)）

- `RequestReceived`ステージのあらゆるイベントを記録しない。
- あらゆるNamespaceにあるSecretをGetするイベント発生時、Metadataレベルで記録
- ns:testにあるすべてのPodのlogに対するイベント発生時、Requestレベルで記録
- ns:testにあるpod:testに対するイベント発生時、RequestResponseレベルで記録
- 上記以外記録しない。

2. 上記ルールファイルを使用してkube-apiserverの監査ログを設定してください。また、監査ログは`/var/log/kube-api/audit.log`に保管し、ログの保持期間を10日、ログの最大世代数を3つ、ログファイルの最大容量は10MBとして設定してください。（[ヒント](https://kubernetes.io/docs/tasks/debug-application-cluster/audit/)）

3. 任意のnsでsecretを作成してください。ns:testを作成してください。ns:testにpod:testを作成してください。また、作成したPodのログを確認してください。

4. masterノードの`/var/log/kube-api/audit.log`を確認してください。SecretおよびPodとlogに関するイベントがログに出力されていることを確認してください。

5. ns:test以外でPodを作成しPodのログを確認してください。

6. masterノードの`/var/log/kube-api/audit.log`を確認してください。上記操作がログに出力されていないことを確認してください。

7. ns:testのすべてのリソース、任意のnsに作成したsecret、podを削除してください。

8. kube-apiserverの監査ログ設定を無効にしてください。

[*解答例*](../ans/api-audit.md)  

---

次： -  
[トップ](../README.md)  
