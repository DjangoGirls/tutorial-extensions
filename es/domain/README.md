# Dominio

PythonAnywhere nos da un dominio gratuito, pero tal vez tu no quieres tener un ".pythonanywhere.com" al final de tu URL de blog. Quizas tu quieres tener ago como "www.infinite-kitten-pictures.org" o "www.3d-printed-steam-engine-parts.com" o "www.antique-buttons.com" o "www.mutant-unicornz.net", o lo que quieras.

Aquí vamos a hablar un poco sobre como obtener un dominio, y como enlazarlo con la aplicación en PythonAnywhere. Sin embargo, debes saber que los dominios cuestan dinero, y PythonAnywhere también te cobra mensualmente una tarifa para usar tu propio nombre de dominio -- no es mucho dinero en total, pero esto es posiblemente que solamente quieras si estas realmente comprometido.


## ¿Dónde registrar un dominio?

Un nombre de dominio cuesta típicamente $15 USD al año. Hay opciones mas baratas y mas caras, dependiendo del probeedor. Existen muchas compañías con las que puedes comprar un dominio: una simple [busqueda en google](https://www.google.com/search?q=register%20domain) te dará muchas opciones.

Nuestra favorita es [I want my name](https://iwantmyname.com/). Su anuncion dice "manejo de dominio sin dolor" y realmente es sin dolor.

También puedes obtener dominios gratis. [dot.tk](http://www.dot.tk) es un lugar para obtener uno, pero debes estar conciente que los dominios gratuitos a veces se sienten baratos -- si tu sitio quiere ser para un negocio profesional, deberías pensar sobre pagar por un dominio "propio" que termine en `.com`.

## ¿Cómo apuntar tu dominio a PythonAnywhere?

Si fuiste a través de *iwantmyname.com*, da click en `Domains` en el menú y escoge tu dominio recién comprado. Entonces localiza y da click en `manage DNS records`:

![](images/4.png)

Ahora necesitas localizar este formulario:

![](images/5.png)

Y llena la siguiente información

- Hostname: www
- Type: CNAME
- Value: your domain from PythonAnywhere (for example djangogirls.pythonanywhere.com)
- TTL: 60

![](images/6.png)

Click en el botón `Add` y Guarda los cambios al final.

> **Note** Si quieres un proveedor de dominio distinto, la interfaz para encontrar tu configuración de DNS/ CNAME será diferente, pero el objetivo es el mismo: Definir un CNAME que apunte tu nuevo dominio a `nombredeusuario.pythonanywhere.com`.

También puede tomar algunos minutos para que tu dominio comience a funcionar. ¡Se paciente!

## Configura el dominio a través de la aplicación en PythonAnywhere.

También necesitas decirle a PythonAnywhere que quieres utilizar tu dominio personalizado.

Ve a la [página de cuenta de PythonAnywhere](https://www.pythonanywhere.com/account/) y mejora tu cuenta. La opcieon mas barata (el plan "Hacker:) está bien para comenzar, y siempre puedes mejorarlo después cuando te vuelvas super famoso y tengas millones de visitas.

Siguiente, ve sobre la [pestaña Web](https://www.pythonanywhere.com/web_app_setup/) y anota un par de cosas:

* Copia la **ruta a tu virtualenv** y colocala en un lugar seguro.
* Da click a través de tu **archivo de configuración wsgi**, copia el contenido y pegalo en un lugar seguro.

Siguiente, **elimina** tu vieja aplicación. No te preocupes, esto no elimina nada de tu código, solamente apaga el dominio en `nombredeusuario.pythonanywhere.com`. Siguiente, crea una nueva aplicación y sigue estos pasos:

* Ingresa tu nombre de dominio
* Escoge "configuración manual"
* Selecciona Python 3.4
* Y estamos listos

Cuando seas enviado de vuelta a la pestaña web

* Pega la ruta del ambiente virtual que guardaste antes.
* Da click através del archivo de configuración, y pega el contenido de tu archivo de configuración viejo.

Da click en actualizar web app y ¡deberías encontrar  tu sitio corriendo en tu nuevo dominio!

Si tienes problemas, da click en "Send Feedback" en el sitio PythonAnywhere, y uno de sus amables administradores te ayudará a solucionarlo en poco tiempo.

