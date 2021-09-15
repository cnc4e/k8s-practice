[トップ](../README.md)  
前： [より安全なコンテナランタイムの使用](runtimeclass.md)  

---

# イメージの脆弱性診断

脆弱性の少ないコンテナイメージを使用したほうが安全です。脆弱性は診断ツールを使って検査します。[trivy](https://github.com/aquasecurity/trivy)などが有名です。

1. trivyをインストールしてください。（[ヒント](https://aquasecurity.github.io/trivy/v0.19.2/getting-started/installation/)）

2. trivyを使用し`nginx`と`nginx:alpine`のイメージ脆弱性をスキャンしてください。`nginx:alpine`の方がより安全であることを確認してください。（[ヒント](https://hub.docker.com/r/aquasec/trivy)）

[*解答例*](../ans/image-scan.md)  

---

次： [SealedSecretsをつかったsecretの暗号化](seald-secret.md)  
[トップ](../README.md)  
