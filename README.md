# K8s practice
## まえがき
K8s practice はこれからKubernetes(以下、K8s)に入門する方を対象とした練習問題集です。kubectlによる操作や基本的なリソースの使い方などの基本を押さえることを目的としています。テーマごとに動作や使い方を学ぶ問題を用意しています。問題の指示や公式ドキュメントを参考に自身で考えながら進めてください。

CKA/CKADの試験対策としても使えると思います。

## カテゴリについて
K8s practice は大きく以下の4つのカテゴリに分けています。カテゴリ順に進んでいくことを想定していますが、気になる/学びたいリソースのところだけ問題を解くこともできます。

|カテゴリ|内容|
|-|-|
|[Beiginner](1.Beginner)|初級カテゴリです。kubectlやPodなど基礎となるリソースについて扱います。|
|[Intermediate](2.Intermediate)|中級カテゴリです。K8s標準で使えるリソースを扱います。|
|[Advanced](3.Advanced)|上級カテゴリです。アドオンを追加することで使用できるリソースを扱います。|
|[Security](4.Security)|セキュリティに特化したカテゴリです。[CKS](https://training.linuxfoundation.org/ja/certification/certified-kubernetes-security-specialist/)で問われる内容について扱います。|

## 前提
K8sクラスタを用意してください。また、kubectlでクラスタに接続できる状態としてください。これらを準備する手順としては[Amazon EKSの開始方法](https://docs.aws.amazon.com/ja_jp/eks/latest/userguide/getting-started.html)などを参考にしてください。

なお、K8s practice の問題は EKS v1.14 をベースに作成しています。そのため、クラウドプロバイダもAWSを前提としています。他のマネージド・サービスやベアメタルのK8sでも問題を解くことはできると思いますが、クラウドプロバイダの部分はそれぞれのクラウドに置き換えてください。


