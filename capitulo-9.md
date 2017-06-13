# Capítulo 9. Vistas QWeb y Kanban

**QWeb** es un motor de plantilla utilizado por Odoo. Se basa en XML y se utiliza para generar fragmentos y páginas HTML. QWeb fue introducido por primera vez en la versión 7.0 para permitir vistas de kanban más ricas y, desde la versión 8.0, también se utiliza para el diseño de informes y páginas web de CMS.

Aquí aprenderás sobre la sintaxis de QWeb y cómo usarla para crear sus propias vistas kanban e informes personalizados. Comencemos aprendiendo más sobre los tableros kanban.

## Acerca de tableros kanban

**Kanban** es una palabra japonesa utilizada para representar un método de gestión de colas de trabajo. Se inspira en el Sistema de Producción Toyota y en la Fabricación Lean. Se ha convertido en popular en la industria del software con la adopción de metodologías Agile.

El tablero kanban **kanban board** es una herramienta para visualizar la cola de trabajo. La junta está organizada en columnas que representan las etapas **stages** del proceso de trabajo. Los objetos de trabajo se representan mediante tarjetas **cards** colocadas en la columna apropiada del tablero. Los nuevos elementos de trabajo empiezan desde la columna más a la izquierda y viajan a través de la tabla hasta llegar a la columna más a la derecha, representando el trabajo terminado.

La simplicidad y el impacto visual de los tableros kanban los hacen excelentes para soportar procesos empresariales simples. Un ejemplo básico de un tablero kanban puede tener tres columnas, como se muestra en la siguiente imagen: **Por Hacer "To Do"**, **Haciendo "Doing"**, and **Hecho "Done"**.

Por supuesto, puede ampliarse a cualquier proceso específico que podamos necesitar:

![kanban](file:img/9-01.jpg)

Créditos de la foto: "A Scrum board suggesting to use kanban" por Jeff.lasovski. Cortesía de Wikipedia.

### Vistas Kanban

Para muchos casos de uso empresarial, un tablero kanban puede ser una forma más eficaz de administrar el proceso correspondiente que el motor de flujo de trabajo normalmente más pesado. Odoo admite las vistas de tablas kanban, junto con la lista clásica y vistas de formulario. Esto facilita la implementación de este tipo de vista. Vamos a aprender cómo usarlos.

En las vistas de formulario, usamos elementos XML específicos, como `<field>` y `<group>`, y pocos elementos HTML, como `<h1>` o `<div>`. Con las vistas kanban, es todo lo contrario; Son plantillas basadas en HTML y admiten sólo dos elementos específicos de Odoo,`<field>` y `<button>`.

El HTML se genera dinámicamente con las plantillas de QWeb. El motor QWeb procesa etiquetas y atributos XML especiales para producir el HTML final que se presentará en el cliente web. Esto aporta mucho control sobre cómo procesar el contenido, pero también hace que el diseño de la vista sea más complejo.

El diseño de la vista kanban es bastante flexible, por lo que haremos todo lo posible para prescribir una manera sencilla de crear rápidamente sus vistas kanban. Un buen enfoque es encontrar una vista kanban existente similar a lo que usted necesita, e inspeccionarla para ideas sobre cómo construir la suya.

Podemos ver dos formas diferentes de usar las vistas de kanban. Una es una lista de tarjetas. Se utiliza en lugares como contactos, productos, directorios de empleados o aplicaciones.

Así es como se ve la vista de kanban de **Contactos "Contacts"**:

![kanban_contacts](file:img/9-02.jpg)

Pero esto no es un tablero kanban verdadero. Se espera que un tablero kanban tenga las tarjetas organizadas en columnas y, por supuesto, la vista kanban también apoya ese diseño. Podemos ver ejemplos en **Sales | My Pipeline** o en **Tareas de Proyecto "Project Tak"**.

Aquí está como se ve ** Sales | My Pipeline**:

![sales_my_pipeline](file:img/9-03.jpg)

La diferencia más llamativa entre los dos es la organización de las tarjetas en columna del tablero de kanban. Esto se logra mediante la función **Agrupar por "Group By"**, similar a lo que proporcionan las vistas de lista. Por lo general, el agrupamiento se realiza en un campo **Etapa "Stage"**. Una característica muy útil de las vistas de kanban es que soporta arrastrar y soltar tarjetas entre columnas, asignando automáticamente el valor correspondiente al campo que agrupa la vista.

Mirando las tarjetas en ambos ejemplos, podemos ver algunas diferencias. De hecho, su diseño es bastante flexible, y no hay una sola manera de diseñar una tarjeta kanban. Pero estos dos ejemplos pueden proporcionar un punto de partida para sus diseños.

Las tarjetas de **Contacto "Contact"** básicamente tienen una imagen en el lado izquierdo, y un título en negrita en el área principal, seguido de una lista de valores. Las tarjetas **My Pipeline** tienen un poco más de estructura. El área de la tarjeta principal también tiene un título seguido por una lista de información relevante, así como un área de pie de página, en este caso con un widget de prioridad en el lado izquierdo, y el usuario responsable en el lado derecho. No es visible en la imagen, pero las tarjetas también tienen un menú de opciones en la parte superior derecha, que se muestra al pasar el puntero del ratón sobre él. Este menú permite, por ejemplo, cambiar el color de fondo de la tarjeta.

