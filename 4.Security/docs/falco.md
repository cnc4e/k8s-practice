[トップ](../README.md)  
前： [SealedSecretsをつかったsecretの暗号化](seald-secret.md)  

---

# Falcoによる挙動の監視

コンテナが不正な動きをしていないか監視するツールとして[Falco](https://falco.org/docs/)があります。

1. ワーカーノードにFalcoをインストールしてください。（[ヒント](https://falco.org/docs/getting-started/installation/)）

2. Flacoのログをsyslogに出力する様に設定してください。（すでにそう設定されていればとくに変えなくてよいです。）（[ヒント](https://falco.org/docs/configuration/)）

3. コンテナの中でパッケージマネジメントのプロセスが動いたら以下のようなメッセージが出力されるようにしてください。

```
Package management process launched in container (<時間> , <Pod名> , <コンテナ名>)
```

4. Falcoを起動してください。

5. nginxのPodをワーカーノードで起動してください。

6. nginxのPodに追加コマンドで`apt-get`を実行してください。

7. ワーカーノードのsyslogにFalcoが検知した動作のログが出ていることを確認してください。また、`Package management process launched in container`のあとのメッセージが設定した`<時間> , <Pod名> , <コンテナ名>`になっていることを確認してください。

8. Podを削除してください。

9. ワーカーノードのFlacoを停止してください。

[*解答例*](../ans/falco.md)  

---

次： [APIサーバーの監査ログ設定](api-audit.md)  
[トップ](../README.md)  
