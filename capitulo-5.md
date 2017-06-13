#  Capítulo 5. Modelos - Estructurando los Datos de la Aplicación.

En los capítulos anteriores, tuvimos un resumen de principio a fin de la creación de nuevos módulos para Odoo. En el Capítulo 2, *Construyendo tu Primera Aplicación Odoo*, construimos una aplicación completamente nueva, y en el Capítulo 3, *Extendiendo Aplicaciones Existentes de Herencia*, exploramos la herencia y cómo usarla para crear un módulo de extensión para nuestra aplicación. En el Capítulo 4, *Datos de Módulos*, discutimos cómo agregar datos iniciales y de demostración a nuestros módulos.

En estas reseñas, hemos abordado todas las capas involucradas en la construcción de una aplicación backend para Odoo. Ahora, en los capítulos siguientes, es el momento de explicar estas varias capas que componen una aplicación con más detalle: modelos, vistas y lógica de negocio.

En este capítulo, aprenderás cómo diseñar las estructuras de datos que soportan una aplicación y cómo representar las relaciones entre ellas.

## Organizando las características de la aplicación en módulos

Como antes, usaremos un ejemplo para ayudarnos a explicar los conceptos.

Las características de herencia de Odoo proporcionan un mecanismo de extensibilidad efectivo. Esto te permite ampliar aplicaciones de terceros existentes sin cambiarlas directamente. Esta composición también permite un patrón de desarrollo orientado a módulos, en el que las aplicaciones grandes se pueden dividir en características más pequeñas, lo suficientemente ricas como para mantenerse por sí solas.

Esto puede ser útil para limitar la complejidad, tanto en el nivel técnico como en el nivel de experiencia del usuario. Desde una perspectiva técnica, dividir un gran problema en partes más pequeñas facilita la solución y es más amigable para el desarrollo incremental de características. Desde la perspectiva de la experiencia del usuario, podemos optar por activar sólo las características que realmente son necesarias para ellos, para una interfaz de usuario más sencilla. Por lo tanto, vamos a mejorar nuestra aplicación To-Do a través de módulos addon adicionales para finalmente formar una aplicación completa.

### Presentando el módulo todo_ui

En el capítulo anterior, primero creamos una aplicación para tareas personales y luego la ampliamos para que la tarea se pudiera compartir con otras personas.

Ahora queremos llevar nuestra aplicación al siguiente nivel mejorando su interfaz de usuario, incluyendo un tablero kanban. El tablero kanban es una herramienta de flujo de trabajo simple que organiza los elementos en columnas, donde estos elementos fluyen de la columna izquierda a la derecha, hasta que se completan. Organizaremos nuestras Tareas en columnas, de acuerdo con sus Etapas, como **Esperar**, **Listo**, **Iniciado** o **Hecho**.

Comenzaremos agregando las estructuras de datos para permitir esta visión. Tenemos que añadir etapas, y será bueno añadir soporte para las etiquetas también,  permitiendo que las tareas se clasifiquen por tema. En este capítulo, nos centraremos únicamente en los modelos de datos. La interfaz de usuario para estas características se describirá en el Capítulo 6, *Vistas - Diseñando la interfaz de usuario* y las vistas de kanban en el Capítulo 9, *Vistas QWeb y Kanban*.

Lo primero que debemos averiguar es cómo serán estructurados nuestros datos para que podamos diseñar los modelos de apoyo. Ya tenemos la entidad central: la Tarea pendiente. Cada tarea estará en una etapa a la vez y las tareas también pueden tener una o más etiquetas en ellas. Necesitaremos agregar estos dos modelos adicionales, y ellos tendrán estas relaciones:

+ Cada tarea tiene una etapa, y puede haber muchas tareas en cada etapa
+ Cada tarea puede tener muchas etiquetas, y cada etiqueta se puede adjuntar a muchas tareas

Esto significa que las Tareas tienen una relación de muchos a uno con las Fases, y las relaciones de muchos a muchos con las Etiquetas. Por otro lado, las relaciones inversas son: Las etapas tienen una relación de uno a muchos con las Tareas y las Etiquetas tienen una relación de muchos a muchos con Tareas.

Comenzaremos por crear el nuevo módulo `todo_ui` y añadiremos los modelos de Tareas pendientes y de Tareas.

Hemos estado usando el directorio `~ / odoo-dev / custom-addons /` para alojar nuestros módulos. Debemos crear un nuevo directorio `todo_ui` dentro de él para los nuevos addons. Desde el shell, podríamos usar los siguientes comandos:

```

$ cd ~/odoo-dev/custom-addons





$ mkdir todo_ui





$ cd todo_ui
```

Comenzamos añadiendo el archivo manifiesto `__manifest__.py`, con este contenido:

```
{
  'name': 'User interface improvements to the To-Do app',
  'description': 'User friendly features.',
  'author': 'Daniel Reis',
  'depends': ['todo_user'] }
```
También debemos añadir un archivo` __init__.py`. Está perfectamente bien que esté vacío por ahora.

Ahora podemos instalar el módulo en nuestra base de datos Odoo y comenzar con los modelos.

## Creando modelos

Para que las tareas pendientes tengan un tablero kanban, necesitamos Etapas. Las etapas son columnas del tablero, y cada tarea cabrá en una de estas columnas:


1. Edita `todo_ui/__init__.py` para importar el submodulo `models`:

```
from . import models 
```

