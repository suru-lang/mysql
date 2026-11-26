[English](README.md) | 日本語

# MySQL(する のデータベースの層「蔵」の駆動)

日本語のプログラミング言語「[する](https://github.com/suru-lang/suru)」から、データベースの層 [「蔵」](https://github.com/suru-lang/db) を通して MySQL を使う駆動です。libmysqlclient を外部の関数として呼びます。

## 入れ方

[kumu](https://github.com/suru-lang/suru/blob/main/README_jp.md#プロジェクトを組むkumu) のプロジェクトで:

```sh
kumu 追加 github.com/suru-lang/mysql
```

「蔵」(suru-lang/db)も依存として一緒に入ります。システムに libmysqlclient(MySQL 8.0 以上)が要ります(組.suru の `外部依存`。無ければ kumu が入れ方を出して止まります)。

する 本体は 0.9.1 以上が要ります(ライブラリが C の関数を「以下は外部の入出力動作です … 以上です」で書くため)。

- macOS: `brew install mysql-client` して `export PKG_CONFIG_PATH="$(brew --prefix mysql-client)/lib/pkgconfig"`
- Debian・Ubuntu: `sudo apt install libmysqlclient-dev`
- Fedora: `sudo dnf install mysql-devel`

## 使い方

```
「蔵」 を 読み込んで 使う
「MySQL」 を 読み込んで 使う
「標準」 を 読み込んで 使う

DB行 を 品表示文にする は 文字列
行 を 品表示文にする とは （行 から 「名前」 を DB列取得して DB文字列取出する） + 「 は 」 + （行 から 「値段」 を DB列取得して DB値文字列にする） + 「 円です」

主処理する とは 入出力手順
  db とは 「」 を MySQL駆動 で DB開封した 結果
  db に 「DROP TABLE IF EXISTS 品; CREATE TABLE 品 (番号 BIGINT AUTO_INCREMENT PRIMARY KEY, 名前 TEXT NOT NULL, 値段 DOUBLE)」 を DB実行する
  db で （db に 「INSERT INTO 品 (名前, 値段) VALUES ('りんご', 120), ('みかん', 80.5), ('ぶどう', 450)」 を DB実行する） を DB取引する
  行群 とは db に 「SELECT 名前, 値段 FROM 品 WHERE 値段 < ? ORDER BY 値段」 を （（200 の DB数） を 空 に 挿入する） で DB行問合した 結果
  （行群 の それぞれ を 品表示文にして 改行 で 結合する） を 表示する
  db に 「DROP TABLE 品」 を DB実行する
  db を DB閉鎖する
  「以上です」 を 表示する

主処理する
```

関数はどれも「蔵」のものです(「蔵」の README を見てください)。`MySQL駆動` を `SQLite駆動` に替えれば、同じプログラムが SQLite で動きます。例は [例/店.suru](例/店.suru) です。

## 決めたこと

1. **つなぐ。** `先 を MySQL駆動 で DB開封する` でつなぎます。`先` は `名前=値` を空白で区切ったものです: `「host=127.0.0.1 port=3306 user=花子 password=… database=店」`(値に空白は書けません)。書かなかった名前は環境変数 `MYSQL_HOST`・`MYSQL_PORT`・`MYSQL_USER`・`MYSQL_PASSWORD`・`MYSQL_DATABASE` から取るので、`「」` なら環境変数だけを使います。文字は utf8mb4 でやりとりします。
2. **プレースホルダ。** 文は「蔵」のどこでも同じく「?」で書きます。駆動は、送る前に値を文の中に字句として入れます: 文字列は `mysql_real_escape_string` で退避したもの、バイト列は `X'…'`、数、`TRUE`・`FALSE`、`NULL` です。文字列と囲んだ名前の中の「?」はそのままで、「?」と値の数が違えば実行時エラーです。
3. **何文でも。** 接続は複数の文を許すので、`DB実行する` で「;」で区切った文を一度に実行できます。結果は全部読み、最後のものを返します。
4. **読んだ値。** 整数・小数・DECIMAL の列は `DB数`、バイナリの列(BINARY・VARBINARY・BLOB)は `DBバイト列`、NULL は `DB無`、ほかは `DB文字列` です。MySQL の BOOLEAN は TINYINT(1) なので、`DB数` の 1 か 0 で読めます(`DB真偽取出する` で真偽にできます)。
5. **最後の番号。** `DB最終番号取得する` は `mysql_insert_id` で、その接続の最後の INSERT の AUTO_INCREMENT の値です。
6. **失敗は実行時エラー。** 「MySQL で『SELEC 1』を実行するのに失敗しました(You have an error in your SQL syntax; …)」のように MySQL の理由が付き、`試行する` で受け止められます。MySQL は CREATE TABLE などの文で暗黙に確定するので、それらは `DB取引する` でも取り消せません。

## 試験

試験は本物の MySQL のサーバーを使います。つなぐ先は環境変数(`MYSQL_HOST`・`MYSQL_PORT`・`MYSQL_USER`・`MYSQL_PASSWORD`・`MYSQL_DATABASE`)で決めます。`人_…` という名前の表を作って消します。

```sh
MYSQL_HOST=127.0.0.1 MYSQL_USER=root MYSQL_PASSWORD=… MYSQL_DATABASE=… kumu 試験
kumu 点検
```

## 今後の候補

- サーバー側のプリペアドステートメント(`mysql_stmt_*`)で、ほかの駆動と同じく値を別に送る
- 日付・時刻の列を、文字列でなく する の値で読む
- MariaDB のクライアントライブラリ(libmariadb)にも対応する

## ライセンス

[MIT](LICENSE)
