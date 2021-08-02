[トップ](../README.md)  
前： -  

---

# ダッシュボードの強化

## ダッシュボードのデプロイ

K8sにはダッシュボードというWeb UIがある。k8s公式は[こちら](https://kubernetes.io/ja/docs/tasks/access-application-cluster/web-ui-dashboard/)、Gitレポジトリは[こちら](https://github.com/kubernetes/dashboard)。このダッシュボードのセキュリティについて知る。

1. ダッシュボードをクラスタにデプロイしてください。

2. ダッシュボードのserviceをtype:NodePortに修正してください。

3. workerのパブリックIP:NodePortでダッシュボードに`https`でアクセスしてください。**接続元の環境によってはNodePort(30000-32767)のアクセスが閉じられてアクセスできないかもしれません。アクセス元を替えてみてください。**

4. ログイン画面ではトークンを選択し、ダッシュボードを起動しているServiceAccountのトークンでログインしてください。(ヒント：ServiceAccountのトークンはSecretにあります。)

5. デフォルトではダッシュボードからリソース情報を取得する権限がないためリソースを表示できません。ダッシュボードを起動しているServiceAccountにClusterRole:viewを付けてください。ダッシュボードでns:kubernetes-dashboardのPod等の情報が表示できることを確認してください。

> ClusterRole:viewをClusterRoleBindingで付けた場合とRoleBindingで付けた場合とで参照できるリソースに違いがあります。前者はクラスタ全体のリソースを参照できますが後者の場合は同じNamespace内のリソースしか参照できません。

上記のようにデフォルトでhttpsやログインがあるため安全です。それでは逆に安全でない状態へとしてみたいと思います。

## ダッシュボードのセキュリティを弱める

デフォルトで安全ですがセキュリティを弱めるやってはいけない設定を試してみます。

1. ダッシュボードのDeplymentを以下の様に修正してください。（ヒント：[ダッシュボードのパラメータ](https://github.com/kubernetes/dashboard/blob/master/docs/common/dashboard-arguments.md)）

- auto-generate-certificatesを削除
- ダッシュボードPodのHTTP公開ポートとして9090を設定
- HTTPのアクセスを許可
- ログインをスキップ可
- livenessProbeを削除
- containerPortを9090に変更

2. ダッシュボードのServiceも修正しHTTPでもアクセスできる様に修正してください。

3. workerのパブリックIP:NodePortでダッシュボードに`http`でアクセスしてください。**接続元の環境によってはNodePort(30000-32767)のアクセスが閉じられてアクセスできないかもしれません。アクセス元を替えてみてください。**

4. ログイン画面はサインインボタンの横にある`スキップ`を選択してください。これでダッシュボードにログインできてしまい、クラスタ内の情報も見れてしまうことを確認してください。

5. ダッシュボード用に作成したRoleBinding(またはClusterRoleBinding)とダッシュボード関連のリソースをすべて削除してください。

この様に設定次第ではhttpでのアクセスや認証なしのログインができてしまいます。とくにインターネット経由でアクセスできる環境の場合、このような設定はしないようにしましょう。

[*解答例*](../ans/dashboard.md)  

---

次： [コンポーネントバイナリの検証](binary.md)  
[トップ](../README.md)  
