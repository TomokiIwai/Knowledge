# Apacheモジュール

## 概要

Apacheのモジュールは、Apacheのリクエストサイクルの代表的なフェーズにフック関数を登録して、任意の処理を実行することができる。

* クライアントの接続
* リクエスト読み込み
* URI変換
* アクセス制御
* 認証
* 微調整(fixup)
* インプットフィルタ
* コンテンツ出力
* アウトプットフィルタ
* ロギング

>[フックに関する詳細](http://d.hatena.ne.jp/dayflower/20081029/1225266220)


## 開発環境

apxs2というシェルスクリプトが必要になるため、MPMに応じて下記のパッケージをインストールする。

| MPM | インストールするパッケージ |
| - | - |
| Prefork | apache2-prefork-dev |
| Theaded, Event | apache2-threaded-dev |


## 開発

### 雛形の作成

モジュールの雛形を作成する。

    apxs2 -g -n test


### コンパイル

    apxs2 -i -a -c mod_test.c

| オプション | 意味 | 備考 | 
| - | - | - |
| i | インストール | モジュールディレクトリにインストールされる | 
| a | LoadModuleディレクティブをhttpd.confに追加 | apache2.confを利用している状況だとエラーで書けないっぽい | 
| c | コンパイル |  | 


### Apacheの設定変更

モジュールの読み込みを追記する

    LoadModule test_module modules/mod_test.so

    <Location "/module_test/">
        SetHandler test
    </Location>


## リクエストヘッダに独自情報を追加するモジュール例

    #include "httpd.h"
    #include "http_config.h"
    #include "http_protocol.h"
    #include "ap_config.h"
    
    /**
    * Handler called "fixups" stage.
     */
    static int fixups_handler(request_rec* r)
    {
        apr_table_set(r->subprocess_env, "hogeKey", "hogeValue");
        return OK;
    }
    
    /**
    * Register hooks.
     */
    static void register_hooks(apr_pool_t* p)
    {
        ap_hook_fixups(fixups_handler, NULL, NULL, APR_HOOK_MIDDLE);
    }
    
    module AP_MODULE_DECLARE_DATA test_module = {
        STANDARD20_MODULE_STUFF,
        NULL,                  /* create per-dir    config structures */
        NULL,                  /* merge  per-dir    config structures */
        NULL,                  /* create per-server config structures */
        NULL,                  /* merge  per-server config structures */
        NULL,                  /* table of config file commands       */
        register_hooks         /* register hooks                      */
    };

>fixups_handler()で、request_recのheader_inではなく、subprocess_envにhogeKey=hogeValueを追加しているのは、mod_cgiとの併用を意識したためである。
>mod_cgiの仕様で、subprocess_envに入っている情報を環境変数へ設定するようになっている。


## リファレンス

### APR(Apache Portable Runtime)

C言語における移植性の確保を行う抽象レイヤーであり、関数名はapr_から始まる。

* メモリプールの管理
* 文字列操作
* 複雑なデータ構造(動的配列、コンテナ型)
* ファイル I/O
* etc...

#### apr_strtok

    /**
    * 文字列を指定したデリミタで分割する。
    * 
    * @param str 対象文字列
    * @param sep デリミタ
    * @param last 残りの文字列
     */
    apr_strtok(char *str, const char *sep, char **last)

使用例

    char* targetString;
    char* element;
    char* last;
    
    // 分割対象文字列(リクエストパラメータ)
    targetString = r->args;
    
    // 対象文字列を&で分割し、レスポンスに出力する
    for (element = apr_strtok(targetString, "&", &last); element != NULL; element = apr_strtok(NULL, "&", &last))
    {
        ap_rputs((char*)pair, r);
    }

### apr_table_set

    /**
    * テーブルの要素を更新する。
     *
    * @param t 対象テーブル
    * @param key キー
    * @param val 値
    * @note setnの方は内部でコピーを作成しない
     */
    void apr_table_set(apr_table_t* t, const char* key, const char* val)
    void apr_table_setn(apr_table_t* t, const char* key, const char* val)

使用例

    apr_table_set(r->subprocess_env, "hogeKey", "hogeValue");

### apr_table_add

    /**
    * テーブルに要素を追加する。
     *
    * @param t 対象テーブル
    * @param key キー
    * @param val 値
    * @note addnの方は内部でコピーを作成しない
     */
    void apr_table_add(apr_table_t* t, const char * key, const char * val)
    void apr_table_addn(apr_table_t* t, const char * key, const char * val)

### apr_table_elts

    /**
    * 全エントリーの配列を取得する。
    * 
    * @param t 対象テーブル
    * @return 全エントリーの配列(apr_array_header_t*)
     */
    const apr_array_header_t* apr_table_elts(const apr_table_t* t)

使用例

    // tableの全エントリー配列を取得
    const apr_array_header_t *arr = apr_table_elts(table);
    
    // 各エントリーのメモリサイズ
    int elt_size = arr->elt_size;
    
    // 全エントリーについてループ
    int i;
    for (i = 0; i < arr->nelts; i++)
    {
        // i番目のエントリーを取得
        apr_table_entry_t *entry = (apr_table_entry_t *)(arr->elts + (elt_size * i));
        
        // キーと値を出力
        printf("%d番目の要素：キー=[%s],値=[%s]
    ",i, entry->key, entry->val);
    }

## 参考

* [Apache Module](http://www.slideshare.net/shebang/apache-module-presentation)
* [Apache Portable Runtime Documentation](http://apr.apache.org/docs/apr/0.9/index.html)