1. Crea el directorio `todo_ui/models` y añádelo al archivo `an __init__.py` con esto:
```
        from . import todo_model 
```
1. Ahora, añadamoslo al archivo de código Pyton `todo_ui/models/todo_model.py`:
```
        # -*- coding: utf-8 -*-
        from odoo import models, fields, api

        class Tag(models.Model):
            _name = 'todo.task.tag'
            _description = 'To-do Tag'
            name = fields.Char('Name', 40, translate=True)
        class Stage(models.Model):
            _name = 'todo.task.stage'
            _description = 'To-do Stage'
            _order = 'sequence,name'

        name = fields.Char('Name', 40, translate=True)
            sequence = fields.Integer('Sequence')
```

Aquí hemos creado los dos nuevos modelos que serán referenciados en las tareas pendientes.

Centrándonos en las etapas de la tarea, tenemos una clase Python, Etapa, basada en la clase `models.Model`, que define un nuevo modelo Odoo llamado `todo.task.stage`. También tenemos dos campos: `nombre y secuencia`. Podemos ver algunos atributos de modelo (prefijados con un subrayado) que son nuevos para nosotros. Echemos un vistazo a ellos.

## Atributos del modelo

Las clases de modelo pueden utilizar atributos adicionales que controlan algunos de sus comportamientos. Estos son los atributos más utilizados:

+ `_name` es el identificador interno del modelo Odoo que estamos creando. Obligatorio cuando se crea un nuevo modelo.
+ `_description` es un título fácil de usar para los registros del modelo, que se muestra cuando se ve el modelo en la interfaz de usuario. Opcional pero recomendado.
+ `_order` establece el orden predeterminado para utilizar cuando se exploran los registros del modelo o se muestran en una vista de lista. Es una cadena de texto que se usará como cláusula `SQL order by`, por lo que puede ser cualquier cosa que puedas utilizar allí, aunque tiene un comportamiento inteligente y admite nombres de campo traducibles y muchos a uno.

Para completar, hay un par de más atributos que se pueden utilizar en casos avanzados:

+ `_rec_name` indica el campo a utilizar como la descripción del registro cuando se hace referencia desde campos relacionados, tales como una relación de varios a uno. De forma predeterminada, utiliza el campo de `nombre`, que es un campo común en los modelos. Pero este atributo nos permite usar cualquier otro campo para ese propósito.
+ `_table` es el nombre de la tabla de la base de datos que soporta el modelo. Por lo general, se deja que se calcule automáticamente, y es el nombre del modelo con los puntos reemplazados por subrayados. Pero es posible establecer para indicar un nombre de tabla específico.

También podemos tener los atributos `_inherit` y `_inherits`, como se explicó en el capítulo 3, *Herencia - Extendiendo las aplicaciones existentes.

### Modelos y clases Python
Los modelos Odoo están representados por clases Python. En el código anterior, tenemos una `Etapa` clase Python, basada en la clase `models.Model`, que define un nuevo modelo Odoo llamado `todo.task.stage`.

Los modelos Odoo se mantienen en un registro central, también conocido como piscina en las versiones más antiguas de Odoo. Es un diccionario que mantiene referencias a todas las clases de modelo disponibles en la instancia, y puede ser referenciado por un nombre de modelo. Específicamente, el código de un método de modelo puede usar `self.env ['x']` para obtener una referencia a una clase que representa el modelo `x`.

Puedes ver que los nombres de los modelos son importantes ya que son las claves utilizadas para acceder al registro. La convención para los nombres de modelo es usar una lista de palabras en minúsculas unidas con puntos, como `todo.task.stage`. Otros ejemplos de los módulos principales son `project.project`, `project.task` o `project.task.type`. Deberíamos utilizar el modelo singular `todo.task` en lugar de `todo.task`. Por razones históricas, es posible encontrar algunos modelos básicos que no siguen esto, como `res.users`, pero no es la regla.

Los nombres de los modelos deben ser globalmente únicos. Debido a esto, la primera palabra debe corresponder a la aplicación principal que módulo se refiere. En nuestro ejemplo, es `todo`. Otros ejemplos de los módulos principales son `project`, `crm` o `sale`.

Las clases de Python, por otro lado, son locales al archivo Python donde se declaran. El identificador utilizado para ellos es sólo significativo para el código en ese archivo. Debido a esto, no se requiere que los identificadores de clase sean prefijados por la aplicación principal con la que se relacionan. Por ejemplo, no hay ningún problema para nombrar nuestra clase `Stage` para el modelo `todo.task.stage`. No hay riesgo de colisión con las posibles clases con el mismo nombre en otros módulos.

Se pueden utilizar dos convenciones diferentes para los identificadores de clase: `snake_case` o `CamelCase`. Históricamente, el código de Odoo usó el caso snake, y todavía es posible encontrar las clases que utilizan esta convención. Pero la tendencia es utilizar el caso case, puesto que es el estándar de Python definido por las convenciones de codificación del PEP8. Puedes haber notado que estamos utilizando la última forma.

### Modelos transitorios y abstractos

En el código anterior y en la gran mayoría de los modelos de Odoo, las clases se basan en la clase `models.Model`. Estos tipos de modelos tienen persistencia permanente de la base de datos: se crean tablas de base de datos para ellos y sus registros se almacenan hasta que se borran explícitamente.

Pero Odoo también proporciona otros dos tipos de modelos que se utilizarán: modelos transitorios y abstractos.

