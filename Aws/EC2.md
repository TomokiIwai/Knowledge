# Amazon Web Service EC2

## OSにAmazon Linuxを採用する場合のリポジトリバージョンコントロール

Amazon Linuxはインスタンスの起動時にリポジトリを最新版へ更新するようになっている。これを抑止したい場合には、下記の設定をインスタンス作成時の```User Data```へ設定しておく必要がある。

    #cloud-config
    repo_releasever: 2013.09
    repo_upgrade: none

>コメントっぽいのも全部必要。バージョンは適宜変更

インスタンス構築後に、下記の通り確認できる。

    cat /etc/issue
    
    Amazon Linux AMI release 2013.09
    Kernel \r on an \m
