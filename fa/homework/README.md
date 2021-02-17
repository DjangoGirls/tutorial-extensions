# Homework: add more to your website!

وبلاگ ما راه درازی را آمده است اما هنوز جا برای توسعه بیشتر، هست. در ادامه ما ویژگی‌هایی‌هایی مانند پیشونویس برای یک پست وبلاگی و نحوه انتشار آن‌ها، اضافه خواهیم کرد. علاوه بر این امکان پاک کردن پست‌هایی که لازم نداریم را هم ایجاد خواهیم کرد. بسیارعالی!

## ذخیره کردن پست‌های جدید به صورت پیش‌نویس 

در حال حاضر وقتی ما به کمک فرم *New post* یک پست وبلاگی جدید می‌سازیم، این پست مستقیما منتشر می‌شود. برای آنکه این پست به صورت پیش نویس ذخیره شود عبارت زیر را در متدهای `post_new` و `post_edit` در فایل `blog/views.py` **حذف** کنید:

```python
post.published_date = timezone.now()
```

با این روش، پست‌های جدید به جای آنکه بلافاصله منتشر شوند، به عنوان پیش‌نویس ذخیره می‌شوند و بعدا می‌توانیم آن‌ها را بازبینی کنیم. چیزی که الان لازم داریم راهی است پست های پیش‌نویس شده را لیست کنیم و انتشار دهیم، برویم سراغش! 

## صفحه‌ای با لیستی از پست‌های منتشر نشده 

بخش مربوط به کوئری ست‌ها را به یاد میآورید؟ ما ویویی به نام `post_list` ساخته‌ایم که فقط پست‌های منتشر شده را نمایش می‌دهد نمایش می‌دهد (پست‌هایی که مقدار `published_date` برای آن‌ها خالی نیست).

حالا زمان آن است که کار مشابهی را برای برای پست‌های پیش‌نویس انجام دهیم.

Let's add a link in `blog/templates/blog/base.html` in the header. We don't want to show our list of drafts to everybody, so we'll put it inside the `{% if user.is_authenticated %}` check, right after the button for adding new posts.

```django
<a href="{% url 'post_draft_list' %}" class="top-menu"><span class="glyphicon glyphicon-edit"></span></a>
```

Next: urls! In `blog/urls.py` we add:

```python
path('drafts/', views.post_draft_list, name='post_draft_list'),
```

Time to create a view in `blog/views.py`:

```python
def post_draft_list(request):
    posts = Post.objects.filter(published_date__isnull=True).order_by('created_date')
    return render(request, 'blog/post_draft_list.html', {'posts': posts})
```

The line `    posts = Post.objects.filter(published_date__isnull=True).order_by('created_date')` makes sure that we take only unpublished posts (`published_date__isnull=True`) and order them by `created_date` (`order_by('created_date')`).

Ok, the last bit is of course a template! Create a file `blog/templates/blog/post_draft_list.html` and add the following:

```django
{% extends 'blog/base.html' %}

{% block content %}
    {% for post in posts %}
        <div class="post">
            <p class="date">created: {{ post.created_date|date:'d-m-Y' }}</p>
            <h1><a href="{% url 'post_detail' pk=post.pk %}">{{ post.title }}</a></h1>
            <p>{{ post.text|truncatechars:200 }}</p>
        </div>
    {% endfor %}
{% endblock %}
```

It looks very similar to our `post_list.html`, right?

Now when you go to `http://127.0.0.1:8000/drafts/` you will see the list of unpublished posts.

Yay! Your first task is done!

## Add publish button

It would be nice to have a button on the blog post detail page that will immediately publish the post, right?

Let's open `blog/templates/blog/post_detail.html` and change these lines:

```django
{% if post.published_date %}
    <div class="date">
        {{ post.published_date }}
    </div>
{% endif %}
```

into these:

```django
{% if post.published_date %}
    <div class="date">
        {{ post.published_date }}
    </div>
{% else %}
    <a class="btn btn-default" href="{% url 'post_publish' pk=post.pk %}">Publish</a>
{% endif %}
```

As you noticed, we added `{% else %}` line here. That means, that if the condition from `{% if post.published_date %}` is not fulfilled (so if there is no `published_date`), then we want to do the line `<a class="btn btn-default" href="{% url 'post_publish' pk=post.pk %}">Publish</a>`. Note that we are passing a `pk` variable in the `{% url %}`.

Time to create a URL (in `blog/urls.py`):

```python
path('post/<pk>/publish/', views.post_publish, name='post_publish'),
```

and finally, a *view* (as always, in `blog/views.py`):

```python
def post_publish(request, pk):
    post = get_object_or_404(Post, pk=pk)
    post.publish()
    return redirect('post_detail', pk=pk)
```

Remember, when we created a `Post` model we wrote a method `publish`. It looked like this:

```python
def publish(self):
    self.published_date = timezone.now()
    self.save()
```

Now we can finally use this!

And once again after publishing the post we are immediately redirected to the `post_detail` page!

![Publish button](images/publish2.png)

Congratulations! You are almost there. The last step is adding a delete button!

## Delete post

Let's open `blog/templates/blog/post_detail.html` once again and add this line:

```django
<a class="btn btn-default" href="{% url 'post_remove' pk=post.pk %}"><span class="glyphicon glyphicon-remove"></span></a>
```

just under a line with the edit button.

Now we need a URL (`blog/urls.py`):

```python
path('post/<pk>/remove/', views.post_remove, name='post_remove'),
```

Now, time for a view! Open `blog/views.py` and add this code:

```python
def post_remove(request, pk):
    post = get_object_or_404(Post, pk=pk)
    post.delete()
    return redirect('post_list')
```

The only new thing is to actually delete a blog post. Every Django model can be deleted by `.delete()`. It is as simple as that!

And this time, after deleting a post we want to go to the webpage with a list of posts, so we are using `redirect`.

Let's test it! Go to the page with a post and try to delete it!

![Delete button](images/delete3.png)

Yes, this is the last thing! You completed this tutorial! You are awesome!
