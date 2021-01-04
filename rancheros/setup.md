#  rancheros setup
- (100) インストール
- (101) isoイメージはusbにddしてブートできる。
- (102) ノートPCに入れるとき、RAM1Gではインストール途中で止まる。一度インストールしてしまえばRAM1Gでもブートできる。
- (103) インストールは、isoイメージでブートしておいて、以下のコマンド。
```bash
sudo ros install -c cloud-config.yml -d /dev/sda --append "rancher.password=<パスワード>"
```
- (104) 「-c cloud-config.yml」で構成を指定できるが、初回は自ホスト名を指定するだけの簡単なものでよい。viでカレントディレクトリに作成する。
```bash
vi cloud-config.yml

#cloud-config
hostname: roshost1
```
- (105) 「-d /dev/sda」でインストール先のディスクを指定する。
- (106) 「--append "rancher.password=<パスワード>"」で初回のログインパスワードを指定しておく。これを指定しておかないと、インストールしたシステムにログインできなくなると思う。

# rancheros config
- (201) 基本的な考え方は、まずベースのシステムが起動してきて、起動中にcloud-config.ymlで定義された内容で更新され、動作を開始する、ということだと思う。ベースのシステムを直接書き換えないので、システムの構成がすべてcloud-config.ymlに記述されており、管理しやすい。
- (202) cloud-config.ymlは/var/lib/rancher/conf以下にある。
- (203) cloud-config.ymlを更新するときは、いきなり編集したりせず、以下の手順を踏む。1)更新部分をymlファイルに記述する。2)構文をチェックする。3)cloud-config.ymlへマージする。
```
1)更新部分をymlファイルに記述する

vi rancheros.yml

2)構文をチェックする

sudo ros config validate -i rancheros.yml

3)cloud-config.ymlへマージする

sudo ros config merge -i rancheros.yml

```
- (204) cloud-config.ymlファイルへの更新内容は、再起動後に有効になる。

- (205) ファイルを追加するときは、cloud-config.ymlへwrite_filesを(203)の方法で追加する。例えば、ノートPCで蓋を閉めてもスリープしないようにするには、以下のようにする。
```yml
$ vi rancheros.yml

write_files:
- container: acpid
  content: |
    exit 0
  owner: root
  path: /etc/acpi/suspend.sh
  permissions: "0644"

$ sudo ros config validate -i rancheros.yml
$ sudo ros config merge -i rancheros.yml

再起動して有効にする。
```

- (206) cloud-config.ymlの内容を見るには、「ros config export」を使う。
