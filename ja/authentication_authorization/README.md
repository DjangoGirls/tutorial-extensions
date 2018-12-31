# 宿題: ウェブサイトをセキュアにする

管理画面を使用する以外で、パスワードを使用する必要がないことに気づいたかもしれません。これは、誰でもあなたのブログの投稿を追加したり編集したりできるということを意味するということにも気づいたかもしれませんね。私はあなたのことを存じ上げませんが、私は他の誰かが私のブログに投稿することは望んでいません。なので、対策をほどこしましょう。

## 投稿の追加／編集の承認

まずはセキュアにしてみましょう。`post_new`, `post_edit`, `post_draft_list`, `post_remove`、 `post_publish`のビューを保護し、ログインしているユーザだけがアクセスできるようにします。そうするために、Djangoには、_デコレータ_ と呼ばれる素敵なヘルパーが同梱されています。今は技術的なことは気にしないでください。あとでフォローします。使いたいデコレータは、Djangoの`django.contrib.auth.decorators`モジュールに同梱された、`login_required`というものです。

それでは、`blog/views.py`を編集し、一番上に次のimport文を追加してください：

```python
from django.contrib.auth.decorators import login_required
```

次に`post_new`, `post_edit`, `post_draft_list`, `post_remove`、 `post_publish`のそれぞれのビューの前に、以下のように１行追加して（ビューをデコレートして）ください：

```python
@login_required
def post_new(request):
    [...]
```

以上です！`http://localhost:8000/post/new/`にアクセスしてみましょう。違いに気づきましたか？

> 空のフォームが出てきたときは、管理画面のチャプターからログインしたままかもしれません。`http://localhost:8000/admin/logout/`にアクセスしてログアウトして、再度`http://localhost:8000/post/new/`にアクセスしてみてください。

大切なエラーの一つが見えましたね。実際、これはとても興味深いです。追加したデコレータはログインページにリダイレクトするように動きますが、ログインページがまだ利用できないので、「Page not found (404)」が表示されています。

`post_edit`, `post_remove`, `post_draft_list`、 `post_publish`にもデコレータを追加することを忘れないでくださいね。

やった！目標に一歩近づきました！！もう、他の人が私たちのブログに投稿することはできません。しかし残念ながら、私たちも投稿することができなくなってしまいました。次はそれを修正しましょう。

## ユーザーログイン

ユーザーとパスワードと認証を実装するために多くの素敵なものを使ってみることもできますが、正しく使うのはやや複雑になります。Djangoは「電池同梱販売」（訳注：すべて揃っていることを表す）なので、ユーザーとパスワードと認証の実装という困難な仕事を誰かが成し遂げています。そこで、Djangoで提供されている認証ツールをさらに活用します。

`mysite/urls.py`の中に、`url(r'^accounts/login/$', views.login, name='login')`というurlを追加します。ファイルは次のようになるでしょう：

```python
from django.conf.urls import include, url
from django.contrib import admin

from django.contrib.auth import views

urlpatterns = [
    url(r'^admin/', admin.site.urls),
    url(r'^accounts/login/$', views.login, name='login'),
    url(r'', include('blog.urls')),
]
```

そして、ログインページのテンプレートが必要なので、`blog/templates/registration`というディレクトリを作って、中に`login.html`というファイルを作成してください：

```django
{% extends "blog/base.html" %}

{% block content %}
    {% if form.errors %}
        <p>Your username and password didn't match. Please try again.</p>
    {% endif %}

    <form method="post" action="{% url 'login' %}">
    {% csrf_token %}
        <table>
        <tr>
            <td>{{ form.username.label_tag }}</td>
            <td>{{ form.username }}</td>
        </tr>
        <tr>
            <td>{{ form.password.label_tag }}</td>
            <td>{{ form.password }}</td>
        </tr>
        </table>

        <input type="submit" value="login" />
        <input type="hidden" name="next" value="{{ next }}" />
    </form>
{% endblock %}
```

ブログの全体的な見た目を統一するために、_基本_ テンプレートを読み込んでいることがわかります。

