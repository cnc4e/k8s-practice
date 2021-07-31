[トップ](../README.md)  
前： -  

---

# ダッシュボードの強化

K8sにはダッシュボードというWeb UIがある。k8s公式は[こちら](https://kubernetes.io/ja/docs/tasks/access-application-cluster/web-ui-dashboard/)、Gitレポジトリは[こちら](https://github.com/kubernetes/dashboard)。

1. ダッシュボードをクラスタにデプロイしてください。

2. ダッシュボードにwebブラウザからhttpでアクセスしてください。（ヒント①：ダッシュボードのパラメータに[insecure-port](https://github.com/kubernetes/dashboard/blob/master/docs/common/dashboard-arguments.md)を追加します。ヒント②：ダッシュボードのServiceをNodePortにします。）**接続元の環境によってはNodePort(30000-32767)のアクセスが閉じられてアクセスできないかもしれません。アクセス元を替えてみてください。**

[*解答例*](../ans/dashboard.md)  

---

次： [コンポーネントバイナリの検証](binary.md)  
[トップ](../README.md)  