Vamos a utilizar esta estructura más elaborada como un modelo para las tarjetas en nuestro tablero kanban de tareas pendientes.

## Diseñando vistas kanban

Vamos a agregar la vista kanban a las tareas pendientes con un nuevo módulo addon. Sería más sencillo añadirlo directamente al módulo `todo_ui`. Sin embargo, para una explicación más clara, usaremos un nuevo módulo y evitaremos muchos cambios, posiblemente confusos, en los archivos ya creados.

Nombraremos este nuevo módulo addon como `todo_kanban` y crearemos los archivos iniciales habituales. Edite el archivo descriptor `todo_kanban/__manifest__.py` como sigue:

```
{'name': 'To-Do Kanban', 
 'description': 'Kanban board for to-do tasks.', 
 'author': 'Daniel Reis', 
 'depends': ['todo_ui'], 
 'data': ['views/todo_view.xml'] } 

```
También agregue un archivo vacío `todo_kanban/__ init__.py`, para que el directorio Python pueda ser importado, como se requiere para los módulos addon de Odoo.

A continuación, crea el archivo XML en el que vaya nuestra nueva vista de kanban brillante y establece kanban como la vista predeterminada en la acción de la ventana de la tarea pendiente. Esto debe ser en `todo_kanban/views/todo_view.xml`, que contiene el siguiente código:

```
<?xml version="1.0"?> 
<odoo> 
  <!-- Add Kanban view mode to the menu Action: -->   
  <act_window id="todo_app.action_todo_task" name="To-Do Tasks"                                        
    res_model="todo.task" view_mode="kanban,tree,form,calendar,graph,pivot"        
    context="{'search_default_filter_my_tasks': True}" /> 
  <!-- Add Kanban view --> 
  <record id="To-do Task Kanban" model="ir.ui.view"> 
    <field name="model">todo.task</field> 
    <field name="arch" type="xml"> 
      <kanban>
        
<!-- Empty for now, but the Kanban will go here! -->



 
      </kanban> 
    </field> 
  </record> 
</odoo> 

```
Ahora tenemos el esqueleto básico para nuestro módulo en su lugar.

Antes de comenzar con las vistas kanban, necesitamos agregar un par de campos al modelo de tareas pendientes.

### Prioridad, estado kanban y color

Aparte de las etapas, algunos campos más son útiles y se utilizan con frecuencia en las juntas kanban.

+ `priority` permite a los usuarios organizar sus elementos de trabajo, señalando lo que debe abordarse en primer lugar.
+ `kanban_state` indica si una tarea está lista para pasar a la siguiente etapa o si está bloqueada por alguna razón. En la capa de definición del modelo, ambos son campos de selección. En la capa de vista, tienen widgets específicos para ellos que se pueden utilizar en forma y vistas kanban.
+ `color` se utiliza para almacenar el color que la tarjeta kanban debe mostrar, y se puede establecer mediante un menú selector de color disponible en las vistas kanban.

Para agregar estos campos a nuestro modelo, agregamos un archivo `models/todo_task_model.py`.

Pero primero, tendremos que hacerlo importable y editar el archivo `todo_kanban/__init__.py` para importar el subdirectorio de modelos:

```
from . import models 
```

A continuación, crea el archivo `models/__init__.py` con:

```
from . import todo_task 
```

Ahora vamos a editar el archivo `models/todo_task.py`:

```
from odoo import models, fields 
class TodoTask(models.Model): 
    _inherit = 'todo.task' 
    color = fields.Integer('Color Index') 
    priority = fields.Selection( 
        [('0', 'Low'),  
         ('1', 'Normal'),  
         ('2', 'High')], 
        'Priority', default='1') 
    kanban_state = fields.Selection( 
        [('normal', 'In Progress'), 
         ('blocked', 'Blocked'), 
         ('done', 'Ready for next stage')], 
        'Kanban State', default='normal') 
```

Ahora podemos trabajar en la vista kanban.

### Elementos de tarjeta Kanban

La arquitectura de vista kanban tiene un elemento superior `<kanban>` y la siguiente estructura básica:

```
<kanban default_group_by="stage_id" class="o_kanban_small_column" > 
  
<!-- Fields to use in expressions... -->



 
  <field name="stage_id" /> 
  <field name="color" /> 
  <field name="kanban_state" /> 
  <field name="priority" /> 
  <field name="is_done" /> 
  <field name="message_partner_ids" />
  
<!-- (...add other used fields). -->




  <templates>
    <t t-name="kanban-box">
      
<!-- HTML QWeb template... -->




    </t>
  </templates>
</kanban>


```

Observa el atributo `default_group_by="stage_id"` utilizado en el elemento `<kanban>`. Lo usamos para que, por defecto, las tarjetas kanban se agrupen por etapas como deberían los tableros kanban. En los kanbans de listas de tarjetas simples, como el de **Contactos "Contacts"**, no necesitamos esto y en su lugar simplemente usaríamos una sencilla etiqueta de apertura `<kanban>`.

El elemento superior `<kanban>` admite algunos atributos interesantes:

