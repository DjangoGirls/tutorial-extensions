# Despliega tu sitio en Heroku (También como PythonAnywhere)

Siempre es bueno como un desarrollador tiene un par de diferentes opciones de despliegue bajo su cinturón. ¿Por qué no tratar de desplegar tu sitio en Heroku, también como PythonAnywhere?

[Heroku](http://heroku.com/) es también gratis para pequeñas aplicaciones que no tienen muchas visitas, pero es un poco mas complicado desplegar.

Te guiaremos por el siguiente tutorial: https://devcenter.heroku.com/articles/getting-started-with-django, pero pegamos aquí para hacerlo mas fácil para ti.

## El archivo `requirements.txt`

Si no lo creaste antes, necesitamos crear un archivo `requirements.txt` para decirle a Heroku que paquetes de Python necesitamos que sean instalados en nuestro servidor.

Pero primero, Heroku necesita que instalemos algunos paquetes. Ve a tu consola con tu `virtualenv` activado y escribe:

```
    (myvenv) $ pip install dj-database-url gunicorn whitenoise
```

Después que la instalación está finalizada, ve a la carpeta `djangogirls` y ejecuta este comando:

```
    (myvenv) $ pip freeze > requirements.txt
```

Esto creará un archivo llamado `requirements.txt` con una lista de paquetes instalados. (Librerías Python que estás usando, por ejemplo Django :)

>__Nota__: `pip freeze` imprime la lista de todas las librerías instaladas en tu ambiente virtual, y el `>` toma la salida de `pip freeze` y la coloca en un archivo. ¡Trata ejecutar `pip freeze` sin el `< requirements.txt` para ver lo que pasa!

Abre este archivo y añada esto al final:

```
    psycopg2==2.6.2
```

Esta línea se necesita para que tu aplicación funcione en Heroku.


## Procfile

Otra cosa que Heroku necesita es un `Procfile`. Esto le dice a Heroku que comandos ejecutar para iniciar nuestro sitio web. Ve a tu editor de código y crea un archivo llamado `Procfile` en la carpeta `djangogirls` y agrega esta línea:

```
    web: gunicorn mysite.wsgi --log-file -
```

Esta línea significa que vas a desplegar una aplicación `web` y lo vas a hacer ejecutando el comando `gunicorn mysite.wsgi` (`gunicorn` es un programa que es la versión mas poderosa del comando de django `runserver`).

Entonces guardalo. ¡Listo!

## El archivo `runtime.txt`

También necesitamos decirle a Heroku que versión de Python vamos a usar. Esto es hecho creando un archivo `runtime.txt` en la carpeta `djangogirls` usando tu editor de texto y colocando el siguiente texto (y nada mas) dentro:

```
    python-3.5.2
```


## `mysite/local_settings.py`

Porque es mas restrictivo que PythonAnywhere, Heroku quiere usar diferentes configuraciones de las que nosotros usamos localmente (en nuestro computador). Heroku quiere que usar Postgres mientras que nosotros usamos SQLite, por ejemplo. Por eso necesitamos crear un archivo separado para las configuraciones que estarán disponibles en nuestro ambiente local.

Ve adelante y crea un archivo `mysite/local_settings.py`. Este debe contener tu configuración de `DATABASE`  de tu archivo `mysite/settings.py`. Justo como esto:

```python
import os
BASE_DIR = os.path.dirname(os.path.dirname(__file__))

DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.sqlite3',
        'NAME': os.path.join(BASE_DIR, 'db.sqlite3'),
    }
}

DEBUG = True
```

¡Luego guardalo! :)

## mysite/settings.py

Otra cosa que debemos hacer es modificar nuestro archivo de sitio web `settings.py`. Abre `mysite/settings.py` en tu editor y cambia las siguientes líneas:

```python
import dj_database_url

...

DEBUG = False

ALLOWED_HOSTS = ['127.0.0.1', '.herokuapp.com']

...

DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql_psycopg2',
        'NAME': 'djangogirls',
        'USER': 'name',
        'PASSWORD': '',
        'HOST': 'localhost',
        'PORT': '',
    }
}

...

db_from_env = dj_database_url.config(conn_max_age=500)
DATABASES['default'].update(db_from_env)
```

Esto hará la configuración necesaria para Heroku.

Luego guarda el archivo.

## mysite/wsgi.py

Abre el archivo `mysite/wsgi.py` y agrega estas líneas al final:

```python
from whitenoise.django import DjangoWhiteNoise
application = DjangoWhiteNoise(application)
```

¡Todo bien!

## Cuenta Heroku 

Necesitas instalar `Heroku toolbelt` el cual encontrarás aquí (puedes saltarte la instalación si ya lo instalaste durante la configuración): https://toolbelt.heroku.com/

> Cuando ejecutas la instalación de Heroku toolbelt en windows asegúrate de escoger "Instalación personalizada" cuando te pregunten que componentes instalar. En la lista de componentes que se muestra luego por favor selecciona "Git y SSH"

> En windows también debes ejecutar el siguiente comando para agregar Git y SSH a tu `PATH` de la línea de comanods: `setx PATH "%PATH%;C:\Program Files\Git\bin"`. Reinicia la línea de comandos después para habilitar el cambio.

