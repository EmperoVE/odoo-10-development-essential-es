# Capítulo 3. Herencia - Ampliando aplicaciones existentes.
Una de las características más potentes de Odoo es la capacidad de agregar características sin modificar directamente los objetos subyacentes.

Esto se logra a través de mecanismos de herencia, funcionando como capas de modificación sobre objetos existentes. Estas modificaciones pueden ocurrir en todos los niveles: modelos, vistas y lógica de negocio. En lugar de modificar directamente un módulo existente, creamos un nuevo módulo para agregar las modificaciones deseadas.

En este capítulo, aprenderá cómo escribir tus propios módulos de extensión, lo que te permitirá aprovechar las aplicaciones existentes de núcleo o comunidad. Como un ejemplo relevante, aprenderás a agregar las funciones sociales y de mensajería de Odoo a sus propios módulos.

## Añadiendo capacidades de uso compartido a la aplicación To - Do

Nuestra aplicación de To-Do ahora permite a los usuarios gestionar de forma privada sus propias tareas pendientes. ¿No será genial llevar la aplicación a otro nivel agregando funcionalidades de colaboración y redes sociales? Podremos compartir tareas y discutirlas con otras personas.

Lo haremos con un nuevo módulo para ampliar la aplicación To-Do previamente creada y agregar estas nuevas características utilizando los mecanismos de herencia. Esto es lo que esperamos lograr al final de este capítulo:

![Mytodo](file:img/3-01.jpg)

Este será nuestro plan de trabajo para las extensiones de características que se implementarán:

+ Amplía el modelo de tareas, como el usuario responsable de la tarea
+ Modifica la lógica de negocio para que sólo funcione en las tareas del usuario actual, en lugar de todas las tareas que el usuario pueda ver
+ Amplía las vistas para agregar los campos necesarios a las vistas
+ Añade funciones de redes sociales: una pared de mensajes y los seguidores

Comenzaremos a crear el esqueleto básico de un nuevo módulo `todo_user` junto al módulo `todo_app`. Siguiendo el ejemplo de instalación del Capítulo 1, *Iniciando con desarrollo Odoo*, estamos hospedando nuestros módulos en `~ / odoo-dev / custom-addons /`. Deberíamos añadir allí un nuevo directorio `todo_user` para el módulo, que contenga un archivo vacío `__init__.py`.

Ahora crea `todo_user / __ manifest__.py`, que contenga este código:

```
{  'name': 'Multiuser To-Do', 
   'description': 'Extend the To-Do app to multiuser.', 
   'author': 'Daniel Reis', 
   'depends': ['todo_app'], }
```


No lo hemos hecho aquí, pero incluir las claves de `summary` y claves `category` puede ser importante al publicar módulos en la tienda de aplicaciones en línea de Odoo.
Observa que hemos añadido la dependencia explícita al módulo `todo_app`. Esto es necesario e importante para que el mecanismo de herencia funcione correctamente. Y de ahora en adelante, cuando el módulo `todo_app` se actualice, todos los módulos dependiendo de él, como el módulo `todo_user`, también serán actualizados.

A continuación, instálalo. Debería ser suficiente actualizar la lista de módulos utilizando la opción de menú **Update Apps List** debajo de **Apps**; Busca el nuevo módulo en la lista de **Apps** y haz clic en el botón **Install**. Ten en cuenta que esta vez tendrás que quitar el filtro de aplicaciones predeterminado para ver el nuevo módulo en la lista, ya que no se marca como una aplicación. Para obtener instrucciones más detalladas sobre cómo descubrir e instalar un módulo, consulta el Capítulo 1, *Iniciando con el desarrollo Odoo*.

Ahora, vamos a empezar a añadir nuevas características a la misma.

## Extediendo modelos

Los nuevos modelos se definen a través de las clases de Python. Extenderlos también se hace a través de clases de Python, pero con la ayuda de un mecanismo de herencia específico de Odoo.

Para extender un modelo existente, usamos una clase Python con un atributo `_inherit`. Esto identifica el modelo a ser extendido. La nueva clase hereda todas las características del modelo Odoo padre, y solo necesitamos declarar las modificaciones que queremos introducir.

De hecho, los modelos Odoo existen fuera de nuestro módulo particular de Python, en un registro central. Este registro, se puede acceder desde los métodos del modelo utilizando `self.env [<model name>]`. Por ejemplo, para obtener una referencia al objeto que representa el modelo `res.partner`, escribiríamos `self.env ['res.partner']`.

