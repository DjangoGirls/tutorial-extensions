# Homework: create comment model

Currently, we only have a Post model. What about receiving some feedback from your readers and letting them comment?

## Creating comment blog model

Let's open `blog/models.py` and append this piece of code to the end of file:

```python
class Comment(models.Model):
    post = models.ForeignKey('blog.Post', related_name='comments')
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

You can go back to the **Django models** chapter in the tutorial if you need a refresher on what each of the field types mean.

In this tutorial extension we have a new type of field:
- `models.BooleanField` - this is true/false field.

The `related_name` option in `models.ForeignKey` allows us to have access to comments from within the Post model.

## Create tables for models in your database

Now it's time to add our comment model to the database. To do this we have to tell Django that we made changes to our model. Type `python manage.py makemigrations blog` in your command line. You should see output like this:

    (myvenv) ~/djangogirls$ python manage.py makemigrations blog
    Migrations for 'blog':
      0002_comment.py:
        - Create model Comment

You can see that this command created another migration file for us in the `blog/migrations/` directory. Now we need to apply those changes by typing `python manage.py migrate blog` in the command line. The output should look like this:

```
    (myvenv) ~/djangogirls$ python manage.py migrate blog
    Operations to perform:
      Apply all migrations: blog
    Running migrations:
      Rendering model states... DONE
      Applying blog.0002_comment... OK
```

Our Comment model exists in the database now! Wouldn't it be nice if we had access to it in our admin panel?

## Register Comment model in admin panel

To register the Comment model in the admin panel, go to `blog/admin.py` and add this line:

```python
admin.site.register(Comment)
```

directly under this line:

```python
admin.site.register(Post)
```

Remember to import the Comment model at the top of the file, too, like this:

```python
from django.contrib import admin
from .models import Post, Comment

admin.site.register(Post)
admin.site.register(Comment)
```

If you type `python manage.py runserver` on the command line and go to [http://127.0.0.1:8000/admin/](http://127.0.0.1:8000/admin/) in your browser, you should have access to the list of comments, and also the capability to add and remove comments. Play around with the new comments feature!

## Make our comments visible

Go to the `blog/templates/blog/post_detail.html` file and add the following lines before the `{% endblock %}` tag:

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

Now we can see the comments section on pages with post details.

But it could look a little bit better, so let's add some CSS to the bottom of the `static/css/blog.css` file:

```css
.comment {
    margin: 20px 0px 20px 20px;
}
```

We can also let visitors know about comments on the post list page. Go to the `blog/templates/blog/post_list.html` file and add the line:

```django
<a href="{% url 'post_detail' pk=post.pk %}">Comments: {{ post.comments.count }}</a>
```

After that our template should look like this:

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

## Let your readers write comments

Right now we can see comments on our blog, but we can't add them. Let's change that!

Go to `blog/forms.py` and add the following lines to the end of the file:

```python
class CommentForm(forms.ModelForm):

    class Meta:
        model = Comment
        fields = ('author', 'text',)
```

Remember to import the Comment model, changing the line:

```python
from .models import Post
```

into:

```python
from .models import Post, Comment
```

Now, go to `blog/templates/blog/post_detail.html` and before the line `{% for comment in post.comments.all %}`, add:

```django
<a class="btn btn-default" href="{% url 'add_comment_to_post' pk=post.pk %}">Add comment</a>
```

If you go to the post detail page you should see this error:

![NoReverseMatch](images/url_error.png)

We know how to fix that! Go to `blog/urls.py` and add this pattern to `urlpatterns`:

```python
url(r'^post/(?P<pk>\d+)/comment/$', views.add_comment_to_post, name='add_comment_to_post'),
```

Refresh the page, and we get a different error!

![AttributeError](images/views_error.png)

To fix this error, add this view to `blog/views.py`:

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

Remember to import `CommentForm` at the beginning of the file:

```python
from .forms import PostForm, CommentForm
```


Now, on the post detail page, you should see the "Add Comment" button.

![AddComment](images/add_comment_button.png)

However, when you click that button, you'll see:

![TemplateDoesNotExist](images/template_error.png)


Like the error tells us, the template doesn't exist yet. So, let's create a new one at `blog/templates/blog/add_comment_to_post.html` and add the following code:

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

Yay! Now your readers can let you know what they think of your blog posts!

## Moderating your comments

Not all of the comments should be displayed. As the blog owner, you probably want the option to approve or delete comments. Let's do something about it.

Go to `blog/templates/blog/post_detail.html` and change lines:

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

to:

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

You should see `NoReverseMatch`, because no URL matches the `comment_remove` and `comment_approve` patterns... yet!

To fix the error, add these URL patterns to `blog/urls.py`:

```python
url(r'^comment/(?P<pk>\d+)/approve/$', views.comment_approve, name='comment_approve'),
url(r'^comment/(?P<pk>\d+)/remove/$', views.comment_remove, name='comment_remove'),
```

Now, you should see `AttributeError`. To fix this error, add these views in `blog/views.py`:

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


You'll need to import `Comment` at the top of the file:

```python
from .models import Post, Comment
```

Everything works! There is one small tweak we can make. In our post list page -- under posts -- we currently see the number of all the comments the blog post has received. Let's change that to show the number of *approved* comments there.

To fix this, go to `blog/templates/blog/post_list.html` and change the line:

```django
<a href="{% url 'post_detail' pk=post.pk %}">Comments: {{ post.comments.count }}</a>
```

to:

```django
<a href="{% url 'post_detail' pk=post.pk %}">Comments: {{ post.approved_comments.count }}</a>
```

Finally, add this method to the `Post` model in `blog/models.py`:

```python
def approved_comments(self):
    return self.comments.filter(approved_comment=True)
```

Now your comment feature is finished! Congrats! :-)