+ `default_group_by` establece el campo a utilizar para los grupos de columnas predeterminados.
+ `default_order` establece un orden por defecto para usar para los elementos kanban.
+ `quick_create="false"` desactiva la opción de creación rápida (el signo *más* grande), disponible en la parte superior de cada columna para crear nuevos elementos proporcionando sólo una descripción de título. El valor false es un literal de JavaScript, y debe estar en minúsculas.
+ `class` añade una clase CSS al elemento raíz de la vista renderizada de kanban. Una clase relevante es `o_kanban_small_column`, lo que hace que las columnas sean algo más compactas que las predeterminadas. Las clases adicionales pueden estar disponibles por el módulo proporcionado CSS personalizado.

A continuación, vemos una lista de campos utilizados en las plantillas. Para ser exactos, sólo los campos utilizados exclusivamente en las expresiones QWeb deben ser declarados aquí, para asegurarse de que sus datos son obtenidos desde el servidor.

A continuación, tenemos un elemento `<templates>`, que contiene una o más plantillas QWeb para generar los fragmentos HTML utilizados. Debemos tener una plantilla llamada `kanban-box`, que renderizará las tarjetas kanban. También se pueden agregar plantillas adicionales, generalmente para definir fragmentos HTML que se reutilizarán en la plantilla principal.

Estas plantillas utilizan HTML estándar y el lenguaje de plantillas QWeb. QWeb proporciona directivas especiales, que se procesan para generar dinámicamente el HTML final que se va a presentar.

#### Tip

Odoo utiliza la biblioteca de estilo web de Twitter Bootstrap 3, por lo que esas clases de estilo están generalmente disponibles dondequiera que se pueda renderizar HTML. Puedes obtener más información sobre Bootstrap en https://getbootstrap.com

Ahora vamos a ver más de cerca las plantillas de QWeb para usarlas en las vistas kanban.


### El diseño de la tarjeta kanban

El área de contenido principal de una tarjeta kanban se define dentro de la plantilla `kanban-box`. Este área de contenido también puede tener un sub-contenedor de pie de página.

Para un único pie de página, usaríamos un elemento `<div>` en la parte inferior del cuadro kanban, con la clase `oe_kanban_footer` CSS. Esta clase dividirá automáticamente sus elementos internos con espacios flexibles, haciendo explícita la alineación izquierda y derecha dentro de ella superflua.

Un botón que abre un menú de acción también puede aparecer en la esquina superior derecha de la tarjeta. Como alternativa, el Bootstrap proporciona las clases `pull-left` y `pull-right` para añadir elementos alineados a la izquierda oa la derecha en cualquier parte de la tarjeta, incluso en el pie `oe_kanban_footer`.

Aquí está nuestra primera iteración en la plantilla QWeb para nuestra tarjeta kanban:

```
<!-- Define the kanban-box template --> 
<t t-name="kanban-box"> 
  <!-- Set the Kanban Card color: --> 
  <div t-attf-class="#{kanban_color(record.color.raw_value)} 
    oe_kanban_global_click"> 
      <div class="o_dropdown_kanban dropdown"> 
        
<!-- Top-right drop down menu here... -->



 
      </div> 
      <div class="oe_kanban_content"> 
        <div class="oe_kanban_footer"> 
          <div>
            
<!-- Left hand footer... -->



 
          </div> 
          <div> 
            
<!-- Right hand footer... -->



 
          </div>
        </div> 
      </div> <!-- oe_kanban_content --> 
      <div class="oe_clear"/> 
  </div> <!-- kanban color --> 
</t> 

```

Esto establece la estructura general de la tarjeta kanban. Puedes notar que el campo `color` se utiliza en el elemento superior `<div>` para establecer dinámicamente el color de la tarjeta. Explicaremos más detalladamente la directiva Qweb `t-attf` en una de las siguientes secciones.

Ahora vamos a trabajar en el área de contenido principal, y elegir qué colocar allí:

```

<!-- Content elements and fields go here... -->




 <div>
   <field name="tag_ids" />
 </div>

 <div>
   <strong>
     <a type="open"><field name="name" /></a>
   </strong>
 </div>

 <ul>
   <li><field name="user_id" /></li>
   <li><field name="date_deadline" /></li>
 </ul>
```

La mayor parte de esta plantilla es HTML normal, pero también vemos el elemento `<field>` usado para renderizar los valores de los campos, y el atributo  `type` utilizado en los botones regulares de vista de formulario, usados aquí en una etiqueta de anclaje `<a>`.

En el pie de página izquierdo, vamos a insertar el widget de prioridad:

```
<div> 
  
<!-- Left hand footer... -->



 
  <field name="priority" widget="priority"/> 
</div> 
```
Aquí podemos ver el campo `priority` añadido, tal como lo haríamos en una vista de formulario.

En el pie derecho colocaremos el widget de estado kanban y el avatar para el propietario de la tarea:

```
<div> 
  
<!-- Right hand footer... -->



 
  <field name="kanban_state" widget="kanban_state_selection"/> 
  <img t-att- t-att-src="kanban_image( 
    'res.users', 'image_small', record.user_id.raw_value)" 
    width="24" height="24" class="oe_kanban_avatar pull-right" /> 
</div> 
```
El estado kanban se agrega utilizando un elemento `<field`, al igual que en las vistas de formulario normales. La imagen del avatar del usuario se inserta mediante la etiqueta HTML `<img>`. El contenido de la imagen se genera dinámicamente con la directiva QWeb `t-att-`, que explicaremos en un momento.