Para modificar un modelo Odoo, obtenemos una referencia a su clase de registro y luego realizamos cambios en el sitio en él. Esto significa que estas modificaciones también estarán disponibles en todas partes donde se utilice este nuevo modelo.

Durante el arranque del servidor Odoo, el módulo que carga la secuencia es relevante: las modificaciones realizadas por un módulo complementario sólo serán visibles para los módulos complementarios cargados posteriormente. Por lo tanto, es importante que las dependencias del módulo se establezcan correctamente, asegurando que los módulos que proporcionan los modelos que usamos estén incluidos en nuestro árbol de dependencias.

### Agregando campos a un modelo

Vamos a ampliar el modelo `todo.task` para añadir un par de campos a ella: el usuario responsable de la tarea y una fecha límite.

Las pautas de estilo de codificación recomendaron tener un subdirectorio `models /` con un archivo por modelo Odoo. Así que debemos comenzar creando el subdirectorio modelo, haciéndolo Python-importable.

Edita el archivo `todo_user / __ init__.py` para tener este contenido:
```
from .import models
```


Crea `todo_user / models / __ init__.py` con el siguiente código:
```
from . import todo_task
```


La línea anterior le indica a Python que busque un archivo llamado odoo_task.py en el mismo directorio y lo importe. Por lo general, tendrías una línea `from` para cada archivo Python en el directorio:
ahora crea el archivo `todo_user/models/todo_task.py` para extender el modelo original:
```
# -*- coding: utf-8 -*- 
from odoo import models, fields, api 
class TodoTask(models.Model): 
    _inherit = 'todo.task' 
    user_id = fields.Many2one('res.users', 'Responsible') 
    date_deadline = fields.Date('Deadline')
```
El nombre de clase `TodoTask` es local para este archivo de Python y, en general, es irrelevante para otros módulos. El atributo de clase `_inherit` es la clave aquí: le dice a Odoo que esta clase está heredando y modificando así el modelo `todo.task`.

#### Nota
Observa que el atributo `_name` está ausente. No es necesario porque ya está heredado del modelo padre.

Las dos líneas siguientes son declaraciones de campo regulares. El campo `user_id` representa un usuario del modelo de usuarios `res.users`. Es un campo `Many2one`, que es equivalente a una clave extranjera en la jerga de la base de datos. El `date_deadline` es un simple campo de fecha. En el capítulo 5, *Modelos - Estructurando de los datos de aplicación*, explicaremos los tipos de campos disponibles en Odoo con más detalle.

Para que los nuevos campos se agreguen a la tabla de base de datos de soporte del modelo, necesitamos realizar una actualización de módulo. Si todo sale como se esperaba, deberías ver los nuevos campos al inspeccionar el modelo `todo.task` en las opciones de menú **Technical | Database Estructure | Models**.

### Modificando campos existentes

Como puedes ver, agregar nuevos campos a un modelo existente es bastante sencillo. Desde Odoo 8, también es posible modificar los atributos en los campos heredados existentes. Se hace agregando un campo con el mismo nombre y estableciendo valores sólo para los atributos que se van a cambiar.

Por ejemplo, para agregar una herramienta de ayuda al campo de nombre, agregamos esta línea a `todo_ task.py`, descrito anteriormente:

```
name = fields.Char(help="What needs to be done?") 
```

Esto modifica el campo con los atributos especificados, dejando sin modificar todos los otros atributos que no se utilizan explícitamente aquí. Si actualizamos el módulo, ve a un formulario de tareas pendientes y haz una pausa en el cursor sobre el campo **Description**; Se mostrará el texto de la herramienta de información.

### Modificando los métodos del modelo

La herencia también funciona en el nivel de lógica empresarial. Agregar nuevos métodos es simple: solo declara sus funciones dentro de la clase de herencia.

Para extender o cambiar la lógica existente, el método correspondiente puede anularse declarando un método con el mismo nombre. El nuevo método reemplazará al anterior, y también puede extender el código de la clase heredada, utilizando el método `super ()` de Python para llamar al método padre. Puede entonces agregar nueva lógica alrededor de la lógica original antes y después de que el método `super ()` sea llamado.

#### Tip

