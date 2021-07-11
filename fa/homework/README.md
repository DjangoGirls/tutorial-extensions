# کار در خانه: چیزهای بیشتری به وب‌سایت اضافه کنیم!

وبلاگ ما راه درازی را آمده است اما هنوز جا برای توسعه بیشتر، هست. در ادامه ما ویژگی‌هایی‌هایی را مانند ایجاد پیش‌نویس برای یک پست وبلاگی و نحوه انتشار آن‌ها، اضافه خواهیم کرد. علاوه بر این امکان پاک کردن پست‌هایی که لازم نداریم را هم ایجاد خواهیم کرد. بسیارعالی!

## ذخیره کردن پست‌های جدید به صورت پیش‌نویس

در حال حاضر وقتی ما به کمک فرم *New post* یک پست وبلاگی جدید می‌سازیم، این پست مستقیماً منتشر می‌شود. برای آنکه این پست به صورت پیش‌نویس ذخیره شود عبارت زیر را در متدهای `post_new` و `post_edit` در فایل `blog/views.py` **حذف** کنید:

```python
post.published_date = timezone.now()
```

با این روش، پست‌های جدید به جای آنکه بلافاصله منتشر شوند، به عنوان پیش‌نویس ذخیره می‌شوند و بعداً می‌توانیم آن‌ها را بازبینی کنیم. چیزی که الان لازم داریم راهی است که پست‌های پیش‌نویس شده را لیست کنیم و انتشار دهیم، برویم سراغش!

## صفحه‌ای با لیستی از پست‌های منتشر نشده

بخش مربوط به کوئری‌ست‌ها را به یاد می‌آورید؟ ما ویویی به نام `post_list` ساخته‌ایم که فقط پست‌های منتشر شده را نمایش می‌دهد (پست‌هایی که مقدار `published_date` برای آن‌ها خالی نیست).

حالا زمان آن است که کار مشابهی را برای پست‌های پیش‌نویس انجام دهیم.

حالا می‌خواهیم در هدر فایل `blog/templates/blog/base.html` یک لینک اضافه کنیم. از آنجا که  نمی‌خواهیم لیست پست‌های پیش‌نویس را به همه نشان بدهیم، پس این لینک رادر عبارت کنترلی {% raw %}`{% if user.is_authenticated %}`{% endraw %} و دقیقاً بعد از دکمه مربوط به اضافه کردن پست جدید، اضافه می‌کنیم.

```django
<a href="{% url 'post_draft_list' %}" class="top-menu"><span class="glyphicon glyphicon-edit"></span></a>
```

حالا وقت اصلاح urlها در فایل `blog/urls.py` است:

```python
path('drafts/', views.post_draft_list, name='post_draft_list'),
```

وقت ساختن یک ویو در فایل `blog/views.py` است:

```python
def post_draft_list(request):
    posts = Post.objects.filter(published_date__isnull=True).order_by('created_date')
    return render(request, 'blog/post_draft_list.html', {'posts': posts})
```

خط `    posts = Post.objects.filter(published_date__isnull=True).order_by('created_date')`  کنترل می‌کند که ما فقط پست‌های منتشر نشده (`published_date__isnull=True`) را گرفته باشیم و آن‌ها را به ترتیب تاریخ ساخت (`order_by('created_date')`) دریافت کرده باشیم.

بسیار عالی، آخرین بخش کار، ساختن یک تمپلیت است! فایل `blog/templates/blog/post_draft_list.html` را بسازید و خطوط زیر را به آن اضافه کنید:

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

این فایل بسیار شبیه فایل `post_list.html` است، اینطور نیست؟

حالا وقتی به آدرس `http://127.0.0.1:8000/drafts/` بروید لیستی از پست‌های منتشر نشده را خواهید دید.

وای!‌ اولین تمرین شما انجام شد!

## اضافه کردن دکمه انتشار

