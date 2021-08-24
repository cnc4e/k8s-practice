[トップ](../README.md)  
前： [セキュリティコンテキストの設定](securitycontext.md)  

---

# PodSecurityPolicyによるポリシー定義

apiserverの設定をしてPodSecurityPolicyを使える様にする。

以下を満たすポリシーを定義する。

- 特権を禁止
- マウント許可するボリュームタイプはemptyDirのみ
- ホストネットワークの接続禁止
- rootの起動を禁止
- コンテナのrootファイルシステムの変更禁止 

上記ポリシーを使うRBACをつくる

作成したポリシーを使うPodを動かす。

ポリシーに違反する動作をさせてみる。

Podを削除する。


[*解答例*](../ans/podsecuritypolicy.md)  

---

次： [アドミッションコントローラの追加](addmissioncontroller.md)  
[トップ](../README.md)  
