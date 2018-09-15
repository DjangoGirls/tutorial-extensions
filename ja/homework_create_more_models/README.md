# 課題:コメントモデルを作ります。

現在、ポストモデルしかありません。 あなたの読者からいくつかのフィードバックを受け取り、彼らにコメントさせるのはどうでしょうか？


## コメントブログモデルを作りましょう！

Let's open `blog/models.py` and append this piece of code to the end of file:
`blog/models.py`を開き、このコードをファイルの最後に追加しましょう：

```python
class Comment(models.Model):
    post = models.ForeignKey('blog.Post', on_delete=models.CASCADE, related_name='comments')
    author = models.CharField(max_length=200)
    text = models.TextField()
    created_date = models.DateTimeField(default=timezone.now)
    approved_comment = models.BooleanField(default=False)

    def approve(self):
        self.approved_comment = True
        self.save()

    def __str__(self):
        return self.text
```

各フィールドタイプの意味を見直す必要がある場合は、チュートリアルの**Django models**の章に戻ることができます。

このチュートリアルエクステンションの中に、新しいフィールドタイプがある：
- `models.BooleanField` - こちらはture/falseのフィールドです。

`models.ForeignKey`の`related_name`オプションは、ポストモデルの中のコメントにアクセスできるようにします。

## データベース内のモデルのテーブルを作りましょう！

今度は、コメントモデルをデータベースに追加します。 これを行うには、モデルを変更したことをDjangoに伝えなければなりません。 あなたのコマンドラインに `python manage.py makemigrations blog`と入力してください。 次のような出力が表示されます。

    (myvenv) ~/djangogirls$ python manage.py makemigrations blog
    Migrations for 'blog':
      0002_comment.py:
        - Create model Comment

このコマンドは、 `blog / migrations /`ディレクトリに別のマイグレーションファイルを作成したことがわかります。 コマンドラインで `python manage.py migrate blog`とタイプして変更を適用する必要があります。 出力は次のようになります。


```
    (myvenv) ~/djangogirls$ python manage.py migrate blog
    Operations to perform:
      Apply all migrations: blog
    Running migrations:
      Rendering model states... DONE
      Applying blog.0002_comment... OK
```

コメントモデルは今データベースに存在します！ 管理パネルでアクセスできるといいですか？

## アドミンパネルにコメントモデルを登録しましょう

アドミンパネルにコメントモデルを登録するため、`blog/admin.py`に行って以下のラインを追加してください。

```python
admin.site.register(Comment)
```

この行の直下：


```python
admin.site.register(Post)
```

コメント・モデルをファイルの先頭にインポートすることも忘れないようにしてください。

```python
from django.contrib import admin
from .models import Post, Comment

admin.site.register(Post)
admin.site.register(Comment)
```

