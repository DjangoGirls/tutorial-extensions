# Lição de casa: adicionar mais ao seu site!

Sim, esta é a última coisa que vamos fazer neste tutorial. Você já aprendeu muito! Hora de usar esse conhecimento.

## Página com a lista de posts não publicados

Lembra-se do capítulo sobre querysets? Criamos uma view `post_list` que exibe no blog apenas os posts publicados (aqueles com `published_date` não vazio).

Hora de fazer algo semelhante, mas para os posts não publicados.

Vamos adicionar um link em `blog/templates/blog/base.html` próximo do botão para adicionar novos posts (logo acima `<h1><a href="/">Django Girls Blog</a></h1>` dessa linha!):

    <a href="{% url 'post_draft_list' %}" class="top-menu"><span class="glyphicon glyphicon-edit"></span></a>

Próximo: urls! Em `blog/urls.py` nós adicionamos:

    url(r'^drafts/$', views.post_draft_list, name='post_draft_list'),

Agora crie a view em `blog/views.py`:

    def post_draft_list(request):
        posts = Post.objects.filter(published_date__isnull=True).order_by('created_date')
        return render(request, 'blog/post_draft_list.html', {'posts': posts})

Esta linha `Post.objects.filter(published_date__isnull=True).order_by('created_date')` pega apenas os posts não publicados (`published_date__isnull=True`) e ordena pelo `created_date` (`order_by('created_date')`).

Ok, a última parte, naturalmente é o template! Crie o arquivo `blog/templates/blog/post_draft_list.html` e adicione o seguinte:

    {% extends 'blog/base.html' %}

    {% block content %}
        {% for post in posts %}
            <div class="post">
                <p class="date">created: {{ post.created_date|date:'d-m-Y' }}</p>
                <h1><a href="{% url 'blog.views.post_detail' pk=post.pk %}">{{ post.title }}</a></h1>
                <p>{{ post.text|truncatechars:200 }}</p>
            </div>
        {% endfor %}
    {% endblock %}

Veja que é muito similar ao nosso `post_list.html`, certo?

Agora, quando você acessar `http://127.0.0.1:8000/drafts/` você verá a lista de posts não publicados.

Boa! Sua primeira tarefa está concluída!

## Adicionar botão publicar

Seria bom ter um botão na página de detalhes do post para publicar imediatamente o post, certo?

Vamos abrir o `blog/template/blog/post_detail.html` e alterar estas linhas:

    {% if post.published_date %}
        {{ post.published_date }}
    {% endif %}

para estas:

    {% if post.published_date %}
        {{ post.published_date }}
    {% else %}
        <a class="btn btn-default" href="{% url 'blog.views.post_publish' pk=post.pk %}">Publicar</a>
    {% endif %}

Como você percebeu, nós adicionamos `{% else %}` esta linha. Isso significa que, se a condição `{% if post.published_date %}` não for cumprida (então não há `published_date`), então nós queremos mostrar a linha `<a class="btn btn-default" href="{% url 'post_publish' pk=post.pk %}">Publicar</a>`. Note que estamos passando um `pk` na variável `{% url %}`.

Agora vamos criar a URL (em `blog/urls.py`):

    url(r'^post/(?P<pk>[0-9]+)/publish/$', views.post_publish, name='post_publish'),

e finalmente, criar a *view* (como sempre, em `blog/views.py`):

    def post_publish(request, pk):
        post = get_object_or_404(Post, pk=pk)
        post.publish()
        return redirect('blog.views.post_detail', pk=pk)

Lembre-se, quando criamos o modelo`Post` nós escrevemos um método `publish`. Ele ficou assim:

    def publish(self):
        self.published_date = timezone.now()
        self.save()

Agora podemos finalmente utilizar este!

E mais uma vez após a publicação do post é imediatamente redirecionado para a página `post_detail`!

![Publish button](images/publish2.png)

Parabéns! Você está quase lá. O último passo é adicionar um botão delete!

## Deletar post

Vamos abrir o `blog/templates/blog/post_detail.html` e mais uma vez adicionar esta linha:

    <a class="btn btn-default" href="{% url 'post_remove' pk=post.pk %}"><span class="glyphicon glyphicon-remove"></span></a>

logo abaixo da linha do botão editar.

Agora precisamos de uma URL (`blog/urls.py`):

    url(r'^post/(?P<pk>[0-9]+)/remove/$', views.post_remove, name='post_remove'),

Agora, é hora de criar a view! Abra `blog/views.py` e adicione este código:

    def post_remove(request, pk):
        post = get_object_or_404(Post, pk=pk)
        post.delete()
        return redirect('blog.views.post_list')

A única coisa nova é realmente excluir uma postagem do blog. Todo modelo Django pode ser excluído por `.delete ()`. É tão simples quanto isso!

E desta vez, após a exclusão de um post queremos ir para a página com uma lista de posts, por isso estamos usando `redirect`.

Vamos testá-lo! Vá para uma página com um post e tente excluí-lo!

![Delete button](images/delete3.png)

Sim, esta é a última parte! Você completou este tutorial! Você é incrível!
