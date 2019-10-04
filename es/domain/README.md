# Dominio

PythonAnywhere nos da un dominio gratuito, pero tal vez tu no quieres tener un ".pythonanywhere.com" al final de tu URL de blog. Quizas tu quieres tener ago como "www.infinite-kitten-pictures.org" o "www.3d-printed-steam-engine-parts.com" o "www.antique-buttons.com" o "www.mutant-unicornz.net", o lo que quieras.

Aquí vamos a hablar un poco sobre como obtener un dominio, y como enlazarlo con la aplicación en PythonAnywhere. Sin embargo, debes saber que los dominios cuestan dinero, y PythonAnywhere también te cobra mensualmente una tarifa para usar tu propio nombre de dominio -- no es mucho dinero en total, pero es posible que solo quieras si estas realmente comprometido.


## ¿Dónde registrar un dominio?

Un nombre de dominio cuesta típicamente $15 USD al año. Hay opciones más baratas y más caras, dependiendo del proveedor. Existen muchas compañías con las que puedes comprar un dominio: una simple [busqueda en google](https://www.google.com/search?q=register%20domain) te dará muchas opciones.

Nuestra favorita es [I want my name](https://iwantmyname.com/). Su anuncio dice "manejo de dominio sin dolor" y realmente es sin dolor.

También puedes obtener dominios gratis. [dot.tk](http://www.dot.tk) es un lugar para obtener uno, pero debes ser consciente que los dominios gratuitos a veces se ven muy baratos -- si tu sitio es para un negocio profesional, deberías pensar en pagar por un dominio "propio" que termine en `.com`.

## ¿Cómo apuntar tu dominio a PythonAnywhere?

Si realizaste el proceso a través de *iwantmyname.com*, haz click en `Domains` en el menú y escoge tu dominio recién comprado. Entonces localiza y haz click en `manage DNS records`:

![](images/4.png)

Ahora necesitas localizar este formulario:

![](images/5.png)

Y llenar la siguiente información

- Hostname: www
- Type: CNAME
- Value: your domain from PythonAnywhere (for example djangogirls.pythonanywhere.com)
- TTL: 60

![](images/6.png)

Haz click en el botón `Add` y Guarda los cambios al final.

> **Note** Si quieres un proveedor de dominio distinto, la interfaz para encontrar tu configuración de DNS/ CNAME será diferente, pero el objetivo es el mismo: Definir un CNAME que apunte tu nuevo dominio a `nombredeusuario.pythonanywhere.com`.

Es posible que tu dominio no comience a funcionar hasta pasados unos minutos. ¡Se paciente!

## Configura el dominio a través de la aplicación en PythonAnywhere.

También necesitas decirle a PythonAnywhere que quieres utilizar tu dominio personalizado.

Ve a la [página de cuenta de PythonAnywhere](https://www.pythonanywhere.com/account/) y mejora tu cuenta. La opción más barata (el plan "Hacker":) está bien para comenzar, y siempre puedes mejorarlo después cuando te vuelvas super famoso y tengas millones de visitas.

Después, ve a la [pestaña Web](https://www.pythonanywhere.com/web_app_setup/) y anota un par de cosas:

* Copia la **ruta a tu virtualenv** y colócala en un lugar seguro.
* Haz click a través de tu **archivo de configuración wsgi**, copia el contenido y pégalo en un lugar seguro.

Seguidamente, **elimina** tu vieja aplicación. No te preocupes, esto no elimina nada de tu código, solamente apaga el dominio en `nombredeusuario.pythonanywhere.com`. Después, crea una nueva aplicación y sigue estos pasos:

* Escrine el nombre de tu dominio
* Escoge "configuración manual"
* Selecciona Python 3.4
* Y estamos listos

Cuando seas enviado de vuelta a la pestaña web

* Pega la ruta del ambiente virtual que guardaste antes.
* Haz click en el archivo de configuración, y pega el contenido de tu archivo de configuración viejo.

Haz click en actualizar web app y ¡deberías encontrar tu sitio funcionando en tu nuevo dominio!

Si tienes problemas, haz click en "Send Feedback" en el sitio PythonAnywhere, y uno de sus amables administradores te ayudará a solucionarlo en poco tiempo.