Es mejor evitar cambiar la firma de la función del método (es decir, mantener los mismos argumentos) para asegurarse de que las llamadas existentes en él seguirán funcionando correctamente. En caso de que necesites agregar parámetros adicionales, haz de ellos argumentos de palabras clave opcionales (con un valor predeterminado).

La acción original **Clear All Done** no es apropiada para nuestro módulo de intercambio de tareas ya que borra todas las tareas, independientemente de su usuario. Tenemos que modificarlo para que borre sólo las tareas del usuario actual.

Para ello, anularemos (o reemplazaremos) el método original por una nueva versión que primero encuentre la lista de tareas completadas para el usuario actual y luego las inactive:
```
@api.multi 
def do_clear_done(self): 
    domain = [('is_done', '=', True), 
               '|', ('user_id', '=', self.env.uid), 
                    ('user_id', '=', False)] 
    dones = self.search(domain) 
    dones.write({'active': False}) 
    return True 
```



Para mayor claridad, primero construimos la expresión de filtro que se utilizará para encontrar los registros que se van a borrar.

Esta expresión de filtro sigue una sintaxis específica de Odoo denominada `domain`: que es una lista de condiciones, donde cada condición es una tupla.

Estas condiciones se unen implícitamente con el operador AND (&). Para la operación OR, una tubería, | , se utiliza en el lugar de una tupla, y se une a las dos condiciones siguientes. Vamos a entrar en más detalles acerca de los dominios en el Capítulo 6, *Vistas - Diseñando de la interfaz de usuario*.

El dominio utilizado aquí filtra todas las tareas hechas `('is_done', '=', True)` que tienen el usuario actual como responsable ('user_id', '=', self.env.uid) o no tienen un set de usuario actual `('user_id', '=', False)`.

A continuación, utilizamos el método de búsqueda para obtener un conjunto de registros con los registros hechos para actuar y, finalmente, hacer una escritura masiva en ellos estableciendo el campo activo a `False`. El valor Python `False` aquí representa el valor `NULL` de la base de datos.

En este caso, hemos sobrescrito completamente el método padre, reemplazándolo con una nueva implementación, pero eso no es lo que normalmente queremos hacer. En su lugar, debemos ampliar la lógica existente con algunas operaciones adicionales. De lo contrario, podríamos romper las características ya existentes.

Para que el método principal mantenga la lógica ya existente, usamos la construcción `super ()` de Python para llamar a la versión del método padre. Veamos un ejemplo de esto.

Podemos mejorar el método `do_toggle_done ()` para que sólo realice su acción en las tareas asignadas al usuario actual. Este es el código para lograrlo:
```
from odoo.exceptions import ValidationError
# ...
# class TodoTask(models.Model):
# ...
@api.multi 
def do_toggle_done(self): 
    for task in self: 
        if task.user_id != self.env.user: 
            raise ValidationError(
                'Only the responsible can do this!') 
    return super(TodoTask, self).do_toggle_done()

```

El método en la clase heredada comienza con un bucle `for` para comprobar que ninguna de las tareas a activar pertenece a otro usuario. Si estos chequeos pasan, entonces continúa llamando al método de clase padre, usando `super ()`. Si no se plantea un error, y debemos utilizar para ello las excepciones incorporadas de Odoo. Los más relevantes son `ValidationError`, utilizado aquí y `UserError`.

Estas son las técnicas básicas para reemplazar y extender la lógica empresarial definida en las clases de modelo. A continuación, veremos cómo ampliar las vistas de la interfaz de usuario.

## Extendiendo vistas
Las formas, listas y vistas de búsqueda se definen utilizando las estructuras de `arch XML`. Para ampliar vistas, necesitamos una forma de modificar este XML. Esto significa localizar elementos XML y luego introducir modificaciones en esos puntos.

Las vistas heredadas permiten justamente eso. Una declaración de vista heredada tiene este aspecto:
```
<record id="view_form_todo_task_inherited"   
  model="ir.ui.view"> 
  <field name="name">Todo Task form - User 
    extension</field> 
  <field name="model">todo.task</field> 
  <field name="inherit_id" 
    ref="todo_app.view_form_todo_task"/> 
  <field name="arch" type="xml"> 
    <!-- ...match and extend elements here! ... --> 
  </field 
</record> 

```
El campo `inherit_id` identifica la vista que debe extenderse consultando su identificador externo usando el atributo especial `ref`. Los identificadores externos se tratarán con más detalle en el Capítulo 4, *Datos del módulo*.