いいところは、_これだけで動作する_ ことです。フォーム送信の処理やパスワードの取り扱い、パスワードの保護は必要ありません。もう少し、やることが残っています。`mysite/settings.py`に設定を追加します：

```python
LOGIN_REDIRECT_URL = '/'
```

ログインページに直接アクセスして、ログインが成功したときにトップレベルのインデックス（ブログのホームページ）にリダイレクトします。

## レイアウトを改善する

投稿を追加したり編集したりするためのボタンを認証されたユーザー（つまり、私たち）のみに見せるようにすでに設定しています。今度は、誰もがアクセスしたときにログインボタンを見せたいですね。

次のようにログインボタンを追加してみましょう：

```django
    <a href="{% url 'login' %}" class="top-menu"><span class="glyphicon glyphicon-lock"></span></a>
```

テンプレートを編集する必要があるので、`blog/templates/blog/base.html`を開いて、`<body>`と`</body>`タグの中を次のように変更してください。

```django
<body>
    <div class="page-header">
        {% if user.is_authenticated %}
            <a href="{% url 'post_new' %}" class="top-menu"><span class="glyphicon glyphicon-plus"></span></a>
            <a href="{% url 'post_draft_list' %}" class="top-menu"><span class="glyphicon glyphicon-edit"></span></a>
        {% else %}
            <a href="{% url 'login' %}" class="top-menu"><span class="glyphicon glyphicon-lock"></span></a>
        {% endif %}
        <h1><a href="/">Django Girls Blog</a></h1>
    </div>
    <div class="content container">
        <div class="row">
            <div class="col-md-8">
            {% block content %}
            {% endblock %}
            </div>
        </div>
    </div>
</body>
```

ここで、あるパターンがあることに気づくでしょう。`if`条件式で、認証されたユーザーには追加／編集ボタンが表示されるようにしています。認証されていない場合は、ログインボタンが表示されます。

## 認証されたユーザーへの追加項目

ログイン状態のユーザーに向けて、テンプレートにもう少し付け加えましょう。まず、ログイン時に表示されるものを追加します。`blog/templates/blog/base.html`を次のように修正します：

```django
<div class="page-header">
    {% if user.is_authenticated %}
        <a href="{% url 'post_new' %}" class="top-menu"><span class="glyphicon glyphicon-plus"></span></a>
        <a href="{% url 'post_draft_list' %}" class="top-menu"><span class="glyphicon glyphicon-edit"></span></a>
        <p class="top-menu">Hello {{ user.username }} <small>(<a href="{% url 'logout' %}">Log out</a>)</small></p>
    {% else %}
        <a href="{% url 'login' %}" class="top-menu"><span class="glyphicon glyphicon-lock"></span></a>
    {% endif %}
    <h1><a href="/">Django Girls Blog</a></h1>
</div>
```

この追加によって、"Hello _&lt;username&gt;_" というふうに、何のユーザーでログインしているか、認証されているかがわかるようになります。また、ブログからログアウトするリンクを追加していますが、お気づきのとおり動いていません。次はそれを修正しましょう！

ログイン処理をDjangoに頼ったように、ログアウト処理も頼れるかどうか見てみましょう。https://docs.djangoproject.com/en/1.10/topics/auth/default/ をチェックしてみてください。

読みました？Djangoのログアウトビュー（例えば`django.contrib.auth.views.logout`）を示すurlを次のように`mysite/urls.py`に追加することを思いついたんではないでしょうか：

```python
from django.conf.urls import include, url
from django.contrib import admin

from django.contrib.auth import views

urlpatterns = [
    url(r'^admin/', admin.site.urls),
    url(r'^accounts/login/$', views.login, name='login'),
    url(r'^accounts/logout/$', views.logout, name='logout', kwargs={'next_page': '/'}),
    url(r'', include('blog.urls')),
]
```

以上です！これらのことをやり終えたら（そして宿題をやったら）、あなたのブログは次のようになっています。

 - ログイン時にはユーザー名とパスワードが必須である
 - 投稿を追加、編集、公開、削除時にはログインが必須である
 - そして、ログアウトできる
