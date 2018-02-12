# 숙제: 안전한 웹사이트 만들기

관리자 화면을 사용하는 경우를 제외하고는 미리 설정한 비밀번호를 그동안 사용하지 않았어요. 그래서 내 블로그에 누구든지 마음대로 글을 작성하거나 수정할 수 있다는 것을 눈치챘을 거예요. '네가 누군지 모르겠지만, 내 블로그에 글 쓰는 것을 원치 않아!'라는 생각이 들 거에요. 그래서 아무나 들어와서 글을 쓰지 못하도록 뭔가 만들 수 있답니다.

## 블로그 글 작성/수정 편집 허가하기

첫 번째로 보안을 위해 무언가를 할거에요. 우리는 로그인 한 사용자만 글에 접근할 수 있도록 `post_new`, `post_edit`, `post_draft_list`, `post_remove` 그리고 `post_publish`의 뷰를 보호할 거에요. Django 는 _decorators_ 를 사용해 도움을 주고 있어요. decorators는 고급 주제이긴 해요. 지금 기술적인 것에 걱정하지 마세요. 여러분은 나중에 이해하게 될 거에요. Django 에서 제공하는 `django.contrib.auth.decorators` 모듈 안의 decorator를 사용하는데, 이것을 `login_required` 라고 해요.

그다음, `blog/views.py` 을 수정하고 import를 하는 영역에 아래와 같이 한 줄을 추가해 볼 거에요.

```python
from django.contrib.auth.decorators import login_required
```

그리고 각각의 `post_new`, `post_edit`, `post_draft_list`, `post_remove` 그리고 `post_publish` 뷰의 앞에 아래와 같이 `@login_required`라는 한 줄을 추가해 주세요.

```python
@login_required
def post_new(request):
    [...]
```

잘했어요! 이제 `http://localhost:8000/post/new/` 로 접속해보세요. 다른 점을 찾으셨나요?

> 만약 비어있는 폼 페이지만 보인다면, 이미 관리자 화면을 통해 로그인한 상태일 거예요. `http://localhost:8000/admin/logout/` 로 로그아웃을 한 다음,  `http://localhost:8000/post/new` 로 다시 접속해보세요.

사랑스런 에러 중 하나를 보게 될 거에요. 정말 흥미로운 사실 중 하나: 추가한 Decorator는 로그인 페이지로 우리를 돌려보낼 거에요. 아직 사용할 수 없지만, "페이지를 찾을 수 없습니다. (404)" 라는 에러를 나타내죠.

`post_edit`, `post_remove`, `post_draft_list`, `post_publish` 위에도 decorator를 추가하는 것을 잊지 마세요.

야호, 목표지점까지 다 왔어요! 이제 다른 사람들이 내 블로그에 마음대로 글을 쓸 수 없답니다. 그런데 관리자인 나도 새 글을 작성할 수가 없네요. 그럼 작성할 수 있게 수정해 봅시다.


## 사용자 로그인

이제 우리는 사용자와 패스워드 그리고 인증을 구현할 수 있는 많은 종류의 마법의 도구를 사용할 수 있었어요. 그러나 모든 도구를 정확히 사용하는 것은 꽤 복잡하지요. Django 에서 제공되는 인증 도구를 좀 더 사용할 수 있게 만들 거에요.

`mysite/urls.py` 에서 `url(r'^accounts/login/$', views.login, name='login')`. 부분에 url을 추가해요. 아래처럼 말이죠.

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

다음으로는 로그인 페이지 템플릿이 필요합니다. `blog/templates/registration` 디렉토리를 생성하고, `login.html` 파일을 그 디렉토리 안에 넣으세요.

```django
{% extends "blog/base.html" %}

{% block content %}
    {% if form.errors %}
        <p>이름과 비밀번호가 일치하지 않습니다. 다시 시도해주세요.</p>
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

전반적인 블로그의 모양과 느낌을 _기본_ 템플릿을 적용해 나타낼 수 있어요.

여기서 좋은 점은 _just works<sup>TM</sup>_ 라는 거에요. 폼을 제출하거나 패스워드와 보완을 처리할 필요가 없어요. 단지 한 가지 남았다면, 우리는 `mysite/settings.py`에 설정만 추가하면 됩니다.:

```python
LOGIN_REDIRECT_URL = '/'
```

이제 바로 직접 로그인하면, 최상 위 index 레벨에서 로그인이 성공적으로 될 거에요.

## 레이아웃 개선하기

지금까지는 인증된 자만 글을 추가/수정/발행하는 버튼을 볼 수 있도록 했어요. 이제는 모두에게 로그인 버튼을 보이게 할거에요.

아래에 보이는 코드처럼 로그인 버튼을 추가해 볼 거에요.:

```django
    <a href="{% url 'login' %}" class="top-menu"><span class="glyphicon glyphicon-lock"></span></a>
```

이렇게 버튼을 추가하기 위해서는 템플릿을 수정해 줘야 해요. `blog/templates/blog/base.html` 파일을 열어 `<body>` 태그를 아래처럼 보이게 변경해 보세요.:

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

어떤 패턴이 있음을 눈치채셨나요? 템플릿 내에서 if 문을 통해 인증 여부를 점검하고 있습니다. 인증이 되었을 시에는 추가/수정 버튼을 노출하고, 인증이 되지 않았을 시에는 로그인 버튼을 노출했습니다.

## 인증된 사용자를 위한 기능 추가하기

템플릿에 달콤한 설탕을 추가해볼게요. 먼저 현재 우리가 로그인 중이라는 표시를 나타내주는 몇 가지 도구들을 추가할 것입니다. `blog/templates/blog/base.html` 를 아래와 같이 수정해봅시다.:

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

" _&lt;이름&gt;_님 안녕하세요."라는 멋진 구문을 추가함으로써 인증된 사용자라는 것을 알려줍니다. 또 블로그에서 로그아웃하는 링크를 추가해봅시다. 하지만 아직 작동하지 않네요. 이런, 우리가 인터넷을 망가뜨렸나 봐요. 자, 고쳐봅시다!

Django 가 로그인의 처리를 하게 했듯이, Django 가 로그아웃까지 되는지 살펴봅시다. https://docs.djangoproject.com/en/1.10/topics/auth/default/ 에서 관련 내용을 찾아봅시다.

다 살펴보았나요? `mysite/urls.py` 파일 안에 `django.contrib.auth.views.logout`
뷰를 가리키는 URL 을 추가해야 한다는 것을 알아챘을 거예요. 이렇게 말이에요.:

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

끝났어요! 여기까지 잘 따라왔다면(그리고 숙제를 했다면), 이런 기능을 갖춘 블로그가 만들어졌을 거예요.

 - 로그인을 위해 사용자 이름과 패스워드를 넣을 수 있습니다.
 - 글 생성/수정/게시(/삭제)하기 위해 로그인을 할 수 있습니다.
 - 다시 로그아웃도 할 수 있습니다.