Siendo XML, la mejor manera de localizar elementos en XML es usar expresiones XPath. Por ejemplo, tomando la vista de formulario definida en el capítulo anterior, una expresión XPath para localizar el elemento `<field name = "is_done">` es `// field [@name] = 'is_done'`. Esta expresión encuentra cualquier elemento `field` con un atributo `name` igual a `is_done`. Puedes encontrar más información sobre XPath en https://docs.python.org/2/library/xml.etree.elementtree.html#xpath-support.

Si una expresión XPath coincide con varios elementos, sólo se modificará la primera. Por lo tanto, deben ser hechos lo más específico posible, utilizando atributos únicos. El uso del atributo `name` es la forma más fácil de asegurar que encontramos los elementos exactos que queremos utilizar un punto de extensión. Por lo tanto, es importante configurarlos en nuestros elementos de vista XML.

Una vez localizado el punto de extensión, puedes modificarlo o tener elementos XML añadidos cerca de él. Como un ejemplo práctico, para agregar el campo `date_deadline` antes del campo `is_done`, escribiremos lo siguiente en `arch`:
```
<xpath expr="//field[@name]='is_done'" position="before"> 
  <field name="date_deadline" /> 
</xpath>
```
Afortunadamente, Odoo proporciona la notación de acceso directo para esto, así que la mayoría de las veces podemos evitar la sintaxis XPath por completo. En lugar del elemento XPath anterior, podemos utilizar sólo la información relacionada con el tipo de tipo de elemento para localizar y sus atributos distintivos, y en lugar de la anterior XPath, escribimos esto:
```
<field name="is_done" position="before"> 
  <field name="date_deadline" /> 
</field>
```
Sólo ten en cuenta que si el campo aparece más de una vez en la misma vista, siempre debe utilizar la sintaxis XPath. Esto es porque Odoo se detendrá en la primera aparición del campo y puede aplicar sus cambios al campo incorrecto.

A menudo, queremos agregar nuevos campos junto a los existentes, por lo que la etiqueta `<field>` se utilizará como localizador con frecuencia. Pero cualquier otra etiqueta se puede utilizar: `<sheet>, <group>, <div>`, etc. El atributo `name` suele ser la mejor opción para los elementos coincidentes, pero a veces, es posible que necesitemos utilizar otra cosa: el elemento de `CSS class`, por ejemplo. Odoo encontrará el primer elemento que tiene al menos todos los atributos especificados.

#### Nota

Antes de la versión 9.0, el atributo `string` (para la etiqueta de texto que se muestra) también podría utilizarse como localizador de extensión. Desde 9.0, esto ya no está permitido. Esta limitación está relacionada con el mecanismo de traducción del lenguaje que opera en esas cadenas.

El atributo `position` utilizado con el elemento localizador es opcional y puede tener los siguientes valores:

+ `after` agrega el contenido al elemento padre, después del nodo coincidente.
+ `before` añade el contenido, antes del nodo coincidente.
+ `inside` (valor predeterminado) agrega el contenido dentro del nodo compatible.
+ `replace` reemplaza el nodo coincidente. Si se utiliza con contenido vacío, elimina un elemento. Puesto que Odoo 10 también permite que se envuelva un elemento con otro marcado, usando $ 0 en el contenido para representar el elemento que se está reemplazando.
+ `attributes` modifica los atributos XML del elemento emparejado. Esto se hace utilizando en los elementos del contenido `<attribute name = "attr-name">` con los nuevos valores de atributo que se deben establecer.

Por ejemplo, en el formulario Tarea, tenemos el campo activo, pero tenerlo visible no es tan útil. Podríamos ocultarlo del usuario. Esto puede hacerse estableciendo su atributo `invisible`:
```
<field name="active" position="attributes"> 
  <attribute name="invisible">1</attribute> 
</field>
```
Establecer el atributo `invisible` para ocultar un elemento es una buena alternativa para usar el localizador `replace` para eliminar nodos. Se debe evitar la eliminación de nodos, ya que puede romper módulos dependientes que pueden depender del nodo eliminado como marcador de posición para agregar otros elementos.

#### Ampliando de la vista de formulario

