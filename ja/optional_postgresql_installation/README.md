# PostgreSQLのインストール

> このチャプターの一部はGeek Girls Carrotsのチュートリアルをもとにしています。(http://django.carrots.pl/).

> このチャプターの一部は、Creative Commons Attribution-ShareAlike 4.0 International Licenseの下で提供される[django-marcador tutorial](http://django-marcador.keimlink.de/)をもとにしています。django-marcador tutorialは、Markus Zapke-Gründemannらに著作権が帰属します。

## Windows

PostgresをWindowsにインストールする一番簡単な方法は、次のリンクから入手できるプログラムを使うことです： http://www.enterprisedb.com/products-services-training/pgdownload#windows

お使いのOSで利用できる最新のバージョンを選んでください。インストーラをダウンロードし、実行し、次のリンクの指示に従ってください：http://www.postgresqltutorial.com/install-postgresql/
インストール先のディレクトリは次のステップで必要ですので、控えておいてください。（通常は `C:\Program Files\PostgreSQL\9.3` のようになります）

## Mac OS X

一番簡単な方法は、Mac OSの他のアプリケーションと同様に、無料の [Postgres.app](http://postgresapp.com/) をダウンロードし、インストールすることです。

ダウンロードし、「アプリケーション」ディレクトリにドラッグし、ダブルクリックで実行してください。これでおしまいです！

[ここ](http://postgresapp.com/documentation/cli-tools.html)に書かれているように、`PATH` 変数にPostgresのコマンドラインツールを追加する必要があります。

## Linux

インストール手順はディストリビューションごとに異なります。以下にあるのは、UbuntuとFedora向けのコマンドです。これら以外のディストリビューションをお使いの場合は、[PostgreSQLのドキュメントを参照してください](https://wiki.postgresql.org/wiki/Detailed_installation_guides#General_Linux)。

### Ubuntu

次のコマンドを実行してください：

    sudo apt-get install postgresql postgresql-contrib

### Fedora

次のコマンドを実行してください：

    sudo yum install postgresql93-server

# データベースを作成する

次に、最初のデータベースと、それにアクセスできるユーザを作ります。PostgreSQLでは好きなだけデータベースとユーザを作成できるので、複数のサイトを稼働させる場合は、サイトごとにデータベースを作るべきです。

## Windows

Windowsをお使いの場合は、完了する必要のあるいくつかの追加のステップがあります。今の時点では、ここで行う設定を理解することは重要ではありませんが、何が行われているか興味がある方はお気軽にコーチに質問してください。

1. コマンドプロンプトを開きます（「スタートメニュー」→「アクセサリ」→「コマンドプロンプト」）
2. 次のように入力して、Enterキーを押して実行します：`setx PATH "%PATH%;C:\Program Files\PostgreSQL\9.3\bin"`。コマンドプロンプトを右クリックして `ペースト` を選択することで、コピー&ペーストもできます。`"%PATH%;"` に続く部分が、インストール時に控えたパスに `\bin` をつけ足したものと同じか確認してくださいね。`SUCCESS: Specified value was saved.` というメッセージが現れるでしょう。
3. コマンドプロンプトを閉じて、再度開きます。

## データベース作成

まずは、`psql` コマンドを実行して、Postgresコンソールを起動しましょう。コンソールの起動の仕方は覚えていますか？
>Mac OS Xでは、`ターミナル`アプリケーションを起動して行います（「アプリケーション」→「ユーティリティ」の中にあります）。Linuxでは、おそらく「アプリケーション」→「アクセサリ」→「ターミナル」となるでしょう。Windowsでは、「スタートメニュー」→「すべてのプログラム」→「アクセサリ」→「コマンドプロンプト」となります。さらに言うと、Windowsでは、インストール時に設定したユーザ名とパスワードを使ってログインすることを `psql` コマンドが求めてくるかもしれません。`psql` コマンドがパスワードの入力を求めていて、入力してもうまくいかない場合は、`psql -U <username> -W` コマンドをまず実行し、パスワードを後で入力してください。

    $ psql
    psql (9.3.4)
    Type "help" for help.
    #

先頭の `$` が `#` に変わりました。これから入力するコマンドはPostgreSQLに送られることを意味します。`CREATE USER name;` コマンドでユーザを作りましょう。（最後にセミコロン(;)を使うことを覚えておいてくださいね）：

    # CREATE USER name;

`name` の部分はご自分のお名前に書き換えてくださいね。アクセント文字や空白文字は使うべきではありません。（例えば `bożena maria` はうまくいきません。`bozena_maria` のように書き換える必要があります。）うまくいくと、コンソールから `CREATE ROLE` という応答が返ってきます。

いよいよあなたのDjangoプロジェクトのデータベースを作るときです：

    # CREATE DATABASE djangogirls OWNER name;

`name` は先ほど設定した名前（例 `bozena_maria`）に置き換えてくださいね。このコマンドは、あなたのプロジェクトで今から使える空っぽのデータベースを作ります。うまくいくと、コンソールから `CREATE DATABASE` という応答が返ってきます。

素晴らしい！データベース全部が並んで表示されています。

# 設定の更新

`mysite/settings.py` ファイルの中から、以下の部分を見つけてください：

```python
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.sqlite3',
        'NAME': os.path.join(BASE_DIR, 'db.sqlite3'),
    }
}
```

その部分を以下のように書き換えます：

```python
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql',
        'NAME': 'djangogirls',
        'USER': 'name',
        'PASSWORD': '',
        'HOST': 'localhost',
        'PORT': '',
    }
}
```

`name` をこのチャプターで作ったユーザの名前に書き換えてくださいね。

# PythonのPostgreSQL用パッケージのインストール

まず、Heroku Toolbelt を https://toolbelt.heroku.com/ からインストールします。これは後であなたのサイトをデプロイするときに主に必要になりますが、Gitも含んでいます。Gitはもう重宝しているかもしれませんね。

次は、PythonがPostgreSQLとやりとりできるようにするためのパッケージをインストールします。`psycopg2` という名前のパッケージです。Windowsの場合と、LinuxまたはOS Xの場合とで、インストール手順はわずかに異なります。

## Windows

Windowsでは、ビルド前のファイルを http://www.stickpeople.com/projects/python/win-psycopg/ からダウンロードしてください。

あなたの使っているPythonのバージョンに対応するものであること（Python 3.4 対応版は最後の行にあります）と、アーキテクチャに対応するものであること（32ビット版は左の列、64ビット版は右の列です）を確認してください。

ダウンロードしたファイルの名前を変更してから移動し、`C:\psycopg2.exe` とします。

ここまで終わったら、コマンドラインに以下のコマンドを入力します（`virtualenv` が有効になっていることを確認して実行してください）：

    easy_install C:\psycopg2.exe

## Linux and OS X

コマンドラインで以下を実行します：

    (myvenv) ~/djangogirls$ pip install psycopg2

うまくいくと、以下のような出力が表示されるでしょう：

    Downloading/unpacking psycopg2
    Installing collected packages: psycopg2
    Successfully installed psycopg2
    Cleaning up...

---

完了したら、`python -c "import psycopg2"` というコマンドを実行します。エラーが表示されなければ、インストールは全て成功しました。

# マイグレーションの適用とsuperuserの作成

新しく作成したデータベースをあなたのウェブサイトのプロジェクトで使うために、全てのマイグレーションを適用する必要があります。仮想環境の中で以下のコマンドを実行してください：

    (myvenv) ~/djangogirls$ python manage.py migrate

ブログに記事を追加するために、以下のコマンドを実行して、superuserを作ります：

    (myvenv) ~/djangogirls$ python manage.py createsuperuser --username name 

これまでのように、コマンドの `name` はこのチャプターで作ったユーザ名に置き換えてください。メールアドレスとパスワードを入力するように指示されます。

以上で、サーバを起動し、superuserのアカウントでアプリケーションにログインし、新しいデータベースに記事を追加することができるようになりました。