A veces queremos tener una pequeña imagen representativa que se mostrará en la tarjeta, como en el ejemplo de **Contactos "Contacts"**. Para hacer referencia, esto se puede hacer agregando lo siguiente como el primer elemento de contenido:

```
<img t-att-src="kanban_image( 'res.partner', 'image_medium', 
  record.id.value)" class="o_kanban_image"/> 
```

### Añadiendo un menú de opciones de tarjeta kanban

Las tarjetas Kanban pueden tener un menú de opciones, situado en la parte superior derecha. Las acciones habituales son editar o eliminar el registro, pero es posible tener cualquier acción que se puede llamar desde un botón. También tenemos un widget para configurar el color de la tarjeta.

El siguiente es un código HTML de línea de base para el menú de opciones que se agregará en la parte superior del elemento `oe_kanban_content`:

```
<div class="o_dropdown_kanban dropdown"> 
  
<!-- Top-right drop down menu here... -->



 
  <a class="dropdown-toggle btn" data-toggle="dropdown" href="#"> 
    <span class="fa fa-bars fa-lg"/> 
  </a> 
  <ul class="dropdown-menu" role="menu" aria-labelledby="dLabel"> 
    
<!-- Edit and Delete actions, if available: -->



 
    <t t-if="widget.editable"> 
      <li><a type="edit">Edit</a></li> 
    </t> 
    <t t-if="widget.deletable"> 
      <li><a type="delete">Delete</a></li>
    </t> 
    
<!-- Call a server-side Model method: -->



 
    <t t-if="!record.is_done.value"> 
      <li><a name="do_toggle_done" type="object">Set as Done</a>
      </li> 
    </t> 
    
<!-- Color picker option: -->



 
    <li> 
      <ul class="oe_kanban_colorpicker" data-field="color"/> 
    </li> 
  </ul> 
</div> 
```
Observa que lo anterior no funcionará a menos que tengamos `<field name="is_done" />` en algún lugar de la vista, porque se usa en una de las expresiones. Si no necesitamos usarla dentro de la plantilla, podemos declararla antes del elemento `<templates>`, como lo hicimos al definir la vista `<kanban>`.

El menú desplegable es básicamente una lista HTML de los elementos `<a>`. Algunas opciones, como **Editar "Edit"** y **Eliminar "Delete"**, sólo están disponibles si se cumplen ciertas condiciones. Esto se hace con la directiva QWeb `t-if`. Más adelante en este capítulo, explicamos estas y otras directivas QWeb con más detalle.

La variable global `widget` representa el objeto `KanbanRecord()` JavaScript actual responsable de la representación de la tarjeta kanban actual. Dos propiedades particularmente útiles son `widget.editable` y `widget.deletable` para inspeccionar si las acciones están disponibles.

También podemos ver cómo mostrar u ocultar una opción dependiendo de los valores del campo de registro. La opción **Establecer como Hecho "Set as Done"** solo se mostrará si el campo `is_done` no está establecido.

La última opción agrega el widget especial del selector de color usando el campo de datos `color` para seleccionar y cambiar el color de fondo de la tarjeta.

### Acciones en vistas kanban

En las plantillas de QWeb, la etiqueta `<a>` para los vínculos puede tener un atributo `type`. Establece el tipo de acción que el enlace realizará para que los enlaces puedan actuar igual que los botones en formas regulares. Así, además de los elementos `<button>`, las etiquetas `<a>` también se pueden usar para ejecutar acciones Odoo.

Al igual que en las vistas de formulario, el tipo de acción puede ser `action` u `object`, y debe ir acompañado de un atributo `name`, que identifica la acción específica a ejecutar. Además, también están disponibles los siguientes tipos de acción:

+ `open` abre la vista de formulario correspondiente
+ `edit` abre la vista de formulario correspondiente directamente en modo de edición
+ `delete` elimina el registro y remueve el elemento de la vista kanban

##  El lenguaje de plantillas QWeb

El analizador QWeb busca directivas especiales en las plantillas y las reemplaza con HTML generado dinámicamente. Estas directivas son atributos de elementos XML y pueden utilizarse en cualquier etiqueta o elemento válido, como `<div>`, `<span>` o `<campo>`.

A veces queremos usar una directiva QWeb pero no queremos colocarla en ninguno de los elementos XML de nuestra plantilla. Para esos casos, tenemos un elemento especial `<t>` que puede tener directivas QWeb, como un `t-if` o un `t-foreach`, pero es silencioso y no tendrá ningún resultado en el XML / HTML final producido.

Las directivas QWeb usan frecuentemente expresiones evaluadas para producir resultados diferentes dependiendo de los valores de registro actuales. Existen dos implementaciones QWeb diferentes: JavaScript en el lado del cliente y Python en el lado del servidor.

 Los informes y las páginas del sitio web utilizan la implementación Python del lado del servidor. Por otro lado, las vistas kanban utilizan la implementación JavaScript del lado del cliente. Esto significa que la expresión QWeb utilizada en las vistas kanban debe escribirse utilizando la sintaxis JavaScript, no Python.

Al mostrar una vista de kanban, los pasos internos son aproximadamente como sigue:

1. Obten el XML de las plantillas para procesar.
1. Llama al método del servidor `read()` para obtener los datos de los campos de las plantillas.
1. Busca la plantilla `kanban-box` y analizala utilizando QWeb para generar los fragmentos HTML finales.
1. Inyecta el HTML en la pantalla del navegador (el DOM).