Juntando todos los elementos de formulario anteriores, podemos agregar los nuevos campos y ocultar el campo `active`. La vista de herencia completa para ampliar el formulario de tareas pendientes es la siguiente:
```
<record id="view_form_todo_task_inherited"   
  model="ir.ui.view"> 
  <field name="name">Todo Task form - User 
    extension</field> 
  <field name="model">todo.task</field> 
  <field name="inherit_id"   
    ref="todo_app.view_form_todo_task"/> 
  
<field name="arch" type="xml"> 
    <field name="name" position="after">
      <field name="user_id">
    </field> 
    <field name="is_done" position="before"> 
      <field name="date_deadline" /> 
    </field> 
    <field name="active" position="attributes"> 
      <attribute name="invisible">1</attribute> 
    </field> 
  </field>



 
</record> 
```
Esto debe agregarse a un archivo `views / todo_task.xml` en nuestro módulo, dentro del elemento `<odoo>`, como se muestra en el capítulo anterior.

#### Nota

Las vistas heredadas también se pueden heredar, pero como esto crea dependencias más intrincadas, debe evitarse. Debes preferir heredar de la vista original siempre que sea posible.

Además, no debemos olvidar añadir el atributo `data` al archivo descriptor `__manifest__.py`:
```
'data': ['views/todo_task.xml'],
```
### Ampliandola vista de árbol y búsqueda

Las extensiones de vista de árbol y búsqueda también se definen utilizando la estructura XML de `arch`, y pueden extenderse de la misma forma que las vistas de formulario. Continuaremos nuestro ejemplo ampliando las vistas de lista y de búsqueda.

Para la vista de lista, queremos añadirle el campo de usuario:
```
<record id="view_tree_todo_task_inherited"   
  model="ir.ui.view"> 
  <field name="name">Todo Task tree - User 
  extension</field> 
  <field name="model">todo.task</field> 
  
<field name="inherit_id"   





    ref="todo_app.view_tree_todo_task"/>



 
  <field name="arch" type="xml"> 
   
 <field name="name" position="after"> 
      <field name="user_id" /> 
    </field>



 
  </field 
</record> 
```
Para la vista de búsqueda, agregamos la búsqueda por el usuario y filtros predefinidos para las propias tareas del usuario y las tareas no asignadas a nadie:
```
<record id="view_filter_todo_task_inherited"    
  model="ir.ui.view"> 
  <field name="name">Todo Task tree - User 
    extension</field> 
  <field name="model">todo.task</field> 
 
 <field name="inherit_id" 





  ref="todo_app.view_filter_todo_task"/>



 
  <field name="arch" type="xml"> 
    
<field name="name" position="after"> 
      <field name="user_id" /> 
      <filter name="filter_my_tasks" string="My Tasks" 
        domain="[('user_id','in',[uid,False])]" /> 
      <filter name="filter_not_assigned" string="Not 





        Assigned" domain="[('user_id','=',False)]" /> 
    </field>



 
  </field 
</record> 

```

No se preocupe demasiado por la sintaxis específica de estas vistas. Los cubriremos con más detalle en el Capítulo 6, *Vistas - Diseñando la interfaz de usuario*.
 
## Más modelos de mecanismos de herencia

Hemos visto la extensión básica de los modelos, llamada *herencia de clase* en la documentación oficial. Este es el uso más frecuente de la herencia, y es más fácil pensar en ello como una *extensión in situ*. Tu tomas un modelo y lo extiendes. A medida que agregas nuevas funciones, se agregan al modelo existente. No se crea un nuevo modelo. También podemos heredar de varios modelos padre, estableciendo una lista de valores para el atributo `_inherit`. Con esto, podemos hacer uso de *clases de mixin*. Las clases de Mixin son modelos que implementan características genéricas que podemos agregar a otros modelos. No se espera que se usen directamente, y son como un contenedor de características listas para ser agregadas a otros modelos.

Si también usamos el atributo `_name` con un valor diferente del modelo padre, obtenemos un nuevo modelo que reutiliza las características del heredado pero con su propia tabla de base de datos y datos. La documentación oficial llama a este *prototipo de herencia*. Aquí tu tomas un modelo y creas uno nuevo que es una copia de la vieja. A medida que agregas nuevas funciones, se agregan al nuevo modelo. El modelo existente no se modifica.

