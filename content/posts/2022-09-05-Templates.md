---
typora-copy-images-to: ../assets/
typora-root-url: ../../
layout: post
slug: templates
conToc: true
title : Templates
render_with_liquid: false
---

Como se explica en el apartado anterior, los controladores son los responsables de manejar cada solicitud que entre en una aplicación Symfony y por lo general, terminan representando una plantilla para generar los contenidos de respuesta.

En realidad, el controlador delega la mayor parte del trabajo pesado a otros lugares, por lo que ese código puede ser probado y reutilizado. Cuando un controlador necesita generar HTML, CSS o cualquier otro contenido, entrega el trabajo al motor de plantillas.

En este manual, aprenderás cómo escribir plantillas poderosas que pueden ser utilizadas para devolver el contenido al usuario, rellenar cuerpos de correo electrónico y más. Aprenderás métodos abreviados, formas inteligentes de extender plantillas y cómo reutilizar código de plantillas.

## Plantillas

Una plantilla es simplemente un archivo de texto que puede generar cualquier formato basado en texto ()`HTML`, `XML`, `CSV`, `LaTeX` ...). El tipo de plantilla más familiar es una plantilla en `PHP`: un archivo de texto analizado por `PHP` que contiene una combinación de texto y código `PHP`:

```php
<?php
<!DOCTYPE html>
<html>
    <head>
        <title>Welcome to Symfony!</title>
    </head>
    <body>
        <h1><?php echo $page_title ?></h1>

        <ul id="navigation">
            <?php foreach ($navigation as $item): ?>
                <li>
                    <a href="<?php echo $item->getHref() ?>">
                        <?php echo $item->getCaption() ?>
                    </a>
                </li>
            <?php endforeach ?>
        </ul>
    </body>
</html>
```

Pero `Symfony` incluye un lenguaje de plantillas aún más poderoso llamado `Twig`. Twig te permite escribir plantillas concisas y legibles que son más amigables para los diseñadores web y, de varias maneras, más poderoso que las plantillas `PHP`:

```html
<!-- Plantilla -->
<!DOCTYPE html>
<html>
    <head>
        <title>Welcome to Symfony!</title>
    </head>
    <body>
        <h1>{{ page_title }}</h1>

        <ul id="navigation">
            {% for item in navigation %}
                <li><a href="{{ item.href }}">{{ item.caption }}</a></li>
            {% endfor %}
        </ul>
    </body>
</html>
```

Twig define tres tipos de sintaxis especial:

`{{...}}`

"Dice algo": imprime una variable o el resultado de una expresión en la plantilla.

`{% ...%}`

"Hace algo": un **tag** que controla la lógica de la plantilla; se usa para ejecutar sentencias como `for-loops` por ejemplo.

`{# ... #}`

"Comenta algo": es el equivalente de PHP a `/* comment */` . Se usa para agregar comentarios de una o varias líneas. El contenido de los comentarios no estará incluido en las páginas representadas.

Twig también contiene **filtros** que modifican el contenido antes de ser procesados. 

Lo siguiente hace que la variable de `title` sea mayúscula antes de la representación:

```php
<?php
{# TWIG #}
    {{ title|upper }}
```

Twig viene con una larga lista de [tags](https://twig.symfony.com/doc/3.x/tags/index.html), [filtros](https://twig.symfony.com/doc/3.x/templates.html#filters) y [funciones](https://twig.symfony.com/doc/3.x/functions/index.html) que están disponibles por defecto. Incluso puedes agregar tus propios filtros _personalizados_, funciones y más a través de [Twig Extensions](https://symfony.com/doc/6.4/templates.html#templates-twig-extension).

El código `Twig` es similar al código `PHP`, con diferencias sutiles y agradables. El siguiente  ejemplo usa una etiqueta estándar `for`  y la función de `cycle()` para imprimir diez etiquetas `div`, alternando clases `odd` y `even`:

```html
{# TWIG #}

{% for i in 1..10 %}
    <div class="{{ cycle(['even', 'odd'], i) }}">
      <!-- some HTML here -->
    </div>
{% endfor %}
```

A lo largo de este artículo, se mostrarán ejemplos de plantillas tanto en Twig como en PHP.

## ¿Por qué Twig?

Las plantillas Twig están diseñadas para ser simples y no procesarán etiquetas PHP. Esto es así por diseño: el sistema de plantillas Twig está destinado a expresar la presentación, no la lógica del programa. Cuanto más uses Twig, más apreciarás y te beneficiarás de esta distinción. Y por supuesto, serás amado por diseñadores web en todas partes.  
Twig también puede hacer cosas que PHP no puede, como el control de espacios en blanco, sandboxing, escapado automático de HTML, escape de salida contextual manual, y la inclusión de funciones personalizadas y filtros que solo afectan a las plantillas. Twig contiene pequeñas características que hacen que escribir plantillas sea más fácil y más conciso. Toma el siguiente ejemplo, que combina un bucle con una declaración _if_:

```html
{# TWIG #}
<ul>
    {% for user in users if user.active %}
        <li>{{ user.username }}</li>
    {% else %}
        <li>No users found</li>
    {% endfor %}
</ul>
```

## Twig Template Caching

Twig es rápido porque cada plantilla se compila en una clase PHP nativa y se almacena en caché. Pero no te preocupes: esto sucede automáticamente y no requiere que hagas nada.  
Y mientras estamos en desarrollo, Twig es lo suficientemente inteligente como para volver a compilar sus plantillas después de hacer cualquier cambio. Eso significa que Twig es rápido en producción, pero fácil de usar mientras estás desarrollando.

## Herencia de plantillas y layouts

La mayoría de las veces, las plantillas en un proyecto comparten elementos comunes, como el encabezado, pie de página, barra lateral o más. En Symfony, se piensa en este problema de manera diferente: una plantilla puede ser decorada por otra. Esto funciona exactamente lo mismo que las clases de PHP: la herencia de la plantilla te permite construir una plantilla base de "layout" que contiene todos los elementos comunes de tu sitio definidos como **blocks** (piensa en una "clase PHP con métodos base"\). Una plantilla hija puede extender el diseño base y sobreescribir cualquiera de sus bloques (piensa en "subclase PHP" que sobreescribe ciertos métodos de su clase principal").

Primero, crea un archivo de diseño base:

```html
{# templates/base.html.twig #}
<!DOCTYPE html>
<html>
    <head>
        <meta charset="UTF-8">
        <title>{% block title %}Test Application{% endblock %}</title>
    </head>
    <body>
        <div id="sidebar">
            {% block sidebar %}
                <ul>
                    <li><a href="/">Home</a></li>
                    <li><a href="/blog">Blog</a></li>
                </ul>
            {% endblock %}
        </div>

        <div id="content">
            {% block body %}{% endblock %}
        </div>
    </body>
</html>
```
