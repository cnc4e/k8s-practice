# 4.Security(セキュリティ)

セキュリティに特化した内容を扱います。

内容的にマスターコンポーネントの設定変更などが含まれます。そのため、EKS等のマネージドK8sだと実践できないものもあるかと思います。kubeadmで自前のクラスタを作成するのに参考となるterraformを用意しましたので活用ください。[k8s-practice-env](https://github.com/moriryota62/k8s-practice-env)

## 目次

- クラスタ設定
  - [4-01.ダッシュボードの強化](docs/dashboard.md)
  - [4-02.コンポーネントバイナリの検証](docs/binary.md)
  - [4-03.etcdの暗号化](docs/etcd.md)
  - [4-04.kube-benchによる構成の検証](docs/kube-bench.md)
  - [4-05.ingressでTLSの終端](docs/ingress-tls.md)
  - [4-06.NetworkPolicyによるアクセス制御](docs/networkpolicy.md)
  - [4-07.metadataサーバへのアクセス制御](docs/metadata-server.md)
- クラスタ強化
  - [4-08.特定の操作しかできないユーザを追加する](docs/user.md)
  - [4-09.ServiceAccountのトークン自動マウントを無効化、デフォルトの権限](docs/serviceaccount.md)
- システムの強化
  - [4-10.apparmorによるファイルアクセスの制限](docs/apparmor.md)
  - [4-11.seccompによるシステムコールの制限](docs/seccomp.md)
- マイクロサービスの脆弱性を抑える
  - [4-12.ファイルシステムのReadOnly化（&一部Write化）](docs/filesystem.md)
  - [4-13.セキュリティコンテキストの設定](docs/securitycontext.md)
  - [4-14.PodSecurityPolicyによるポリシー定義](docs/podsecuritypolicy.md)
  - [4-15.アドミッションコントローラの追加](docs/addmissioncontroller.md)
  - [4-16.より安全なコンテナランタイムの使用](docs/runtimeclass.md)
- サプライチェーンのセキュリティ
  - [4-17.イメージの脆弱性診断](docs/image-scan.md)
  - [4-18.SealedSecretsをつかったsecretの暗号化](docs/seald-secret.md)
- モニタリング、ロギング、ランタイムセキュリティ
  - [4-19.Falcoによる挙動の監視](docs/falco.md)
  - [4-20.APIサーバーの監査ログ設定](docs/api-audit.md)