También existe el método de *delegación de herencia*, utilizando el atributo `_inherits`. Este permite que un modelo contenga otros modelos de manera transparente para el observador mientras que, detrás de escenas, cada modelo maneja sus propios datos. Tu tomas un modelo y lo extiendes. A medida que agrega nuevas funciones, se agregan al nuevo modelo. El módulo existente no se cambia. Los registros del nuevo modelo tienen un enlace a un registro en el modelo original y los campos del modelo original están expuestos y se pueden usar directamente en el nuevo modelo.

Exploremos estas posibilidades con más detalle.

### Copiando características con herencia de prototipo

El método que usamos antes para extender un modelo utiliza sólo el atributo `_inherit`. Definimos una clase heredando el modelo `todo.task` y le agregamos algunas características. El atributo de clase `_name` no se estableció explícitamente; Implícitamente, era `todo.task`.

Sin embargo, el uso del atributo `_name` nos permite crear un nuevo modelo copiando las características de las heredadas. Aquí hay un ejemplo:
```
from odoo import models 
class TodoTask(models.Model): 
    _name = 'todo.task' 
    _inherit = 'mail.thread'
```

Esto amplía el modelo `todo.task` copiando en él las características del modelo `mail.thread`. El modelo `mail.thread` implementa las características de mensajes y seguidores de Odoo y es reutilizable para que sea fácil agregar esas características a cualquier modelo.

Copiar significa que los métodos y campos heredados también estarán disponibles en el modelo de herencia. Para los campos, esto significa que también se crearán y almacenarán en las tablas de la base de datos del modelo de destino. Los registros de datos de los modelos originales (heredados) y los nuevos (hereditarios) se mantienen sin relación. Sólo se comparten las definiciones.

En un momento, vamos a discutir en detalle cómo usar esto para agregar `mail.thread` y sus características de redes sociales a nuestro módulo. En la práctica, al usar mixins, rara vez heredamos de modelos regulares porque esto causa duplicación de las mismas estructuras de datos.

Odoo también proporciona el mecanismo de herencia de delegación que evita la duplicación de estructura de datos, por lo que suele ser preferido cuando se hereda de modelos regulares. Miremos esto con más detalle.

### Incorporando modelos mediante la herencia de delegación

La herencia de delegación se utiliza menos frecuentemente, pero puede proporcionar soluciones muy convenientes. Se utiliza a través del atributo `_inherits` (nota la `s` adicional) con mapeo de diccionario modelos heredados con campos que se enlazan a ellos.

Un buen ejemplo de esto es el modelo de usuario estándar, `res.users`; Tiene un modelo de Socio incrustado en él:
```
from odoo import models, fields 
class User(models.Model): 
    _name = 'res.users' 
    _inherits = {'res.partner': 'partner_id'} 
    partner_id = fields.Many2one('res.partner') 
```
Con la herencia de delegación, el modelo `res.users` incrusta el modelo heredado `res.partner` de forma que cuando se crea una nueva clase `User`, también se crea un socio y se mantiene una referencia en el campo `partner_id` de la clase `User`. Tiene algunas similitudes con el concepto de polimorfismo en la programación orientada a objetos.

A través del mecanismo de delegación, todos los campos del modelo heredado y del Socio están disponibles como si fueran campos de `user`. Por ejemplo, los campos Nombre y dirección del asociado se exponen como campos de usuario, pero de hecho, se almacenan en el modelo Socio asociado y no se produce duplicación de datos.

La ventaja de esto, en comparación con la herencia de prototipo, es que no hay necesidad de repetir estructuras de datos, como direcciones, a través de varias tablas. Cualquier modelo nuevo que necesite incluir una dirección puede delegarla a un modelo de socio incrustado. Y si las modificaciones se introducen en los campos de la dirección del socio, ¡éstas están inmediatamente disponibles para todos los modelos que lo incrustan!

#### Nota

Tenga en cuenta que con la herencia de delegación, los campos se heredan, pero los métodos no.

### Agregando las funciones de redes sociales

El módulo de red social (nombre técnico `mail`) proporciona el panel de mensajes que se encuentra en la parte inferior de muchos formularios y la función **Seguidores**, así como la lógica de los mensajes y las notificaciones. Esto es algo que a menudo queremos añadir a nuestros modelos, así que vamos a aprender cómo hacerlo.

Las características de mensajería de redes sociales son proporcionadas por el modelo `mail.thread` del módulo de `mail`. Para agregarlo a un modelo personalizado, debemos hacer lo siguiente:

