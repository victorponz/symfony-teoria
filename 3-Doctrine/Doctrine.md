---
typora-copy-images-to: assets
---

Symfony no proporciona un componente para trabajar con bases de datos, pero proporciona una estrecha integración con una biblioteca de terceros llamada `Doctrine`.

Instalar Doctrine
-------------------

Primero, instalamos `Doctrine`, así como `MakerBundle`, que ayudará a generar algo de código:

```bash
composer require doctrine maker
```
## Configurar la base de datos

La información de conexión de la base de datos se almacena como una variable de entorno llamada `DATABASE_URL`.  Para el desarrollo, puedes encontrar y personalizar esto dentro de `.env`:

```php
# .env
# customize this line!
DATABASE_URL="mysql://db_user:db_password@127.0.0.1:3306/db_name"
```

Ahora que los parámetros de conexión están configurados, `Doctrine` puede crear la base de datos `db_name` por nosotros:

```bash
$ php bin/console doctrine:database:create
```


> **Tip**
> Hay muchos más comandos de Doctrine . Ejecuta `php bin/console list doctrine` para ver una lista completa

Crear una Clase de Entidad (Entity Class)
------------------------

Supongamos que estamos creando una aplicación donde deben mostrarse productos. Sin siquiera pensar en `Doctrine` o bases de datos, ya sabemos que necesitamos un objeto **Product** para representar esos productos. Usa el comando `make: entity` para crear esta clase:

```bash
$ php bin/console make:entity Product
```
Ahora tenemos un nuevo fichero `src/Entity/Product.php`:

```php
// src/Entity/Product.php
namespace App\Entity;

use Doctrine\ORM\Mapping as ORM;

/**
 * @ORM\Entity(repositoryClass="App\Repository\ProductRepository")
 */
class Product
{
    /**
     * @ORM\Id
     * @ORM\GeneratedValue
     * @ORM\Column(type="integer")
     */
    private $id;

    // add your own fields
}
```
Esta clase se llama una "entidad". Y pronto, podremos guardar y consultar objetos de tipo **Product** en una tabla de `product` en nuestra base de datos.

Mapeando más Campos / Columnas
-----------------------------

Cada propiedad en la entidad **Product** se puede asignar a una columna en la tabla de productos. Al agregar alguna configuración de mapeo, `Doctrine` podrá guardar un objeto **Product** en la tabla `product` y realizar consultas en la tabla `product` y convertir esos datos en objetos **Product**:

