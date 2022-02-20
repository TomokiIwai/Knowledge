# Apache の設定ファイル

## 概要

ディストリビューションにより、httpd.confやapache2.confといった名前の場合がある。

## 基本的な書き方

コメント

    # This is comment.


外部設定ファイル

    Include subconf/*.conf

>パスの指定は```ServerRoot```からの相対パス


## 主な項目の意味

| 項目 | 意味 | 設定例 |
| ---- | ---- | ------ |
| ThreadsPerChild | サーバーが常時起動するスレッド数。 | ThreadsPerChild 250 | 
| MaxRequestsPerChild | プロセス再起動タイミング。メモリリークなどが発生しているアプリケーションが存在している場合に備えるなどの意味がある。0を設定すると再起動は行われない。 | MaxRequestsPerChild 0 | 
| ServerRoot | サーバーの設定ファイル、ログファイルなどを格納するディレクトリのルートディレクトリ設定。 | ServerRoot "/etc/httpd" | 
| Listen | リクエストを受け付けるIPとPortを指定することができる。 | Listen 80 | 
| LoadModule | DSOモジュールの組み込みを行う。 | LoadModule python_module modules/mod_python.so | 
| ServerAdmin | アドミニストレーターのメールアドレス。エラーページなどで表示される。 | ServerAdmin fooAdmin@foo.co.jp | 
| ServerName | リダイレクト時に指定するサーバー名もしくはIPアドレス。[参考](http://his.luky.org/ML/linux-users.5/msg04998.html) | ServerName 219.5.172.12 | 
| DocumentRoot | ドキュメントを設置するルートディレクトリ。シンボリック、エイリアスなどを使えば他階層ディレクトリへアクセスさせることも可能。 | DocumentRoot "/var/www/Wikitten" | 
| CustomLog | アクセスログファイルの位置とフォーマットを指定する。 | CustomLog logs/access_log common | 
| LogFormat | ログフォーマットのニックネーム指定 | LogFormat "%h %l %u %t \\"%r\\" %>s %b" common | 
| DefaultType | サーバがデフォルトで扱うMIMEタイプを指定する。これはファイル名の拡張子で、ファイルタイプを決定できなかった場合などに使用される。 | DefaultType text/plain | 


## Directoryディレクティブの例

### Apache 2.4未満

 特定ディレクトリとそのサブディレクトリだけに適用する 設定ディレクティブのグループを作成する。

    <Directory>
        Options FollowSymLinks
        AllowOverride None
        Order deny,allow
        Deny from all
        Allow from 127.0.0.1
        Satisfy all
    </Directory>

| 項目 | 意味 | 
| ---- | ---- |
| Options | どのサーバー機能が使用できるかを制御する。例としてindex.htmlがない場合にディレクトリ内一覧表示機能を提供するか否かなど。 | 
| AllowOverride | .htaccess ファイルによる設定の上書きを許可するか否か。 | 
| Order | Allow, Deny ディレクティブの評価順序を制御する。 | 
| Allow | ディレクトリにアクセスできるクライアントの指定。All, ドメイン名, IPアドレス, 部分IPアドレス, ネットワーク/ネットワークマスクのペアなどが設定可能 | 
| Deny | ディレクトリにアクセスできないクライアントの指定。同上 | 
| Satisfy | ユーザ名/パスワード とクライアントのホストのアドレスで制限されている場合に、両方をテストするか片方のみ満たしていれば許可するかを設定。 | 


### Apache 2.4以降

    <Directory>
        Options FollowSymLinks
        AllowOverride None
        Require all denied
        Require ip 127.0.0.1
    </Directory>

| 項目 | 意味 |
| ---- | ---- |
| Require | ディレクトリにアクセスできるクライアントの指定<br>all granted, all denied, env method, expr, user, group, valid-user, ipが指定可能 |


## よく使う設定例

### サーバー情報出力の抑止

    # 'Full' needed by mod_security SecServerSignature
    ServerTokens Full
    ServerSignature Off
    TraceEnable Off
    ServerAdmin tomoki@tomoki.com

    <IfModule security2_module>
      SecServerSignature "Microsoft-IIS/8.5"
    </IfModule>

>```ServerTokens```は、mod_securityを併用して書き換える場合に```Full```とする。それ以外の場合は```Prod```にすべき


### ログ指定

日次でログローテーションし、出力内容にReferer, UserAgent, 処理時間を出力する例

     #
     # a remote ip
     # u remote user
     # t datetime
     # r 1st line of request
     # s status
     # b size of response(byte)
     # {Referer} referer header
     # {User-Agent} user agent
     # D request time(micro seconds)
     #
    LogFormat "%a %u %t \"%r\" %>s %b \"%{Referer}i\" \"%{User-Agent}i\" %D" myformat
    
    CustomLog "|/usr/sbin/rotatelogs /var/log/httpd/default/access_log.%Y%m%d 86400 540" myformat


### 基本設定

アクセスコントロール

    DocumentRoot "/var/www/html"

    <Directory />
        AllowOverride none
        Require all denied
    </Directory>
    
    <Directory "/var/www/html">
        Options Indexes
        AllowOverride None
        Require all granted
    </Directory>
    
    <Files ".ht*">
        Require all denied
    </Files>

Apache実行ユーザとグループの指定

    <IfModule unixd_module>
        User apache
        Group apache
    </IfModule>

ディレクトリインデックスの指定(「/」でアクセスされた場合のデフォルト)

    <IfModule dir_module>
        DirectoryIndex index.html
    </IfModule>

拡張子とMIMEタイプのマッピング指定
 
    <IfModule mime_module>
        TypesConfig /etc/httpd/conf/mime.types
        AddType application/x-compress .Z
        AddType application/x-gzip .gz .tgz
    </IfModule>

>mime.typesファイルは、mailcapパッケージのインストールが必要(CentOS 6)


### バーチャルホスト設定

    <VirtualHost _default_:80>
        # DNSリバインディング対策用の仮想ホスト
        ServerName default-virtual-host.invalid
        
        DocumentRoot /var/www/html
        <Directory "/var/www/html">
            AllowOverride none
            Require all granted
        </Directory>
    </VirtualHost>

    <VirtualHost *:80>
        ServerName hogehoge.com
        ServerAlias hogehoge.com
      
        KeepAlive Off
      
        AddDefaultCharset UTF-8
        DocumentRoot /var/www/sample/htdocs
        
        ErrorDocument 400 "BAD REQUEST"
        ErrorDocument 413 https://hogehoge.com/error/
        LimitRequestBody 7864320
        
        # AWS ELB対応
        RemoteIPHeader X-Forward-For
      
        <Directory "/var/www/sample/htdocs">
            Options FollowSymLinks
            AllowOverride All
            Require all granted
        </Directory>
        
        <Location "/img">
            FileETag Mtime Size
        </Location>
        <Location "/css">
            FileETag Mtime Size
        </Location>
        <Location "/js">
            FileETag Mtime Size
        </Location>
    </VirtualHost>

### FastCGI設定

    <IfModule mod_fastcgi.c>
        ScriptAlias /fcgi-bin/ /var/www/fcgi-bin/
        FastCGIExternalServer /var/www/fcgi-bin/php-fpm -socket /var/run/php-fpm/php-fpm.sock
        AddHandler php-fastcgi .php
        Action php-fastcgi /fcgi-bin/php-fpm
      
        <Directory /var/www/fcgi-bin>
            AllowOverride none
            Require all granted
        </Directory>
    </IfModule>

>/var/www/fcgi-bin/ディレクトリが必要（空で良い）

>```mod_fastcgi```を想定。```mod_fcgid```とは異なるので注意。
