# Herokuにデプロイしよう（PythonAnywhereだけでなく）

開発者にとって、いくつかのデプロイ先を経験しておくことは得策です。ぜひ、PythonAnywhereだけでなくHerokuにもサイトをデプロイしてみませんか？

[Heroku](http://heroku.com/) は、アクセスがそんなに多くない小さなアプリケーションでは無料ですが、デプロイするのは少し難しいです。

こちらのチュートリアルを参考にしてください：https://devcenter.heroku.com/articles/getting-started-with-django
ですがここでは、より簡単に説明したいと思います。


## `requirements.txt`ファイル

もし作成していなかったら、`requirements.txt`ファイルを作成しましょう。サーバーに何のパッケージがインストールされている必要があるかをHerokuに教えるためのファイルです。

ですがまず、Herokuにデプロイするためにいくつかの新しいパッケージが必要です。`virtualenv`が有効になっているコンソール画面にいって、次のようにタイプします：

    (myvenv) $ pip install dj-database-url gunicorn whitenoise

インストールが終わったら、`djangogirls`のディレクトリ下で次のコマンドをたたきます：

    (myvenv) $ pip freeze > requirements.txt

このコマンドで、インストールされているパッケージ（例えばDjangoなど、あなたが使用しているPythonライブラリ）のリストが書かれた`requirements.txt`というファイルが作成されます。

> __注__：`pip freeze`コマンドは、virtualenv内にインストールされているすべてのPythonライブラリのリストを出力します。`>`をつけると、`pip freeze`の出力をファイルに書き出します。試しに、`> requirements.txt`なしで`pip freeze`コマンドだけたたいて何が起こるか見てみてください！

`requirements.txt`ファイルを開いて、次の行を一番下に追加しましょう：

    psycopg2==2.7.6.1

上記の行は、Heroku上であなたのアプリケーションを動かすのに必要です。


## Procfile

また、ProcfileというものがHerokuには必要です。これは、ウェブサイトを起動するために実行するコマンドをHerokuに伝えます。コードエディタで`djangogirls`ディレクトリ下に`Procfile`というファイルを作成し、次の行を追加してください：

    web: gunicorn mysite.wsgi --log-file -

この行は、`web`というアプリケーションをデプロイすることを意味し、そのために`gunicorn mysite.wsgi`コマンドを走らせようとしています（`gunicorn`とは、Djangoの`runserver`コマンドのより強力なバージョンみたいなものです）。

保存して完了！

## `runtime.txt`ファイル

また、どのバージョンのPythonを使いたいのかをHerokuに知らせる必要があります。コードエディタで`djangogirls`ディレクトリ下に`runtime.txt`というファイルを作成して、次のテキストを書き込んでください（これ以外には何もありません！）：

    python-3.6.4

## `mysite/local_settings.py`

PythonAnywhereよりも制限が厳しいため、Herokuでは私たちがローカルで使用しているもの（コンピュータ上のもの）とは異なる設定を使います。例えば、HerokuではSQLiteではなく、Postgresを使います。そのため、ローカル環境でしか利用できない設定用に、別々のファイルを作成することにします。

それでは、`mysite/local_settings.py`ファイルを作成しましょう。この中には`mysite/settings.py`ファイル内の`DATABASE`設定が含まれます。次のように：

```python
import os
BASE_DIR = os.path.dirname(os.path.dirname(__file__))

DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.sqlite3',
        'NAME': os.path.join(BASE_DIR, 'db.sqlite3'),
    }
}

DEBUG = True
```

では保存しましょう！ :)

## mysite/settings.py

あと一つ必要なのは、 `settings.py` ファイルの修正です。エディタで `mysite/settings.py` を開いて、次のように変更／追加します：

```python
import dj_database_url

...

DEBUG = False

ALLOWED_HOSTS = ['127.0.0.1', '.herokuapp.com']

...

DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql_psycopg2',
        'NAME': 'djangogirls',
        'USER': 'name',
        'PASSWORD': '',
        'HOST': 'localhost',
        'PORT': '',
    }
}

...

db_from_env = dj_database_url.config(conn_max_age=500)
DATABASES['default'].update(db_from_env)
```
Herokuに必要な設定になります。

ファイルを保存しましょう。

## mysite/wsgi.py

 `mysite/wsgi.py` ファイルを開いて、最終行に次の行を追加します：

```python
from whitenoise.django import DjangoWhiteNoise
application = DjangoWhiteNoise(application)
```

できた！

## Herokuアカウント

Heroku CLI（旧Heroku toolbelt）をインストールする必要があります（セットアップ中に既にインストールしている場合は、このステップをスキップしましょう）。: https://devcenter.heroku.com/articles/heroku-cli

