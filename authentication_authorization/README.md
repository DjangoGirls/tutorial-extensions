# Lição de casa: Adicionando segurança para o seu site

Você deve ter notado que você não tem que usar sua senha, independentemente de quando usamos a interface de administração. Você também pode ter notado que isto significa que qualquer pessoa pode adicionar ou editar mensagens em seu blog. Eu não sei você, mas eu não quero que qualquer pessoa poste no meu blog. Então vamos fazer algo sobre isso.

## Autorizando adicionar/editar posts

Primeiro vamos tornar as coisas seguras. Vamos proteger o nosso `post_new`, `post_edit`, `post_draft_list`, `post_remove` e `post_publish` views, de modo que somente usuários registrados possam acessá-los. Django fornece algumas agradáveis ajudas para uso, o tipo de tópico avançado, _decorators_. Não se preocupe com os detalhes técnicos agora, você pode ler sobre isso mais tarde. O decorator que vamos usar se encontra no módulo do Django `django.contrib.auth.decorators` e é chamado `login_required`.

Então edite seu `blog/views.py` e adicione estas linhas na parte superior junto com o resto das importações:

```
from django.contrib.auth.decorators import login_required
```

Em seguida, adicione uma linha antes de cada `post_new`, `post_edit`, `post_draft_list`, `post_remove` and `post_publish` views (decorá-los) como o seguinte:

```
@login_required
def post_new(request):
    [...]
```

É isso aí! Agora tente acessar `http://localhost:8000/post/new/`, notou a diferença?

> Se você só tem o formulário vazio, você provavelmente ainda está conectado na parte da administração. Vá para `http://localhost:8000/admin/logout/` para sair, em seguida, Vá para `http://localhost:8000/post/new` novamente.

Você deve obter um dos erros amados. Este erro é bastante interessante, na verdade: O decorator que nós adicionamos antes está redirecionando para a página de login. Mas isso ainda não está disponível, por isso está dando "página não encontrada (404)".

Não se esqueça de adicionar o decorator na parte de cima no `post_edit`, `post_remove`, `post_draft_list` e `post_publish` também.

Uaaau, nós alcançamos parte da meta! Outras pessoas não podem mais criar posts em nosso blog. Infelizmente, não podemos criar mais posts também. Então, vamos consertar isso no próximo passo.

## Acesso aos usuários

Agora nós podemos tentar fazer muita coisa mágica para implementar usuários e senhas e autenticação, mas fazer esse tipo de coisa corretamente é bastante complicado. Como Django é "batteries included", alguém fez o trabalho duro para nós, por isso vamos continuar a fazer uso do material de autenticação fornecido.

No seu `mysite/urls.py` adicione a url `url(r'^accounts/login/$', 'django.contrib.auth.views.login')`. Portanto, o arquivo agora deve ser semelhante a este:

```
from django.conf.urls import patterns, include, url

from django.contrib import admin
admin.autodiscover()

urlpatterns = patterns('',
    url(r'^admin/', include(admin.site.urls)),
    url(r'^accounts/login/$', 'django.contrib.auth.views.login'),
    url(r'', include('blog.urls')),
)
```

Então precisamos de um modelo para a página de login, assim criamos um diretório `blog/templates/registration` e um arquivo dentro chamado `login.html`:

```
{% extends "blog/base.html" %}

{% block content %}

{% if form.errors %}
<p>Seu nome de usuário e senha não coincidem. Por favor, tente novamente.</p>
{% endif %}

<form method="post" action="{% url 'django.contrib.auth.views.login' %}">
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

<input type="submit" value="Entrar" />
<input type="hidden" name="next" value="{{ next }}" />
</form>

{% endblock %}
```

Você vai ver que isso também faz uso do nosso modelo-base para a aparência geral do seu blog.

O bom aqui é que isso _simplesmente funciona[TM]_. Nós não temos que lidar com a manipulação de envio de formulários nem com senhas e assegurar-lhes. Só uma coisa devemos fazer aqui, devemos acrescentar uma configuração em `mysite/settings.py`:

```
LOGIN_REDIRECT_URL = '/'
```

