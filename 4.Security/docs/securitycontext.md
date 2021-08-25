[トップ](../README.md)  
前： [ファイルシステムのReadOnly化（&一部Write化）](filesystem.md)  

---

# セキュリティコンテキストの設定

セキュリティコンテキストでより細かいセキュリティ設定ができます。セキュリティコンテキストはPodレベルとコンテナレベルの設定があります。

1. 以下を満たすPodを作成してください。マニフェストは[ファイルシステムのReadOnly化（&一部Write化）](filesystem.md)で作成したものを流用して良いです。(ヒント：[PodレベルのSecurityContext](https://kubernetes.io/docs/reference/kubernetes-api/workload-resources/pod-v1/#security-context)、[ContainerレベルのSecurityContext](https://kubernetes.io/docs/reference/kubernetes-api/workload-resources/pod-v1/#security-context-1))

- コンテナのrootファイルシステムは書き込み禁止
- nginxの起動に必要な領域のみ書き込み可
- コンテナの起動ユーザはnginx(id:101)を指定
- 外部ボリュームはnginxグループ(gid:101)を指定
- コンテナは特権を許可しない

2. 作成したPodにログインしてください。ログイン時のユーザがnginxであることを確認してください。また、外部ボリュームをマウントした領域のグループがnginxになっていることを確認してください。

3. ワーカーノードでプロセスを確認しnginxのプロセスがid:101のユーザで起動していることを確認してください。（プロセスは101で起動していますがコンテナの中から見た時とホストから見た時では紐づくユーザが違います。）

4. Podを削除してください。

[*解答例*](../ans/securitycontext.md)  

---

次： [PodSecurityPolicyによるポリシー定義](podsecuritypolicy.md)  
[トップ](../README.md)  
