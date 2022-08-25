# K8s practice

## まえがき

K8s practice はこれからKubernetes(以下、K8s)に入門する方を対象とした練習問題集です。
kubectlを使用した操作や基本的なリソースの使い方などの基本を押さえることを目的としています。
テーマごとに動作や使い方を学ぶ問題を用意しています。
問題の指示や公式ドキュメントを参考に自身で考えながら進めてください。

CKA/CKADの試験対策としても使えると思います。

## カテゴリについて

K8s practice は大きく以下の4つのカテゴリに分けています。
カテゴリ順に進んでいくことを想定していますが、気になる/学びたいリソースのところだけ問題を解くこともできます。

|カテゴリ|内容|
|-|-|
|[Beginner](1.Beginner)|初級カテゴリです。kubectlやPodなど基礎となるリソースについて扱います。|
|[Intermediate](2.Intermediate)|中級カテゴリです。K8s標準で使えるリソースを扱います。|
|[Advanced](3.Advanced)|上級カテゴリです。アドオンを追加することで使用できるリソースを扱います。|
|[Security](4.Security)|セキュリティに特化したカテゴリです。[CKS](https://training.linuxfoundation.org/ja/certification/certified-kubernetes-security-specialist/)で問われる内容について扱います。|

## 前提

以下に本問題集を実施するための前提条件を記載します。

### 実習環境

本問題集ではkubectlのインストールやK8sクラスタ作成などの実習環境の構築は取り扱いません。
実習環境は別途用意して、kubectlでクラスタに接続できる状態としてください。

これらを準備する手順としては[Amazon EKSの開始方法](https://docs.aws.amazon.com/ja_jp/eks/latest/userguide/getting-started.html)などを参考にしてください。

なおEKSクラスタで使用するノードは`マネージド型ノード`を選択してください。
AWS Fargateは機能制約があるため、Amazon EBSを使用した問題など、一部の練習問題が実施できません。

### 各コンポーネントのバージョン

本問題集のベース環境は以下の通りです。
その他のマネージド・サービスやベアメタルのK8sでも問題を解くことはできると思いますが、
その場合は環境差異にあたる部分を適宜読み替えてください。

| 環境 | バージョン |
|-|-|
| kubectl | 1.23 |
|Amazon Elastic Kubernetes Service (EKS) | 1.22 |