> コマンドプロンプトを再起動した後は、 `djangogirls` のフォルダに戻って仮想環境を有効にするのを忘れないでください！（[「Djangoのインストール」の章をチェック](https://tutorial.djangogirls.org/ja/django_installation/#%E4%BB%AE%E6%83%B3%E7%92%B0%E5%A2%83%E3%81%AE%E6%93%8D%E4%BD%9C)）

こちらでHerokuのフリーアカウントを作成しましょう：https://signup.heroku.com/www-home-top

次にこのコマンドを実行して、あなたのコンピュータでHerokuアカウントを認証します：

    $ heroku login

SSHキーがない場合は、このコマンドによって自動的に作成されます。 SSHキーは、Herokuにソースコードをプッシュするために必要です。

## Gitコミット

Herokuはデプロイするのにgitを使います。PythonAnywhereとは違って、Githubを経由せずに直接Herokuにプッシュできます。しかし、最初にいくつか微調整する必要があります。

`djangogirls` ディレクトリの中の `.gitignore` ファイルを開いて、そこに `local_settings.py` を追加してください。gitに `local_settings` を無視させることで、それは私たちのローカルコンピュータにとどまり、Herokuにアップされません。

    *.pyc
    db.sqlite3
    myvenv
    __pycache__
    local_settings.py

そして変更をコミットします。

    $ git status
    $ git add -A .
    $ git commit -m "additional files and changes for Heroku"


## アプリケーション名をつける

あなたのブログを `[あなたのブログの名前].herokuapp.com` で公開しますので、他の誰も取得していない名前を選ぶ必要があります。この名前はDjangoの `blog` アプリや `mysite` 、あるいはこれまでに作成したものに関連している必要はありません。名前は何でもかまいませんが、Herokuでは使える文字の種類が非常に限られています。使用可能なのは、英数小文字とダッシュ( `-` )だけです（大文字やアクセント文字は使えません）。

名前を考えたら（あなたの名前やニックネームが含まれているなど）、このコマンドを実行します。 `djangogirlsblog` のところを自分のアプリケーション名に置き換えてください：

    $ heroku create djangogirlsblog

> __注__: `djangogirlsblog` の部分を、あなたのHerokuのアプリケーション名に置き換えるのを忘れないでください。

名前が思いつかなかったら、代わりに次のように実行することができます。

    $ heroku create

すると、Herokuのほうで自動的に未使用の名前をつけてくれます（おそらく `enigmatic-cove-2527` のようなもの）。

Herokuアプリケーションの名前を変更したい場合は、いつでもこのコマンドで変更できます（ `the-new-name` を使用したい新しい名前に置き換えてください）：

    $ heroku apps:rename the-new-name

> __注__: アプリケーション名を変更した後は、 `[the-new-name].herokuapp.com` にアクセスして確認してください。

## Herokuにデプロイ！

たくさんの設定やインストールをしてきましたが、大丈夫？でも、それらは一度だけでいいんです。もうデプロイできますよ！

`heroku create` コマンドを実行したときに、あなたのリポジトリにHerokuのリモートサーバーが自動的に追加されています。次は、あなたのアプリケーションをデプロイするのに、単にgitでプッシュするだけです：

    $ git push heroku master

> __注__: Herokuがpsycopgをコンパイルしてインストールするので、これを最初に実行したときはおそらく*大量*のログが出力されます。ログの最後のほうで、 `https://yourapplicationname.herokuapp.com/ deployed to Heroku` のような記述を見つけたら、デプロイに成功したと思っていいでしょう。

## アプリケーションにアクセス

あなたのソースコードをHerokuにデプロイし、そしてプロセスタイプを `Procfile`で指定しました（以前に `web` プロセスタイプを選んでいます）。Herokuにこの `web process` を開始するように指示できます。

そのためには、次のコマンドを実行します：

    $ heroku ps:scale web=1

これはHerokuに `web` プロセスのインスタンスを1つだけ実行するように伝えます。私たちのブログアプリケーションは非常に単純なので、あまりパワーを必要とせず、1つのプロセスだけを実行しても問題ありません。Herokuにもっと多くのプロセスを実行するようにすることも可能です（ところで、Herokuはこれらのプロセスを "Dynos"と呼んでいますので、この用語を見ても驚かないでください）。ですが、それはフリープランではできません。

これで、 `heroku open` コマンドを使うと、ブラウザでアプリにアクセスできます。

    $ heroku open

> __注__: エラーページが見えていますね！もう少し説明を加えます。

ブラウザに [https://djangogirlsblog.herokuapp.com/]() のようなURLが表示され、現時点ではおそらくエラーページが表示されます。

このエラーは、Herokuにデプロイしたときにデータベースが新規作成され、それが空であるために起こっています。PythonAnywhereにデプロイしたときと同じように、 `migrate` コマンドと `createsuperuser` コマンドを実行する必要があります。ここでは、あなたのコンピュータ上で特別なコマンドラインが用意されています。 `heroku run` です：

    $ heroku run python manage.py migrate

    $ heroku run python manage.py createsuperuser

コマンドプロンプトから、ユーザー名とパスワードをもう一度入力するように求められます。これらはリモートサーバー上のウェブサイトの管理者ページのログイン情報になります。

ブラウザで更新してください。これで、２つの異なるホスティングプラットフォームにデプロイする方法がわかりました。お気に入りを選んでくださいね :)
