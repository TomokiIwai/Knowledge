# 初期設定的なこと

## 7.x

### ネットワーク設定

まず、Minimalでインストールするとネットワークサービスが無効になっているっぽいので、有効化してネットワークサービスを再起動する。

    nmcli connection modify enp0s3 connection.autoconnect yes

自動起動が設定され、```/etc/sysconfig/network-scripts/ifcfg-enp0s3```へ書き込まれる。

    systemctl restart network.service

ネットワークサービスが再起動される。

    ip addr

```enp0s3```については、```ip addr```コマンドで確認する。正常にIPアドレスが取得できれば、```ip addr```で確認できる。

### yum設定

下準備

    yum -y install yum-plugin-priorities
    yum -y update
    yum -y groupinstall "Base" "Development tools" "Japanese Support"

リポジトリ追加(EPEL, Remi, RPMForge)

    yum -y install epel-release
    rpm -ivh http://rpms.famillecollet.com/enterprise/remi-release-7.rpm
    yum -y install http://pkgs.repoforge.org/rpmforge-release/rpmforge-release-0.5.3-1.el7.rf.x86_64.rpm

### SELInux無効化

再起動後に無効とする。

    vim /etc/selinux/config
    
    SELINUX=disabled

とりあえず、動きを阻害しないPassiveモードに変更しておく

    setenforce 0
    
    getenforce