+ Tener un módulo que dependa de `mail`
+ Tener la clase heredada de `mail.thread`
+ Tener añadidos los seguidores y widgets de subproceso a la vista de formulario
+ Opcionalmente, necesitamos establecer reglas de registro para los seguidores.

Sigamos la siguiente lista de verificación.

En cuanto al primer punto, nuestro módulo de extensión necesitará la dependencia adicional de `mail` en el módulo `__manifest__.py manifest file`:
```
'depends': ['todo_app', 'mail'],
```
Con respecto al segundo punto, la herencia en `mail.thread` se hace usando el atributo `_inherit` que utilizamos antes. Pero nuestra extensión de tareas pendientes ya está usando el atributo `_inherit`. Afortunadamente, puede aceptar una lista de modelos de los que heredar, por lo que podemos usar esto para que también incluya la herencia en `mail.thread`:
```
_name = 'todo.task' 
_inherit = ['todo.task', 'mail.thread']
```
`mail.thread` es un modelo abstracto. Los **modelos abstractos** son como modelos regulares, excepto que no tienen una representación de base de datos; no se crean tablas reales para ellos. Los modelos abstractos no están destinados a ser utilizados directamente. En su lugar, se espera que se utilizan como clases mixin, como acabamos de hacer. Podemos pensar en ellos como plantillas con características listas para usar. Para crear una clase abstracta, solo la necesitamos para usar models.AbstractModel en lugar de models.Model para la clase que los define.

Para el tercer punto, queremos agregar los widgets de red social en la parte inferior del formulario. Esto se hace extendiendo la definición de vista de formulario. Podemos reutilizar la vista heredada que ya hemos creado, `view_form_todo_task_inherited`, y añadirla a sus datos `arch`:
```
<sheet position="after"> 
  <div class="oe_chatter"> 
    <field name="message_follower_ids" 
      widget="mail_followers" /> 
    <field name="message_ids" widget="mail_thread" /> 
  </div> 
</sheet> 
```

Los dos campos agregados aquí no han sido declarados explícitamente por nosotros, pero son proporcionados por el modelo `mail.thread`.

El paso final, que es el paso cuatro, consiste en establecer reglas de registro para seguidores: control de acceso a nivel de fila. Esto sólo es necesario si nuestro modelo es requerido para limitar el acceso de otros usuarios a los registros. En este caso, queremos que cada registro de tarea también sea visible para cualquiera de sus seguidores.

Ya tenemos un Registro de Reglas definido en el modelo de tareas pendientes, por lo que debemos modificarlo para agregar este nuevo requisito. Esa es una de las cosas que haremos en la próxima sección.
##Modificando los datos
A diferencia de las vistas, los registros de datos regulares no tienen una estructura `XML arch` y no se pueden ampliar utilizando expresiones XPath. Pero todavía se pueden modificar modificando los valores en sus campos.

Los elementos de carga de datos `<record id = "x" model = "y">` realizan realmente una operación `insert` o `update` en el modelo `y`: si el modelo `x` no existe, se crea; De lo contrario, se actualiza / sobre escribe.

Dado que se puede acceder a los registros de otros módulos con un identificador global `<model>. <Identifier>`, es posible que nuestro módulo sobrescriba algo que fue escrito antes por otro módulo.

#### Nota

Ten en cuenta que dado que el punto está reservado para separar el nombre del módulo del identificador de objeto, no se puede utilizar en los nombres de identificador. En su lugar, utiliza la opción subrayado.

### Modificando el menú y acciones de registro
`
Como ejemplo, cambiemos la opción de menú creada por el módulo `todo_app` a `My To-Do`. Para ello, podemos agregar lo siguiente al archivo `todo_user / views / todo_task.xml`:
```
<!-- Modify menu item --> 
<record id="todo_app.menu_todo_task" model="ir.ui.menu"> 
  <field name="name">My To-Do</field> 
</record>
```
También podemos modificar la acción utilizada en el ítem menú. Las acciones tienen un atributo contextual opcional. Puede proporcionar valores predeterminados para campos de vista y filtros. Lo utilizaremos para activar de forma predeterminada el filtro **Mis tareas**, definido anteriormente en este capítulo:
```
<!-- Action to open To-Do Task list --> 
<record model="ir.actions.act_window"
  id="todo_app.action_todo_task"> 
  <field name="context"> 
    {'search_default_filter_my_tasks': True} 
  </field> 