+ **Los modelos transitorios** se basan en la clase `models.TransientModel` y se utilizan para la interacción del usuario estilo asistente. Sus datos aún se almacenan en la base de datos, pero se espera que sea temporal. Un trabajo periódico de vacío limpia los datos antiguos de estas tablas. Por ejemplo, la ventana de diálogo `Load a Lenguage`, que se encuentra en el menú `Setings | Translations`, utiliza un modelo Transient para almacenar selecciones de usuarios e implementar la lógica del asistente.
+ **Los modelos abstractos** se basan en la clase models.AbstractModel y no tienen ningún almacenamiento de datos adjunto a ellos. Actúan como conjuntos de funciones reutilizables que se mezclan con otros modelos, utilizando las capacidades de herencia de Odoo. Por ejemplo, `mail.thread` es un modelo abstracto, proporcionado por el addon `Discuss`, utilizado para agregar funciones de mensaje y seguidores a otros modelos.

### Inspeccionando los modelos existentes

Los modelos y campos creados a través de las clases Python tienen su metadata disponíble a través de la interface de usuario. En el menú superior **Settings** navega al ítem de menú ** Technical | Database Structure | Models**.
Aquí, hallarás la lista de todos los modelos disponíbles en la base de datos. Haciendo click en un modelo en la lista, abrirá una forma con estos detalles:

![Modelo_de_lista](file:img/5-01.jpg)

esta es una buena herramienta para inspeccionar la estructura de un modelo, ya que en un lugar, puedes ver los resultados de la personaliación de diferentes modulos. En este caso, tal como lo puedes ver en en la esquina superior derecha en el campo **In Apps**, las definiciones `todo.task`para este modelo provienen de ambos modulos `todo_app` y `todo_user`.

En el área inferior, tenemos algunas pestañas de información disponíbles: una referencia rápida para los modelos **Campos**, los **Derechos de Acceso** garantizados en grupos de seguridad y también enlista las **Vistas** disponíbles para este modelo.

Podemos hallar el **Identificador Externo** del modelo utilizando, desde el menú **Desarrollador**, la opción **Metadata View**. Los identificadores de modelo externo, o IDs XML, son generadas automáticamente por el ORM pero justamente predecible: para el modelo `todo.task`, el identificador externo es `model_todo_task`.

### Tip
¡El formulario **Modelos** es editable!. Es posible crear y modificar modelos, campos, y vistas desde aqui. Puedes utilizar este para construír prototipos antes de persistir en modulos.

## Creando campos
Luego de crear un nuevo modelo, el próximo paso es añadir campos a éste. Odoo soporta todos los tipos de datos básicos que se esperan, tales como cadenas  de texto,   la base de datos, enteros, números de punto flotante, Booleanos, fechas,  

Algunos nombres de campo son especiales, se marchitan porque están reservados por el ORM para propósitos especiales, o porque algunas características incorporadas usan por defecto algunos nombres de campo predeterminados.

Vamos a explorar los diversos tipos de campos disponibles en Odoo.

### Tipos de campos básicos

Ahora tenemos un modelo de `Etapa` y lo ampliaremos para agregar algunos campos adicionales. Debemos editar el archivo `todo_ui / models / todo_model.py` y añadir definiciones de campo adicionales para que se vea así:

```
class Stage(models.Model): 
    _name = 'todo.task.stage' 
    _description = 'To-do Stage' 
    _order = 'sequence,name' 
    # String fields: 
    name = fields.Char('Name', 40) 
    desc = fields.Text('Description') 
    state = fields.Selection( 
        [('draft','New'), ('open','Started'),
        ('done','Closed')],'State') 
    docs = fields.Html('Documentation') 
    # Numeric fields: 
    sequence = fields.Integer('Sequence') 
    perc_complete = fields.Float('% Complete', (3, 2)) 
    # Date fields: 
    date_effective = fields.Date('Effective Date') 
    date_changed = fields.Datetime('Last Changed') 
    # Other fields: 
    fold = fields.Boolean('Folded?') 
    image = fields.Binary('Image')
```
Aquí, tenemos una muestra de los tipos de campo no relacional disponibles en Odoo con los argumentos posicionales esperados por cada uno.

En la mayoría de los casos, el primer argumento es el título del campo, que corresponde al argumento del campo de la `string`; Se utiliza como texto predeterminado para las etiquetas de la interfaz de usuario. Es opcional, y si no se proporciona, un título se generará automáticamente a partir del nombre del campo.

Para los nombres del campo de fecha, hay una convención para usar la fecha como un prefijo. Por ejemplo, deberíamos usar el campo `date_effective` en lugar de `effective_date`. Convenciones similares también se aplican a otros campos, como `amount_`, `price_` o `qty_`.

Estos son los argumentos de posición estándar esperados por cada uno de los tipos de campo:

