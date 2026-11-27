# 「MySQL」

The MySQL driver for Suru, used with the database layer 「蔵」. It calls libmysqlclient through external functions.
Add it with `kumu 追加 github.com/suru-lang/mysql` and write `「MySQL」 を 読み込んで 使う` in your file.

`先 を MySQL駆動 で DB開封する` connects. 先 is 「名前=値」 pairs separated by spaces, like 「host=127.0.0.1 port=3306 user=花子 password=… database=店」 (values cannot contain spaces). What is not written is taken from the environment variables MYSQL_HOST, MYSQL_PORT, MYSQL_USER, MYSQL_PASSWORD and MYSQL_DATABASE.
Then use the functions of 「蔵」 (`DB実行する`, `DB差込問合する`, `DB行問合する`, `DB取引する` and so on).

- Text is exchanged as utf8mb4.
- Each 「?」 is filled with the value made into text by mysql_real_escape_string (bytes as X'…'). A 「?」 inside a string or a quoted name stays as it is.
- `DB実行する` without values can run several statements separated by 「;」 (the result is that of the last one).
- `DB最終番号取得する` is mysql_insert_id (the last AUTO_INCREMENT value).
- Values read: integers, decimals and DECIMAL are `DB数`, BINARY, VARBINARY and BLOB are `DBバイト列`, and the rest are `DB文字列`. BOOLEAN is TINYINT(1), so it is `DB数` 1 or 0; `DB真偽取出する` turns it into a truth value.

A failure is a runtime error such as 「MySQL で『SELEC 1』を実行するのに失敗しました(You have an error in your SQL syntax; …)」, which `試行する` can catch.

```{toctree}
:maxdepth: 2
:caption: API
:glob:

api/*
```
