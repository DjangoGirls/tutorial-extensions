# کار در خانه: وبسایت خود را امن‌تر کنید

ممکن است توجه کرده باشید که شما، برعکس مواقعی که وارد پنل ادمین می‌شوید، برای کار با وبلاگ از پسورد استفاده نمی‌کنید. احتمالاً توجه کرده‌اید که این به معنی آن است که هر کسی می‌تواند پست‌های وبلاگ شما را اصلاح کند یا پستی به آن اضافه کند. در مورد شما نمی‌دانم ولی من دوست ندارم هرکسی بتواند به جای من پست در وبلاگم بگذارد. پس بیایید برای این موضوع کاری بکنیم.

## مجوز‌دار کردن اضافه/اصلاح کردن پست‌ها

اول بیایید کمی امنیت بیشتر ایجاد کنیم. ما می‌خواهیم ویوهای `post_new`، `post_edit`، `post_draft_list`، `post_remove` و `post_publish` را فقط برای افرادی که در سایت لاگین کرده باشند قابل دسترس کنیم. جنگو با ابزارهای خوبی برای این کار تجهیز شده است که به آن _دکوراتور_ می‌گوییم. نگران توضیحات فنی آن نباشید بعداً می‌توانید در مورد دکوراتورها بخوانید. دکوراتوری که ما می‌خواهیم از آن استفاده کنیم در ماژول `django.contrib.auth.decorators` قرار داده شده و نام آن `login_required` است.

پس فایل را اصلاح کنید و این خط را به بالای فایل و در کنار سایر import ها قرار دهید:

```python
from django.contrib.auth.decorators import login_required
```

سپس دقیقاً بالای هرکدام از تابع‌های `post_new`, `post_edit`, `post_draft_list`, `post_remove` و `post_publish`، یک خط مانند زیر اضافه کنید. انگار که به آن تزیینی (decorator) اضافه کرده‌اید.

```python
@login_required
def post_new(request):
    [...]
```

همین بود! حالا سعی کنید آدرس `http://localhost:8000/post/new/` را باز کنید. تفاوت‌ها را  متوجه شدید؟

> اگر یک فرم خالی می‌بینید احتمالاً هنوز به واسطه ورود به بخش ادمین، شما در وب‌سایت لاگین هستید برای لاگ‌اوت کردن به آدرس `http://localhost:8000/admin/logout/` بروید سپس دوباره آدرس `http://localhost:8000/post/new` را باز کنید. 

شما یکی از خطاهای مورد علاقه ما را دریافت خواهید کرد. در واقع این خطا نمونه جالبی است: دکوراتوری که ما اضافه کردیم شما را به صفحه لاگین هدایت می‌کند، اما چون این صفحه هنوز وجود ندارد خطای "Page not found (404)" را نشان می‌دهد.

فراموش نکنید که این دکوراتور را به توابع `post_edit`, `post_remove`, `post_draft_list` و `post_publish` نیز اضافه کنید.

هورا، ما به بخشی از اهدافمان رسیدیم! حالا کسی غیر از ما نمی‌تواند در وبلاگ ما پست منتشر کند. اما متأسفانه خودمان هم دیگر نمی‌توانیم پستی منتشر کنیم، پس بیایید این موضوع را حل کنیم.


## ورود کاربران

الان ما می‌توانیم کارهای جادویی زیادی برای کاربران، پسوردهایشان و سیستم کنترل اعتبار آن‌ها انجام دهیم اما انجام دادن درست این‌ها، کاری نسبتاً پیجیده است. از آنجا که جنگو از قبل مجهز شده و افرادی کارهای سخت آن را برای ما انجام داده‌اند، ما از این ابزارهای اعتبارسنجی استفاده بیشتری خواهیم کرد.

در فایل `mysite/urls.py` یک url به این شکل `path('accounts/login/', views.LoginView.as_view(), name='login')` اضافه کنید، بنابراین فایل شما شبیه به این خواهد شد:

```python
from django.urls import path, include
from django.contrib import admin

from django.contrib.auth import views

urlpatterns = [
    path('admin/', admin.site.urls),
    path('accounts/login/', views.LoginView.as_view(), name='login'),
    path('', include('blog.urls')),
]
```

حالا یک تمپلیت برای صفحه ورود کاربر نیاز داریم، پس یک دایرکتوری به نام `blog/templates/registration`درست کنید و فایلی به نام `login.html` درون آن بسازید و این خطوط را به آن اضافه کنید:

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

