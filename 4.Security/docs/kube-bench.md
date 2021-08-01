[トップ](../README.md)  
前： [etcdの暗号化](etcd.md)  

---

# kube-benchによる構成の検証

kubernetes構成のセキュリティベストプラクティスとして[CIS Benchmark](https://www.cisecurity.org/benchmark/kubernetes/)があります。このベストプラクティスへの対応状況を検査するツールとして[kube-bench](https://github.com/aquasecurity/kube-bench)というものがあります。これを使って現在のクラスタのセキュリティ状況をチェックします。

## マスターのチェック

1. masterノードでkube-benchを実行してください。kube-benchはmasterをターゲットに実行してください。Controller ManagerおよびSchedulerに対するprofilingの設定に関する指摘を確認してください。

2. 上記指摘されたController ManagerおよびSchedulerに対するprofilingの設定を修正してください。

3. 再度masterノードでkube-benchを実行してください。kube-benchはmasterをターゲットに実行してください。Controller ManagerおよびSchedulerに対するprofilingの設定に関する指摘が出ていないことを確認してください。

## ワーカーのチェック

ワーカーも同じくチェック可能です。kube-benchの対象ターゲットを変えます。

1. workerノードでkube-benchを実行してください。kube-benchはnodeをターゲットに実行してください。

[*解答例*](../ans/kube-bench.md)  

---

次： [ingressでTLSの終端](ingress-tls.md)  
[トップ](../README.md)  
