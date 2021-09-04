[トップ](../README.md)  
前： [セキュリティコンテキストの設定](securitycontext.md)  

---

# PodSecurityPolicyによるポリシー定義

PodSecurityPolicyを使うとPodのセキュリティ設定を強制できます。ただし、PodSecurityPolicyはデフォルトで使用できないためkube-apiserverの設定で有効にします。なお。PodSecurityPolicyは将来的(v1.25)に削除される予定です。ポリシーの定義はOPAなどの別ツールを使用したほうが良いでしょう。

1. kube-apiserverの設定を修正しPodSecurityPolicyを使用できるようにしてください。

2. 以下を満たすPodSecurityPolicyを作成してください。([ヒント](https://kubernetes.io/docs/concepts/policy/pod-security-policy/#host-namespaces))

- 特権を禁止
- マウント許可するボリュームタイプはemptyDirのみ
- ホストネットワークの接続禁止
- rootの起動を禁止
- コンテナのrootファイルシステムの変更禁止 

3. 上記作成したPodSecurityPolicyを使用する`ClusterRole`を作成してください。

4. ns:testを作成してください。

5. ns:test内のすべてのServiceAccountで上記作成したPodSecurityPolicyを使用できるようにしてください。

6. ns:testでnginxのPodを起動してください。起動できないことを確認してください。
   
7. 原因を特定しns:testでnginxを起動できるようにマニフェストを修正しデプロイしてください。（nginxユーザのidは101です。）

8. 作成したPod、RBAC、Namespaec、PSPをすべて削除してください。

[*解答例*](../ans/podsecuritypolicy.md)  

---

次： [アドミッションコントローラの追加](addmissioncontroller.md)  
[トップ](../README.md)  