Agora, quando o login é acessado diretamente, ele irá redirecionar para o index se o login for bem-sucedido.

## Melhorar o layout

Então, agora temos a certeza que somente usuários autorizados (ie. us) podem adicionar, editar ou publicar mensagens. Mas todos ainda consegue ver os botões adicionar ou editar mensagens, vamos esconder os botões para os usuários que não estão logados. Para isso, precisamos editar os modelos, então vamos começar com o modelo base `blog/templates/blog/base.html`:

```
    <body>
        <div class="page-header">
            {% if user.is_authenticated %}
            <a href="{% url 'post_new' %}" class="top-menu"><span class="glyphicon glyphicon-plus"></span></a>
            <a href="{% url 'post_draft_list' %}" class="top-menu"><span class="glyphicon glyphicon-edit"></span></a>
            {% else %}
            <a href="{% url 'django.contrib.auth.views.login' %}" class="top-menu"><span class="glyphicon glyphicon-lock"></span></a>
            {% endif %}
            <h1><a href="{% url 'blog.views.post_list' %}">Django Girls</a></h1>
        </div>
        <div class="content">
            <div class="row">
                <div class="col-md-8">
                {% block content %}
                {% endblock %}
                </div>
            </div>
        </div>
    </body>
```

Você pode reconhecer o padrão aqui. Há uma condição 'se' dentro do modelo que verifica a existência de usuários autenticados para mostrar os botões de edição. Caso contrário, ele mostra um botão de login.

*Lição de casa*: Edite o modelo `blog/templates/blog/post_detail.html` para mostrar apenas os botões de edição para usuários autenticados.

## Mais informações sobre usuários autenticados

Vamos adicionar algumas coisas bem legais em nossos modelos para melhorar a experiência do usuário. Primeiro vamos adicionar algumas coisas para mostrar que está logado. Edite o `blog/templates/blog/base.html` assim:

```
        <div class="page-header">
            {% if user.is_authenticated %}
            <a href="{% url 'post_new' %}" class="top-menu"><span class="glyphicon glyphicon-plus"></span></a>
            <a href="{% url 'post_draft_list' %}" class="top-menu"><span class="glyphicon glyphicon-edit"></span></a>
            <p class="top-menu">Olá {{ user.username }}<small>(<a href="{% url 'django.contrib.auth.views.logout' %}">Sair</a>)</small></p>
            {% else %}
            <a href="{% url 'django.contrib.auth.views.login' %}" class="top-menu"><span class="glyphicon glyphicon-lock"></span></a>
            {% endif %}
            <h1><a href="{% url 'blog.views.post_list' %}">Django Girls</a></h1>
        </div>
```

Isso adiciona um agradável "Olá &lt;username&gt;" para nos lembrar quem somos e que estamos autenticados. Além disso, isso adiciona um link para deslogar do blog. Mas, como você pode notar, isso não está funcionando ainda. Oh Nooz, nós quebramos a internetz! Vamos corrigi-lo!

Decidimos contar com Django para lidar com o login, vamos ver se Django também pode lidar com o deslogar para nós. Verifique https://docs.djangoproject.com/en/1.8/topics/auth/default/ e veja se você encontra algo.

Terminou a leitura? Você agora deve estar pensando em adicionar uma url (em `mysite/urls.py`) apontando para a `django.contrib.auth.views.logout` view. Assim:

```
from django.conf.urls import patterns, include, url

from django.contrib import admin
admin.autodiscover()

urlpatterns = patterns('',
    url(r'^admin/', include(admin.site.urls)),
    url(r'^accounts/login/$', 'django.contrib.auth.views.login'),
    url(r'^accounts/logout/$', 'django.contrib.auth.views.logout', {'next_page': '/'}),
    url(r'', include('blog.urls')),
)
```

É isso aí! Se você seguiu todos os itens acima até este ponto (e fez o dever de casa), agora você tem um blog onde você

 - precisa de um nome de usuário e senha para entrar,
 - precisa estar logado para adicionar/editar/publicar(/excluir) posts
 - e pode sair novamente