> ¡Después de reiniciar tu línea de comandos, no olvides ir a tu carpeta `djangogirls` de nuevo y habilitar el ambiente virtual! (Truco: Ve al [capítulo de instalación de Django](http://tutorial.djangogirls.org/en/django_installation/index.html#working-with-virtualenv))

Por favor también crea una cuenta gratuita de Heroku aquí: https://id.heroku.com/signup/www-home-top

Luego autenticate con tu cuenta Heroku en tu computador ejecutando este comando:

```
    $ heroku login
```

En cas que no tengas una llave SSH este comando automáticamente creará una. Las llaves SSH son requeridas para colocar código en Heroku.

## Git commit

Heroku usa git para sus propios despliegues. A diferencia de PythonAnywhere, puedes hacer push a Heroku directamente, sin Github. Pero necesitamos hacer un par de cosas antes.

Abre un archivo llamado `.gitignore` en tu carpeta `djangogirls` y agrega `local_settings.py` a el. Queremos que git ignore `local_settings`, entonces este permanece en nuestro computador local y no termina en Heroku.
Open the file named `.gitignore` in your `djangogirls` directory and add `local_settings.py`

```
    *.pyc
    db.sqlite3
    myvenv
    __pycache__
    local_settings.py
```

Y guarda los cambios

```
    $ git status
    $ git add -A .
    $ git commit -m "additional files and changes for Heroku"
```

## Escoge un nombre de aplicación

Vamos a hacer que tu blog esté disponible en la web en `[tu nombre de blog].herokuapp.com` así vamos a escoger un nombre que nadie mas haya tomado. Este nombre no tiene que estar relacionado con el blog de `Django` o `mysite` o nada de lo que hemos creado hasta ahora. El nombre debe estar en minúscula (sin letras mayúsculas o acentos), nombres y guiones (`-`).

Una ves tengas un nombre (tal vez algo con tu nombre o nick en el), ejecuta este comando, reemplazaod `djangogirlsblo` con tu propio nombre de aplicación:

```
    $ heroku create djangogirlsblog
```

> __Nota__: Recuerda reemplazar `djangogirlsblog` con el nombre de tu aplicación en Heroku.

Si no puedes pensar en un nombre, puedes en lugar ejecutar:

```
    $ heroku create
```

Y heroku escogerá un nombre no disponible por ti (posiblemente algo como `enigmatic-cove-2527`).

Si tu alguna vez sientes que debes cambiar el nombre de tu aplicación Heroku, puedes hacerlo en cualquier momento con est eomando (reemplaza `the-new-game` con el nuevo nombre que quieres usar):

```
    $ heroku apps:rename the-new-name
```

> __Nota__: Recuerda que luego que cambias el nombre de tu aplicación, necesitas visitar `[the-new-name].herokuapp.com` para ver tu sitio

## ¡Despliega a Heroku!

Esto fue mucho de configuración e instalación, ¿cierto? ¡Pero esto solo lo necesitas hacer una vez! ¡Ahora puedes desplegar!

Cuando tu ejecutas `heroku create`, el automáticamente agregará un `remote` de Heroku a nuestro repositorio de aplicaciones. Ahora simplemente necesitamos hacer push a Heroku para desplegar nuestra aplicación:

```
    $ git push heroku master
```

> __Nota__: Esto posiblemente produzca *un montón* de salida si es la primera vez que lo ejecutas, como Heroku compila e instala psycopg. Verás que tu comando funcionó si ves algo como `https://yourapplicationname.herokuapp.com/ deployed to Heroku` cerca al final de output.

## Visita tu aplicación

Tu has desplegado tu código a Heroku, y especificado los tipos de proceso en un archivo `Procfile` (nosotros escogimos un proceso `web` anteriormente). Ahora puedes decirle  a Heroku que inicie este proceso `web`

Para hacerlo, ejecuta el siguiente comando:

```
    $ heroku ps:scale web=1
```

Esto le dice a Heroku que ejecute solamente una instancia de nuestro proceso `web`. Desde que nuestra aplicación es muy simple, nosotros no necesitamos mucho poder, y por eso está bien un solo proceso. Es posible solicitar a Heroku que ejecute mas procesos (por cierto, Heroku llama a estos procesos "Dynos" así que no te sorprendas si ves estos términos) pero ya no serán gratis.

Puedes visitar la app en tu navegador ejecutando `heroku open`.

```
    $ heroku open
```

> __Nota__: ¡verás una página de error! Hablaremos de eso en un minuto.

Esto abrirá una url como [https://djangogirlsblog.herokuapp.com/]() en tu navegador, y por el momento, posiblemente veas una página de error.

El error que ves es porque cuando desplegamos a Heroku, creamos una nueva base de datos y está vacía. Necesitamos ejecutar los comandos `migrate` y `createsuperuser`, justo como hicimos en PythonAnywhere. Esta vez, ellos vienen en una versión especial en nuestro computador, `heroku run`:

```
    $ heroku run python manage.py migrate

    $ heroku run python manage.py createsuperuser
```

Este comando te preguntará que escojas un nombre de usuario y contraseña de nuevo. Estas serán tus credenciales de acceso a la página de administración de tu sitio.

¡Refresca en tu navegador y ahí estás! Ahora sabes como desplegar a dos diferentes plataformas de hosting. Escoge tu favorita :)
