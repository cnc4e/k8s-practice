# 2.Intermediate(中級)

## はじめに

中級では標準で利用できるさまざまなK8sのAPIリソースについて学んでいます。
また、クラウドプロバイダと連携した機能やPod設定の詳細な部分についてもこちらで触れます。

## manifestについて

初級では、使用するmanifestは全て用意してありましたが、
中級以降では公式ドキュメントなどを参考にして`自身でmanifestを作成`してもらう形式になります。
これは`公式ドキュメントを読み解く能力`を身に着けてもらいたいためです。

初級と比較するとぐっと難易度は上がりますが、試行錯誤しながら各章を進めてください。

## 目次

- クラウドプロバイダ連携機能
  - [2-01.PersistentVolume](docs/PersistentVolume.md)
  - [2-02.Service(LB)](docs/Service-LB.md)
- Pod設定
  - [2-03.resources](docs/Pod-resources.md)
  - [2-04.liveness/readiness Probe](docs/Pod-Probe.md)
  - [2-05.initContainer](docs/Pod-initContainer.md)
  - [2-06.nodeSelector](docs/Pod-nodeSelector.md)
  - [2-07.postStart/preStop](docs/Pod-lifecycle.md)
- K8sリソース
  - [2-08.Namespace](docs/Namespace.md)
  - [2-09.DaemonSet](docs/DaemonSet.md)
  - [2-10.StatefulSet](docs/StatefulSet.md)
  - [2-11.StatefulSet(volumeClaimTemplate)](docs/StatefulSet-volumeClaimTemplate.md)
  - [2-12.ConfigMap(env)](docs/ConfigMap-env.md)
  - [2-13.ConfigMap(mount)](docs/ConfigMap-mount.md)
  - [2-14.Secret](docs/Secret.md)
  - [2-15.Job](docs/Job.md)
  - [2-16.CronJob](docs/CronJob.md)
  - [2-17.LimitRange](docs/LimitRange.md)
  - [2-18.ResourceQuota](docs/ResourceQuota.md)
  - [2-19.RBAC](docs/RBAC.md)
- [章末問題](docs/Practice.md)