+ `Char` espera un segundo tamaño de argumento opcional para el tamaño máximo de texto. Se recomienda no usarlo a menos que exista un requisito de negocio que lo requiera, como un número de seguro social con una longitud fija.
+ `Text`o difiere de `Char`, ya que puede albergar contenido de texto multilínea, pero espera los mismos argumentos.
+ `Selection` es una lista de selección desplegable. El primer argumento es la lista de opciones seleccionables y el segundo es el título de la cadena. El elemento de selección es una lista de tuplas (`'value'`, `'Títle'`), para el valor almacenado en la base de datos y la correspondiente descripción de interfaz de usuario. Cuando se extiende a través de la herencia, el argumento `selection_add` está disponible para añadir nuevos elementos a una lista de selección existente.
+ `Html` se almacena como un campo de texto, pero tiene un manejo específico en la interfaz de usuario, para la presentación de contenido HTML. Por razones de seguridad, se desinfectan de forma predeterminada, pero este comportamiento se puede sobreescribir.
+ `Integer` sólo espera un argumento de cadena para el título del campo.
+ `Float` tiene un segundo argumento opcional, una tupla (`x,y`) con la precisión del campo: `x` es el número total de dígitos; De éstos, `y` son dígitos decimales.
+ Los campos `Date` y `Datetime` sólo esperan la cadena de texto como un argumento de posicional. Por razones históricas, el ORM maneja sus valores en un formato de cadena. Las funciones auxiliares se deben utilizar para convertirlas en objetos de fecha real. También los valores de fecha y hora se almacenan en la base de datos en tiempo UTC pero presentadas en hora local, utilizando las preferencias de zona horaria del usuario. Esto se discute con más detalle en el Capítulo 6, *Vistas - Diseñando la interfaz de usuario*.
+ `Boolean` tiene valores `True` o `False`, como puedes esperar, y sólo tiene un argumento de posición para la cadena de texto.
+ `Binary` almacena datos binarios de tipo archivo y también espera sólo el argumento de cadena. Pueden ser manejados por código Python usando cadenas codificadas en `base64`.

Aparte de estos, también tenemos los campos relacionales, que serán presentados más adelante en este capítulo. Pero ahora, todavía hay más para aprender acerca de estos tipos de campo y sus atributos.

### Atributos de campo comunes

Los campos tienen atributos que se pueden establecer al definirlos. Dependiendo del tipo de campo, algunos atributos pueden ser pasados ​​en posición, sin una palabra clave de argumento, como se muestra en la sección anterior.

Por ejemplo, `name=fields.Char('Name', 40)` podría hacer uso de argumentos posicionales. Usando los argumentos de la palabra clave, lo mismo se podría escribir como `name=fields.Char (size=40, string='Name')`. Puedes encontrar más información sobre argumentos de palabras clave en la documentación oficial de Python en https://docs.python.org/2/tutorial/controlflow.html#keyword-arguments.

Todos los atributos disponibles se pueden pasar como un argumento de palabra clave. Estos son los atributos generalmente disponibles y las palabras clave de argumento correspondientes:

+ `string` es la etiqueta por defecto del campo, que se utilizará en la interfaz de usuario. Excepto para los campos de selección y relacionales, es el primer argumento posicional, por lo que la mayoría de las veces no se utiliza como argumento de palabra clave.
+ `default` establece un valor predeterminado para el campo. Puede ser un valor estático, como una cadena o una referencia callable, ya sea una función con nombre o una función anónima (una expresión lambda).
+ `size` sólo se aplica a los campos `Char` y puede establecer un tamaño máximo permitido. La mejor práctica actual es no usarla a menos que sea realmente necesaria.
+ `translate` se aplica sólo a los campos `Char`, `Text` y `Html`, y hace que el contenido del campo se pueda traducir, manteniendo valores diferentes para diferentes idiomas.
+ `help` proporciona el texto para las sugerencias que se muestran a los usuarios.
+ `readonly=True` hace que el campo por defecto no sea editable por la interfaz de usuario. Esto no se aplica a nivel API; Es sólo una configuración de interfaz de usuario.
+ `required=True` hace obligatorio el campo por defecto en la interfaz de usuario. Esto se aplica en el nivel de base de datos mediante la adición de una restricción `NOT NULL` en la columna.
+ `index=True` creará un índice de base de datos en el campo.
+ `copy=False` tiene el campo ignorado cuando se utiliza la función de registro duplicado, método ORM `copy ()`. Los campos no relacionales son `copyable` de forma predeterminada.
+ `groups` permite limitar el acceso y la visibilidad del campo a sólo algunos grupos. Espera una lista separada por comas de IDs XML para grupos de seguridad, como `groups='base.group_user, base.group_system'`.

+ `states` espera un diccionario que asigna valores para los atributos UI  que dependen de los valores del campo `satate`. Por ejemplo: `states = {'done': [('readonly', True)]}`. Los atributos que se pueden utilizar son `readonly`, `required` e `invisible`.

#### Nota

Ten en cuenta que el campo`satates` es equivalente al atributo `attrs` en las vistas. Nota que las vistas admiten un atributo `states`, pero tiene un uso diferente: acepta una lista de estados separados por comas para controlar cuando el elemento debe ser visible.

Para completar, a veces se utilizan otros dos atributos cuando se actualiza entre versiones Odoo principales:

+ `deprecated=True` registra una advertencia cada vez que se utiliza el campo.
+ `oldname='field'` se utiliza cuando un campo se renombra en una versión más reciente, permitiendo que los datos en el campo antiguo se copien automáticamente en el nuevo campo.

### Nombres de campos especiales

Algunos nombres de campo están reservados para ser utilizados por el ORM.

El campo `id` es un número automático que identifica de manera única cada registro y se utiliza como la clave principal de la base de datos. Se agrega automáticamente a cada modelo.

Los siguientes campos se crean automáticamente en los nuevos modelos, a menos que se establezca el atributo `_log_access=False`:

+ `create_uid` es para el usuario que creó el registro
+ `create_date` es la fecha y la hora en que se crea el registro
+ `write_uid` es para que el último usuario modifique el registro
+ `write_date` es la última fecha y hora en que se modificó el registro

