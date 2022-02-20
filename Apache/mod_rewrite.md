# mod_rewrite

## 概要

ApacheでリクエストURLを任意に書き換えることができるモジュール。

## クエストストリング（パラメータ）の引き継ぎに関する注意

クエストストリングは、RewriteRuleの書き方によって挙動がことなるので注意が必要。

クエストストリングが自動的に引き継がれる書き方

    RewriteCond %{REQUEST_URI}    ^/hoge/$
    RewriteRule .*                /foo [L]

クエストストリングが消える書き方

    RewriteCond %{REQUEST_URI}    ^/hoge/$
    RewriteRule .*                /foo? [L]

クエストストリングを書き足す方法

    RewriteCond %{REQUEST_URI}    ^/hoge/$
    RewriteRule .*                /foo?a=1 [L, QSA]
