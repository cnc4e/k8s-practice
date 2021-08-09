[トップ](../README.md)  
前： [ServiceAccountのトークン自動マウントを無効化、デフォルトの権限](serviceaccount.md)  

---

**本プラクティスはubuntuのworkerを想定しています。**

# AppArmorによるファイルアクセスの制限

AppArmorはプログラムのファイルアクセスを制御する機能です。UbuntuやSUSEなどのディストリビューションで使用できます。CentやRHELには代わりにSELinuxがあります。

AppArmorを使うことでコンテナごとにアクセスできるファイルを制御できます。普段そのプログラムが使用するファイルにのみアクセス権を与え、それ以外へのアクセスを禁止します。これによりPodが乗っ取られたとしてもホスト上のファイルを保護できます。また、新たな脆弱性が発見されてもその被害を抑えることができます。AppArmorはプロファイルを定義します。プロファイルはホスト上に配置します。Podの設定でどのプロファイルをどのコンテナで使用するか設定できます。

1. Deploymentを作成してください。busyboxのイメージを使用したコンテナを2つ起動してください。どちらもworkerノードの/tmpをコンテナの/tmpにマウントしてください。コマンドは`sh -c sleep 3600`など指定してください。作成したら各コンテナがworkerの/tmpをマウントしており、書き込みも問題なくできることを確認してください。確認したらDeploymentは削除してください。（この時のマニフェストを残しておくと後で楽です。）

2. workerノードでAppArmorカーネルモジュールが有効になっていることを確認してください。また、AppArmorのサービスが起動していることを確認してください。（ヒント：[AppArmorを使用してコンテナのリソースへのアクセスを制限する](https://kubernetes.io/docs/tutorials/clusters/apparmor/)、ubuntuの場合はデフォルトで有効になっているはずです。）

3. workerノードにロード済のAppArmorプロファイルの一覧を表示してください。（ヒント：[AppArmorを使用してコンテナのリソースへのアクセスを制限する](https://kubernetes.io/docs/tutorials/clusters/apparmor/)）

4.  workerノードに以下AppArmorプロファイルをロードしてください。ロードしたらプロファイル一覧を表示し`k8s-apparmor-example-deny-write`がロードされていることを確認してください。（ヒント：[AppArmorを使用してコンテナのリソースへのアクセスを制限する](https://kubernetes.io/docs/tutorials/clusters/apparmor/)）

```
#include <tunables/global>

profile k8s-apparmor-example-deny-write flags=(attach_disconnected) {
  #include <abstractions/base>

  file,

  # Deny all file writes.
  deny /** w,
}
```
5. 再びDeploymentを作成してください。今度は2つある内の1のつのコンテナに対し、`k8s-apparmor-example-deny-write`プロファイルを適用して起動してください。作成したらプロファイルを適用したコンテナでは/tmpへ書き込みができないことを確認してください。もう1方のコンテナは問題なく/tmpに書き込みできることを確認してください。確認したら/tmp内に作成したテスト用ファイルを削除し、Deploymentも削除してください。（ヒント：[AppArmorを使用してコンテナのリソースへのアクセスを制限する](https://kubernetes.io/docs/tutorials/clusters/apparmor/)）


[*解答例*](../ans/apparmor.md)  

---

次： [seccompによるシステムコールの制限](seccomp.md)  
[トップ](../README.md)  
