# PostgreSQL installation

> Part of this chapter is based on tutorials by Geek Girls Carrots (http://django.carrots.pl/).

> Parts of this chapter is based on the [django-marcador
tutorial](http://django-marcador.keimlink.de/) licensed under Creative Commons
Attribution-ShareAlike 4.0 International License. The django-marcador tutorial
is copyrighted by Markus Zapke-Gründemann et al.


## Windows

PostgresをWindowsにインストールする一番簡単な方法は、次のリンクから入手できるプログラムを使うことです： http://www.enterprisedb.com/products-services-training/pgdownload#windows

お使いのOSで利用できる最新のバージョンを選んでください。インストーラをダウンロードし、実行し、次のリンクの指示に従ってください：http://www.postgresqltutorial.com/install-postgresql/
インストール先のディレクトリは次のステップで必要ですので、控えておいてください。（通常は `C:\Program Files\PostgreSQL\9.3` のようになります）

## Mac OS X

一番簡単な方法は、Mac OSの他のアプリケーションと同様に、無料の [Postgres.app](http://postgresapp.com/) をダウンロードし、インストールすることです。

ダウンロードし、「アプリケーション」ディレクトリにドラッグし、ダブルクリックで実行してください。これでおしまいです！

[ここ](http://postgresapp.com/documentation/cli-tools.html)に書かれているように、`PATH` 変数にPostgresのコマンドラインツールを追加する必要があります。

## Linux

インストール手順はディストリビューションごとに異なります。以下にあるのは、UbuntuとFedora向けのコマンドです。これら以外のディストリビューションをお使いの場合は、[PostgreSQLのドキュメント
を参照してください](https://wiki.postgresql.org/wiki/Detailed_installation_guides#General_Linux)。

### Ubuntu

次のコマンドを実行してください：

    sudo apt-get install postgresql postgresql-contrib

### Fedora

次のコマンドを実行してください：

    sudo yum install postgresql93-server

# Create database

次に、最初のデータベースと、それにアクセスできるユーザを作ります。PostgreSQLでは好きなだけデータベースとユーザを作成できるので、複数のサイトを稼働させる場合は、サイトごとにデータベースを作ることになるでしょう。

## Windows

Windowsをお使いの場合は、完了する必要のあるいくつかの追加のステップがあります。今の時点では、ここで行う設定を理解することは重要ではありませんが、何が行われているか興味がある方はお気軽にコーチに質問してください。

1. コマンドプロンプトを開きます（「スタートメニュー」→「アクセサリ」→「コマンドプロンプト」）
2. 次のように入力して、Enterキーを押して実行します：`setx PATH "%PATH%;C:\Program Files\PostgreSQL\9.3\bin"`。コマンドプロンプトを右クリックして `ペースト` を選択することで、コピー&ペーストもできます。`"%PATH%;"` に続く部分が、インストール時に控えたパスに `\bin` をつけ足したものと同じか確認してくださいね。`SUCCESS: Specified value was saved.` というメッセージが現れるでしょう。
3. コマンドプロンプトを閉じて、再度開きます。

## Create the database

ますは、`psql` コマンドを実行して、Postgresコンソールを起動しましょう。コンソールの起動の仕方は覚えていますか？
>Mac OS Xでは、`ターミナル`アプリケーションを起動して行います（「アプリケーション」→「ユーティリティ」の中にあります）。Linuxでは、おそらく「アプリケーション」→「アクセサリ」→「ターミナル」となるでしょう。Windowsでは、「スタートメニュー」→「すべてのプログラム」→「アクセサリ」→「コマンドプロンプト」となります。さらに言うと、Windowsでは、インストール時に

>On Mac OS X you can do this by launching the `Terminal` application (it's in Applications → Utilities). On Linux, it's probably under Applications → Accessories → Terminal. On Windows you need to go to Start menu → All Programs → Accessories → Command Prompt. Furthermore, on Windows, `psql` might require logging in using the username and password you chose during installation. If `psql` is asking you for a password and doesn't seem to work, try `psql -U <username> -W` first and enter the password later.

    $ psql
    psql (9.3.4)
    Type "help" for help.
    #

Our `$` now changed into `#`, which means that we're now sending commands to PostgreSQL. Let's create a user with `CREATE USER name;` (remember to use the semicolon):

    # CREATE USER name;

Replace `name` with your own name. You shouldn't use accented letters or whitespace (e.g. `bożena maria` is invalid - you need to convert it into `bozena_maria`). If it goes well, you should get `CREATE ROLE` response from the console.

Now it's time to create a database for your Django project:

    # CREATE DATABASE djangogirls OWNER name;

Remember to replace `name` with the name you've chosen (e.g. `bozena_maria`).  This creates an empty database that you can now use in your project. If it goes well, you should get `CREATE DATABASE` response from the console.

Great - that's databases all sorted!

# Updating settings

Find this part in your `mysite/settings.py` file:

```python
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.sqlite3',
        'NAME': os.path.join(BASE_DIR, 'db.sqlite3'),
    }
}
```

And replace it with this:

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

Remember to change `name` to the user name that you created earlier in this chapter.

# Installing PostgreSQL package for Python

First, install Heroku Toolbelt from https://toolbelt.heroku.com/ While we will need this mostly for deploying your site later on, it also includes Git, which might come in handy already.

Next up, we need to install a package which lets Python talk to PostgreSQL - this is called `psycopg2`. The installation instructions differ slightly between Windows and Linux/OS X.

## Windows

For Windows, download the pre-built file from http://www.stickpeople.com/projects/python/win-psycopg/

Make sure you get the one corresponding to your Python version (3.4 should be the last line) and to the correct architecture (32 bit in the left column or 64 bit in the right column).

Rename the downloaded file and move it so that it's now available at `C:\psycopg2.exe`.

Once that's done, enter the following command in the terminal (make sure your `virtualenv` is activated):

    easy_install C:\psycopg2.exe

## Linux and OS X

Run the following in your console:

    (myvenv) ~/djangogirls$ pip install psycopg2

If that goes well, you'll see something like this

    Downloading/unpacking psycopg2
    Installing collected packages: psycopg2
    Successfully installed psycopg2
    Cleaning up...

---

Once that's completed, run `python -c "import psycopg2"`. If you get no errors, everything's installed successfully.

# Applying migrations and creating a superuser

In order to use the newly created database in your website project, you need to apply all the migrations. In your virtual environment run the following code:

    (myvenv) ~/djangogirls$ python manage.py migrate

To add new posts to your blog, you also need to create a superuser by running the code:

    (myvenv) ~/djangogirls$ python manage.py createsuperuser --username name

Remember to replace `name` with the username. You will be prompted for email and password.

Now you can run the server, log into your application with the superuser account and start adding posts to your new database.