Esta información está disponible desde el cliente web, navegando hasta el menú **Developer Mode** y seleccionando la opción **View Metadata**.

Algunas características API incorporadas por defecto esperan nombres de campos específicos. Debemos evitar el uso de estos nombres de campo para propósitos diferentes a los que se pretenden. Algunos de ellos son incluso reservado y no se puede utilizar para otros fines en absoluto:

+ `name` se utiliza de forma predeterminada como el nombre para mostrar para el registro. Normalmente es un campo de tipo  `Char`, pero también puede ser un `Text` o un `Many2one`. Todavía podemos establecer otro campo para ser utilizado para el nombre de visualización, utilizando el atributo del modelo `_rec_name`.
+ `Active`, de tipo `Boolean`, permite inactivar registros. Los registros con `active==False` se excluirán automáticamente de las consultas. Para acceder a ellos debe añadirse una condición `('active', '=', False)` al dominio de búsqueda, o `'active_test': False` Se debe agregar al contexto actual.
+ `Sequence`, de tipo `Integer`, si está presente en una vista de lista, permite definir manualmente el orden de los registros. Para que funcione correctamente, no debes olvidar usarlo con el atributo del modelo `_order`.
+ `State`, de tipo `Selection`, representa los estados básicos del ciclo de vida del registro y puede ser utilizado por el atributo de campo del estado para modificar dinámicamente la vista: algunos campos de formulario se pueden `readonly` o `invisible` en estados de registro específicos.
+ `parent_id`, `parent_left` y `parent_right`, de tipo `Integer`, tienen un significado especial para las relaciones jerárquicas padre/hijo. Lo analizaremos en detalle en la siguiente sección.

Hasta ahora, hemos discutido campos no relacionales. Pero una buena parte de una estructura de aplicación de datos es acerca de describir las relaciones entre entidades. Ahora vamos a mirar esto.


## Relaciones entre modelos
Mirando de nuevo el diseño de nuestro módulo, tenemos estas relaciones:

+ Cada tarea tiene una etapa. Esa es una relación de muchos a uno, también conocida como clave extranjera. La inversa es una relación uno-a-muchos, lo que significa que cada etapa puede tener muchas tareas.
+ Cada tarea puede tener muchas etiquetas. Esa es una relación de muchos a muchos. La relación inversa, por supuesto, es también un mucho a muchos, ya que cada etiqueta puede estar en muchas tareas.

El siguiente diagrama de relación de entidad puede ayudar a visualizar las relaciones que estamos a punto de crear en el modelo. Las líneas que terminan con un triángulo representan muchos lados de las relaciones:

![EntityRelationshipDiagram](file:img/5-02.jpg)

Añadamos los correspondientes campos de relación a las tareas pendientes en nuestro archivo `todo_model.py`:

```
class TodoTask(models.Model): 
    _inherit = 'todo.task' 
    stage_id = fields.Many2one('todo.task.stage', 'Stage') 
    tag_ids = fields.Many2many('todo.task.tag', string='Tags')
```

El código anterior muestra la sintaxis básica de estos campos, estableciendo el modelo relacionado y el título del campo `string`. La convención para los nombres de campos relacionales es añadir `_id` o `_ids` a los nombres de campo, para a-una y a-muchas relaciones, respectivamente.

Como ejercicio, puedes intentar también agregar las relaciones inversas correspondientes a los modelos relacionados:

+ La inversa de la relación `Many2one` es un campo `One2many` en las etapas, ya que cada etapa puede tener muchas tareas. Deberíamos añadir este campo a la clase Etapas.
+ La inversa de la relación `Many2many` es también un campo `Many2many` en Etiquetas, ya que cada etiqueta también se puede usar en muchas Tareas.

Echemos un vistazo más de cerca a las definiciones de campos relacionales.

### Relaciones de muchos a uno (Many-to-one)

La relación `Many2one` acepta dos argumentos posicionales: el modelo relacionado (correspondiente al argumento de la palabra clave `comodel`) y el título `string`. Crea un campo en la tabla de base de datos con una clave externa a la tabla relacionada.

Algunos argumentos con nombre adicionales también están disponibles para utilizar con este tipo de campo:

+ `ondelete` define lo que ocurre cuando se elimina el registro relacionado. Su valor predeterminado es `set null`, lo que significa que se establece un valor vacío cuando se elimina el registro relacionado. Otros valores posibles son `restrict`, generando un error que impida la eliminación y `cascade` también eliminar este registro.
+ `context` es un diccionario de datos, significativo para las vistas del cliente web, para llevar información al navegar a través de la relación. Por ejemplo, para establecer vales predeterminados. Se explicará mejor en el Capítulo 6, *Vistas - Diseñando la interfaz de usuario*.
+ `domain` es una expresión de dominio, una lista de tuplas, filtra los registros disponibles para el campo de relación.
+ `auto_join=True` permite al ORM utilizar combinaciones de SQL cuando se realizan búsquedas utilizando esta relación. Si se usan, las reglas de seguridad de acceso serán anuladas y el usuario podría tener acceso a registros relacionados que las reglas de seguridad no permitirían, pero las consultas SQL serán más eficientes y se ejecutarán más rápido.

### Relaciones de muchos a muchos (Many-to-many)
La firma mínima `Many2many` acepta un argumento para el modelo relacionado, y se recomienda proporcionar también el argumento `strings` con el título del  campo.

