# mod_security

## 概要

Apacheモジュールとして提供されているWeb Application Firewall。

## 導入方法

ソースコードからビルドする必要があるため、下記パッケージはインストール済みであること。

    gcc、make、gcc-c++、libcurl-devel、libxml2-devel、openldap-devel

ソースコード取得

    cd /usr/local/src
    git clone git://github.com/SpiderLabs/ModSecurity.git
    
    cd ModSecurity
    git checkout refs/tags/v2.8.0
    
ビルド

    sh autogen.sh
    sh configure
    
    make
     make install

/etc/httpd/modules/mod_security2.soが出来ていればOK

## 設定

/etc/httpd/conf/extra/modsecurity.conf

    <IfModule security2_module>
      SecServerSignature "Microsoft-IIS/8.5"
    </IfModule>

>SecServerSignatureをいじる場合は、Apacheの設定で```ServerTokens```が```Full```になっている必要がある