Esto no pretende ser técnicamente exacto. Es sólo un mapa mental que puede ser útil para entender cómo funcionan las cosas en las vistas kanban.

A continuación, aprenderemos acerca de la evaluación de expresiones de QWeb y exploraremos las directivas QWeb disponibles, usando ejemplos que mejoren nuestra tareas pendientes de la tarjeta kanban.

### La evaluación del contexto Qweb Javascript

Muchas de las directivas QWeb utilizan expresiones que se evalúan para producir algún resultado. Cuando se utiliza desde el lado del cliente, como es el caso de las vistas kanban, estas expresiones están escritas en JavaScript. Se evalúan en un contexto que tiene algunas variables útiles disponibles.

Un objeto `record` está disponible, representando el registro que se procesa, con los campos solicitados desde el servidor. Se puede acceder a los valores de campo utilizando los atributos `raw_value` o `value`:

+ `raw_value` es el valor devuelto por el método de servidor `read()`, por lo que es más adecuado para usar en expresiones de condición.
+ `value` se formatea de acuerdo con la configuración del usuario, y está destinado a ser utilizado para mostrar en la interfaz de usuario. Esto es típicamente relevante para los campos date/datetime y float/monetary.

La evaluación de contexto QWeb también tiene referencias disponibles para la instancia de cliente web JavaScript. Para hacer uso de ellos, se necesita una buena comprensión de la arquitectura de cliente web, pero no vamos a ser capaces de entrar en eso en detalle. Para fines de referencia, los siguientes identificadores están disponibles en la evaluación de la expresión QWeb:

+ `widget` es una referencia al objeto de widget `KanbanRecord()` actual, responsable de la representación del registro actual en una tarjeta kanban. Expone algunas funciones auxiliares útiles que podemos usar.
+ `record` es un atajo para `widget.records` y proporciona acceso a los campos disponibles, usando la notación de puntos.
+ `read_only_mode` indica si la vista actual está en modo de lectura (y no en el modo de edición). Es un acceso directo para `widget.view.options.read_only_mode`.
+ `instance` es una referencia a la instancia completa del cliente web.

También es de destacar que algunos caracteres no están permitidos dentro de las expresiones. El signo inferior a (`<`) es tal caso. Esto es debido al estándar XML, donde dichos caracteres tienen un significado especial y no deben usarse en el contenido XML. Un `>=` negado es una alternativa válida, pero la práctica común es utilizar los siguientes símbolos alternativos que están disponibles para las operaciones de desigualdad:

+ `lt` es por menos que
+ `lte` es por menor o igual que
+ `gt` es por mayor que
+ `gte`es por mayor o igual que

### Utilizando t-attf para la sustitución de atributos de cadena

Nuestra tarjeta kanban utiliza la directiva QWeb `t-attf` para establecer dinámicamente una clase en el elemento superior `<div>` para que la tarjeta se coloree dependiendo del valor del campo `color`. Para ello, se utilizó la directiva QWeb `t-attf-`.

La directiva `t-attf-` genera dinámicamente atributos de etiqueta mediante sustitución de cadena. Esto se permite para las partes de cadenas más grandes generadas dinámicamente, como una dirección URL o nombres de clase CSS.

La directiva busca bloques de expresión que serán evaluados y reemplazados por el resultado. Estos son delimitados por {{and}} o por #{and}. El contenido de los bloques puede ser cualquier expresión JavaScript válida y puede utilizar cualquiera de las variables disponibles para expresiones QWeb, como `record` y `widget`.

En nuestro caso, también utilizamos la función JavaScript `kanban_color()`, proporcionada especialmente para asignar números de índice de color a los nombres de color de la clase CSS.

Como un ejemplo más elaborado, podemos usar esta directiva para cambiar dinámicamente el color de la **Fecha Límite "Deadline Date"**, de modo que las fechas vencidas se muestren en rojo.

Para ello, reemplaza `<field name="date_deadline"/>` en nuestra tarjeta kanban con esto:

```
<li t-attf-class="oe_kanban_text_{{ 
  record.date_deadline.raw_value and 
  !(record.date_deadline.raw_value > (new Date())) 
  ? 'red' : 'black' }}"> 
  <field name="date_deadline"/> 
</li> 
```

Esto resulta en  `class="oe_kanban_text_red"` o en `class="oe_kanban_text_black", dependiendo de la fecha límite. Tenga en cuenta que, aunque la clase CSS `oe_kanban_text_red` está disponible en las vistas kanban, la clase CSS `oe_kanban_text_black` no existe y se utilizó para explicar mejor el punto.

#### Tip 

El signo más bajo que `<`, no está permitido en las expresiones, y elegimos trabajar alrededor de esto usando una comparación negada mayor que. Otra posibilidad sería utilizar la función `&lt`; (Menor que) símbolo de escape en su lugar.


### Utilizando t-att para atributos dinámicos

La directiva QWeb `t-att-` genera dinámicamente un valor de atributo mediante la evaluación de una expresión. Nuestra tarjeta kanban lo usa para establecer dinámicamente algunos atributos en la etiqueta `<img>`.

El elemento `títle` se procesa dinámicamente usando:

```
-att-
```

El campo `.value` devuelve su representación de valores como debería mostrarse en la pantalla, para campos de muchos-a-uno, generalmente este es el valor `name` del registro relacionado. Para los usuarios, este es el nombre de usuario. Como resultado, al pasar el puntero del ratón sobre la imagen, verá el nombre de usuario correspondiente.

La etiqueta `src` también se genera dinámicamente, para proporcionar la imagen correspondiente al usuario responsable. Los datos de imagen son proporcionados por la función JavaScript de ayuda, `kanban_image()`:

```
t-att-src="kanban_image('res.users', 'image_small', 
  record.user_id.raw_value)" 
