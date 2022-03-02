---
typora-copy-images-to: assets
---


# Forms

Tratar con formularios HTML es una de las tareas más comunes y desafiantes para un desarrollador web. Symfony integra un componente de formulario que facilita el manejo de formularios. En este artículo, crearás un formulario complejo desde cero, aprendiendo las características más importantes de la biblioteca de formularios a lo largo del camino.

Instalación
------------
Ejecuta este comando para instalar la función de formulario antes de usarlo:

```bash
$ composer require form
```

> **Nota**
> El componente Symfony Form es una biblioteca independiente que se puede usar fuera de los proyectos de Symfony. Para obtener más información, consulta [Form component documentation](https://symfony.com/doc/current/components/form.html) en GitHub.

## Crear un formulario simple

Supongamos que estás creando una aplicación de lista de tareas pendientes que deberá mostrar "tareas". Como tus usuarios necesitarán editar y crear tareas, necesitarás crear un formulario. Pero antes de comenzar, primero concéntrate en la clase de `Task` genérica que representa y almacena los datos para una única tarea:
[Código Fuente](./assets/Task.php.txt)

```php
// src/Entity/Task.php
namespace App\Entity;

class Task
{
  protected $task;
  protected $dueDate;

  public function getTask()
  {
    return $this->task;
  }

  public function setTask($task)
  {
    $this->task = $task;
  }

  public function getDueDate()
  {
    return $this->dueDate;
  }

  public function setDueDate(\DateTime $dueDate = null)
  {
    $this->dueDate = $dueDate;
  }
}
```

Esta clase es un "objeto simple de PHP" porque, hasta el momento, no tiene nada que ver con Symfony ni con ninguna otra biblioteca. Es simplemente un objeto PHP normal que resuelve directamente un problema dentro de tu aplicación (es decir, la necesidad de representar una tarea en su aplicación). Por supuesto, al final de este artículo, podrás enviar datos a una instancia `Task` (a través de un formulario HTML), validar sus datos y conservarlos en la base de datos.


## Crear el Form
Ahora que has creado una clase `Task`, el siguiente paso es crear y representar el formulario HTML real. En Symfony, esto se hace construyendo un objeto de formulario y luego representándolo en una plantilla. Por ahora, todo esto se puede hacer desde dentro de un controlador:

[Código Fuente](./assets/DefaultController.php.txt)

```php
// src/Controller/DefaultController.php
namespace App\Controller;

use App\Entity\Task;
use Symfony\Bundle\FrameworkBundle\Controller\Controller;
use Symfony\Component\HttpFoundation\Request;
use Symfony\Component\Form\Extension\Core\Type\TextType;
use Symfony\Component\Form\Extension\Core\Type\DateType;
use Symfony\Component\Form\Extension\Core\Type\SubmitType;

class DefaultController extends Controller
{
  public function new(Request $request)
  {
    // creates a task and gives it some dummy data for this example
    $task = new Task();
    $task->setTask('Write a blog post');
    $task->setDueDate(new \DateTime('tomorrow'));

    $form = $this->createFormBuilder($task)
      ->add('task', TextType::class)
      ->add('dueDate', DateType::class)
      ->add('save', SubmitType::class, array('label' => 'Create Task'))
      ->getForm();

    return $this->render('default/new.html.twig', array(
      'form' => $form->createView(),
    ));
  }
}
```
> **Consejo**
> Este ejemplo muestra cómo construir su formulario directamente en el controlador. Más  adelante, en la sección [Crear Form Classes](#crear-form-classes) , aprenderás cómo crear tu formulario en una clase independiente, que se recomienda para que el formulario se vuelva más reutilizable.

Crear un formulario requiere relativamente poco código porque los objetos de formulario de Symfony se crean con un "generador de formularios". El propósito del creador de formularios es permitirle escribir "recetas" de forma simple y hacer que todo el trabajo pesado de la creación del formulario sea realmente fácil.

En este ejemplo, se ha agregado dos campos al formulario - task y `dueDate` - que corresponden a las propiedades `task` y `dueDate` de la clase Task. También se asigna a cada uno un "tipo" (por ejemplo,  `TextType` y `DateType`), representado por su nombre de clase completamente calificado (*fully qualified class name*). Entre otras cosas, determina qué etiqueta (s) de formulario HTML se representa para ese campo.

Finalmente, agregamos un botón de enviar con una etiqueta personalizada para enviar el formulario al servidor.

Symfony viene con muchos tipos incorporados que se detallan más adelante (ver [Tipos de campos integrados](#Tipos-de-campos-integrados)).

## Renderizar el Form
Ahora que se ha creado el formulario, el siguiente paso es renderizarlo. Esto se hace pasando un objeto formulario especial "view" a tu plantilla (observa el `$form->createView()` en el controlador de arriba) y usando un conjunto de funciones de ayuda de formulario:

Con `twig`:

```twig
{# templates/default/new.html.twig #}
{{ form_start(form) }}
{{ form_widget(form) }}
{{ form_end(form) }}
```
Con `php`
```php+html
<!-- templates/default/new.html.php -->
<?php echo $view['form']->start($form) ?>
<?php echo $view['form']->widget($form) ?>
<?php echo $view['form']->end($form) ?>
```
![_images/simple-form.png](https://symfony.com/doc/current/_images/simple-form.png)


> **Nota**
> Este ejemplo asume que se envía el formulario en una solicitud "POST" y en la 
> misma URL en la que se mostró. Más adelante aprenderás cómo cambiar el 
> método de solicitud y la URL de destino del formulario.

¡Eso es todo! Solo se necesitan tres líneas para representar el formulario completo:

* `form_start(form)` Renderiza la etiqueta de inicio del formulario, incluido el atributo de tipo correcto al usar cargas de archivos (*file uploads*).

* `form_widget(form)`  Renderiza todos los campos, que incluyen el elemento de campo en sí, una etiqueta y cualquier mensaje de error de validación para el campo.

* `form_end(form)` Renderiza la etiqueta de cierre del formulario y cualquier campo que aún no se haya procesado, en caso de que represente cada campo por ti mismo. Esto es útil para renderizar campos ocultos y aprovechar la función automática [CSRF Protection](https://symfony.com/doc/current/security/csrf.html)

Tan fácil como esto, no es muy flexible (todavía). Generalmente, querrás entregar cada campo de formulario individualmente para que pueda controlar cómo se ve el formulario. Aprenderás a hacer esto en [Cómo controlar el renderizado de un Form](https://symfony.com/doc/current/form/rendering.html).

Antes de continuar, observa cómo el campo de input `task`  tiene el valor de la propiedad `task` del objeto `$task` (es decir, "Write a blog post"). Este es el primer trabajo de un formulario: tomar datos de un objeto y traducirlo a un formato que sea adecuado para representarlo en un formulario HTML.

> **Consejo**
> El sistema de formularios es lo suficientemente inteligente como para acceder al valor de la propiedad protegida `Task` a través de los métodos `getTask()` y `setTask()` en la clase `Task`. A menos que una propiedad sea pública, debe tener un método "getter" y "setter" para que el componente Form pueda obtener y poner datos en la propiedad. Para una propiedad booleana, puedes usar un método "isser" o "hasser" (por ejemplo, `isPublished()` o `hasReminder()`) en lugar de un getter (por ejemplo, `getPublished()` o `getReminder()`).

## Manejo de Form Submissions
De forma predeterminada, el formulario enviará una solicitud POST al mismo controlador que la procesa. Aquí, el segundo trabajo de un formulario es traducir los datos enviados por el usuario a las propiedades de un objeto. Para que esto suceda, los datos enviados por el usuario deben escribirse en el objeto Form. Agrega la siguiente funcionalidad al controlador:

[Código Fuente](./assets/DefaultController2.php.txt)

```php
// ...
use Symfony\Component\HttpFoundation\Request;

public function new(Request $request)
{
  // just setup a fresh $task object (remove the dummy data)
  $task = new Task();

  $form = $this->createFormBuilder($task)
    ->add('task', TextType::class)
    ->add('dueDate', DateType::class)
    ->add('save', SubmitType::class, array('label' => 'Create Task'))
    ->getForm();

  $form->handleRequest($request);

  if ($form->isSubmitted() && $form->isValid()) {
    // $form->getData() holds the submitted values
    // but, the original `$task` variable has also been updated
    $task = $form->getData();

    // ... perform some action, such as saving the task to the database
    // for example, if Task is a Doctrine entity, save it!
    // $em = $this->getDoctrine()->getManager();
    // $em->persist($task);
    // $em->flush();

    return $this->redirectToRoute('task_success');
  }

  return $this->render('default/new.html.twig', array(
    'form' => $form->createView(),
  ));
}
```
> **Precaución**
> Ten en cuenta que se debe llamar al método `createView()` después de llamar a `handleRequest()`. De lo contrario, los cambios realizados en los eventos `*_SUBMIT` no se aplican a la vista (igual que los errores de validación).

Este controlador sigue un patrón común para el manejo de formularios y tiene tres *caminos* posibles:

* Cuando inicialmente carga la página en un navegador, el formulario se crea y se procesa. [handleRequest()](http://api.symfony.com/4.0/Symfony/Component/Form/FormInterface.html#method_handleRequest) reconoce que el formulario no se envió y no hace nada. [isSubmitted()](http://api.symfony.com/4.0/Symfony/Component/Form/FormInterface.html#method_isSubmitted) devuelve falso si el formulario no se envió.
* Cuando el usuario envía el formulario,  [handleRequest()](http://api.symfony.com/4.0/Symfony/Component/Form/FormInterface.html#method_handleRequest) lo reconoce e inmediatamente vuelve a escribir los datos enviados en `task` y la propiedad `dueDate` del objeto `$task`. Entonces este objeto es validado. Si no es válido (la validación se trata en la siguiente sección), [isValid()](http://api.symfony.com/4.0/Symfony/Component/Form/FormInterface.html#method_isValid) devuelve falso y el formulario se procesa nuevamente, pero ahora con errores de validación.
* Cuando el usuario envía el formulario con datos válidos, los datos enviados se 
  vuelven a escribir en el formulario, pero esta vez [isValid()](http://api.symfony.com/4.0/Symfony/Component/Form/FormInterface.html#method_isValid) devuelve true. Ahora  tienes la oportunidad de realizar algunas acciones utilizando el objeto `$task` (por ejemplo, persistir en la base de datos) antes de redirigir al usuario a alguna otra página (por ejemplo, una página de "agradecimiento" o "éxito").

 > **Nota**
 > Redirigir a un usuario después de enviar correctamente el formulario evita que el usuario pueda presionar el botón "Actualizar" de su navegador y volver a publicar los datos. Si se realiza por ajax, no hace falta redirigir ya que no se guarda en el historial del navegador.

Si necesitas más control sobre cuando se envía su formulario o qué datos se le pasan, puedes usar el método [submit()](http://api.symfony.com/4.0/Symfony/Component/Form/FormInterface.html#method_submit). Más información al respecto en [Calling Form::submit() manually](https://symfony.com/doc/current/form/direct_submit.html#form-call-submit-directly).

## Validación del Form
En la sección anterior, aprendiste cómo se puede enviar un formulario con datos válidos o no válidos. En Symfony, la validación se aplica al objeto subyacente (por ejemplo, `Task`). En otras palabras, la pregunta no es si el "formulario" es válido, sino si el objeto `$task` es o no válido después de que el formulario le haya aplicado los datos enviados. Llamar `$form->isValid()` es un atajo que le pide al objeto `$task` si tiene o no datos válidos.

Antes de usar la validación, agrega soporte para ella en tu aplicación:

```bash
$ composer require validator
```
La validación se realiza agregando un conjunto de reglas (llamadas restricciones) a una clase. Para ver esto en acción, agrega restricciones de validación para que el campo `task` no pueda estar vacío y el campo `dueDate` no puede estar vacío y debe ser un objeto válido `\DateTime`.

[Código Fuente](./assets/Task2.php.txt)

```php
// src/Entity/Task.php
namespace App\Entity;

use Symfony\Component\Validator\Constraints as Assert;

class Task
{
  /**
   * @Assert\NotBlank()
  */
  public $task;

  /**
   * @Assert\NotBlank()
   * @Assert\Type("\DateTime")
  */
  protected $dueDate;
}
```
En `yaml`
```yaml
# config/validation.yaml
App\Entity\Task:
	properties:
		task:
        	- NotBlank: ~
		dueDate:
			- NotBlank: ~
			- Type: \DateTime
```

¡Eso es todo! Si vuelves a enviar el formulario con datos no válidos, verás los errores correspondientes impresos con el formulario.

La validación es una característica muy poderosa de Symfony y tiene su propio [artículo dedicado](https://symfony.com/doc/current/validation.html).

> **Validación HTML5**
> Gracias a HTML5, muchos navegadores pueden aplicar de forma nativa ciertas restricciones de validación en el lado del cliente. La validación más común se activa al presentar un atributo `required` en los campos que se requieren. Para los navegadores compatibles con HTML5, esto dará como resultado que se muestre un mensaje del navegador nativo si el usuario intenta enviar el formulario con ese campo en blanco.
>
> Los formularios generados aprovechan al máximo esta nueva característica al agregar atributos HTML sensibles que activan la validación. Sin embargo, la validación del lado del cliente se puede desactivar agregando el atributo `novalidate` a la etiqueta `form` o `formnovalidate` a la etiqueta submit. Esto es especialmente útil cuando quieres probar las restricciones de validación del lado del servidor, pero su navegador lo impide, por ejemplo, al enviar campos en blanco.
>
> Con `twig`:
>
> ```twig
> {# templates/default/new.html.twig #}
> {{ form(form, {'attr': {'novalidate': 'novalidate'}}) }}
> ```
> Con `php`:
> ```php+html
> <!-- templates/default/new.html.php -->
> <?php echo $view['form']->form($form, array(
>   'attr' => array('novalidate' => 'novalidate'),
> )) ?>
> ```


## Tipos de campos integrados

Symfony viene de serie con un gran grupo de tipos de campo que cubren todos los campos de formularios comunes y tipos de datos que encontrarás:

**Campos de texto**
* [TextType](https://symfony.com/doc/current/reference/forms/types/text.html)
* [TextareaType](https://symfony.com/doc/current/reference/forms/types/textarea.html)
* [EmailType](https://symfony.com/doc/current/reference/forms/types/email.html)
* [IntegerType](https://symfony.com/doc/current/reference/forms/types/integer.html)
* [MoneyType](https://symfony.com/doc/current/reference/forms/types/money.html)
* [NumberType](https://symfony.com/doc/current/reference/forms/types/number.html)
* [PasswordType](https://symfony.com/doc/current/reference/forms/types/password.html)
* [PercentType](https://symfony.com/doc/current/reference/forms/types/percent.html)
* [SearchType](https://symfony.com/doc/current/reference/forms/types/search.html)
* [UrlType](https://symfony.com/doc/current/reference/forms/types/url.html)
* [RangeType](https://symfony.com/doc/current/reference/forms/types/range.html)
* [TelType](https://symfony.com/doc/current/reference/forms/types/tel.html)
* [ColorType](https://symfony.com/doc/current/reference/forms/types/color.html)

**Campos de elección**
* [ChoiceType](https://symfony.com/doc/current/reference/forms/types/choice.html)
* [EntityType](https://symfony.com/doc/current/reference/forms/types/entity.html)
* [CountryType](https://symfony.com/doc/current/reference/forms/types/country.html)
* [LanguageType](https://symfony.com/doc/current/reference/forms/types/language.html)
* [LocaleType](https://symfony.com/doc/current/reference/forms/types/locale.html)
* [TimezoneType](https://symfony.com/doc/current/reference/forms/types/timezone.html)
* [CurrencyType](https://symfony.com/doc/current/reference/forms/types/currency.html)

**Campos de fecha y hora**
* [DateType](https://symfony.com/doc/current/reference/forms/types/date.html)
* [DateIntervalType](https://symfony.com/doc/current/reference/forms/types/dateinterval.html)
* [DateTimeType](https://symfony.com/doc/current/reference/forms/types/datetime.html)
* [TimeType](https://symfony.com/doc/current/reference/forms/types/time.html)
* [BirthdayType](https://symfony.com/doc/current/reference/forms/types/birthday.html)

**Otros campos**
* [CheckboxType](https://symfony.com/doc/current/reference/forms/types/checkbox.html)
* [FileType](https://symfony.com/doc/current/reference/forms/types/file.html)
* [RadioType](https://symfony.com/doc/current/reference/forms/types/radio.html)

**Campos agrupados**
* [CollectionType](https://symfony.com/doc/current/reference/forms/types/collection.html)
* [RepeatedType](https://symfony.com/doc/current/reference/forms/types/repeated.html)

**Campos ocultos**
* [HiddenType](https://symfony.com/doc/current/reference/forms/types/hidden.html)

**Botones**
* [ButtonType](https://symfony.com/doc/current/reference/forms/types/button.html)
* [ResetType](https://symfony.com/doc/current/reference/forms/types/reset.html)
* [SubmitType](https://symfony.com/doc/current/reference/forms/types/submit.html)

**Campos base**
* [FormType](https://symfony.com/doc/current/reference/forms/types/form.html)

También puedes crear sus propios tipos de campos personalizados. Consulta [How to create a Custom Form Field Type](https://symfony.com/doc/current/form/create_custom_field_type.html) para obtener más información.

## Opciones en los tipos de campo
Cada tipo de campo tiene varias opciones que se pueden usar para configurarlo. Por ejemplo, el campo [dueDate]() se está representando actualmente como 3 cuadros de selección. Sin embargo, el tipo [DateType](https://symfony.com/doc/current/reference/forms/types/date.html) se puede configurar para que se represente como un cuadro de texto único (donde el usuario ingresaría la fecha como una cadena en el cuadro):

```php
->add('dueDate', DateType::class, array('widget' => 'single_text'))
```

![_images/simple-form-2.png](https://symfony.com/doc/current/_images/simple-form-2.png)

Cada tipo de campo tiene una cantidad de opciones diferentes que se le pueden pasar. Muchos de estos son específicos para el tipo de campo y los detalles se pueden encontrar en la documentación para cada tipo.

> **La opción `required`** 
>
> La opción más común es la opción `required`, que se puede aplicar a cualquier  campo. De forma predeterminada, la opción `required` se establece en `true`, lo que significa que los navegadores compatibles con HTML5 aplicarán la validación del lado del cliente si el campo se deja en blanco. Si no deseas estos comportamientos, deshabilita la validación HTML5 o establece la opción `required` en tu campo a [false]():
>
> ```php
> ->add('dueDate', DateType::class, array(
>   'widget' => 'single_text',
>   'required' => false
> ))
> ```
>
> También ten en cuenta que establecer la opción  `required` en `true` **no** dará lugar a que se aplique la validación del lado del servidor. En otras palabras, si un usuario envía un valor en blanco para el campo (ya sea con un navegador o servicio web antiguo, por ejemplo), se aceptará como un valor válido a menos que use la restricción de validación `NotBlank` o `NotNull` de Symfony.
>
> En otras palabras, la opción `required` es "buena", pero siempre se debe usar la validación verdadera del lado del servidor.

> **La opción `label`** 
>
> La etiqueta para el campo de formulario se puede establecer usando la opción `label`, que se puede aplicar a cualquier campo:
>
> ```php
> ->add('dueDate', DateType::class, array(
>   'widget' => 'single_text',
>   'label'  => 'Due Date',
> ))
> ```
> La etiqueta de un campo también se puede establecer en la plantilla que representa el formulario, ver a continuación. Si no necesitas una etiqueta asociada a tu entrada, puedes desactivarla estableciendo su valor en `false`.

## Adivinar el tipo de campo
Ahora que has agregado metadatos de validación a la clase `Task`, Symfony ya sabe un poco sobre tus campos. Si lo permites, Symfony puede "adivinar" el tipo de tu campo y configurarlo por ti. En este ejemplo, Symfony puede adivinar a partir de las reglas de validación que tanto el campo `task` es un campo de tipo `TextType` normal y que el campo `dueDate` es un campo de tipo `DateType`:

[Código Fuente](./assets/Task3.php.txt)

```php
public function new()
{
  $task = new Task();

  $form = $this->createFormBuilder($task)
    ->add('task')
    ->add('dueDate', null, array('widget' => 'single_text'))
    ->add('save', SubmitType::class)
    ->getForm();
}
```
La "adivinación" se activa cuando omites el segundo argumento para el método `add()` (o si le pasas `null`). Si pasas un `array` de opciones como tercer argumento (hecho para `dueDate` ), estas opciones se aplican al campo adivinado.

### Adivinar opciones para los campos

Además de adivinar el "tipo" de un campo, Symfony también puede intentar adivinar los valores correctos de varias opciones del mismo.

> **Consejo**
>
> Cuando se configuran estas opciones, el campo se representará con atributos HTML especiales que proporcionan validación del lado del cliente HTML5. Sin embargo, no genera las restricciones equivalentes del lado del servidor (por ejemplo, `Assert\Length`). Y aunque deberás agregar manualmente su validación del lado del servidor, estas opciones de tipo de campo se pueden deducir a partir de esa información.

* `required`
  La opción `required` se puede adivinar en función de las reglas de validación (es decir, es el campo `NotBlank` o `NotNull`?) o los metadatos de Doctrine (es decir, es el campo `nullable`?). Esto es muy útil, ya que la validación del lado del cliente coincidirá automáticamente con tus reglas de validación.
* `maxlength`
  Si el campo es un tipo de campo de texto, entonces el atributo de opción `maxlength` se puede adivinar a partir de las restricciones de validación (si se usa `Length` o `Range`) o desde los metadatos de Doctrine (a través de la longitud del campo).

> **Precaución**
> Estas opciones de campo solo se adivinan si estás utilizando Symfony para adivinar el tipo de campo (es decir, si omites o pasas `null` como el segundo argumento para `add()`).

Si deseas cambiar uno de los valores adivinados, puedes anularlo pasando la opción en el array de opciones de campo:

```php
->add('task', null, array('attr' => array('maxlength' => 4)))
```
## Crear Form Classes

Como has visto, se puede crear un formulario y usarlo directamente en un controlador. Sin embargo, una mejor práctica es construir el formulario en una clase separada e independiente de PHP, que luego se puede reutilizar en cualquier parte de la aplicación. Crea una nueva clase que contendrá la lógica para crear el formulario de tareas:

[Código Fuente](./assets/TaskType.php.txt)

```php
// src/Form/TaskType.php
namespace App\Form;

use Symfony\Component\Form\AbstractType;
use Symfony\Component\Form\FormBuilderInterface;
use Symfony\Component\Form\Extension\Core\Type\SubmitType;

class TaskType extends AbstractType
{
  public function buildForm(FormBuilderInterface $builder, array $options)
  {
    $builder
      ->add('task')
      ->add('dueDate', null, array('widget' => 'single_text'))
      ->add('save', SubmitType::class)
      ;
  }
}
```

Esta nueva clase contiene todas las instrucciones necesarias para crear el formulario de tareas. Se puede usar para crear rápidamente un objeto de formulario en el controlador:

[Código Fuente](./assets/DefaultController3.php.txt)

```php
// src/Controller/DefaultController.php
use App\Form\TaskType;

public function new()
{
  $task = ...;
  $form = $this->createForm(TaskType::class, $task);

  // ...
}
```
Colocar la lógica de formulario en su propia clase significa que el formulario se puede reutilizar fácilmente en otro lugar de tu proyecto. Esta es la mejor manera de crear formularios, pero la decisión depende de ti.

**Configurar ``data_class``**

Cada formulario necesita saber el nombre de la clase que contiene los datos subyacentes (por ejemplo, `App\Entity\Task`). Por lo general, esto se adivina basándose en el objeto pasado al segundo argumento a `createForm()` (es decir, `$task`). Más tarde, cuando comiences a incrustar formularios en otras partes de tu aplicación, esto ya no será suficiente. Por lo tanto, aunque no siempre es necesario, generalmente es una buena idea especificar explícitamente la opción `data_class` agregando lo siguiente a su clase de tipo de formulario:

[Código Fuente](./assets/TaskType2.php.txt)

```php
// src/Form/TaskType.php
use App\Entity\Task;
use Symfony\Component\OptionsResolver\OptionsResolver;

// ...
public function configureOptions(OptionsResolver $resolver)
{
  $resolver->setDefaults(array(
    'data_class' => Task::class,
  ));
}
```
> **Consejo**
> Al asignar (*mapping*) formularios a objetos, se asignan todos los campos. Cualquier campo en el formulario que no exista en el objeto mapeado causará una excepción. En los casos en que necesites campos adicionales en el formulario (por ejemplo, una casilla de verificación "¿Está de acuerdo con estos términos?") que no se asignarán al objeto subyacente, debes establecer la opción ` mapped` en `false`:
>
> [Código Fuente](./assets/TaskType3.php.txt)
>
> ```php
> use Symfony\Component\Form\FormBuilderInterface;
> public function buildForm(FormBuilderInterface $builder, array $options)
> {
> $builder
> 
>  ->add('task')
>  ->add('dueDate')
>  ->add('agreeTerms', CheckboxType::class, array('mapped' => false))
>  ->add('save', SubmitType::class)
>  ;
> }
> ```
>

Además, si hay campos en el formulario que no están incluidos en los datos enviados, esos campos se establecerán explícitamente como `null`.

Se puede acceder a los datos del campo en un controlador con:

```php
$form->get('agreeTerms')->getData();
```
Además, los datos de un campo no mapeado también se pueden modificar directamente:

```php
$form->get('agreeTerms')->setData(true);
```

## Pensamientos finales
Al crear formularios, ten en cuenta que el primer objetivo del mismo es traducir datos de un objeto (`Task`) a un formulario HTML para que el usuario pueda modificar esos datos. El segundo objetivo de un formulario es tomar los datos enviados por el usuario y volver a aplicarlos al objeto.

Hay mucho más por aprender y muchos trucos poderosos en el sistema de formularios.