# このソフトウェアについて

　SQLite3のデータベースとテーブルをディレクトリやSQLファイルから作成するスクリプトの試作2。

　前回はディレクトリ、SQLファイル、TSVファイルからDBファイルを作成した。今回はテキストファイルから作成したい。

## 前回

* https://github.com/ytyaru/Shell.Sqlite3.SqliteDbCreator.20181129170000

# 案

　以下のようなテキストファイルでDBファイルを作成する。

Databases.txt
```
MyDb1                                   ... ${DbName}
    Parent                              ... ${TableName}
        Id      INTEGER PRYMARY KEY     -+
        CId     INTEGER NOT NULL         |_ ${ColumnDefine}
        Name    TEXT                     |
        foreign CId Child(Id)           -+
    Child                               ... ${TableName}
        Id      INTEGER PRYMARY KEY     -+
        Name    TEXT                     |- ${ColumnDefine}
        Created TEXT                    -+
```

　列の制約をさらにインデントできる。長いときは複数行にもできる。

Databases.txt
```
MyDb1                                   ... ${DbName}
    Parent                              ... ${TableName}
        Id                              ... ${ColumnName}
            INTEGER                     -+_ ${ColumnConstraint}
            PRYMARY KEY                 -+
        CId
            INTEGER
            NOT NULL         
        Name
            TEXT
        foreign CId Child(Id)
    Child
        Id
            INTEGER
            PRYMARY KEY
        Name
            TEXT
        Created
            TEXT
```

　外部参照は略記できる。

Databases.txt
```
MyDb1
    Parent
        Id
            INTEGER
            PRYMARY KEY
        CId Child.Id      ... foreign CId Child(Id)
            INTEGER
            NOT NULL
        Name
            TEXT
    Child
        Id
            INTEGER
            PRYMARY KEY
        Name
            TEXT
        Created
            TEXT
```


* output/
    * ${DbName}.sqlite3
    * ${DbName}.${TableName}.sql
    * ${DbName}.${TableName}.tsv
    * app.ini: 各種ファイルの出力パス設定
    * relocate.sh: app.ini編集後に実行すると各種ファイルを再配置する
    * import.sh: tsv編集後に実行するとレコード挿入される

　なお、出力先はINI設定ファイルで自由に変更できるものとする。出力先のルートディレクトリは起動オプションで設定できる。デフォルトはカレントディレクトリ。

app.ini
```ini
[Output]
database=${DbName}.sqlite3
sql=${DbName}.${TableName}.sql
tsv=${DbName}.${TableName}.tsv
import=import.sh
```

　サブディレクトリも作れる。

app.ini
```ini
[Output]
database=db/${DbName}.sqlite3
sql=sql/${DbName}/${TableName}.sql
tsv=tsv/${DbName}/${TableName}.tsv
import=import.sh
```

　レコード単位でDBファイルを作成することもできる。

Databases.txt
```
AccountDb
    AccountTable
        Id      INTEGER PRYMARY KEY
        Username    TEXT
        Password    TEXT
RepositoryDb.${AccountDb.AccountTable.Username}     ... ${DB名.Table名.Column名}
    RepositoryTable
        Id      INTEGER PRYMARY KEY
        Name    TEXT
```

* ${AccountDb.AccountTable.Username}
    * 名前が定数でないので置き換え構文を使うと処理に困るし、統一性がないので混乱する？
* $(AccountDb, AccountTable, Username)
    * 関数の引数として渡せば変数でもOK
    * 関数名や構文はこれでいいのか？

　たとえばUsernameのレコードに`user1`, `user2`があったとき、`RepositoryDb.user1.sqlite3`, `RepositoryDb.user2.sqlite3`という2つのDBファイルが作成される。この特性上、親DB側にTSVでデータ挿入した後でないと子DBを作成できない。この依存関係があることで処理手順が複雑になる。
　これは[トリガー](https://www.dbonline.jp/sqlite/trigger/)で実装できない。SQL文発行でなく、`sqlite3`コマンドでDBファイルを作成する必要があるため。

　[sqlite3 C++ライブラリ](https://code.i-harness.com/ja/docs/sqlite/c3ref/update_hook)としてデータ変更時のイベントをキャッチし、コールバック関数の処理として子DB作成コマンドを実行すれば可能だろう。

　また、DBファイル名定義にてサブディレクトリを作成できる。

Databases.txt
```
AccountDb
    AccountTable
        Id      INTEGER PRYMARY KEY
        Username    TEXT
        Password    TEXT
RepositoyDb/${AccountDb.AccountTable.Username}     ... サブディレクトリ作成
    RepositoryTable
        Id      INTEGER PRYMARY KEY
        Username    TEXT
        Password    TEXT
```

app.ini
```ini
[Output]
database=db/${DbName}.sqlite3
sql=sql/${DbName}/${TableName}.sql
tsv=tsv/${DbName}/${TableName}.tsv
import=import.sh
```



```sql
create table ${TableName} (
    ${ColumnDefine}
);
```

　テーブル作成用SQLの発行コマンドは以下のいずれか。

```bash
$ sqlite3 ${DbName} "create table ${TableName} (${ColumnDefine})"
```
```bash
$ sqlite3 ${DbName} < ${DbName}.${TableName}.sql
```

　さらに、テストデータ挿入用TSVファイルの雛形を作成する

Parent.tsv
```tsv
Id  CId Name
```

```bash
$ sqlite3 ${DbName} ".mode tabs" ".import ./Tsv/${DbName}/${TableName}.tsv ${TableName}"
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

