[トップ](../README.md)  
前： [seccompによるシステムコールの制限](seccomp.md)  

---

# ファイルシステムのReadOnly化（&一部Write化）

コンテナのファイルシステムをReadOnlyにすることでコンテナが乗っ取られた場合の影響を抑えることができます。

1. コンテナのrootファイルシステムをReadOnlyにしたnginxのPodを起動してください。起動に失敗し、失敗した原因を確認してください。（[ヒント](https://kubernetes.io/docs/reference/kubernetes-api/workload-resources/pod-v1/)）

2. Podを削除してください。

3. rootファイルシステムをReadOnlyに設定したままnginxのPodを起動できるようにしてください。（ヒント：書き込みしたい領域に外部ボリュームをマウントすれば良いです。）

4. Podを削除してください。

[*解答例*](../ans/filesystem.md)  

---

次： [セキュリティコンテキストの設定](securitycontext.md)  
[トップ](../README.md)  
