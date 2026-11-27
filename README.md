English | [日本語](README_jp.md)

# MySQL (a driver for Suru's database layer 蔵)

A driver that lets [Suru (「する」)](https://github.com/suru-lang/suru), a programming language in Japanese, use MySQL through its database layer [「蔵」](https://github.com/suru-lang/db). It calls libmysqlclient as foreign functions.

## Installing

In a [kumu](https://github.com/suru-lang/suru#building-projects-kumu) project:

```sh
kumu 追加 github.com/suru-lang/mysql
```

「蔵」 (suru-lang/db) comes along as a dependency. libmysqlclient (MySQL 8.0 or later) is required on the system (the `外部依存` in 組.suru; if it is missing, kumu shows how to install it and stops).

Suru 0.9.1 or later is required (the library writes its C functions with 「以下は外部の入出力動作です … 以上です」).

- macOS: `brew install mysql-client`, then `export PKG_CONFIG_PATH="$(brew --prefix mysql-client)/lib/pkgconfig"`
- Debian, Ubuntu: `sudo apt install libmysqlclient-dev`
- Fedora: `sudo dnf install mysql-devel`

## Usage

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

All the functions are those of 「蔵」 (see its README). Replacing `MySQL駆動` with `SQLite駆動` runs the same program on SQLite. An example is [例/店.suru](例/店.suru).

## Decisions

1. **Connecting.** `先 を MySQL駆動 で DB開封する` connects. `先` is `名前=値` pairs separated by spaces: `「host=127.0.0.1 port=3306 user=花子 password=… database=店」` (values can't contain spaces). Names left out are taken from the environment variables `MYSQL_HOST`, `MYSQL_PORT`, `MYSQL_USER`, `MYSQL_PASSWORD` and `MYSQL_DATABASE`, so `「」` uses only them. Text is exchanged as utf8mb4.
2. **Placeholders.** Statements are written with 「?」 as in all of 「蔵」. The driver puts each value into the statement as a literal before sending it: strings escaped with `mysql_real_escape_string`, byte strings as `X'…'`, numbers, `TRUE`/`FALSE` and `NULL`. 「?」 inside strings and quoted names is left alone, and a different number of 「?」 and values is a runtime error.
3. **Several statements.** The connection allows several statements, so `DB実行する` can run statements separated by 「;」 at once; all results are read and the last one is returned.
4. **Values read back.** Integer, decimal and DECIMAL columns become `DB数`, binary columns (BINARY, VARBINARY, BLOB) `DBバイト列`, NULL `DB無`, and everything else `DB文字列`. BOOLEAN is TINYINT(1) in MySQL, so it reads back as the `DB数` 1 or 0 (`DB真偽取出する` turns it into a truth value).
5. **The last id.** `DB最終番号取得する` is `mysql_insert_id`: the AUTO_INCREMENT value of the last INSERT on the connection.
6. **Failures are runtime errors.** They carry MySQL's message, like 「MySQL で『SELEC 1』を実行するのに失敗しました(You have an error in your SQL syntax; …)」 ("failed to run 『SELEC 1』 in MySQL (…)"), and are catchable with `試行する`. Note that MySQL commits implicitly on statements like CREATE TABLE, so they can't be rolled back by `DB取引する`.

## API documentation

The API documentation is made from the comments in `MySQL.suru`: the head comment of the file and the comment right above each public definition, written in English. It is published at https://suru-lang-mysql.readthedocs.io/en/latest/.

```sh
kumu 文書                                                            # writes docs/api/MySQL.md
pip install -r docs/requirements.txt && sphinx-build docs docs/_build   # HTML in docs/_build
```

`docs/` and `.readthedocs.yaml` were written by `kumu 文書 --雛形` (Suru 0.11.0 or later). On every build, Read the Docs downloads kumu 0.11.0, runs `kumu 文書 --出力 docs/api` and then Sphinx, so the generated pages are not committed. The editor hover shows the same descriptions.

## Tests

The tests use a real MySQL server, chosen with the environment variables (`MYSQL_HOST`, `MYSQL_PORT`, `MYSQL_USER`, `MYSQL_PASSWORD`, `MYSQL_DATABASE`). They create and drop tables named `人_…`.

```sh
MYSQL_HOST=127.0.0.1 MYSQL_USER=root MYSQL_PASSWORD=… MYSQL_DATABASE=… kumu 試験
kumu 点検
```

## Future work

- Server-side prepared statements (`mysql_stmt_*`), to send values separately like the other drivers
- Read date and time columns into Suru values instead of strings
- Support MariaDB's client library (libmariadb) as well

## License

[MIT](LICENSE)
