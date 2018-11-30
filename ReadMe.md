# このソフトウェアについて

　SQLite3のデータベースとテーブルをディレクトリやSQLファイルから作成するスクリプトの試作2。

　前回はディレクトリ、SQLファイル、TSVファイルからDBファイルを作成した。今回はテキストファイルから作成したい。

## 前回

* https://github.com/ytyaru/Shell.Sqlite3.SqliteDbCreator.20181129170000

# 案

　以下のようなテキストファイルでDBファイルを作成する。

Databases.txt
```
MyDb1
    Parent
        Id      INTEGER PRYMARY KEY
        CId     INTEGER NOT NULL
        Name    TEXT
        foreign CId Child(Id)
    Child
        Id      INTEGER PRYMARY KEY
        Name    TEXT
        Created TEXT
MyDb2
    ...
```

```bash
$ sqlite3 ${DbName} "create table ${TableName} (${ColumnDefine})"
```

　さらに、テストデータ挿入用TSVファイルの雛形を作成する

* Tsv
    * ${DbName}/
        * ${TableName}.tsv
    * import.sh

Parent.tsv
```tsv
Id  CId Name
```

# 参考

* https://qiita.com/Kunikata/items/61b5ee2c6a715f610493

# 開発環境

* [Raspberry Pi](https://ja.wikipedia.org/wiki/Raspberry_Pi) 3 Model B+
    * [Raspbian](https://www.raspberrypi.org/downloads/raspbian/) GNU/Linux 9.0 (stretch) 2018-06-27
        * [SQLite version 3.22.0 2018-01-22 18:45:57](http://ytyaru.hatenablog.com/entry/2019/01/31/000000)

# 使い方

1. データベース定義を作成するルートディレクトリを作成する `$ mkdir -p /tmp/DbDefines`
1. 作成したいデータベースのディレクトリを作成する `$ mkdir -p /tmp/DbDefines/SomeDb`
1. データベースに作成するテーブルを定義したSQLファイルを作成する
1. SQLファイルを出力するディレクトリを用意する `$ mkdir -p /tmp/DbOutput`
1. 本ツールを実行する

```bash
$ ./SqliteDbCreator.sh -i /tmp/DbDefines -o /tmp/DbOutput
```

　以下が作成される。

* `.sqlite3`ファイル
* 初期データ作成用の雛形TSVファイル
* `import.sh`

　TSVファイルを作成したら`import.sh`を実行すると、初期データが挿入される。

# ライセンス

　このソフトウェアはCC0ライセンスである。

[![CC0](http://i.creativecommons.org/p/zero/1.0/88x31.png "CC0")](http://creativecommons.org/publicdomain/zero/1.0/deed.ja)