می‌بنید که این فایل هم از تمپلیت  _base_  به عنوان شکل‌دهنده کلی صفحه و برای شبیه کردن این صفحه به بقیه وبلاگ، استفاده می‌کند.

نکته بسیار حالب آن این است که کاملاً  _کار_ می‌کند. لازم نیست که برای ثبت فرم‌ها یا پسورد یا امن کردن آن‌ها هیچ کاری انجام بدهیم. تنها کاری که باقی مانده است آن است که برخی تنظیمات را به فایل `mysite/settings.py` اضافه کنیم:

```python
LOGIN_REDIRECT_URL = '/'
```

بنابراین وقتی صفحه لاگین به طور مستقیم در دسترس قرار می‌گیرد، هر لاگین موفق را به بالاترین سطح ایندکس وب‌سایت (یعنی صفحه اصلی وبلاگ ما)، هدایت خواهد کرد.

## بهبود صفحه‌بندی

ما الان همه چیز را مرتب کرده‌ایم که فقط کاربران تأیید شده (مانند خودمان) دکمه‌های اضافه کردن یا اصلاح هر پست را ببینند. حالا می‌خواهیم مطمئن شویم که یک دکمه لاگین کردن برای سایر افراد نمایش داده می‌شود.

ما یک کلید لاگین کردن مانند زیر اضافه خواهیم کرد: 

```django
    <a href="{% url 'login' %}" class="top-menu"><span class="glyphicon glyphicon-lock"></span></a>
```

برای این کار نیاز است که تمپلیت را اصلاح کنیم، پس فایل `blog/templates/blog/base.html` را باز کنید و آن را به شکلی تغییر دهید که بخش بین تگ‌های `<body>`، مانند زیر باشد:

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

احتمالاً الگوی آن را متوجه شده‌اید. یک عبارت شرطی یا if-condition در تمپلیت وجود دارد که معتبر بودن کاربران را بررسی می‌کند تا در صورت معتبر بودن، دکمه اضافه کردن یا اصلاح کردن پست را نمایش دهد، در صورتی که کاربر تأیید شده نباشد دکمه لاگین را نمایش می‌دهد.

## چیزهای بیشتری برای کاربر تأیید اعتبار شده 

بیایید حالا که در این تمپلیت هستیم دستی به سر و گوش آن بکشیم. اول از همه کاری می‌کنیم که نشان دهد ما چه زمانی لاگین کرده‌ایم. فایل `blog/templates/blog/base.html` را مانند زیر اصلاح کنید:

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

این کار یک عبارت "Hello _&lt;username&gt;_"  زیبا را اضافه می‌کند که به ما یادآوری کند با چه نام کاربری لاگین کرده‌ایم. همچنین یک لینک به صفحه لاگ اوت را نیز نمایش می‌دهد، اما احتمالاً متوجه شده‌اید که این دکمه هنوز کارنمی‌کند پس بیایید درستش کنیم!

ما تصمیم گرفتیم برای مدیریت لاگین، به جنگو تکیه کنیم، بیایید ببینیم آیا جنگو لاگ‌اوت را هم مدیریت می‌کند. آدرس https://docs.djangoproject.com/en/2.0/topics/auth/default/ را نگاهی بیندازید و ببینید آیا چیزی می‌فهمید.

مطالعه کردید؟ ممکن است تا الان به این نتیجه رسیده باشید که لازم است یک URL به فایل `mysite/urls.py` اضافه کنید که به صفحه لاگ‌اوت جنگو (مثلاً `django.contrib.auth.views.logout`) ارجاع دهد. چیزی شبیه به این:

```python
from django.urls import path, include
from django.contrib import admin

from django.contrib.auth import views

urlpatterns = [
    path('admin/', admin.site.urls),
    path('accounts/login/', views.LoginView.as_view(), name='login'),
    path('accounts/logout/', views.LogoutView.as_view(next_page='/'), name='logout'),
    path('', include('blog.urls')),
]
```

همین بود! اگر همه مراحل بالا تا اینجا (و همچنین کارهای در خانه) را انجام داده باشید، وبلاگی دارید که

 - یک نام کاربری و یک پسورد برای ورود لازم دارد،
 - لازم است که لاگین کنید تا بتوانید پست‌های وبلاگ را اضافه، حذف، اصلاح و یا منتشر کنید،
 - و می‌توانید از وبلاگ، لاگ‌اوت کنید.