En el nivel de base de datos, no se agrega ninguna columna a las tablas existentes. En su lugar, crea automáticamente una nueva tabla de relación que tiene sólo dos campos de ID con las claves externas para las tablas relacionadas. El nombre de la tabla de relación y los nombres del campo se generan automáticamente. El nombre de tabla de relación es el nombre de ambas tablas unidos con un subrayado con `_rel` añadido a él.

En algunas ocasiones podemos necesitar anular estos valores predeterminados automáticos.

Uno de estos casos es cuando los modelos relacionados tienen nombres largos, y el nombre de la tabla de relaciones generado automáticamente es demasiado largo, superando el límite de 63 caracteres de PostgreSQL. En estos casos, debemos elegir manualmente un nombre para la tabla de relaciones, para que se ajuste al límite de tamaño de nombre de tabla.

Otro caso es cuando necesitamos una segunda relación muchos-a-muchos entre los mismos modelos. En estos casos, necesitamos proporcionar manualmente un nombre para la tabla de relaciones, para que no colisione con el nombre de tabla que ya se está utilizando para la primera relación.

Hay dos alternativas para anular manualmente estos valores: ya sea utilizando argumentos posicionales o argumentos de palabra clave.

Utilizando argumentos posicionales para la definición de campo tenemos:

```
# Task <-> Tag relation (positional args): 
tag_ids = fields.Many2many( 
    'todo.task.tag',      # related model 
    'todo_task_tag_rel',  # relation table name 
    'task_id',            # field for "this" record 
    'tag_id',             # field for "other" record 
    string='Tags')
```

#### Nota

Ten en cuenta que los argumentos adicionales son opcionales. Podríamos simplemente establecer el nombre de la tabla de relaciones y dejar que los nombres de campo usen los valores predeterminados automáticos.

En su lugar, podemos utilizar argumentos de palabras clave, que algunas personas prefieren para la legibilidad:

```
# Task <-> Tag relation (keyword args): 
tag_ids = fields.Many2many( 
    comodel_name='todo.task.tag',  # related model 
    relation='todo_task_tag_rel',# relation table name 
    column1='task_id',      # field for "this" record 
    column2='tag_id',       # field for "other" record 
    string='Tags')
```

Al igual que los campos muchos a uno, los campos muchos-a-muchos también admiten los atributos de palabras clave `domain` y `context`.

#### Nota

Actualmente hay una limitación en el diseño de ORM, con respecto a los modelos abstractos, que cuando se obliga a los nombres de la tabla de relación y las columnas, ya no se pueden heredar. Así que esto no debería hacerse en modelos abstractos.

El inverso de la relación `Many2many` es también un campo `Many2many`. Si también añadimos un campo `Many2many` al modelo `Tags`, Odoo infiere que esta relación de muchos-a-muchos es la inversa de la del modelo `Task`.

La relación inversa entre Tareas y Etiquetas se puede implementar de la siguiente manera:

```
class Tag(models.Model):    
    _name = 'todo.task.tag'   
    
# Tag class relationship to Tasks:    





    task_ids = fields.Many2many(       





        'todo.task',    # related model        





        string='Tasks')


```

### Relaciones inversas Uno a muchos (One-to-many)

Un inverso de un `Many2one` se puede agregar al otro extremo de la relación. Esto no tiene ningún impacto en la estructura de la base de datos real, pero nos permite navegar fácilmente desde **un** lado de los **muchos** registros relacionados. Un caso de uso típico es la relación entre un encabezado de documento y sus líneas.

En nuestro ejemplo, una relación inversa `One2many` en Etapas nos permite listar fácilmente todas las Tareas en esa Etapa. El código para agregar esta relación inversa a Etapas es:

```
class Stage(models.Model):
    _name = 'todo.task.stage'

    # Stage class relationship with Tasks:






    tasks = fields.One2many(






        'todo.task',   # related model






        'stage_id', # field for "this" on related model






        'Tasks in this stage')


```
El `One2many` acepta tres argumentos posicionales: el modelo relacionado, el nombre del campo en ese modelo que hace referencia a este registro y la cadena de título. Los dos primeros argumentos posicionales corresponden a los argumentos de palabra clave `comodel_name` y `inverse_name`.

Los parámetros de palabras clave adicionales disponibles son los mismos que para `Many2one : context`, `domain`, `ondelete` (aquí actúa en el lado **muchos** de la relación) y `auto_join`.

### Relaciones jerárquicas

Las relaciones de árbol padre-hijo se representan usando una relación `Many2one` con el mismo modelo, de modo que cada registro hace referencia a su padre. Y la inversa `One2many` hace que sea fácil para un padre mantener el seguimiento de sus hijos.

Odoo proporciona un soporte mejorado para estas estructuras de datos jerárquicas, para una navegación más rápida a través de hermanos de árbol y para una búsqueda más fácil usando el operador adicional de expresiones de dominio `child_of`.

Para habilitar estas características necesitamos establecer la bandera de atributo `_parent_store` y añadir al modelo los campos auxiliares: `parent_left` y `parent_right`. Ten en cuenta que esta operación adicional se produce en tiempo de almacenamiento y penalidades de tiempo de ejecución, por lo que es mejor utilizarla cuando se espera leer con más frecuencia que escribir, como en el caso de un árbol de categorías.

Revisitando el modelo `Tags`, definido en el archivo `todo_model.py`, debemos editarlo para que parezca lo siguiente:

