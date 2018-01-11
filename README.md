### サーバー側プロセスのメソッド実行スタックについて

クライアント/サーバー版アプリケーションの場合，集計などの「重い」処理を**サーバー側プロセス**で実行すれば，ネットワーク経由のデータベースアクセス数が減り，パフォーマンスの向上が期待できます。

クライアント/サーバー版アプリケーションの場合，集計などの「重い」処理をサーバー側プロセスで実行すれば，ネットワーク経由のデータベースアクセス数が減り，パフォーマンスの向上が期待できます。

4D Serverには，メソッドをサーバー側で実行する方法が幾つか用意されています。

1. [サーバー上で実行](http://doc.4d.com/4Dv16/4D/16/Execute-on-Server-attribute.300-3047542.ja.html)メソッドプロパティ
1. [Execute on server()](http://doc.4d.com/4Dv16/4D/16.2/Execute-on-server.301-3433448.ja.html)
1. [実行モード: 4D Server上](http://doc.4d.com/4Dv16R4/4D/16-R4/Executing-methods.300-3330269.ja.html)オプション（デザインモード > 実行 > メソッド）