</record> 
```

### Modificando las reglas de registro de seguridad

La aplicación Tareas incluye una regla de registro para garantizar que cada tarea sólo fuera visible para el usuario que la creó. Pero ahora, con la adición de características sociales, necesitamos que los seguidores de las tareas también tengan acceso a ellas. El módulo de red social no maneja esto por sí mismo.

Además, ahora las tareas pueden tener usuarios asignados a ellos, por lo que tiene más sentido tener las reglas de acceso para trabajar en el usuario responsable en lugar del usuario que creó la tarea.

El plan sería el mismo que lo hicimos para el elemento del ítem menú: sobreescribir `todo_app.todo_task_user_rule` para modificar el campo `domain_force` a un nuevo valor.




La idea es mantener los archivos de seguriad relacionados en un subdirectorio `segurity`, por lo que crearemos un archivo `security / todo_access_rules.xml` con el siguiente contenido:
```
<?xml version="1.0" encoding="utf-8"?> 
<odoo> 
  <data noupdate="1"> 
    <record id="todo_app.todo_task_per_user_rule"    
      model="ir.rule"> 
      <field name="name">ToDo Tasks for owner and 
        followers</field> 
      <field name="model_id" ref="model_todo_task"/> 
      <field name="groups" eval="[(4, 
        ref('base.group_user'))]"/> 
      <field name="domain_force"> 
        ['|',('user_id','in', [user.id,False]), 
        ('message_follower_ids','in',
        [user.partner_id.id])] 
      </field> 
    </record> 
  </data> 
</odoo> 
```

Esto sobrescribe la regla de registro `todo_task_per_user_rule` del módulo `todo_app`. El nuevo filtro de dominio ahora hace que una tarea sea visible para el usuario responsable, `user_id` o para todos si el usuario responsable no está establecido (equivale a `False`); Es visible para todos los seguidores de la tarea también.

La regla de registro se ejecuta en un contexto en el que una variable `user` está disponible y representa el registro para el usuario de sesión actual. Dado que los seguidores son socios, no `users`, en lugar de `user.id`, necesitamos usar `user.partner_id`.

El campo de grupos es una relación de muchos. La edición de datos en estos campos utiliza una notación especial. El código `4` utilizado aquí es para añadir a la lista de registros relacionados. También se utiliza con frecuencia el código `6`, para reemplazar completamente los registros relacionados con una nueva lista. Podremos analizar esta notación con mayor detalle en el Capítulo 4, *Datos del Módulo*.

El atributo `noupdate = "1"` del elemento de registro significa que estos datos de registro sólo se escribirán en las acciones de instalación y se ignorarán en las actualizaciones del módulo. Esto permite su personalización, sin asumir los riesgos de sobrescribir personalizaciones y perderlas al hacer una actualización de módulo en algún momento en el futuro.
####Tip
Trabajar en archivos de datos con `<data noupdate ="1">` en el momento del desarrollo puede ser complicado porque las ediciones posteriores de la definición XML serán ignoradas en las actualizaciones de módulos. Para evitar esto, puedes volver a instalar el módulo. Esto es más fácil hecho a través de la línea de comandos usando el `-i`

Como de costumbre, no debemos olvidar agregar el nuevo archivo al atributo de datos en el `__manifest__.py`:
```
'data': ['views/todo_task.xml', 'security/todo_access_rules.xml'], 
```

# Resumen

Ahora deberías ser capaz de crear tus propios módulos extendiendo los existentes.

Para demostrar cómo hacerlo, ampliamos el módulo Tareas que creamos en el capítulo anterior, añadiendo nuevas características a las varias capas que componen una aplicación.

Hemos ampliado un modelo Odoo para agregar nuevos campos y ampliar sus métodos de lógica de negocio. A continuación, modificamos las vistas para poner a su disposición los nuevos campos. Finalmente, aprendimos a heredar características de otros modelos y usarlas para agregar las funciones de red social a la aplicación de tareas pendientes.

Los tres primeros capítulos nos dieron una visión general de las actividades comunes involucradas en el desarrollo Odoo, desde la instalación y configuración de Odoo hasta la creación y extensión de módulos.

Los próximos capítulos se centrarán en un área específica del desarrollo Odoo, la mayoría de los cuales visitamos brevemente en estos primeros capítulos. En el capítulo siguiente, abordaremos la serialización de datos y el uso de archivos de datos XML y CSV con más detalle.



