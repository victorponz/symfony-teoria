# Crear y usar plantillas

Como se explica en el apartado anterior, los controladores son los responsables de manejar cada solicitud que entre en una aplicación Symfony y por lo general, terminan representando una plantilla para generar los contenidos de respuesta.

En realidad, el controlador delega la mayor parte del trabajo pesado a otros lugares, por lo que ese código puede ser probado y reutilizado. Cuando un controlador necesita generar HTML, CSS o cualquier otro contenido, entrega el trabajo al motor de plantillas.

En este manual, aprenderás cómo escribir plantillas poderosas que pueden ser utilizadas para devolver el contenido al usuario, rellenar cuerpos de correo electrónico y más. Aprenderás métodos abreviados, formas inteligentes de extender plantillas y cómo reutilizar código de plantillas.

## Plantillas

Una plantilla es simplemente un archivo de texto que puede generar cualquier formato basado en texto \(`HTML`, `XML`, `CSV`, `LaTeX` ...\). El tipo de plantilla más familiar es una plantilla en `PHP`: un archivo de texto analizado por `PHP` que contiene una combinación de texto y código `PHP`:

```php
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
"Hace algo": una ** tag ** que controla la lógica de la plantilla; se usa para ejecutar sentencias como for-loops por ejemplo.

`{# ... #}`  
"Comenta algo": es el equivalente de PHP a `/* comment */` . Se usa para agregar comentarios de una o varias líneas. El contenido de los comentarios no estará incluido en las páginas representadas.

Twig también contiene ** filtros **, que modifican el contenido antes de ser procesados.   
Lo siguiente hace que la variable de `title` sea mayúscula antes de la representación:

```php
{# TWIG #}
    {{ title|upper }}
```

Twig viene con una larga lista de [tags](https://twig.symfony.com/doc/2.x/tags/index.html), [filtros](https://twig.symfony.com/doc/2.x/filters/index.html) y [funciones](https://twig.symfony.com/doc/2.x/functions/index.html) que están disponibles por defecto. Incluso puedes agregar tus propios filtros _personalizados_, funciones \(y más\) a través de [Twig Extensions](http://symfony.com/doc/current/templating/twig_extension.html).

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

La mayoría de las veces, las plantillas en un proyecto comparten elementos comunes, como el encabezado, pie de página, barra lateral o más. En Symfony, se piensa en este problema de manera diferente: una plantilla puede ser decorada por otra. Esto funciona exactamente lo mismo que las clases de PHP: la herencia de la plantilla te permite construir una plantilla base de "layout" que contiene todos los elementos comunes de tu sitio definidos como ** blocks ** \(piensa en una "clase PHP con métodos base"\). Una plantilla hija puede extender el diseño base y sobreescribir cualquiera de sus bloques \(piensa en "subclase PHP" que sobreescribe ciertos métodos de su clase principal"\).

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

Esta plantilla define el esqueleto del documento HTML base como una página de dos columnas. En este ejemplo, se definen tres áreas `{% block % }` \(`title`, `sidebar` y `body`\). Cada bloque puede ser sobreescrito por una plantilla secundaria o dejado con su implementación predeterminada. Esta plantilla también podría ser renderizada directamente. En ese caso, `title`, `sidebar` y `body` usarían los valores predeterminados utilizados en esta plantilla.

Una plantilla hija puede verse así:

```php
{# templates/blog/index.html.twig #}
{% extends 'base.html.twig' %}

{% block title %}My cool blog posts{% endblock %}

{% block body %}
    {% for entry in blog_entries %}
        <h2>{{ entry.title }}</h2>
        <p>{{ entry.body }}</p>
    {% endfor %}
{% endblock %}
```

La clave para la herencia de plantillas es la etiqueta `{% extends %}`. Esto dice al motor de plantillas que evalúe primero la plantilla base, que configura el diseño y define varios bloques. La plantilla hija se procesa, en ese punto se reemplazan  los bloques del padre por los de la plantilla hija.   
Dependiendo del valor de `blog_entries`,  la salida podría verse así:

```html
<!DOCTYPE html>
<html>
    <head>
        <meta charset="UTF-8">
        <title>My cool blog posts</title>
    </head>
    <body>
        <div id="sidebar">
            <ul>
                <li><a href="/">Home</a></li>
                <li><a href="/blog">Blog</a></li>
            </ul>
        </div>

        <div id="content">
            <h2>My first post</h2>
            <p>The body of the first post.</p>

            <h2>Another post</h2>
            <p>The body of the second post.</p>
        </div>
    </body>
</html>
```

Ten en cuenta que dado que la plantilla hija no define un bloque de barra lateral \(`sidebar`\), el valor de la plantilla primaria se usa en su lugar. El contenido dentro de un `{% block %}` de la plantilla principal siempre se usa de manera predeterminada.

> **Sugerencia**  
> ¡Puedes usar tantos niveles de herencia como quieras! Consulta el documento [How to organize your twig templates using inheritance](http://symfony.com/doc/current/templating/inheritance.html) para más información.

Cuando trabajes con la herencia de la plantilla, he aquí algunos consejos para tener en cuenta:

* Si usas _extends_ en una plantilla, debe ser la primera etiqueta en esa plantilla;

* Cuantas más etiquetas _block_ tengas en tus plantillas base, mejor. Recuerda, las plantillas secundarias no tienen que definir todos los bloques padre, así que crea tantos bloques en tus plantillas base como desees y da a cada uno un valor defecto. Cuantos más bloques tengan tus plantillas base, más flexible será el diseño.

* Si te encuentras duplicando contenido en una serie de plantillas, probablemente significa que debe mover ese contenido a un _block_ en una plantilla principal.  En algunos casos, una solución mejor puede ser mover el contenido a una nueva plantilla  e incluirlo \(ver [Including other templates](http://symfony.com/doc/current/templating.html#including-templates)\);

* Si necesitas obtener el contenido de un bloque de la plantilla principal, puedes usar la función `parent()`. Esto es útil si quieres agregar a los contenidos de un bloque padre en lugar de sobreescribirlo por completo:

```php
{% block sidebar %}
    <h3>Table of Contents</h3>

    {# ... #}

    {{ parent() }}
{% endblock %}
```

## Nombres y ubicaciones de plantillas

Por defecto, las plantillas pueden vivir en dos ubicaciones diferentes:

`templates/`  
El directorio de vistas de la aplicación puede contener plantillas base para toda la aplicación \(es decir, los diseños y las plantillas de su aplicación del paquete de aplicaciones\) como así como las plantillas que sobreescriben las plantillas de paquetes de terceros \(ver [How to override templates from third-party bundles](http://symfony.com/doc/current/templating/overriding.html)\).

`vendor/path/to/CoolBundle/Resources/views/`  
Cada paquete de terceros alberga sus plantillas en el directorio  `Resources/views/` \(y subdirectorios\). Cuando planees compartir tu paquete \(bundle\), deberías poner las plantillas en el paquete, en lugar del directorio `templates/`.

La mayoría de las plantillas que usarás estarán en el directorio `/templates`. La ruta que usarás será relativa a este directorio. Por ejemplo, para renderizar/extender `templates/base.html.twig`, usarás la ruta a `base.html.twig` y para renderizar/extender `templates/blog/index.html.twig`, usarás la ruta `blog/index.html.twig`.

## Sufijo de plantilla

Cada nombre de plantilla también tiene dos extensiones que especifican el _formato_ y _motor_ para esa plantilla.

| Nombre de archivo | Formato | Motor |
| :--- | :--- | :--- |
| `blog/index.html.twig` | HTML | Twig |
| `blog/index.html.php` | HTML | PHP |
| `blog/index.css.twig` | CSS | Twig |

Por defecto, cualquier plantilla de Symfony se puede escribir en Twig o PHP, y la última parte de la extensión \(por ejemplo, .twig o .php\) especifica cuál de estos dos _motores_ se debe utilizar. La primera parte de la extensión, \(por ejemplo, .`html`, .`css`, etc.\) es el formato final que la plantilla generará. A diferencia del motor, que determina cómo Symfony analiza la plantilla, esto es simplemente una táctica organizacional utilizada en caso de que el mismo recurso necesite representarse como HTML \(index.html.twig\), XML \(index.xml.twig\), o cualquier otro formato. Para obtener más información, lee el documento [How to work with different output formats in templates](http://symfony.com/doc/current/templating/formats.html)

## Etiquetas y ayudantes \(helpers\)

Ya entiendes los conceptos básicos de las plantillas, cómo se llaman y cómo usar la herencia de plantillas. Las partes más difíciles ya están superadas. En esta sección, aprenderás sobre un gran grupo de herramientas disponibles para ayudar a realizar las tareas de plantilla más comunes, como incluir otras plantillas, enlazar a páginas e incluir imágenes.

Symfony viene incluido con varias etiquetas Twig especializadas y funciones que facilitar el trabajo del diseñador de la plantilla.

Ya has visto algunas etiquetas Twig integradas \(`{% block %}` y `{% extends %}`\). 

Aquí aprenderás un poco mas.

## Incluyendo otras plantillas

A menudo querrás incluir la misma plantilla o fragmento de código en varias páginas. Por ejemplo, en una aplicación con "artículos de noticias", el código de la plantilla que muestra un artículo se puede usar en la página de detalles del artículo, en una página que muestra los artículos más populares, o en una lista de los últimos artículos.

Cuando necesitamos reutilizar un fragmento de código PHP, generalmente mueves el código a una nueva clase o función PHP. Lo mismo es cierto para las plantillas. Al mover el código reutilizado de plantilla a su propia plantilla, se puede incluir desde cualquier otra plantilla. Primera, crea la plantilla que necesitas volver a usar.

```html
{# templates/article/article_details.html.twig #}
<h2>{{ article.title }}</h2>
<h3 class="byline">by {{ article.authorName }}</h3>

<p>
    {{ article.body }}
</p>
```

Incluir esta plantilla desde cualquier otra plantilla es simple:

```php
{# templates/article/list.html.twig #}
{% extends 'layout.html.twig' %}

{% block body %}
    <h1>Recent Articles<h1>

    {% for article in articles %}
        {{ include('article/article_details.html.twig', { 'article': article }) }}
    {% endfor %}
{% endblock %}
```

La plantilla se incluye utilizando la función `{{ include() }}`. Ten en cuenta que el nombre de la plantilla sigue la misma convención típica. El `article_details.html.twig` de la plantilla usa una variable `article`, que le pasamos a ella. En este caso, podrías evitar hacer esto completamente, ya que todas las variables disponibles en `list.html.twig` también están disponibles en `article_details.html.twig` \(a menos que establezcas [with\_context](https://twig.symfony.com/doc/2.x/functions/include.html) a false\).  
Si quieres pasar más parámetros, se usa la misma sintaxis:

```php
{# templates/article/list.html.twig #}
{% extends 'layout.html.twig' %}

{% block body %}
    <h1>Recent Articles<h1>

    {% for article in articles %}
        {{ include('article/article_details.html.twig', { 'article': article, 'foo': foo, 'bar': bar }) }}
    {% endfor %}
{% endblock %}
```

## Enlace a páginas

Crear enlaces a otras páginas en tu aplicación es uno de los más comunes trabajos para una plantilla. En lugar de codificar \(_hardcodear_\) URLs en plantillas, usa la función Twig `path` \(o el helper `router` de PHP\) para generar URLs basadas en la configuración de enrutamiento. Más tarde, si deseas modificar la URL de una página particular, todo lo que tienes que hacer es cambiar la configuración de enrutamiento: las plantillas generarán automáticamente la nueva URL.

Primero, el enlace a la página "welcome", a la que se puede acceder mediante el siguiente enrutamiento:

```php
// src/Controller/WelcomeController.php

// ...
use Symfony\Component\Routing\Annotation\Route;

class WelcomeController extends Controller
{
    /**
     * @Route("/", name="welcome")
     */
    public function index()
    {
        // ...
    }
}
```

Para enlazar a esta página, simplemente use la función `path()` Twig y haz una referencia a la misma:

```html
{# TWIG #}

<a href="{{ path('welcome') }}"> Inicio </a>
```

Como se esperaba, esto generará la URL /. 

Ahora, para una ruta más complicada:

```php
// src/Controller/ArticleController.php

// ...
use Symfony\Component\Routing\Annotation\Route;

class ArticleController extends Controller
{
    /**
     * @Route("/article/{slug}", name="article_show")
     */
    public function show($slug)
    {
        // ...
    }
}
```
En este caso, debes especificar tanto el nombre de la ruta \(`article_show`\) como un valor para el parámetro `{slug}`. Usando esta ruta, revisa la plantilla `recent_list.html.twig` de la sección anterior y enlaza a los artículos correctamente:

```php
{# templates/article/recent_list.html.twig #}
{% for article in articles %}
    <a href="{{ path('article_show', {'slug': article.slug}) }}">
        {{ article.title }}
    </a>
{% endfor %}

```

# Enlace a los assets

Las plantillas también se refieren comúnmente a imágenes, JavaScript, hojas de estilo y otros archivos, a los que en el argot informático llamamos _assets_. Por supuesto, puedes codificar la ruta a estos recursos \(por ejemplo, `/images/logo.png`\), pero Symfony proporciona una opción más dinámica a través de la función `asset()` de Twig.

Para usar esta función, instala el paquete _asset_:

```bash
$ composer require asset
```

Ahora puedes usar la función `asset()`:

```html
{# TWIG #}
<img src="{{ asset('images/logo.png') }}" alt="Symfony!" />

<link href="{{ asset('css/blog.css') }}" rel="stylesheet" />
```

El objetivo principal de la función `asset()` es hacer que tu aplicación sea más portátil. Si tu aplicación reside en la raíz de su host \(por ejemplo, `http://example.com`\), entonces las rutas renderizadas deberían ser `/images/logo.png`. Pero si tu aplicación vive en un subdirectorio \(por ejemplo, `http://example.com/my_app`\), cada ruta de asset debe representarse con el subdirectorio \(por ejemplo, `/my_app/images/logo.png`\). La función `asset()` se ocupa de esto al determinar cómo se está usando tu aplicación y generando las rutas correctas en consecuencia.

Si necesitas URL absolutas para los assets, usa la función Twig `absolute_url()` como sigue:

```html
{# TWIG #}
<img src="{{ absolute_url(asset('images/logo.png')) }}" alt="Symfony!" />
```

## Incluyendo hojas de estilo y JavaScripts en Twig

Ningún sitio estaría completo sin incluir archivos JavaScript y hojas de estilo. En Symfony, la inclusión de estos assets se maneja elegantemente tomando ventaja de la herencia de plantilla de Symfony.

Comienza agregando dos bloques a tu plantilla base que contendrán tus assets: uno llamado `stylesheets` dentro de la etiqueta `head` y otro llamado `javascripts` justo encima de la etiqueta de cierre de `body`. Estos bloques contendrán todos las hojas de estilo y JavaScripts que necesitará en su sitio:

```html
{# templates/base.html.twig #}
<html>
    <head>
        {# ... #}

        {% block stylesheets %}
            <link href="{{ asset('css/main.css') }}" rel="stylesheet" />
        {% endblock %}
    </head>
    <body>
        {# ... #}

        {% block javascripts %}
            <script src="{{ asset('js/main.js') }}"></script>
        {% endblock %}
    </body>
</html>
```

¡Eso es bastante fácil! Pero, ¿qué sucede si necesitas incluir una hoja de estilo adicional o JavaScript desde una plantilla secundaria? Por ejemplo, supongamos que tienes una página de contacto y necesitas incluir una hoja de estilo `contact.css` _sólo_ en esa página. Desde dentro de la plantilla de esa página de contacto, haz lo siguiente:

```php
{# templates/contact/contact.html.twig #}
{% extends 'base.html.twig' %}

{% block stylesheets %}
    {{ parent() }}

    <link href="{{ asset('css/contact.css') }}" rel="stylesheet" />
{% endblock %}

{# ... #}
```

En la plantilla secundaria, simplemente sobreescribe el bloque de hojas de estilo y coloca tu nueva etiqueta de hoja de estilo dentro de ese bloque. Por supuesto, ya que quieres  agregar contenido al bloque principal \(y no _reemplazarlo_\), debes usar la función `parent()` de Twig para incluir todas las hojas de estilo del bloque de la plantilla base.

## Referencia a Request, User o Session

Symfony también te proporciona una variable `app` global en Twig que se puede usar para acceder al usuario actual, el objeto Request y más.

Ver [How to Access the User, Request, Session & more in Twig via the app Variable](http://symfony.com/doc/current/templating/app_variable.html) para más detalles.

## Escapando la salida

Twig realiza un "escape de salida" automático al procesar cualquier contenido para protegerlo de ataques Cross Site Scripting \(XSS\).

Supongamos que `description` es igual a `I <3 this product`:

```php
{# TWIG #}
<!-- output escaping is on automatically -->
{{ description }} <!-- I &lt;3 this product -->

<!-- disable output escaping with the raw filter -->
{{ description|raw }} <!-- I <3 this product -->
```

Para obtener más detalles, consulte [How to Escape Output in Templates](http://symfony.com/doc/current/templating/escaping.html).

## Pensamientos finales

El sistema de plantillas es solo _una_ de las muchas herramientas en Symfony. Y su trabajo es simple: nos permite renderizar resultados HTML dinámicos y complejos para que esto pueda finalmente ser devuelto al usuario, enviado por correo electrónico u otra cosa.