```
class Tags(models.Model):
    _name = 'todo.task.tag'
    _description = 'To-do Tag'

    _parent_store = True




    # _parent_name = 'parent_id'
    name = fields.Char('Name')

    parent_id = fields.Many2one(






       'todo.task.tag', 'Parent Tag', ondelete='restrict')






    parent_left = fields.Integer('Parent Left', index=True)






    parent_right = fields.Integer('Parent Right', index=True)

```
Aquí, tenemos un modelo básico, con un campo `parent_id` para referenciar el registro principal y el atributo `additional` `_parent_store`  para agregar soporte de búsqueda jerárquica. Al hacer esto, los campos `parent_left` y `parent_right` también se deben agregar.

El campo que se refiere al padre se espera que se nombre `parent_id`, pero cualquier otro nombre de campo se puede utilizar siempre y cuando lo declaremos en el atributo `_parent_name`.

Además, a menudo es conveniente agregar un campo con los hijos directos del registro:

```
child_ids = fields.One2many(
    'todo.task.tag', 'parent_id', 'Child Tags')
```

## Campos de referencia que utilizan relaciones dinámicas

Los campos relacionales regulares hacen referencia a un comodelo fijo. El tipo de campo de referencia no tiene esta limitación y admite relaciones dinámicas, de modo que el mismo cam po puede referirse a más de un modelo.

Por ejemplo, podemos usarlo para añadir un campo `Refers to` a las tareas pendientes, que puede referirse a un `User` o a un `Partner`:
```
# class TodoTask(models.Model):
    refers_to = fields.Reference(
    [('res.user', 'User'), ('res.partner', 'Partner')],
    'Refers to')
```

Como puedes ver, la definición de campo es similar a un campo de selección, pero aquí la lista de selección contiene los modelos que se pueden utilizar. En la interfaz de usuario, el usuario primero seleccionará un modelo de la lista disponible av, y luego escogerá un registro de ese modelo.

Esto puede llevarse a otro nivel de flexibilidad: existe una tabla de configuración de **Modelos de Referenciables** que puede ser utilizada en los campos de **Referencia**. Está disponible en el menú **Settings|Technical|database Structure**. Al crear tal campo podemos configurarlo para usar cualquier modelo registrado allí, con la ayuda de la función `referenceable_models (` en el módulo `odoo.addons.res.res_request`.

Utilizando la configuración de **Modelos Referenciables**, una versión mejorada del campo `Refers to` luciría así:


```

from odoo.addons.base.res.res_request import referenceable_models




# class TodoTask(models.Model):

    refers_to = fields.Reference(






        referenceable_models, 'Refers to')





```

Ten en cuenta que en Odoo 9.0 esta función utiliza una ortografía ligeramente diferente y todavía estaba utilizando la API antigua. Así que en la versión 9.0, antes de usar el código mostrado antes, tenemos que añadir algún código en la parte superior de nuestro archivo Python para envolverlo para que utilice la nueva API:

```
from openerp.addons.base.res import res_request
    def referenceable_models(self):
        return res_request.referencable_models( 
            self, self.env.cr, self.env.uid, context=self.env.context)
```

## Campos computados

Los campos pueden tener valores calculados por una función, en lugar de simplemente leer un valor almacenado en una base de datos. Un campo computado se declara igual que un campo regular, pero tiene el argumento de adicional `compute` que define la función utilizada para calcularlo.

En la mayoría de los casos, los campos computados implican escribir alguna lógica de negocio, por lo que desarrollaremos este tema más en el Capítulo 7, *Aplicación Lógica de ORM - Soportando procesos de negocios`. Seguiremos explicándolos aquí, pero mantendremos la lógica de negocios lo más simple posible.

Vamos a trabajar en un ejemplo: Las etapas tienen un campo de plegado `fold`. Vamos a añadir a las Tareas pendientes un campo computado con la etiqueta **Folded?** para la Etapa correspondiente.

Debemos editar el modelo `TodoTask` en el archivo `todo_model.py` para agregar lo siguiente:

```
# class TodoTask(models.Model):
    stage_fold = fields.Boolean(
        'Stage Folded?',
        compute='_compute_stage_fold')

    @api.depends('stage_id.fold')
    def _compute_stage_fold(self):
        for task in self:
            task.stage_fold = task.stage_id.fold
```
El código anterior agrega un nuevo campo `stage_fold` y el método `_compute_stage_fold` utilizado para computarlo. El nombre de la función se pasó como una cadena, pero también se le permite pasar como una referencia llamable (el identificador de función sin comillas). En este caso, debemos asegurarnos de que la función esté definida en el archivo Python antes de que lo sea el campo.

El decorador `@api.depends` es necesario cuando la computación depende de otros campos, como suele ocurrir. Permite al servidor saber cuándo volver a calcular los valores almacenados o en datos caché. Uno o más nombres de campo se aceptan como argumentos y la notación de puntos se puede utilizar para seguir relaciones de campo.

Se espera que la función de computación asigne un valor al campo o a los campos a computar. Si no lo hace, se producirá un error. Dado que `self` es un objeto de registro, nuestro cálculo aquí es simplemente para obtener el campo **Folded?** utilizando `stage_id.fold`. El resultado se logra asignando ese valor (escribiéndolo) al campo computado, `stage_fold`.

No vamos a estar trabajando todavía en las vistas de este módulo, pero puede hacer ahora una edición rápida en el formulario de tarea para confirmar si el campo computado está funcionando como se esperaba: usando el **Developer Mode** selecciona la opción **Edit View** y agregua el archivo directamente en el formato XML. No te preocupes: será reemplazado por la vista de módulo limpio en la próxima actualización.

### Búscando y escribiendo en campos computados

El campo computado que acabamos de crear se puede leer, pero no puede ser buscado o escrito. Para habilitar estas operaciones, primero necesitamos implementar funciones especializadas para ellos. Junto con la función `compute`, también podemos configurar una función `search`, implementando la lógica de búsqueda, y la función `inverse`, implementando la lógica de escritura.

Utilizando estos, nuestra declaración de campo computado se convierte así:

```
# class TodoTask(models.Model):
    stage_fold = fields.Boolean(
        string='Stage Folded?',
        compute='_compute_stage_fold',
        # store=False,  # the default

        search='_search_stage_fold',






        inverse='_write_stage_fold'



)
```

Y las funciones de soporte son:

```
def _search_stage_fold(self, operator, value):
    return [('stage_id.fold', operator, value)]