```

Los parámetros de la función son: el modelo para leer la imagen, el nombre del campo a leer y el ID del registro. Aquí usamos `.raw_value`, para obtener el ID de la base de datos del usuario en lugar de su texto de representación.

No se detiene allí, y `t-att-NAME` y `t-attf-NAME` se pueden hacer para renderizar cualquier atributo, ya que el nombre del atributo generado se toma del sufijo `NAME` utilizado.

### Usando t-foreach para bucles

Un bloque de HTML se puede repetir iterando a través de un bucle. Podemos usarlo para agregar los avatares de los seguidores de tareas a la tarjeta kanban de la tarea.

Comencemos por representar sólo los ID de socio de la tarea, de la siguiente manera:

```
<t t-foreach="record.message_partner_ids.raw_value" t-as="rec"> 
  <t t-esc="rec" />; 
</t>
```

La directiva `t-foreach` acepta una expresión JavaScript que evalúa una colección para iterar. En la mayoría de los casos, este será sólo el nombre de un campo de relación *a-muchos*. Se utiliza con una directiva `t-as` para establecer el nombre a utilizar para referirse a cada elemento de la iteración.

La directiva `t-esc` utilizada a continuación evalúa la expresión proporcionada, sólo el nombre de la variable `rec` en este caso, y lo hace como código HTML con seguridad.

En el ejemplo anterior, pasamos por los seguidores de tareas, almacenados en el campo `message_parter_ids`. Puesto que hay espacio limitado en la tarjeta kanban, podríamos haber usado la función JavaScript `slice()` para limitar el número de seguidores a mostrar, como se muestra en lo siguiente:

```
t-foreach="record.message_partner_ids.raw_value.slice(0, 3)" 
```

La variable `rec`contine cada valor de iteración, una ID de Socio en este caso. Con esto, podemos reescribir los loops de los seguidores como se muestra a continuación:

```
<t t-foreach="record.message_parter_ids.raw_value.slice(0, 3)" 
  t-as="rec"> 
  <img t-att-src="kanban_image('res.partner', 'image_small', rec)" 
    class="oe_avatar" width="24" height="24" /> 
</t>
```

Por ejemplo, esto podría agregarse junto a la imagen del usuario responsable, en el pie de página derecho.

Algunas variables auxiliares también están disponibles. Su nombre tiene como prefijo el nombre de variable definido en `t-as`. En nuestro ejemplo, utilizamos `rec`, por lo que las variables auxiliares disponibles son las siguientes:

+ `rec_index` es el índice de iteración, a partir de cero
+ `rec_size` es el número de elementos de la colección
+ `rec_first` es verdadero en el primer elemento de la iteración
+ `rec_last` es verdadero en el último elemento de la iteración
+ `rec_even` es verdadero en índices pares
+ `rec_odd` es verdadero en índices impares
+ `rec_parity` es impar `odd` o par `even`, dependiendo del índice actual
+ `rec_all` representa el objeto sobre el que se está iterando
+ `rec_value` al iterar a través de un diccionario, `{key: value}`, contiene el valor (`rec` contiene el nombre de la clave)

Por ejemplo, podríamos hacer uso de lo siguiente para evitar una coma de arrastre en nuestra lista de ID:

```
<t t-foreach="record.message_parter_ids.raw_value.slice(0, 3)" 
  t-as="rec"> 
  <t t-esc="rec" />
  <t t-if="!rec_last">;</t> 
</t> 
```

### Usando t-if para renderizado condicional

Nuestra vista kanban utilizó la directiva `t-if` en el menú de opciones de la tarjeta para hacer que algunas opciones estén disponibles dependiendo de algunas condiciones. La directiva `t-if` espera que una expresión se evalúe en JavaScript al representar vistas kanban en el lado del cliente. La etiqueta y su contenido se renderizan sólo si la condición se evalúa como verdadera.

Como otro ejemplo, para mostrar la estimación del esfuerzo de tarea en la tarjeta kanban, sólo si tiene un valor, agregue lo siguiente después del campo date_deadline:

```
<t t-if="record.effort_estimate.raw_value gt 0"> 
  <li>Estimate <field name="effort_estimate"/></li> 