![_images/mapping_single_entity.png](https://symfony.com/doc/current/_images/mapping_single_entity.png)

Vamos a dar a la clase de entidad **Product** tres propiedades más y asignarlas a columnas en la base de datos. Esto generalmente se hace con anotaciones

```php
    // src/Entity/Product.php
    // ...

    // this use statement is needed for the annotations
    use Doctrine\ORM\Mapping as ORM;

    class Product
    {
        /**
         * @ORM\Id
         * @ORM\GeneratedValue
         * @ORM\Column(type="integer")
         */
        private $id;

        /**
         * @ORM\Column(type="string", length=100)
         */
        private $name;

        /**
         * @ORM\Column(type="decimal", scale=2, nullable=true)
         */
        private $price;
    }
```


```yaml
    # config/doctrine/Product.orm.yml
    App\Entity\Product:
        type: entity
        id:
            id:
                type: integer
                generator: { strategy: AUTO }
        fields:
            name:
                type: string
                length: 100
            price:
                type: decimal
                scale: 2
                nullable: true
```

`Doctrine` admite una amplia variedad de diferentes tipos de campos, cada uno con sus propias opciones.

Para ver una lista completa de tipos y opciones, consulta [Doctrine's Mapping Types documentation](http://docs.doctrine-project.org/projects/doctrine-orm/en/latest/reference/basic-mapping.html).

> **Cuidado**
> Ten cuidado de no utilizar palabras clave SQL reservadas como sus nombres de tabla o columna (por ejemplo, `GROUP` o `USER`). Consulta la documentación de palabras clave reservadas de SQL de Doctrine para obtener detalles sobre cómo evitarlos

Migraciones: creación de las tablas de la base de datos / esquema
-----------------------------------------------

La clase **Product** está completamente configurada y lista para guardar en una tabla `product`.

Por supuesto, la base de datos aún no tiene la tabla de productos. Para agregar la tabla, podemos aprovechar el `DoctrineMigrationsBundle` que ya está instalado:

```bash
$ php bin/console doctrine:migrations:diff
```

Si todo va bien,  veremos algo como esto:

```bash
Generated new migration class to
"/path/to/project/doctrine/src/Migrations/Version20171122151511.php"
from schema differences.```
```
¡Si abres este archivo, contiene el `SQL` necesario para actualizar su base de datos! Para ejecutar ese SQL, ejecuta las migraciones:

```bash
$ php bin/console doctrine:migrations:migrate
```
Este comando ejecuta todos los archivos de migración que aún no se han ejecutado en tu base de datos.

También se puede hacer con el comando:

```bash
$ php ./bin/console doctrine:schema:update --force
```

Aunque se recomienda hacerlo con migraciones.

Migraciones y agregar más campos
-------------------------------

Pero si necesitamos añadir más campos a **Product**, como `description`:

```diff
// src/Entity/Product.php
// ...

class Product
{
    // ...

+     /**
+      * @ORM\Column(type="text")
+      */
+     private $description;
}
```
La nueva propiedad está asignada, pero aún no existe en la tabla de productos. ¡No hay problema! Solo genera una nueva migración:

```bash
$ php bin/console doctrine:migrations:diff
```
Esta vez, el `SQL` en el archivo generado se verá así:

```sql
ALTER TABLE product ADD description LONGTEXT NOT NULL
```
El sistema de migración es *inteligente*. ¡Compara todas tus entidades con el estado actual de la base de datos y genera el `SQL` necesario para sincronizarlas! Al igual que antes, ejecuta tus migraciones:

```bash
$ php bin/console doctrine:migrations:migrate
```
Esto solo ejecutará el nuevo archivo de migración, porque `DoctrineMigrationsBundle` sabe que la primera migración ya se ejecutó antes. Entre bastidores, administra automáticamente una tabla `migration_versions` para rastrear esto.

Cada vez que realices un cambio en tu esquema, ejecuta estos dos comandos para generar la migración y luego ejecútala. Asegúrate de confirmar los archivos de migración y ejecutarlos cuando despliegues.


Generar Getters y Setters
------------------------------

`Doctrine` ahora sabe cómo persistir un objeto **Product** en la base de datos. Pero la clase en sí no es útil todavía. ¡Todas las propiedades son privadas, por lo que no hay forma de establecer datos en ellas!

Por esa razón, debes crear `getters` y `setters` públicos para todos los campos que necesites modificar desde fuera de la clase:

```php
// src/Entity/Product
// ...

class Product
{
    // all of your properties

    public function getId()
    {
        return $this->id;
    }

    public function getName()
    {
        return $this->name;
    }

    public function setName($name)
    {
        $this->name = $name;
    }

    // ... getters & setters for price & description
}
```


> **Tip**
>
> En realidad, no necesitarás un método `setId()`: `Doctrine` lo configurará automáticamente

Persistir objetos en la base de datos
----------------------------------

¡Es hora de guardar un objeto **Product** en la base de datos! Vamos a crear un nuevo controlador para experimentar:

```bash
$ php bin/console make:controller ProductController
```
¡Dentro del controlador, puedes crear un nuevo objeto **Product**, establecer datos en él y guardarlo!

```php
// src/Controller/ProductController.php
namespace App\Controller;

// ...
use App\Entity\Product;

class ProductController extends Controller
{
    /**
     * @Route("/product", name="product")
     */
    public function index()
    {
        // you can fetch the EntityManager via $this->getDoctrine()
        // or you can add an argument to your action: index(EntityManagerInterface $em)
      	// Sección 1
        $em = $this->getDoctrine()->getManager();

      	// Sección 2
        $product = new Product();
        $product->setName('Keyboard');
        $product->setPrice(19.99);
        $product->setDescription('Ergonomic and stylish!');

        // tell Doctrine you want to (eventually) save the Product (no queries yet)
      	// Sección 3
        $em->persist($product);

		// Sección 4
        // actually executes the queries (i.e. the INSERT query)
        $em->flush();

        return new Response('Saved new product with id '.$product->getId());
    }
}
```
¡Pruébalo!

    http://localhost/product
¡Felicitaciones! Acabas de crear tu primera fila en la tabla `product`. Para probarlo, puedes consultar la base de datos directamente:

```bash
$ php bin/console doctrine:query:sql 'SELECT * FROM product'
```
Echemos un vistazo al ejemplo anterior con más detalle:

* **Sección 1** El método `$this->getDoctrine()->getManager()` obtiene el objeto del administrador de entidades de `Doctrine`, que es el objeto más importante en `Doctrine`. Es responsable de guardar objetos y recuperar objetos de la base de datos.
* **Sección 2** En esta sección, creamos una instancia y trabajamos con el objeto `$product` como cualquier otro objeto PHP normal.
* **Sección 3 ** La llamada `persist($product)`  le dice a `Doctrine` que *gestione* el objecto `$product`. Esto **no** causa que se haga una consulta a la base de datos.
* **Sección 4** Cuando se llama al método `flush()`, `Doctrine` examina todos los objetos que está gestionando para ver si necesitan persistir en la base de datos. En este ejemplo, los datos del objeto `$product` no existen en la base de datos, por lo que el administrador de entidades ejecuta una consulta **INSERT**, creando una nueva fila en la tabla del producto. Si la llamada a `flush()` falla, se produce una excepción `Doctrine\ORM\ORMException`


Ya sea que estemos creando o actualizando objetos, el flujo de trabajo siempre es el mismo: `Doctrine` es lo suficientemente inteligente como para saber si debe INSERTAR o ACTUALIZAR tu entidad.

Obteniendo objetos de la base de datos
----------------------------------

Recuperar un objeto de la base de datos es aún más fácil. Supongamos que quieres poder ir a `/product/1` para ver tu nuevo producto :

```php
// src/Controller/ProductController.php
// ...

/**
 * @Route("/product/{id}", name="product_show")
 */
public function showAction($id)
{
    $product = $this->getDoctrine()
        ->getRepository(Product::class)
        ->find($id);

    if (!$product) {
        throw $this->createNotFoundException(
            'No product found for id '.$id
        );
    }

    return new Response('Check out this great product: '.$product->getName());

    // or render a template
    // in the template, print things with {{ product.name }}
    // return $this->render('product/show.html.twig', ['product' => $product]);
}
```

¡Prúebalo!

    http://localhost/product/1
Cuando consultas un tipo particular de objeto, siempre usas lo que se conoce como su "repositorio". Puedes pensar en un repositorio como una clase PHP cuyo único trabajo es ayudarte a buscar entidades de una determinada clase.

Una vez que tienes un objeto de repositorio, tiene muchos métodos de ayuda:

```php
$repository = $this->getDoctrine()->getRepository(Product::class);
// query for a single Product by its primary key (usually "id")
$product = $repository->find($id);

// query for a single Product by name
$product = $repository->findOneBy(['name' => 'Keyboard']);
// or find by name and price
$product = $repository->findOneBy([
    'name' => 'Keyboard',
    'price' => 19.99,
]);

// query for multiple Product objects matching the name, ordered by price
$products = $repository->findBy(
    ['name' => 'Keyboard'],
    ['price' => 'ASC']
);

// find *all* Product objects
$products = $repository->findAll();
```
Obtención automática de Objetos (ParamConverter)
-----------------------------------------------

En muchos casos, puedes usar `SensioFrameworkExtraBundle` para realizar la consulta automáticamente. Primero, instala el paquete en caso de que no lo tengas:

```bash
    $ composer require annotations sensio/framework-extra-bundle
```

Ahora, simplifica el controlador:
```php
// src/Controller/ProductController.php

use App\Entity\Product;
// ...

/**
     * @Route("/product/{id}", name="product_show")
     */
public function showAction(Product $product)
{
  // use the Product!
  // ...
}
```

¡Eso es todo! El paquete usa el `{id}` de la ruta para consultar el **Product** por la columna `id`. Si no se encuentra, se genera una página 404.

Hay muchas más opciones que se pueden usar. Lee más sobre [ParamConverter](http://symfony.com/doc/current/bundles/SensioFrameworkExtraBundle/annotations/converters.html).

Actualizar un objeto
------------------

Una vez que hemos obtenido un objeto de `Doctrine`, actualizarlo es fácil:

```php
/**
 * @Route("/product/edit/{id}")
 */
public function updateAction($id)
{
    $em = $this->getDoctrine()->getManager();
    $product = $em->getRepository(Product::class)->find($id);

    if (!$product) {
        throw $this->createNotFoundException(
            'No product found for id '.$id
        );
    }

    $product->setName('New product name!');
    $em->flush();

    return $this->redirectToRoute('product_show', [
        'id' => $product->getId()
    ]);
}
```

La actualización de un objeto implica solo tres pasos:

1. Obtener el objeto de `Doctrine`;
2. modificarlo;
3. llamar `flush()` en el gestor de entidades (*entity manager*).

Puede llamar `$em->persist($product)` pero no es necesario: `Doctrine` ya está "observando" el objeto en busca de cambios.

Eliminar un objeto
------------------

Eliminar un objeto es muy similar, pero requiere una llamada al método `remove()` del administrador de entidades:

```php
$em->remove($product);
$em->flush();
```
Como era de esperar, el método `remove()` notifica a `Doctrine` que deseas eliminar el objeto dado de la base de datos. La consulta DELETE no se ejecuta realmente hasta que se llama al método `flush()`.

Consulta de objetos: el repositorio
------------------------------------

Ya has visto cómo el objeto repositorio permite ejecutar consultas básicas sin ningún trabajo :

```php
// from inside a controller
$repository = $this->getDoctrine()->getRepository(Product::class);

$product = $repository->find($id);
```
Pero, ¿y si necesitas una consulta más compleja? Cuando generaste la entidad con `make:entity`, el comando también generó una clase `ProductRepository`:

```php
// src/Repository/ProductRepository.php
namespace App\Repository;

use App\Entity\Product;
use Doctrine\Bundle\DoctrineBundle\Repository\ServiceEntityRepository;
use Symfony\Bridge\Doctrine\RegistryInterface;

class ProductRepository extends ServiceEntityRepository
{
    public function __construct(RegistryInterface $registry)
    {
        parent::__construct($registry, Product::class);
    }
}
```
Cuando recuperas tu repositorio (es decir,`->getRepository(Product::class)`), ¡en realidad es una instancia de este objeto! Esto se debe a la configuración `repositoryClass` que se generó encima de la clase de entidad del **Product**.

Supongamos que deseas consultar todos los objetos del **Product** que superen un determinado precio. Agrega un nuevo método para esto a tu repositorio:

```php
// src/Repository/ProductRepository.php
// ...
class ProductRepository extends ServiceEntityRepository
{
  public function __construct(RegistryInterface $registry)
  {
    parent::__construct($registry, Product::class);
  }

  /**
   * @param $price
   * @return Product[]
  */
  public function findAllGreaterThanPrice($price): array
  {
    // automatically knows to select Products
    // the "p" is an alias you'll use in the rest of the query
    $qb = $this->createQueryBuilder('p')
      ->andWhere('p.price > :price')
      ->setParameter('price', $price)
      ->orderBy('p.price', 'ASC')
      ->getQuery();

    return $qb->execute();

    // to get just one result:
    // $product = $qb->setMaxResults(1)->getOneOrNullResult();
  }
}
```

Esto usa [Query Builder](http://docs.doctrine-project.org/projects/doctrine-orm/en/latest/reference/query-builder.html) de `Doctrine`: una forma muy poderosa y fácil de usar para escribir consultas personalizadas. Ahora, puedes llamar a este método en el repositorio:

```php
// from inside a controller
$minPrice = 10;
$products = $this->getDoctrine()
  ->getRepository(Product::class)
  ->findAllGreaterThanPrice($minPrice);

// ...
```
Consultas con DQL o SQL
------------------------

Además del generador de consultas, también puedes consultar con [Doctrine Query Language](http://docs.doctrine-project.org/projects/doctrine-orm/en/latest/reference/dql-doctrine-query-language.html):

```php
// src/Repository/ProductRepository.php
// ...

public function findAllGreaterThanPrice($price): array
{
    $em = $this->getEntityManager();
    
    $query = $em->createQuery(
        'SELECT p
        FROM App\Entity\Product p
        WHERE p.price > :price
        ORDER BY p.price ASC'
    )->setParameter('price', 10);

    // returns an array of Product objects
    return $query->execute();
}
```

O directamente con `SQL` si lo necesitas:

```php
// src/Repository/ProductRepository.php
// ...

public function findAllGreaterThanPrice($price): array
{
    $conn = $this->getEntityManager()->getConnection();

    $sql = '
        SELECT * FROM product p
        WHERE p.price > :price
        ORDER BY p.price ASC
        ';
    $stmt = $conn->prepare($sql);
    $stmt->execute(['price' => 10]);

    // returns an array of arrays (i.e. a raw data set)
    return $stmt->fetchAll();
}
```

Con `SQL`, obtendrás datos en bruto, no objetos (a menos que uses la funcionalidad [NativeQuery](http://docs.doctrine-project.org/projects/doctrine-orm/en/latest/reference/native-sql.html)).

Relaciones y asociaciones
------------------------------

`Doctrine` proporciona toda la funcionalidad que necesitas para administrar las relaciones de bases de datos (también conocidas como asociaciones),  incluidas las relaciones ManyToOne, OneToMany, OneToOne y ManyToMany.

Para más información consulta [How to Work with Doctrine Associations / Relations](https://symfony.com/doc/current/doctrine/associations.html).