コマンドラインで`python manage.py runserver`を入力し、あなたのブラウザで
 [http://127.0.0.1:8000/admin/](http://127.0.0.1:8000/admin/) に行くと、あなたのコメントリストにアクセスすることができ、コメントの追加と削除の機能が使えるようになります。新しいコメント機能で遊んでみましょう！

## コメントを見えるようにしましょう！

`blog/templates/blog/post_detail.html`というファイルに行き、`{% endblock %}`タグの前に、以下のラインを追加してください。

```django
<hr>
{% for comment in post.comments.all %}
    <div class="comment">
        <div class="date">{{ comment.created_date }}</div>
        <strong>{{ comment.author }}</strong>
        <p>{{ comment.text|linebreaks }}</p>
    </div>
{% empty %}
    <p>No comments here yet :(</p>
{% endfor %}
```

ここで、投稿の詳細が記載されたページのコメントセクションを見ることができます。

しかしそれは少し良く見えるかもしれませんので、 `static / css / blog.css`ファイルの一番下にいくつかのCSSを追加しましょう：

```css
.comment {
    margin: 20px 0px 20px 20px;
}
```

ポストリストページのコメントについては、訪問者に知らせることもできます。 `blog / templates / blog / post_list.html`ファイルに移動し、次の行を追加します：

```django
<a href="{% url 'post_detail' pk=post.pk %}">Comments: {{ post.comments.count }}</a>
```

その後、テンプレートは次のようになります。


```django
{% extends 'blog/base.html' %}

{% block content %}
    {% for post in posts %}
        <div class="post">
            <div class="date">
                {{ post.published_date }}
            </div>
            <h1><a href="{% url 'post_detail' pk=post.pk %}">{{ post.title }}</a></h1>
            <p>{{ post.text|linebreaksbr }}</p>
            <a href="{% url 'post_detail' pk=post.pk %}">Comments: {{ post.comments.count }}</a>
        </div>
    {% endfor %}
{% endblock content %}
```

## あなたの読者がコメントを書くことができます

今は私たちのブログでコメントを見ることができますが、追加することはできません。 それを変えましょう！


`blog/forms.py`に行き、以下のラインをファイルの最後に追加します。


```python
class CommentForm(forms.ModelForm):

    class Meta:
        model = Comment
        fields = ('author', 'text',)
```

行を変更して、Commentモデルをインポートすることを忘れないでください：

```python
from .models import Post
```

それに：

```python
from .models import Post, Comment
```

今、`blog/templates/blog/post_detail.html`に行き、`{% for comment in post.comments.all %}`の前に以下の行を追加します。

```django
<a class="btn btn-default" href="{% url 'add_comment_to_post' pk=post.pk %}">Add comment</a>
```

もし投稿詳細ページに行くと、以下のエラーが表示されます。

![NoReverseMatch](images/url_error.png)


私たちはそれを修正する方法を知っています！ `blog / urls.py`に行き、このパターンを` urlpatterns`に追加してください：

```python
url(r'^post/(?P<pk>\d+)/comment/$', views.add_comment_to_post, name='add_comment_to_post'),
```

ページを更新したら、また違うエラーが出てきます！

![AttributeError](images/views_error.png)

こちらのエラーを修正するために、以下のビューを追加してください：

```python
def add_comment_to_post(request, pk):
    post = get_object_or_404(Post, pk=pk)
    if request.method == "POST":
        form = CommentForm(request.POST)
        if form.is_valid():
            comment = form.save(commit=False)
            comment.post = post
            comment.save()
            return redirect('post_detail', pk=post.pk)
    else:
        form = CommentForm()
    return render(request, 'blog/add_comment_to_post.html', {'form': form})
```

`CommentForm`をファイルの先頭にインポートすることを忘れないでください：

```python
from .forms import PostForm, CommentForm
```

今、詳細投稿ページで”コメント追加”というボタンが表示されます。

![AddComment](images/add_comment_button.png)

しかし、そのボタンをクリックしたら、次のように表示されます。

![TemplateDoesNotExist](images/template_error.png)


表示されたエラーのように、テンプレートは存在していないですので、`blog/templates/blog/add_comment_to_post.html`で新しいテンプレートを作り、以下のコードを追加してください：

```django
{% extends 'blog/base.html' %}

{% block content %}
    <h1>New comment</h1>
    <form method="POST" class="post-form">{% csrf_token %}
        {{ form.as_p }}
        <button type="submit" class="save btn btn-default">Send</button>
    </form>
{% endblock %}
```

わーい！ 今あなたの読者は、彼らがあなたのブログ記事をどう思っているかを知らせることができます！

## あなたのコメントをモデレートしましょう！

すべてのコメントを表示する必要はありません。 ブログの所有者は、コメントを承認または削除するオプションが必要な場合があります。 それについて何かしましょう。


`blog/templates/blog/post_detail.html`に行き、以下の行を変更してください。

```django
{% for comment in post.comments.all %}
    <div class="comment">
        <div class="date">{{ comment.created_date }}</div>
        <strong>{{ comment.author }}</strong>
        <p>{{ comment.text|linebreaks }}</p>
    </div>
{% empty %}
    <p>No comments here yet :(</p>
{% endfor %}
```

以下に変更してね。

```django
{% for comment in post.comments.all %}
    {% if user.is_authenticated or comment.approved_comment %}
    <div class="comment">
        <div class="date">
            {{ comment.created_date }}
            {% if not comment.approved_comment %}
                <a class="btn btn-default" href="{% url 'comment_remove' pk=comment.pk %}"><span class="glyphicon glyphicon-remove"></span></a>
                <a class="btn btn-default" href="{% url 'comment_approve' pk=comment.pk %}"><span class="glyphicon glyphicon-ok"></span></a>
            {% endif %}
        </div>
        <strong>{{ comment.author }}</strong>
        <p>{{ comment.text|linebreaks }}</p>
    </div>
    {% endif %}
{% empty %}
    <p>No comments here yet :(</p>
{% endfor %}
```


`comment_remove`と` comment_approve`パターンに一致するURLがないので、 `NoReverseMatch`が表示されます。まだ終わってないです！


こちらのエラーを修正するため、以下URLパターンを`blog/urls.py`に追加してください。

```python
url(r'^comment/(?P<pk>\d+)/approve/$', views.comment_approve, name='comment_approve'),
url(r'^comment/(?P<pk>\d+)/remove/$', views.comment_remove, name='comment_remove'),
```

さて、あなたは`AttributeError`が見えます。このエラーを修正し、以下のビューを
`blog/views.py`に追加してください。

```python
@login_required
def comment_approve(request, pk):
    comment = get_object_or_404(Comment, pk=pk)
    comment.approve()
    return redirect('post_detail', pk=comment.post.pk)

@login_required
def comment_remove(request, pk):
    comment = get_object_or_404(Comment, pk=pk)
    comment.delete()
    return redirect('post_detail', pk=comment.post.pk)
```

ファイルの先頭に `Comment`をインポートする必要があります：

```python
from .models import Post, Comment
```

すべて動作します！私たちが作ることができる小さな微調整があります。投稿リストページ（投稿の下）には、現在、ブログ投稿が受け取ったすべてのコメントの数が表示されます。※承認されたコメントの数を表示するように変更しましょう。

これを修正するには、 `blog / templates / blog / post_list.html`に行き、行を変更してください：


```django
<a href="{% url 'post_detail' pk=post.pk %}">Comments: {{ post.comments.count }}</a>
```

下記行に変更してくださいね。

```django
<a href="{% url 'post_detail' pk=post.pk %}">Comments: {{ post.approved_comments.count }}</a>
```
最後に、このメソッドを `blog / models.py`の` Post`モデルに追加してください：

```python
def approved_comments(self):
    return self.comments.filter(approved_comment=True)
```

これであなたのコメント機能は終了です！ おめでとう！ :-)

