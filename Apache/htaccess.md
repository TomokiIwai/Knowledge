# .htaccess

## 概要

DirectoryでAllowOverrideを指定されたディレクトリにおいて、設定を上書きできる。

## DirectoryとLocationを併用する際の注意点

### 適用順序

1. Directory（正規表現無し）
1. そのディレクトリ内に設置されている.htaccessのDirectory
1. DirectoryMatch>およびDirectory（正規表現あり）
1. FilesとFilesMatch
1. LocationとLocationMatch

>Includeについてはあたかもそこに記述されているかのように振る舞う


### DirectoryとLocationでは、優先順位が異なる

Directoryはリクエストパスに合致する短い設定から順に適用し、上書きしてゆく。
(参考:[うまい棒blog](http://d.hatena.ne.jp/hogem/20091029/1256827891))

>/hoge/fugaは、設定Aが適用された後に、設定Bが適用される。

    <Directory /path/to/docroot/hoge/fuga>
        ## 設定B
    </Directory>

    <Directory /path/to/docroot/hoge>
        ## 設定A
    </Directory>

Locationは出現順に適用し、上書きしてゆく。
>/hoge/fugaは、設定Bが適用された後に、設定Aが適用される。

    <Location /hoge/fuga>
        ## 設定B
    </Location>
    
    <Location /hoge>
        ## 設定A
    </Location>

* [うまい棒blog](http://d.hatena.ne.jp/hogem/20091029/1256827891)


## 特定UAの場合にのみIP制限をかける方法

Apache 2.4以降を前提とする。

    <!-- フィーチャーフォンの場合にIPアドレスを制限 -->
    <If "%{HTTP_USER_AGENT} =~ /^(DoCoMo|KDDI|SoftBank|Vodafone).*$/i">
      Require ip 113.43.73.18
      Require ip 203.152.210.1
      Require ip 203.152.210.2
    </If>
