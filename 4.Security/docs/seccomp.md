[トップ](../README.md)  
前： [apparmorによるファイルアクセスの制限](apparmor.md)  

---

# seccompによるシステムコールの制限

seccompを使うとPodが使えるシステムコールを制限できます。どのシステムコールを使うか洗い出すのはとても大変なため、[docker-slim](https://github.com/docker-slim/docker-slim)を使ってプロファイルの雛形を作成します。

1. ワーカーノードで以下のseccompプロファイル(json)を作成してください。プロファイルは`/var/lib/kubelet/seccomp`ディレクトリ以下に作成してください。これはすべてのシステムコールを禁止します。

``` json
{
    "defaultAction": "SCMP_ACT_ERRNO"
}
```

2. 上記作成したseccompプロファイルを使ってnignxを動かすPodを起動してください。([ヒント](https://kubernetes.io/docs/tutorials/clusters/seccomp/)) Podが起動に失敗することを確認してください。

3. ワーカーノードに[docker-slim](https://github.com/docker-slim/docker-slim)をインストールしてください。

4. docker-slimを使ってnginxのseccompプロファイルを作成してください。
   
5. 作成したプロファイルを`/var/lib/kubelet/seccomp`ディレクトリ以下に移動してください。また、`fstatfs`のシステムコールを許可するルールを追加してください。

6. docker-slimで作成したseccompプロファイルを指定してnginxを動かすPodを起動してください。起動できることを確認してください。

7. Podを削除してください。

[*解答例*](../ans/seccomp.md)  

---

次： [ファイルシステムのReadOnly化（&一部Write化）](filesystem.md)  
[トップ](../README.md)  
