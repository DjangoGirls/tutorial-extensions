# Tarea: Agregar mas a tu sitio web

Nuestro blog ha recorrido un gran camino hasta aquí, pero aún hay espacio para la mejora. Lo siguiente, vamos a agregar nuevas características para borradores y su publicación. También vamos a agregar eliminación de post que ya no queremos mas.

## Guarda nuevos post como borradores

Actualmente, nosotros creamos nuevos post usando nuestro formulario *New Post* y el post es publicado directamente. En lugar de publicar el post vamos a guardarlos com borradores. **elimina** esta linea en `blog/views.py` en los méetodos `post_new` y `post_edit`:

```python
post.published_date = timezone.now()
```

De esta manera los nuevos post serán guardados como borradores y se podrán revisar después en vez de ser publicados instantáneamente. Todo lo que necesitamos es una manera de listar los borradores. ¡Vamos a hacerlo!

## Página con los post no publicados

¿Recuerdas el capítulo sobre los querysets? Creamos una vista `post_list` que muestra solamente los post publicados (aquellos que tienen un `publication_date` no vacío).

Es tiempo de hacer algo similiar, pero con borradores.

Vamos a añadir un enlace en `blog/templates/blog/base.html` en el encabezado. No queremos mostrar nuestro borradores a todo el mundo, entonces vamos a colocarlo dentro de la verificación `{% raw %}{% if user.is_authenticated %}{% endraw %}`, justo después del botón de agregar posts.

```django
<a href="{% url 'post_draft_list' %}" class="top-menu"><span class="glyphicon glyphicon-edit"></span></a>
```

Siguiente: ¡urls! en `blog/urls.py` vamos a agregar:

```python
path('drafts/', views.post_draft_list, name='post_draft_list'),
```

Tiempo de crear una nueva vista en `blog/views.py`

```python
def post_draft_list(request):
    posts = Post.objects.filter(published_date__isnull=True).order_by('created_date')
    return render(request, 'blog/post_draft_list.html', {'posts': posts})
```

Esta línea `    posts = Post.objects.filter(published_date__isnull=True).order_by('created_date')` se asegura de que solamente vamos a tomar post no publicados (`published_date__isnull=True`) y los ordena por `created_date` (`order_by('created_date')`).

Ok, el último pedazo es la plantilla. Crea un archivo `blog/templates/blog/post_draft_list.html` y agrega lo siguiente:

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

Se ve muy similar a nuestr `post_list.html` ¿verdad?

Ahora cuando vayas a `http://127.0.0.1:8000/drafts/` vas a ver la lista de post no publicados.

¡Bien! ¡Nuestra primera tarea hecha!

## Agrega un botón de publicación

Sería bueno tener un botón en el detalle del post que publique el post inmediatamente, ¿no?

Vamos a abrir `blog/templates/blog/post_detail.html` y cambiar estas líneas:

```django
{% if post.published_date %}
    <div class="date">
        {{ post.published_date }}
    </div>
{% endif %}
```

por estas:

```django
{% if post.published_date %}
    <div class="date">
        {{ post.published_date }}
    </div>
{% else %}
    <a class="btn btn-default" href="{% url 'post_publish' pk=post.pk %}">Publish</a>
{% endif %}
```

Como puedes ver, hemos agregado la línea `{% raw %}{% else %}{% endraw %}`. Esto significa, que la condición de `{% raw %}{% if post.published_date %}{% endraw %}` no es cumplida (entonces no hay `publication_date`), entonces queremos agregar la línea `{% raw %}<a class="btn btn-default" href="{% url 'post_publish' pk=post.pk %}">Publish</a>{% endraw %}`. Nota que estamos pasando la variable `pk` en el `{% raw %}{% url %}{% endraw %}`.

Tiempo de crear una URL (en `blog/urls.py`):

```python
path('post/<pk>/publish/', views.post_publish, name='post_publish'),
```

Y finalmente una *vista* (como siempre, en `blog/views.py`):

```python
def post_publish(request, pk):
    post = get_object_or_404(Post, pk=pk)
    post.publish()
    return redirect('post_detail', pk=pk)
```

Recuerda, cuando creamos el modelo `Post` escribimos un método `publush`. Se veía como esto:

```python
def publish(self):
    self.published_date = timezone.now()
    self.save()
```

¡Ahora finalmente podemos usarlo!

Y de nuevo al publicar el post, ¡somos redirigidos inmediatamente a la página `post_detail`!

![Publish button](images/publish2.png)

¡Felicitaciones! ¡Casi terminas, el último paso es un botón de eliminar!

## Eliminar post

Vamos a abrir `blog/templates/blog/post_detail.html` de nuevo y vamos a añadir esta línea

```django
<a class="btn btn-default" href="{% url 'post_remove' pk=post.pk %}"><span class="glyphicon glyphicon-remove"></span></a>
```

Justo debajo de la línea co el botón editar.

Ahora necesitamos una URL (`blog/urls.py`):

```python
path('post/<pk>/remove/', views.post_remove, name='post_remove'),
```

Ahora, ¡Tiempo para la vista! Abre `blog/views.py` y agrega este código:

```python
def post_remove(request, pk):
    post = get_object_or_404(Post, pk=pk)
    post.delete()
    return redirect('post_list')
```

La única cosa nueva es en realidad, eliminar el post. Cada modelo en django puede ser eliminado con `.delete()`. ¡Así de simple!

Y en este momento, después de eliminar un post, queremos ir a la página web con la lista de posts, por eso usamos el `redirect`.

¡Vamos a probarlo! ¡Vamos a la página con los post e intentemos eliminarlos!

![Delete button](images/delete3.png)

Si, ¡esto es lo último! ¡Acabas de completar este tutorial! ¡Eres asombroso!