def _write_stage_fold(self):
    self.stage_id.fold = self.stage_fold
```

La función `search` se llama siempre que una condición (`field, operator, value`) en este campo se encuentra en una expresión de dominio de búsqueda. Recibe al `operator` y el `value` para la búsqueda y se espera que traduzca el elemento de búsqueda original en una expresión de búsqueda de dominio alternativa.

La función `inverse` realiza la lógica inversa del cálculo, para encontrar el valor a escribir en los campos fuente de la computación. En nuestro ejemplo, esto significa escribir de nuevo en el campo `stage_id.fold`.

### Almacenando campos computados

Los valores del campo computado también se pueden almacenar en la base de datos, estableciendo `store = True` en su definición. Serán recomputados cuando cambien cualquiera de sus dependencias. Dado que los valores están ahora almacenados, se pueden buscar como campos regulares y no se necesita una función de búsqueda.
### Campos relacionados

El campo computado que implementamos en la sección anterior sólo copia un valor de un registro relacionado en el propio campo del modelo. Sin embargo, este es un uso común que puede ser manejado automáticamente por Odoo.

El mismo efecto se puede lograr utilizando campos relacionados. Ponen a disposición, directamente en un modelo, los campos que pertenecen a un modelo relacionado, accesible mediante una cadena de punto-notación. Esto los hace utilizables en situaciones donde la notación de punto no se puede usar, como las vistas de formulario de UI.

Para crear un campo relacionado, declaramos un campo del tipo necesario, al igual que con campos computados regulares, pero en lugar de calcular usamos el atributo relacionado con la cadena de campo de notación de puntos para alcanzar el campo deseado.

Las Tareas pendientes se organizan en etapas personalizables y éstas se convierten en estados básicos. Haremos que el valor de estado esté disponible directamente en el modelo de Tarea, de modo que pueda ser usado para alguna lógica del lado del cliente en el próximo capítulo.

De forma similar a `stage_fold`, agregaremos un campo computado en el modelo de tarea, pero esta vez usando el campo relacionado más simple:

```
# class TodoTask(models.Model):
    stage_state = fields.Selection(
        related='stage_id.state',
        string='Stage State')

```

Detrás del escenario, los campos relacionados son sólo campos computados que implementan convenientemente métodos `search` y `inverse`. Esto significa que podemos buscar y escribir en ellos fuera de la caja, sin necesidad de escribir un código adicional.

## Restricciones del modelo

Para reforzar la integridad de los datos, los modelos también admiten dos tipos de restricciones: SQL y Python

Las restricciones de SQL se añaden a la definición de la tabla de la base de datos y son aplicadas directamente por PostgreSQL. Se definen mediante el atributo de clase `_sql_constraints`. Es una lista de tuplas con: el nombre del identificador de restricción; El SQL para la restricción; Y el mensaje de error a utilizar.

Un caso de uso común es agregar restricciones únicas a los modelos. Supongamos que no queremos permitir dos tareas activas con el mismo título:

```
# class TodoTask(models.Model): 
    _sql_constraints = [ 
        ('todo_task_name_uniq', 
         'UNIQUE (name, active)', 
         'Task title must be unique!')] 
```
Las restricciones de Python pueden usar una pieza de código arbitrario para comprobar las condiciones. La función de verificación debe estar decorada con `@api.constraints`, indicando la lista de campos implicados en el chequeo. La validación se activa cuando cualquiera de ellos se modifica y generará una excepción si la condición falla.

Por ejemplo, para validar que un nombre de tarea tiene al menos cinco caracteres, podríamos agregar la siguiente restricción:

```
rom odoo.exceptions import ValidationError
# class TodoTask(models.Model):
    @api.constrains('name')
    def _check_name_size(self):
        for todo in self:
            if len(todo.name) < 5:
                raise ValidationError('Must have 5 
                chars!')
```

## Resumen
Pasamos por una explicación detallada de los modelos y los campos, utilizandolos para ampliar la aplicación de tareas pendientes con etiquetas y etapas en las tareas. Aprendiste cómo definir relaciones entre modelos, incluyendo relaciones jerárquicas entre padres e hijos. Finalmente, vimos ejemplos simples de campos computados y restricciones usando código Python.

En el siguiente capítulo, trabajaremos en la interfaz de usuario para estas características del modelo de backend, haciéndolas disponibles en las vistas utilizadas para interactuar con la aplicación.