</t>
```

Utilizamos un elemento `<t t-if="...">` para que si la condición es falsa, el elemento no produce salida. Si es cierto, sólo el elemento `<li>` contenido se renderiza a la salida. Observe que la expresión de condición usó el símbolo `gt` en lugar de `>`, para representar el operador *mayor que*.

### Utilizando t-esc y t-raw para generar valores

Utilizamos el elemento `<field>` para representar el contenido del campo. Pero los valores de campo también se pueden presentar directamente sin una etiqueta `<field>`.

La directiva `t-esc` evalúa una expresión y la procesa como un valor de escape HTML, como se muestra a continuación:

```
<t t-esc="record.message_parter_ids.raw_value" /> 
```

En algunos casos, y si se garantiza que los datos de origen son seguros, `t-raw` se puede utilizar para renderizar el valor crudo de campo sin ningún escape, como se muestra en el ejemplo siguiente:

```
<t t-raw="record.message_parter_ids.raw_value" />
```

#### Tip

Por razones de seguridad, es importante evitar el uso de `t-raw` tanto como sea posible. Su uso debe ser estrictamente reservado para la salida de datos HTML que fue preparado específicamente sin datos de usuario en él, o donde cualquier usuario de datos se escapó explícitamente para HTML caracteres especiales.

### Usando t-set para establecer valores en variables

Para una lógica más compleja, podemos almacenar el resultado de una expresión en una variable para usarla más adelante en la plantilla. Esto se debe hacer usando la directiva `t-set`, nombrando la variable a establecer seguida por la directiva `t-value` con la expresión calculando el valor a asignar.

Como ejemplo, el siguiente código hace que los plazos perdidos estén en rojo, como en la sección anterior, pero utiliza una variable `red_or_black` para usar la clase CSS, como se muestra a continuación:

```
<t t-set="red_or_black" t-value=" record.date_deadline.raw_value and 
  record.date_deadline.raw_value lte (new Date()) 
  ? 'oe_kanban_text_red' : ''" /> 
<li t-att-class="red_or_black"> 
  <field name="date_deadline" /> 
</li> 

```
Las variables también se pueden asignar contenido HTML a una variable, como en el ejemplo siguiente:

```
<t t-set="calendar_sign"> 
  <span class="oe_e">Using t-set to set values on variables

</span> 
</t> 
<t t-raw="calendar_sign" /> 
```

La clase `oe_e` CSS utiliza la fuente de pictograma Entypo. La representación HTML del signo de calendario se almacena en una variable que puede usarse cuando sea necesario en la plantilla. El conjunto de iconos de **Fuente Impresionante "Font Awesome"** también está disponible fuera de la caja, y podría haber sido utilizado.


### Usando t-call para insertar otras plantillas

Las plantillas de QWeb pueden ser fragmentos de HTML reutilizables, que se pueden insertar en otras plantillas. En lugar de repetir los mismos bloques HTML una y otra vez, podemos diseñar bloques de construcción para componer vistas de interfaz de usuario más complejas.

Las plantillas reutilizables se definen dentro de la etiqueta `<templates>` y se identifican por un elemento superior con un `t-name` distinto de `kanban-box`. Estas otras plantillas pueden ser incluidas usando la directiva `t-call`. Esto es cierto para las plantillas declaradas junto a la misma vista kanban, en otro lugar del mismo módulo addon o en un addon diferente.

La lista de avatar del seguidor es algo que podría aislarse en un fragmento reutilizable. Vamos a volver a trabajar para usar una sub-plantilla. Debemos comenzar agregando otra plantilla a nuestro archivo XML, dentro del elemento `<templates>`, después del nodo `<t t-name="kanban-box">`, como se muestra a continuación:


```
<t t-name="follower_avatars"> 
  <div> 
    <t t-foreach="record.message_parter_ids.raw_value.slice(0, 3)" 
      t-as="rec"> 
      <img t-att-src="kanban_image('res.partner', 'image_small', rec)" 
        class="oe_avatar" width="24" height="24" /> 
    </t> 
  </div> 
</t> 
```

Llamarlo desde la plantilla `kanban-box` principal es bastante sencillo. En lugar del elemento `<div>` que contiene el parámetro para la directiva `for each`, debemos utilizar lo siguiente:

```
<t t-call="follower_avatars" /> 
```

Para llamar a las plantillas definidas en otros módulos addon, necesitamos usar el identificador completo `module.name`, como lo hacemos con las otras vistas. Por ejemplo, este fragmento puede referirse utilizando el identificador completo `todo_kanban.follower_avatars`.

La plantilla llamada se ejecuta en el mismo contexto que la persona que llama, por lo que cualquier nombre de variable disponible en la persona que llama también está disponible al procesar la plantilla llamada.

Una alternativa más elegante es pasar argumentos a la plantilla llamada. Esto se hace mediante el establecimiento de variables dentro de la etiqueta `t-call`. Éstos se evaluarán y se pondrán a disposición en el contexto de la sub-plantilla solamente, y no existirán en el contexto de la persona que llama.

Podríamos utilizar esto para tener el número máximo de avatares de seguidores establecidos por la persona que llama en lugar de ser codificados en la sub-plantilla. Primero, necesitamos reemplazar el valor fijo, 3 con una variable, `arg_max` por ejemplo:

```
<t t-name="follower_avatars"> 
  <div> 
    <t t-foreach="record.message_parter_ids.raw_value.slice(0, arg_max)"
      t-as="rec"> 
      <img t-att-src="kanban_image('res.partner', 'image_small', rec)" 
        class="oe_avatar" width="24" height="24" /> 
    </t> 
  </div> 
</t> 
```

A continuación, define el valor de esa variable al realizar la llamada de sub-plantilla de la siguiente manera:

```
<t t-call="follower_avatars"> 
  <t t-set="arg_max" t-value="3" /> 
