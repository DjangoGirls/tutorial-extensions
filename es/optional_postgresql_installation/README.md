# Opcional: Instalación de PostgreSQL

> Parte de este capítulo está basado en tutoriales de Geek Girls Carrots (http://django.carrots.pl/).

> Parte de este capítulo está basado en [django-marcador
tutorial](http://django-marcador.keimlink.de/) Licenciado bajo la licencia Creative Commons Attribution-ShareAlike 4.0. La licencia del tutorial le pertenece a Zapke-Gründemann.

## Windows

La manera mas fácil de instalar Postgres en Windows es usando un programa que puedes encontra aquí: http://www.enterprisedb.com/products-services-training/pgdownload#windows

Escoge la versión mas nueva disponible para tu sistema operativo. Descarga el instalador, ejecútalo y sigue las instrucciones disponibles aquí http://www.postgresqltutorial.com/install-postgresql/. Toma nota del directorio donde se instaló porque lo necesitarás en el siguiente paso (tipicamente, es `C:\Program Files\PostgreSQL\9.3).

## Mac OS X

La manera mas fácil es descargar gratis [Postgres.app](http://postgresapp.com/) e instalalo como cualquier otra aplicación en tu sistema operativo.

Descargalo, y arrástralo al directorio de Aplicaciones, y ejecutalo dando doble click en el. ¡Eso es todo!

Ahora debes agregar las herramientas de línea de comandos de Postgres a tu variable `PATH`, que es descrita [aquí](http://postgresapp.com/documentation/cli-tools.html).

## Linux

Los pasos de instalación varían de distribucióna  distribución. A continuación encontraremos comandos para Ubuntu y Fedora, pero si estás usando una distribución diferente [mira la documentación de PostgreSQL](https://wiki.postgresql.org/wiki/Detailed_installation_guides#General_Linux).

### Ubuntu

Ejecuta el siguiente comando:

```
    sudo apt-get install postgresql postgresql-contrib
```

### Fedora

Ejecuta el siguiente comando:

```
    sudo yum install postgresql93-server
```

# Crea la base de datos

Lo siguiente será crear nuestra primera base de datos, y un usuario que pueda acceder a ella. PostgreSQL nos deja crear tantas bases de datos como usuarios queramos, entonces si vamos a ejecutar mas de un sitio, deberíamos crear una base de datos para cada uno.

## Windows

Si estás usando Windows, aquí hay un par de pasos adicionales que necesitas completas. Por ahora no es importante si no entiendes la configuración que estamos haciendo aquí, pero sientente libre de preguntarle a tu Coach si estas curioso de lo que estás haciendo.

1. Abre la línea de comandos (Inicio → Todos los programas → Accesorios → Linea de comandos)
1. Ejecuta la siguiente configuración y presiona enter:`setx PATH "%PATH%;C:\Program Files\PostgreSQL\9.3\bin"`. Puedes pegar cosas a la línea de comandos dando click derecho y seleccionando `Pegar`. Asegúrate de que el la carpeta sea la misma que anotaste cuando estabas instalando con un `\bin` al final. Deberías ver el mensaje `SUCCESS: Specified value was saved.`.
1. Cierra y vuelve a abrie la línea de comando

## Crea la base de datos

Primero, vamos a iniciar la consola dePostgres  ejecutando `psql`. ¿Recuerdas como lanzar la consola?

> En Mac OS X debes hacer esto lanzando la aplicación `Terminal` (está en Aplicaciones → Utilidades). En linux, posiblemente esté bajo Aplicaciones → Accesorios → Terminal. En Windows necesitas ir a Inicio → Todos los programas → Accesorios → Línea de Comandos. Mas allá, en Windows, `psql` requiere que ingreses con un nombre de usuario y contraseña, el que escogiste durante la instalación. Si `psql` está pregunando por una contras´ña y no parece funcionar, trata `psql -U <username> -W` primero y luego presioa Enter.

```
    $ psql
    psql (9.3.4)
    Type "help" for help.
    #
```

Nuestro `$` ahora ha cambiado a `#`, lo cual significa que ahora estamos enviando comandos a PostgreSQL. Vamos a crear un usuario con `CREATE USER name;` (recuerda usar el punto y coma):

```
    # CREATE USER name;
```

Reemplaza `nombre` con tu propio nombre. No deberías usar letras acentadas o espacios en blanco. (ejemplo `bożena maria` es inválida - necesitas convertirla en `bozena_maria`). Si esto va bien, puedes tener una respuesta `CREATE ROLE` de la consola.  

Ahora es tiempo de crear una base de datos para tu proyecto en Django:

```
    # CREATE DATABASE djangogirls OWNER name;
```

Recuerda reemplazar `name` con el nombre que hayas escogido (ejemplo `bozena_maria`). Esto crea una base de datos vacía que puedes ahora usar en tus proyectos. Si esto va bien, puedes obtener una respuesta `CREATE DATABASE` de la consola.

¡Genial - las bases de dates están en orden!

# Actualizar la configuración

Encuentra esta parte en tu archivo `mysite/settings.py`:

```python
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.sqlite3',
        'NAME': os.path.join(BASE_DIR, 'db.sqlite3'),
    }
}
```

Y reemplazalo con esto:

```python
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql',
        'NAME': 'djangogirls',
        'USER': 'name',
        'PASSWORD': '',
        'HOST': 'localhost',
        'PORT': '',
    }
}
```

Recuerda cambiar `name` con el nombre de usuario que creaste antes en este capítulo.

# Instalando el paquete PostgreSQL para Python

Primero, instala `Heroku Toolbelt` desde https://toolbelt.heroku.com/ porque lo necesitaremos mas adelante para desplegar tu sitio, esto también include Git, lo cual puede ser util.

Lo siguiente es instalar un paquete que le permita a Python hablar con PostgreSQL - este es llamado `psycopg2`. Las intrucciones de instalación difieren un poco entre Windows y Linux/OS X.

## Windows

Para Windows, descarga el arhchivo pre construido de http://www.stickpeople.com/projects/python/win-psycopg/

Asegúrate de que tienes la versión de Python correspondiente (3.4 debería ser la última línea) y la arquitectura correcta (32 bit en la columna de la izquiera o 64 bit en la columna derecha).

Renombra el archivo descargado y muévelo para que estédisponible en `C:\psycopg2.exe`.

Una vez esté terminado, ejecuta el siguiente comando en la terminal (asegúrate que el `virtualenv` esté activado):

```
    easy_install C:\psycopg2.exe
```

## Linux y OS X

Ejecuta el siguiente código en la consola:

```
    (myvenv) ~/djangogirls$ pip install psycopg2
```

Si eso sale bien, verás algo así:

```
    Downloading/unpacking psycopg2
    Installing collected packages: psycopg2
    Successfully installed psycopg2
    Cleaning up...

---
```

Una vez esto esté completo, ejecuta `python -c "import psycopg2"`. Si no tienes ningún error, todo está instalado correctamente.

# Aplicando las migraciones y creando un super usuario

Para poder usar la base de datos recién creada en nuestro proyecto, necesitarás aplicar las migraciones en tu ambiente virtual ejecutando el siguiente código:

```
    (myvenv) ~/djangogirls$ python manage.py migrate
```

Para agregar nuevos post en nuestro blog, también necesitas crear un super usuario, ejecutando el siguiente código:

```
    (myvenv) ~/djangogirls$ python manage.py createsuperuser --username name 
``` 
Recuerda reemplazar `name` con el nombre de usuario. Se te preguntará por un email y una contraseña.

Ahora puedes ejecutar el servidor. Entra en la aplicación con la cuenta de super usuario y comienza a agregar posts a tu nueva base de datos.
