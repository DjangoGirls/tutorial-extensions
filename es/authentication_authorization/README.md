# Tarea: Asegura tu sitio

Puedes haberte dado cuenta que no usaste tu contraseña, además de cuando iniciaste en la interfaz de administrador. También debes haber notado que esto significa que cualquiera puede editar tus post en tu blog. No se tu, pero a mi no me gustaría que cualquiera editara mi blog, así que vamos a hacer algo al respecto.

## Autorizando la edición y creación de post.

Primero vamos a hacer las cosas seguras. Vamos a proteger nuestras vistas `post_new`, `post_edit`, `post_draft_list`, `post_remove` y `post_publish` para que solo usuarios que hayan ingresado puedan acceder a ellas. Django nos da algunas ayudas para hacer esto, llamados _decoradores_. No te preocupes por las tecnicalidades ahora; puedes leer sobre eso después. El decorador que vamos a usar está en el módulo `django.contrib.auth.decorators` y es llamado `login_required`.

Entonces editemos tu `blog/views.py` y agrega estas lineas al comienzo del archivo junto con las demás importaciones:

```python
from django.contrib.auth.decorators import login_required
```

Entonces añade la línea antes de cada una de las vistas `post_new`, `post_edit`, `post_draft_list`, `post_remove` y `post_publish`, (decorándolas) como a continuación:

```python
@login_required
def post_new(request):
    [...]
```

¡Eso es todo! Ahora intenta acceder a `http://127.0.0.1:8000/post/new/. ¿Notas la diferencia?

> Si obtienes un formulario vacío, posiblemente sigues logeado desde el capítulo en la interfaz de administrador. Ve a `http://127.0.0.1:8000/admin/logout/` para salir, y luego `http://127.0.0.1:8000/post/new` de nuevo.

Puedes obtener uno de nuestros amados errores. Esto es muy interesante: El decoradr que añadimos nos redireccionará a la página de ingreso, pero como no está disponible, retorna un "Page not found (404)".

No te olvides del decorador encima de `post_edit`, `post_remove`, `post_draft_list` y `post_publish` también.

!Perfecto!, !hemos alcanzado parte de nuestro objetivo! Ahora otras personas no podrán crear post en nuestro blog. Desafortunadamente, nosortos tampoco podemos crearlos. Así que vamos a arreglarlo a continuación.

## Ingresar usuarios

Ahora podemos intentar hacer muchas cosas mágicas para implementar usuarios y contraseñas y autenticación, pero hacer esto correctamente es complicado. Como django viene con "baterías incluidas", alguien ya ha hecho el trabajo duro por nosotros, así que vamos a utilizarlas.

En `mysite/urls.py` agrega una nueva entrada `path('accounts/login/', views.login, name='login')`. El contenido del archivo debería verse similar a:

```python
from django.urls import include, path
from django.contrib import admin

from django.contrib.auth import views

urlpatterns = [
    path('admin/', admin.site.urls),
    path('accounts/login/', views.login, name='login'),
    path('', include('blog.urls')),
]
```

Luego necesitamos agregar una plantilla para la página de ingreso, así que crearemos el directorio `blog/templates/registration` y dentro un archivo llamado `login.html`.

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

Verás  que también hace uso de la plantilla _base_ para mantener el estilo de tu blog.

La cosa buena quí es que funciona. No necesitamos lidiar con el manejo del for o las contraseñas y asegurarlas. Solamente una cosa mas para hacer. Entonces vamos a agregar esta configuración en `mysite/settings.py`:

```python
LOGIN_REDIRECT_URL = '/'
```

Entonces cuando la página es accesada directamente, la redirecionará a la página de primer nivel, el index (la página de inicio de blog).

## Mejorando el diseño

Ya definimos como autorizar usuarios. Vemos los botones para añadir post. Ahora queremos asegurarnos que el botón de ingreso le aparezca a todos.

Vamos a agregar el botón de ingreso así:

```django
    <a href="{% url 'login' %}" class="top-menu"><span class="glyphicon glyphicon-lock"></span></a>
```

Para esto necesitamos editar la plantilla, así que vamos a abrir `blog/templates/blog/base.html` y cambiarlo, así que la parte en medio de `<body>` se verá así:

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

Debes reconocer el patrón aquí. Hay una condición `if` en la plantilla que verifica si el usuario está autenticado para mostrar los botones de agregar y editar. De otra manera muestra el botón de ingreso.

## Más para usuarios autenticados

Vamos a agregarle algo de dulce a nuestras plantillas mientras estamos ahí. Primero vamos a mostrar algunos detalles de cuando ingresamos. Editemos el archivo `blog/templates/blog/base.html` así:

```django
<div class="page-header">
    {% if user.is_authenticated %}
        <a href="{% url 'post_new' %}" class="top-menu"><span class="glyphicon glyphicon-plus"></span></a>
        <a href="{% url 'post_draft_list' %}" class="top-menu"><span class="glyphicon glyphicon-edit"></span></a>
        <p id="logout" class="top-menu">
            Hello {{ user.username }} 
            <small>
                (<form method="POST" action="{% url 'logout' %}" style="display:inline;">
                    {% csrf_token %}
                    <button type="submit" style="padding:0; border:none; background:none; color:#337ab7; text-decoration:underline; cursor:pointer;">
                        Log out
                    </button>
                </form>)
            </small>
        </p>    
    {% else %}
        <a href="{% url 'login' %}" class="top-menu"><span class="glyphicon glyphicon-lock"></span></a>
    {% endif %}
    <h1><a href="/">Django Girls Blog</a></h1>
</div>
```

Esto agrega un lindo "Hello _&lt;username&gt;_" para recordarle al usuario como ingresó, y que está autenticado. También agrega un enlace de salida del blog -- como puedes ver, aún no funciona. ¡Vamos a arreglarlo!

Decidimos apoyarnos en Django para manejar el inicio de sesión, así que vamos ver si Django nos permite manejar el cierre de la sesión. Mira https://docs.djangoproject.com/en/2.0/topics/auth/default/ y ve si encuentras algo.

¿Terminaste de leer? Por ahora vamos a pensar en agregar una URL en `mysite/urls.py` apuntando a la vista de salida (`django.contrib.auth.views.logout`) así:

```python
from django.urls import include, path
from django.contrib import admin

from django.contrib.auth import views

urlpatterns = [
    path('admin/', admin.site.urls),
    path('accounts/login/', views.LoginView.as_view(), name='login'),
    path('accounts/logout/', views.LogoutView.as_view(next_page='/'), name='logout'),
    path('', include('blog.urls')),
]
```

¡Eso es todo! Si has segido todo lo anterior, en este punto (e hiciste la tarea), tu blog ahora:

 - necesita un nombre de usuario y contraseña para ingresar,
 - necesita haber ingresado para agregar, editar, publicar o eliminar posts
 - puede salir de nuevo
