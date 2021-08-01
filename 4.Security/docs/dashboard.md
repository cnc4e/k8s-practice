[トップ](../README.md)  
前： -  

---

# ダッシュボードの強化

K8sにはダッシュボードというWeb UIがある。k8s公式は[こちら](https://kubernetes.io/ja/docs/tasks/access-application-cluster/web-ui-dashboard/)、Gitレポジトリは[こちら](https://github.com/kubernetes/dashboard)。このダッシュボードのセキュリティについて知る。

1. ダッシュボードをクラスタにデプロイしてください。

2. ダッシュボードのserviceをtype:NodePortに修正してください。

3. workerのパブリックIP:NodePortでダッシュボードにアクセスしてください。**接続元の環境によってはNodePort(30000-32767)のアクセスが閉じられてアクセスできないかもしれません。アクセス元を替えてみてください。**

4. ログイン画面ではトークンを選択し、ダッシュボードを起動しているServiceAccountのトークンでログインしてください。(ヒント：ServiceAccountのトークンはSecretにあります。)

5. デフォルトではダッシュボードからリソース情報を取得する権限がないためリソースを表示できません。ダッシュボードを起動しているServiceAccountにClusterRole:viewを付けてください。ダッシュボードでns:kubernetes-dashboardのPod等の情報が表示できることを確認してください。

> ClusterRole:viewをClusterRoleBindingで付けた場合とRoleBindingで付けた場合とで参照できるリソースに違いがあります。前者はクラスタ全体のリソースを参照できますが後者の場合は同じNamespace内のリソースしか参照できません。

上記のようにデフォルトでhttpsやログインがあるため安全です。では逆に安全でない状態へしてみたいと思います。

[*解答例*](../ans/dashboard.md)  

---

次： [コンポーネントバイナリの検証](binary.md)  
[トップ](../README.md)  