</t>
```

El contenido completo dentro del elemento `t-call` también está disponible para la sub-plantilla a través de la variable mágica `0`. En lugar de variables de argumento, podemos definir un fragmento de código HTML que se puede usar en la sub-plantilla con `<t t-raw="0" /> `.


### Más formas de utilizar t-attf

Hemos pasado por las directivas QWeb más importantes, pero hay algunas más que deberíamos tener en cuenta. Vamos a hacer una breve explicación de ellos.

Hemos visto los atributos de etiqueta dinámica del estilo `t-att-NAME` y `t-attf-NAME`. Además, se puede utilizar la directiva `t-att` fija. Acepta un mapeo de diccionario de clave-valor o un par (una lista de dos elementos).

Utiliza la siguiente asignación:

```
<p t-att="{'class': 'oe_bold', 'name': 'test1'}" /> 


```

Esto da como resultado lo siguiente:

```
<p class="oe_bold" name="test1" />
```

Utiliza el siguiente par:

```
<p t-att="['class', 'oe_bold']" />
```

Esto da como resultado lo siguiente:

```
<p class="oe_bold" /> 
```

## Herencia en vistas kanban

Las plantillas utilizadas en las vistas e informes de kanban se amplían utilizando las técnicas habituales utilizadas para otras vistas, por ejemplo, mediante expresiones XPath. Consulta el Capítulo 3, *Herencia - Ampliando las aplicaciones existentes*, para obtener más detalles.

Un caso común es usar los elementos `<field>` como selector, para luego agregar otros elementos antes o después de ellos. En el caso de las vistas kanban, el mismo campo se puede declarar más de una vez, por ejemplo, una vez antes de las plantillas, y de nuevo dentro de las plantillas. Aquí el selector coincidirá con el primer elemento de campo y no agregará nuestra modificación dentro de la plantilla, como se pretende.

Para evitar esto, necesitamos usar expresiones XPath para asegurarse de que el campo dentro de la plantilla es el que coincide. Por ejemplo:

```
<record id="res_partner_kanban_inherit" model="ir.ui.view"> 
  <field name="name">Contact Kanban modification</field> 
  <field name="model">res.partner</field> 
  <field name="inherit_id" ref="base.res_partner_kanban_view" /> 
  <field name="arch" type="xml"> 
    
<xpath expr="//t[@t-name='kanban- box']//field[@name='display_name']"



 
      position="before"> 
      <span>Name:</span> 
    </xpath> 
  </field> 
</record> 
```

En el ejemplo anterior, XPath busca un elemento `<field name="display_name">` dentro de un elemento `<t tname="kanban-box">`. Esto elimina el mismo elemento de campo fuera de la sección `<templates>`.

Para estas expresiones XPath más complejas, podemos explorar la sintaxis correcta usando algunas herramientas de línea de comandos. La utilidad de línea de comandos `xmllint` probablemente ya esté disponible en su sistema Linux y tiene una opción `--xpath` para realizar consultas en archivos XML.

Otra opción, que proporciona resultados más agradables, es el comando `xpath` del paquete Debian/Ubuntu `libxml-xpath-perl`:

```

$ sudo apt-get install libxml-xpath-perl 





$ xpath -e "//record[@id='res_partner_kanban_view']" -e "//field[@name='display_name']]" /path/to/*.xml


```

## Activos personalizados CSS y JavaScript

Como hemos visto, las vistas kanban son en su mayoría HTML y hacen uso intensivo de las clases CSS. Hemos introducido algunas clases de CSS utilizadas con frecuencia por el producto estándar. Pero para obtener mejores resultados, los módulos también pueden agregar su propio CSS.

No vamos a entrar en detalles aquí sobre cómo escribir código CSS, pero es relevante para explicar cómo un módulo puede agregar sus propios activos web CSS (y JavaScript). Los activos Odoo para el backend se declaran en la plantilla `assets_backend`. Para agregar nuestros activos de módulo, debemos ampliar esa plantilla. El archivo XML para esto generalmente se coloca dentro de un subdirectorio `views/module`.

A continuación se muestra un archivo XML de ejemplo para agregar un archivo CSS y JavaScript al módulo `todo_kanban`, y podría estar en `todo_kanban/views/todo_kanban_assets.xml`:

```
<?xml version="1.0" encoding="utf-8"?> 
<odoo> 
  <template id="assets_backend" inherit_id="web.assets_backend" 
    name="Todo Kanban Assets" > 
    <xpath expr="." position="inside"> 
      <link rel="stylesheet" 
        href="/todo_kanban/static/src/css/todo_kanban.css"/> 
      <script type="text/javascript" 
       src="/todo_kanban/static/src/js/todo_kanban.js"> 
      </script> 
    </xpath> 
  </template> 
</odoo> 
```
Como de costumbre, se debe hacer referencia en el archivo descriptor `__manifest__.py`. Observa que los activos se encuentran dentro de un subdirectorio `/static/src`. Aunque esto no es necesario, es una convención generalmente usada.

## Resúmen

Aprendiste acerca de los tableros kanban y cómo construir las vistas kanban para implementarlas. También introdujimos el modelo QWeb y cómo se puede utilizar para diseñar tarjetas kanban. QWeb es también el motor de renderizado que potencia el sitio web CMS, por lo que está creciendo en importancia en el conjunto de herramientas de Odoo. En el próximo capítulo, seguiremos usando QWeb, pero en el lado del servidor, para crear nuestros informes personalizados.