بسیار جذاب خواهد بود که دکمه‌ای در صفحه جزییات پست داشته باشیم که با زدن آن بلافاصله یک پست پیش‌نویس، منتشر شود، درست است؟

فایل `blog/templates/blog/post_detail.html` را باز کنید و خط‌های زیر را:

```django
{% if post.published_date %}
    <div class="date">
        {{ post.published_date }}
    </div>
{% endif %}
```

به این شکل تغییر دهید:

```django
{% if post.published_date %}
    <div class="date">
        {{ post.published_date }}
    </div>
{% else %}
    <a class="btn btn-default" href="{% url 'post_publish' pk=post.pk %}">Publish</a>
{% endif %}
```

همانطور که توجه کرده‌اید ما خط {% raw %}`{% else %}`{% endraw %} را اضافه کرده‌ایم. این خط به این معنی است که اگر شرایط موجود در عبارت {% raw %}`{% if post.published_date %}`{% endraw %} برآورده نشد (یعنی اگر هیچ تاریخ انتشار یا `published_date` وجود نداشت) می‌خواهیم که خط `<a class="btn btn-default" href="{% url 'post_publish' pk=post.pk %}">Publish</a>` اجرا شود. توجه کنید که ما یک متغیر `pk` را به عبارت {% raw %}`{% url %}`{% endraw %} ارجاع داده‌ایم.

زمان ساختن یک URL (در فایل `blog/urls.py`) است:

```python
path('post/<pk>/publish/', views.post_publish, name='post_publish'),
```

و در نهایت یک *ویو* (مانند همیشه در فایل `blog/views.py`) می‌سازیم:

```python
def post_publish(request, pk):
    post = get_object_or_404(Post, pk=pk)
    post.publish()
    return redirect('post_detail', pk=pk)
```

به یاد داشته باشید، وقتی ما یک مدل `Post` ساختیم یک متد `publish` هم نوشتیم که شبیه به این بود:

```python
def publish(self):
    self.published_date = timezone.now()
    self.save()
```

حالا بالاخره می‌توانیم از آن استفاده کنیم!

و یک بار دیگر بعد از انتشار پست ما بلافاصله به صفحه جزییات پست یا `post_detail` هدایت خواهیم شد!  

![Publish button](images/publish2.png)

تبریک! تقریبا به نتیجه رسیده‌اید. آخرین مرحله اضافه کردن یک کلید حذف پست است.

## حذف پست

بیایید یک بار دیگر فایل `blog/templates/blog/post_detail.html` را باز کنید و این خط را به آن اضافه کنید:

```django
<a class="btn btn-default" href="{% url 'post_remove' pk=post.pk %}"><span class="glyphicon glyphicon-remove"></span></a>
```

دقیقاً زیر خطی که دکمه اصلاح یا Edit button قرار دارد.

حالا یک URL لازم داریم (`blog/urls.py`):

```python
path('post/<pk>/remove/', views.post_remove, name='post_remove'),
```

و الان زمان اضافه کردن یک ویو است! فایل `blog/views.py` را باز کنید و این کد را به آن اضافه کنید:

```python
def post_remove(request, pk):
    post = get_object_or_404(Post, pk=pk)
    post.delete()
    return redirect('post_list')
```

تنها کار باقی مانده آن است که به صورت واقعی پست وبلاگی مورد نظر را پاک کنیم. هر مدل جنگو می‌تواند با دستور `.delete()` حذف شود. واقعاً به همین سادگی است!

و این بار، بعد از پاک کردن یک پست، ما می‌خواهیم به صفحه‌ای شامل لیست همه پست‌ها برویم، بنابراین ما از `redirect` استفاده خواهیم کرد.

حالا بیایید آن را امتحان کنیم! به یک صفحه که در آن پستی هست بروید و آن را پاک کنید!

![Delete button](images/delete3.png)

بله این آخرین مرحله بود! شما این آموزش را تمام کردید! فوق العاده هستید!
