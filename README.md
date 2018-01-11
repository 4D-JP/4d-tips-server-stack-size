### サーバー側プロセスのメソッド実行スタックについて

クライアント/サーバー版アプリケーションの場合，集計などの「重い」処理を**サーバー側プロセス**で実行すれば，ネットワーク経由のデータベースアクセス数が減り，パフォーマンスの向上が期待できます。

クライアント/サーバー版アプリケーションの場合，集計などの「重い」処理をサーバー側プロセスで実行すれば，ネットワーク経由のデータベースアクセス数が減り，パフォーマンスの向上が期待できます。

4D Serverには，メソッドをサーバー側で実行する方法が幾つか用意されています。

1. [サーバー上で実行](http://doc.4d.com/4Dv16/4D/16/Execute-on-Server-attribute.300-3047542.ja.html)メソッドプロパティ
1. [Execute on server()](http://doc.4d.com/4Dv16/4D/16.2/Execute-on-server.301-3433448.ja.html)
1. [実行モード: 4D Server上](http://doc.4d.com/4Dv16R4/4D/16-R4/Executing-methods.300-3330269.ja.html)オプション（デザインモード > 実行 > メソッド）

[サーバー上で実行](http://doc.4d.com/4Dv16/4D/16/Execute-on-Server-attribute.300-3047542.ja.html)メソッドプロパティは，v11で追加されたメソッドプロパティであり，**同期処理**に向いています。メソッドはサーバー側のトリガプロセスで実行され，クライアント側のプロセスはサーバー側の処理が完了するまで待機します。返り値だけでなく，ポインター引数を経由して入出力データを交換することもできます。Execute on serverのようなポーリングによる待ち合わせではなく，サーバーから値がプッシュされる仕組みになっているので，ネットワーク使用が非常に効率的です。

[Execute on server()](http://doc.4d.com/4Dv16/4D/16.2/Execute-on-server.301-3433448.ja.html)は，ストアドプロシージャーを起動するための標準的な手法であり，**非同期処理**に向いています。メソッドはサーバー側の新規プロセスで実行されるので，クライアント側で結果を受け取るためには，何らかの待ち合わせ処理が必要です。

[実行モード: 4D Server上](http://doc.4d.com/4Dv16R4/4D/16-R4/Executing-methods.300-3330269.ja.html)オプションは，開発者あるいは管理者のようなパワーユーザー向けのオプションであり，**応急的な処理**に向いています。メソッドはサーバー側の新規プロセス（ストアドプロシージャー）で実行されます。結果をクライアント側で受け取るための手段は特に用意されていません。

#### スタックサイズ

メソッドを実行するためには，汎用的なLIFO（Last in first out）メモリ，つまりスタックメモリが必要です。スタックメモリはヒープ領域に確保され，一時的な情報を記憶するために使用されます。スタックを使用するおもなオブジェクトは下記のとおりです。

* ローカル変数
* 引数
* 返り値
* プラグインコール
* メソッド連鎖（ローカル変数・引数・プラグインコール）

**多重サブルーチンコール**や**再帰的メソッドコール**は，スタックを多く使用します。BLOBは参照カウントされないため，サイズのおおきな**BLOB引数**の受け渡しはスタックを余分に消費します。テキスト・ピクチャ・オブジェクト・コレクションは参照カウントされるので，スタックサイズを節約するためにポインター引数を経由する必要はありません。

プロセスの合計スタックサイズ，およびフリースタックサイズは[GET MEMORY STATISTICS](http://doc.4d.com/4Dv16R4/4D/16-R4/GET-MEMORY-STATISTICS.301-3318258.ja.html)で調べることができます。

```
GET MEMORY STATISTICS(1;$names;$values;$count)

$total:=$values{Find in array($names;"Stack memory")}/(1 << 10)
$free:=$values{Find in array($names;"Free stack memory")}/(1 << 10)
```

通常，スタックサイズを明示的に指定する必要はありません。[Execute on server ()](http://doc.4d.com/4Dv16/4D/16.3/Execute-on-server.301-3651704.ja.html)および[New process ()](http://doc.4d.com/4Dv16/4D/16.3/New-process.301-3651687.ja.html)は，スタックサイズに``0``を指定すれば，自動的にプラットフォームの推奨スタックサイズ（Windows 10であれば``565,248``）が採用されるようになっています。

**注記**: 初期の4Dマニュアルには``16*1024``あるいは``32*1024``といった例題が掲載されていました。これは，当時のマシンスペックを考慮したスタックサイズであり，現在のオペレーションシステムでは，特殊な場合を除き，実際的な値ではないことに留意する必要があります。

サーバー側プロセスのスタックサイズが指定できるのは[Execute on server ()](http://doc.4d.com/4Dv16/4D/16.3/Execute-on-server.301-3651704.ja.html)だけです。[サーバー上で実行](http://doc.4d.com/4Dv16/4D/16/Execute-on-Server-attribute.300-3047542.ja.html)メソッドプロパティ，および[実行モード: 4D Server上](http://doc.4d.com/4Dv16R4/4D/16-R4/Executing-methods.300-3330269.ja.html)オプションは，実験してみると，それぞれデフォルトのスタックサイズ（``552``KB）に固定されていることがわかります。

[サーバー上で実行](http://doc.4d.com/4Dv16/4D/16/Execute-on-Server-attribute.300-3047542.ja.html)メソッドプロパティは非常に便利ですが，サーバー側プロセスのスタックサイズが足りないようであれば，下記の対策を講じる必要があるかもしれません。

* [Execute on server ()](http://doc.4d.com/4Dv16/4D/16.3/Execute-on-server.301-3651704.ja.html)を使用する

* サーバー側で[New process ()](http://doc.4d.com/4Dv16/4D/16.3/New-process.301-3651687.ja.html)を使用する

* 過剰なメソッド連鎖を減らす

* おおきなBLOB引数の安易な代入を避ける


