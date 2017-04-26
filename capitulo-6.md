#Capítulo 6. Vistas - Diseñando la interfaz de usuario.

Este capítulo te ayudará a aprender cómo crear la interfaz gráfica para que los usuarios interactúen con la aplicación de tareas pendientes. Descubrirá los distintos tipos de vistas y widgets disponibles, comprenderás qué son contexto y dominio y cómo utilizarlos para proporcionar una buena experiencia de usuario.

Continuaremos trabajando con el módulo `todo_ui`. Ya tiene la capa de modelo lista, y ahora necesita la capa de vista para la interfaz de usuario.
##Definiendo la interfaz de usuario con archivos XML

Cada componente de la interfaz de usuario se almacena en un registro de base de datos, al igual que los registros de negocios. Los módulos añaden elementos de interfaz de usuario a la base de datos cargando los datos correspondientes de archivos XML.

Esto significa que un nuevo archivo de datos XML para nuestra interfaz de usuario debe agregarse al módulo `todo_ui`. Podemos comenzar editando el archivo `__manifest__.py` para declarar estos nuevos archivos de datos:

```
{ 
  'name': 'User interface improvements to the To-Do app', 
  'description': 'User friendly features.', 
  'author': 'Daniel Reis', 
  'depends': ['todo_user'], 
  
'data': [ 
    'security/ir.model.access.csv', 
    'views/todo_view.xml', 
    'views/todo_menu.xml', 
  ]



} 
```

####Nota
Nota

Recuerda que los archivos de datos se cargan en el orden especificado. Esto es importante porque sólo puede referenciar IDs XML que se definieron antes de que ser utilizadas.


También podemos crear el subdirectorio y los archivos `views/todo_view.xml` y `views/todo_menu.xml` con una estructura mínima:

```
<?xml version="1.0"?> 
<odoo> 
  <!-- Content will go here... --> 
</odoo>
```

En el capítulo 3, *Herencia - Ampliando las aplicaciones existentes*, se dio un menú básico a nuestra aplicación, pero ahora queremos mejorarlo. Por lo tanto, vamos a añadir nuevos elementos de menú y las acciones de la ventana correspondiente, que se activarán cuando se seleccionan.

###Elementos de menú

Los elementos de menú se almacenan en el modelo `ir.ui.menu` y se pueden consultar a través del menú **Settings** en **Technical|User Interface|Menu Items.

El complemento `todo_app` creó un menú de nivel superior para abrir las tareas de la aplicación Tareas pendientes. Ahora queremos modificarlo a un menú de segundo nivel y tener otras opciones de menú junto a él.

Para ello, agregaremos un nuevo menú de nivel superior para la aplicación y modificaremos la opción de menú de tareas pendientes existente. Para `views / todo_menu.xml`, añada:

```
<!-- Menu items -->
<!-- Modify top menu item -->
<menuitem id="todo_app.menu_todo_task" name="To-Do" />
<!-- App menu items -->
<menuitem id="menu_todo_task_view"
  name="Tasks"
  parent="todo_app.menu_todo_task"
  sequence="10"
  action="todo_app.action_todo_task" />
<menuitem id="menu_todo_config"
  name="Configuration"
  parent="todo_app.menu_todo_task"
  sequence="100"
  groups="base.group_system" /> 
<menuitem id="menu_todo_task_stage"
  name="Stages"
  parent="menu_todo_config"
  sequence="10"
  action="action_todo_stage" />
```
En lugar de usar elementos `<record model = "ir.ui.menu">`, podemos usar el elemento de acceso directo más conveniente `<menuitem>`, que proporciona una forma abreviada de definir el registro que se va a cargar.

Nuestro primer elemento de menú es para la entrada del menú superior de la aplicación de tareas pendientes, con sólo el atributo `name` y se usará como padre para las dos opciones siguientes.

Observa que utiliza la ID XML existente `todo_app.menu_todo_task`, reescribiendo así el elemento de menú, definido en el módulo `todo_app`, sin ninguna acción adjunta a él. Esto se debe a que vamos a agregar elementos de menú hijo, y la acción para abrir las vistas de tarea ahora se llamará desde uno de ellos.

Los siguientes elementos de menú se colocan bajo el elemento de nivel superior, a través del atributo `parent="todo_app.menu_todo_task"`.

El segundo menú es el que abre las vistas Tarea, a través del atributo `action="todo_app.action_todo_task"`. Como puedes ver en el ID XML utilizado, está reutilizando una acción creada por el módulo `todo_app`.

El tercer elemento de menú añade la sección **Configuration** de nuestra aplicación. Queremos que esté disponible sólo para super usuarios, por lo que también utilizamos el atributo grupos para que sea visible sólo para la **Administration|Settings** del grupo de seguridad.

Por último, en el menú **Configuration** agregamos la opción para la tarea Etapas. Lo usaremos para mantener las etapas que usarán por la función kanban que agregaremos a las Tareas pendientes.

En este punto, si intentamos actualizar el complemento debemos obtener errores porque no hemos definido las IDs XML utilizados en los atributos de `action`. Los agregaremos en la siguiente sección.

###Acciones de ventana

Una **acción de ventana** da instrucciones al cliente de GUI, y usualmente se usa en elementos de menú o botones en vistas. Le dice a la GUI en qué modelo trabajar, y qué vistas para poner a disposición. Estas acciones pueden forzar que sólo un subconjunto de los registros sea visible, utilizando un filtro `domain`. También pueden establecer valores predeterminados y filtros a través del atributo `context`.

Agregaremos acciones de ventana al archivo de datos `views/todo_menu.xml`, que será utilizado por los elementos de menú creados en la sección anterior. Edita el archivo y asegúrate de que se agreguen antes de los elementos del menú:

```
<!-- Actions for the menu items --> 
<act_window id="action_todo_stage" 
  name="To-Do Task Stages" 
  res_model="todo.task.stage" 
  view_mode="tree,form" 
  target="current" 
  context="{'default_state': 'open'}" 
  domain="[]" 
  limit="80" 
/> 
<act_window id="todo_app.action_todo_task" 
  name="To-Do Tasks" 
  res_model="todo.task" 
  view_mode="tree,form,calendar,graph,pivot" 
  target="current" 
  context="{'search_default_filter_my_tasks': True}" 
/> 
<!-- Add option to the "More" button --> 
<act_window id="action_todo_task_stage" 
  name="To-Do Task Stages" 
  res_model="todo.task.stage" 
  src_model="todo.task"  
  multi="False" 
/> 

```

Las acciones de la ventana se almacenan en el modelo `ir.actions.act_window` y se pueden definir en archivos XML utilizando el método abreviado `<act_window>` utilizado en el código anterior.

La primera acción abrirá el modelo Etapas de tareas e incluirá los atributos más relevantes para las acciones de ventana:

+ `name` es el título que se mostrará en las vistas abiertas a través de esta acción.
+ `res_model` es el identificador del modelo de destino.
+ `view_mode` es el tipo de vista disponible y su orden. La primera es la abierta por defecto.
+ `target`, si se establece en `new`, abrirá la vista en una ventana de diálogo emergente. Por defecto es `current`, abriendo la vista en línea, en el área de contenido principal.
+ `context` establece información de contexto en las vistas de destino, que puede establecer valores predeterminados o activar filtros, entre otras cosas. Lo veremos con más detalles en un momento.
+ `domain` es una expresión de dominio que obliga a un filtro para los registros que serán explorables en las vistas abiertas.
+ `límit` es el número de registros para cada página, en la vista de lista.

La segunda acción definida en el XML reemplaza la acción original de Tareas pendientes del complemento `todo_app` para que muestre los otros tipos de vistas que exploraremos más adelante en este capítulo: calendario y gráfico. Una vez instalados estos cambios, verás botones adicionales en la esquina superior derecha, después de los botones de lista y forma; Sin embargo, no funcionarán hasta que creemos las vistas correspondientes.

También añadimos una tercera acción, no utilizada en ninguno de los elementos del menú. Nos muestra cómo agregar una opción al menú **More**s, disponible en la parte superior derecha de la lista y vistas de formulario. Para ello, utiliza dos atributos específicos:
+ `src_model` indica en qué modelo esta acción debe estar disponible.
+ `multi`, cuando se establece en `True`, lo hace disponible en la vista de lista para que pueda aplicarse a una selección múltiple de registros. El valor predeterminado es `False`, como en nuestro ejemplo, hará que la opción sólo esté disponible en la vista de formulario, por lo que sólo puede aplicarse a un registro a la vez.

##Contexto y dominio

Nos hemos topado con contexto y dominio varias veces. Hemos visto que las acciones de ventana son capaces de establecerlos y los campos relacionales en los modelos también pueden tenerlos como atributos.
###Datos de contexto

El **contexto** es un diccionario que lleva datos de sesión que se pueden utilizar tanto del lado del cliente en la interfaz de usuario como del lado del servidor de la ORM y la lógica de negocio.

En el lado del cliente puede transportar información de una vista a la siguiente, como la ID del registro activo en la vista anterior, después de seguir un enlace o un botón o proporcionar valores predeterminados para ser utilizados en la vista siguiente.
You can see the lang key with the user language, tz with the time zone information, and uid with the current user ID.

When opening a form from a link or a button in a previous view, an active_id key is added to the context, with the ID of record we were positioned at, in the origin form. In the particular case of list views, we have an active_ids context key containing a list of the record IDs selected in the previous list.

On the client side, the context can be used to set default values or activate default filters on the target view, using keys with the default_ or default_search_ prefixes. Here are some examples:

To set the current user as a default value of the user_id field, we will use the following:
En el lado del servidor, algunos valores de campo del conjunto de registros pueden depender de la configuración local proporcionada por el contexto. En particular, la clave `lang` afecta al valor de los campos traducibles. El contexto también puede proporcionar señales para el código del lado del servidor. Por ejemplo, la clave `active_test` cuando se establece en `False` cambia el comportamiento del método de ORM `search ()` para que no filtre los registros inactivos.

Un contexto inicial del cliente web se ve así:

```
{'lang': 'en_US', 'tz': 'Europe/Brussels', 'uid': 1} 

```

Puedes ver la clave `lang` con el idioma del usuario, `tz` con la información de zona horaria y `uid` con el ID de usuario actual.

Al abrir un formulario desde un enlace o un botón en una vista anterior, se agrega una clave `active_id` al contexto, con el ID de registro en el que estábamos ubicados, en el formulario de origen. En el caso particular de las vistas de lista, tenemos una clave de contexto `active_ids` que contiene una lista de los identificadores de registro seleccionados en la lista anterior.

En el lado del cliente, el contexto se puede utilizar para establecer valores predeterminados o activar filtros predeterminados en la vista de destino, utilizando claves con los prefijos `default_` o `default_search_`. Aquí hay unos ejemplos:

Para configurar el usuario actual como un valor predeterminado del campo `user_id`, utilizaremos lo siguiente:

```
{'
default



_user_id': uid} 

To have a filter_my_tasks filter activated by default on the target view, we will use this:

{'
default_search



_filter_my_tasks': 1} 

```

###Expresiones de dominio

El **dominio** se utiliza para filtrar registros de datos. Utilizan una sintaxis específica que la ORD Odoo analiza para producir las expresiones SQL WHERE que consultarán la base de datos.

Una expresión de dominio es una lista de condiciones. Cada condición es una tupla `('field_name', 'operator', value ')`. Por ejemplo, esta es una expresión de dominio válida, con una sola condición: `[('is_done', '=', False)]`.

La siguiente es una explicación de cada uno de estos elementos:

El **nombre del campo** es el campo que se está filtrando y puede usar la notación de puntos para los campos de los modelos relacionados.

El **valor** se evalúa como una expresión de Python. Puede usar valores literales, como números, Booleanos, cadenas o listas, y puede usar campos e identificadores disponibles en el contexto de la evaluación. En realidad hay dos posibles contextos de evaluación para los dominios:

+ Cuando se utilizan en el lado del cliente, como en las acciones de ventana o los atributos de campo, los valores de campo sin procesar utilizados para procesar la vista actual están disponibles, pero no podemos usar la notación de puntos en ellos.
+ Cuando se utiliza en el lado del servidor, como en las reglas de registro de seguridad y en el código Python del servidor, la notación de puntos se puede utilizar en los campos, ya que el registro actual es un objeto.

El **operador** puede ser:

+ Los operadores de comparación habituales son `<,>, <=,> =, =,! =`.
+ `'= like'` coincide con un patrón, donde el símbolo de subrayado, `_`, coincide con cualquier carácter individual, y el símbolo de porcentaje,`%`, coincide con cualquier secuencia de caracteres.
+ `'like'` coincide con un patrón `'%value%'`. El `'ilike'` es similar, pero no es sensible a las mayúsculas. Los operadores  `"not like"` y "not ilike"` también están disponibles.
+ `'child of'` encuentra los valores de los hijos en una relación jerárquica, para los modelos configurados para soportarlos.
+ `'in'` y `'not in'` se utilizan para verificar la inclusión en una lista dada, por lo que el valor debe ser una lista de valores. Cuando se utiliza en un campo de relación "a-muchos", el operador `in` se comporta como un operador `contains`.

Una expresión de dominio es una lista de elementos y puede contener varias tuplas de condición. Por defecto, estas condiciones se combinarán implícitamente con el ADN operador lógico. Esto significa que sólo devolverá registros que cumplan todas estas condiciones.

También se pueden utilizar operadores de lógica explícita: el símbolo ampersand, `"&"`, para operaciones AND (el predeterminado) y el símbolo de tubo, `'|'` , Para las operaciones de OR. Éstos funcionarán en los dos elementos siguientes, trabajando de una manera recursiva. Veremos esto con más detalle en un momento.

El signo de exclamación, `'!'` , Representa al operador NOT, también está disponible y opera en el siguiente elemento. Por lo tanto, debe colocarse antes de que el elemento sea negado. Por ejemplo, la expresión `['!', ('Is_done', '=', True)]` filtraría todos los registros no hechos.

El "elemento siguiente" también puede ser un elemento del operador que actúa sobre sus elementos siguientes, definiendo condiciones anidadas. Un ejemplo puede ayudarnos a entender mejor esto.

En las reglas del lado del servidor de registro, podemos encontrar expresiones de dominio similares a esta:

```
['|', ('message_follower_ids', 'in',
      [user.partner_id.id]), 
      '|', ('user_id', '=', user.id), 
      ('user_id', '=', False) 
] 
```

Este dominio filtra todos los registros donde el usuario actual está en la lista de seguidores, es el usuario responsable o no tiene un grupo de usuarios responsable.

El primer operador '|' (OR) actúa sobre la condición del seguidor más el resultado de la condición siguiente. La condición siguiente es de nuevo la unión de otras dos condiciones: registros donde el ID de usuario es el usuario de sesión actual o no está establecido.

El siguiente diagrama ilustra esta resolución de los operadores anidados:

![nestedoperationsresolutions](file:///home/dticucv/Escritorio/OEBPS/Image00020.jpg)

##Las vistas de formulario

Como hemos visto en capítulos anteriores, las vistas de formulario pueden seguir un diseño simple o un diseño de documento comercial, similar a un documento en papel.

Ahora veremos cómo diseñar estas vistas de documentos empresariales y cómo usar los elementos y widgets disponibles. Normalmente haríamos esto heredando y extendiendo las vistas de `todo_app`. Pero en aras de la claridad, vamos a crear vistas completamente nuevas para anular las originales.

###Tratando con varias vistas del mismo tipo

El mismo modelo puede tener más de una vista del mismo tipo. Esto puede ser útil ya que una acción de ventana puede indicar la vista específica que debe utilizarse, a través de su ID XML. Así que tenemos la flexibilidad de tener dos elementos de menú diferentes para abrir el mismo modelo utilizando diferentes vistas. Esto se hace añadiendo un atributo `view_id` a la acción de la ventana, con el ID XML de la vista a utilizar. Por ejemplo, podríamos haber usado esto en la acción `todo_app.action_todo_task`, con algo similar a: `view_id = "view_form_todo_task_ui"`.

Pero, ¿qué sucede si no se define una vista específica? En ese caso, la que se utilice será la primera que se devuelva al consultar las vistas. Esta será la de menor prioridad. Si añadimos una nueva vista y la configuramos con una prioridad inferior a las existentes, será la que se use. El efecto final es que parece que esta nueva vista está sobreponiendo a la original.

Dado que el valor predeterminado para la prioridad de vista es 16, cualquier valor inferior bastaría, por lo que una prioridad 15 funcionará.

No es la ruta más utilizada, para ayudar a mantener nuestros ejemplos tan legibles como sea posible, usaremos el enfoque de prioridad en nuestros próximos ejemplos.

###Vistas del documento de negocios

Las aplicaciones empresariales son a menudo sistemas de registro - para productos en un almacén, facturas en un departamento de contabilidad, y muchos más. La mayoría de los datos grabados se pueden representar como un documento en papel. Para una mejor experiencia de usuario, las vistas de formulario pueden imitar estos documentos en papel. Por ejemplo, en nuestra aplicación, podríamos pensar en una Tarea pendiente como algo que tiene un simple formulario de papel que rellenar. Proporcionaremos una vista de formulario que siga este diseño.

Para agregar una vista XML con el esqueleto básico de una vista de documento de negocios, debemos editar el archivo `views/ todo_views.xml` y añadirlo a la parte superior:

```
<record id="view_form_todo_task_ui" 
  model="ir.ui.view"> 
  <field name="model">todo.task</field> 
  <field name="priority">15</field> 
  <field name="arch" type="xml"> 
    <form> 

      <header> 



 
        <!-- To add buttons and status widget --> 

      </header>





      <sheet>



 
        <!-- To add form content -->  

      </sheet>



 
      <!-- Discuss widgets for history and 
      communication: --> 

      <div class="oe_chatter">



  
        <field name="message_follower_ids"  
          widget="mail_followers" /> 
        <field name="message_ids" widget="mail_thread" /> 
      
</div>



 
    </form>
  </field> 
</record> 

```
El nombre de la vista es opcional y generado automáticamente si falta. Para simplificar, aprovechamos esto y omitimos el elemento `<field name="name">` del registro de vista.

Podemos ver que las vistas de documentos empresariales suelen utilizar tres áreas principales: la barra de estado del encabezado **header**, la hoja **sheet** del contenido principal y una sección inferior **history and communication** de historial y comunicación, también conocida como **chatter** o charla.

La sección de historial y comunicación, en la parte inferior, utiliza los widgets de redes sociales proporcionados por el módulo addon de correo. Para poder usarlos, nuestro modelo debe heredar el modelo `mixin mail.thread`, como vimos en el Capítulo 3, *Herencia - Extendiendo las Aplicaciones Existentes*.

####El encabezado

El encabezado en la parte superior por lo general cuenta con el ciclo de vida o los pasos que el documento seguirá a través y los botones de acción.

Estos botones de acción son botones de forma regular, y los pasos siguientes más importantes se pueden resaltar, usando `class="oe_highlight"`.

El ciclo de vida del documento utiliza el widget de barra de estado`statusbar`  en un campo que representa el punto en el ciclo de vida en el que se encuentra actualmente el documento. Normalmente se trata de un campo de selección de estado **State** o de una etapa **Stage** de varios a uno. Estos dos campos se pueden encontrar a través de varios módulos de núcleo de Odoo.

la etapa es un campo de muchos a uno que utiliza un modelo de apoyo para configurar los pasos del proceso. Debido a esto puede ser configurado dinámicamente por los usuarios finales para adaptarse a su proceso de negocio específico, y es perfecto para apoyar las juntas kanban.

El estado es una lista de selección que contiene algunos pasos, bastante estables, en un proceso, como `New`, `In progress`, y `Done`. No es configurable por los usuarios finales, pero, ya que es estático, es mucho más fácil de utilizar en la lógica de negocio. Los campos de vista tienen incluso un soporte especial para ello: el atributo de estado permite que un campo esté disponible para el usuario o no, dependiendo del estado del documento.

Históricamente, las etapas se introdujeron más tarde que los estados. Ambos han coexistido, pero la tendencia en el núcleo de Odoo es para las etapas de reemplazar a los estados. Pero como se ve en la explicación anterior, los estados todavía proporcionan algunas características que las etapas no lo hacen.

Todavía es posible beneficiarse de lo mejor de ambos mundos, mapeando las etapas en estados. Esto fue lo que hicimos en el capítulo anterior, añadiendo un campo de estado en el modelo de etapas de tareas y haciéndolo disponible también en los documentos de Tarea pendiente a través de un campo computado, permitiendo el uso del atributo de campo de estado.

En el archivo `views/todo_view.xml` ahora podemos expandir el encabezado básico para agregar una barra de estado:

```
<header>  
  <field name="state" invisible="True" />
  <button name="do_toggle_done" type="object" 
    attrs="{'invisible':[('state','in',['draft'])]}"
    string="Toggle Done"
    class="oe_highlight" />
  <field name="stage_id"
    widget="statusbar"  
    clickable="True"
    options="{'fold_field': 'fold'}" />
</header>
```

Aquí añadimos `state` como un campo oculto. Necesitamos esto para obligar al cliente a incluir también ese campo en las solicitudes de datos enviadas al servidor. De lo contrario, no estará disponible para su uso en expresiones.

####Tip

Es importante recordar que cualquier campo que desees utilizar, en un dominio o expresión `attrs`, debe cargarse en la vista, por lo que los campos quedarán invisibles en cualquier momento que los necesite, pero no es necesario que los usuarios los vean.

A continuación, se agrega un botón a la barra de estado, para permitir al usuario cambiar el indicador de tarea realizada **Done**.

Los botones que aparecen en la barra de estado deben cambiarse en función del lugar del ciclo de vida del documento actual.

 Utilizamos el atributo `attrs` para ocultar el botón cuando el documento está en estado `draft`. La condición para hacer esto utiliza el campo `state`, no mostrado en el formulario, por lo que tuvimos que añadirlo como un campo oculto.

Si tenemos un campo de selección `state`, podemos usar el atributo `states`. En este caso lo hacemos, y el mismo efecto podría lograrse usando `states="open, done"`. Aunque no es tan flexible como el atributo `attrs`, es más conciso.

Estas características de visibilidad también se pueden utilizar en otros elementos de vista, como campos. Los exploraremos con más detalle más adelante en este capítulo.

El atributo `clickable` permite al usuario cambiar la etapa del documento haciendo clic en la barra de estado. Normalmente queremos habilitar esto, pero también hay casos en los que no lo hacemos, como cuando necesitamos más control sobre el flujo de trabajo, y requerimos que los usuarios progresen a través de las etapas usando sólo los botones de acción disponibles, para que éstos puedan realizar validaciones antes de moverse entre etapas.

Cuando se utiliza un widget de barra de estado con etapas, podemos tener ocultas las etapas raramente usadas en un grupo de Más etapas. Para esto, el modelo de etapas debe tener una bandera para configurar las que ocultar, usualmente llamado `fold`. Y el widget de `statusbar` debe utilizar un atributo `options`, como se muestra en el código anterior, para proporcionar ese nombre de campo a la opción `fold_field`.

Cuando se utiliza el widget de barra de estado con un campo de estado, se puede lograr un efecto similar con el atributo `statusbar_visible`, utilizado para enumerar estados que deben estar siempre visibles y ocultar estados de excepción necesarios para casos menos comunes. Por ejemplo:

```
<field name="stage_id" widget="statusbar"
  clickable="True"
  statusbar_visible="draft,open" />
```

####La hoja

El lienzo de hoja es el área principal del formulario donde se colocan los elementos de datos reales. Está diseñado para que parezca un documento en papel real, y es común ver que los registros en Odoo se conocen como **documents**.

Normalmente, una estructura de hoja de documento tendrá estas áreas:

+ Un título de documento y un subtítulo en la parte superior.
+ Un cuadro de botones en la esquina superior derecha.
+ Otros campos de encabezado del documento.
+ Un cuaderno para campos adicionales organizados en pestañas o páginas. Las líneas de documento también irían aquí, generalmente en la primera página del cuaderno.

Vamos a pasar por cada una de estas áreas.

####Título y subtítulos

Los campos fuera de un elemento `<group>` no tienen automáticamente etiquetas renderizadas para ellos. Este será el caso de los elementos de título, por lo que el elemento `<label for "..." />` debe ser utilizado para procesarla. A expensas de un trabajo extra, esto tiene la ventaja de dar más control sobre la pantalla de etiquetas.

El HTML regular, incluyendo elementos del estilo CSS, también se puede utilizar para hacer brillar el título. Para obtener mejores resultados, el título debe estar dentro de un `<div>` con la clase `oe_title`.

Aquí está el elemento `<sheet>` expandido para incluir el título además de algunos campos adicionales como subtítulos:

```
<sheet> 
  <div class="oe_title"> 
    <label for="name" class="oe_edit_only"/> 
    <h1><field name="name"/></h1> 
    <h3> 
      <span class="oe_read_only>By</span> 
      <label for="user_id" class="oe_edit_only"/> 
      <field name="user_id" class="oe_inline" /> 
    </h3> 
  </div> 
  <!-- More elements will be added from here... --> 
</sheet> 
```
Aquí podemos ver que usamos elementos HTML normales, como `div`, `span`, `h1` y `h2`. El elemento `<label>` nos permite controlar cuándo y dónde se mostrará. El atributo `for` identifica el campo del que deberíamos obtener el texto de la etiqueta. Otra posibilidad es utilizar el atributo de `string` para proporcionar un texto específico para utilizar en la etiqueta. Nuestro ejemplo también utiliza el atributo `class="oe_edit_only"` para que sea visible sólo en modo de edición.

En algunos casos, como Socios o Productos, se muestra una imagen representativa en la esquina superior izquierda. Suponiendo que teníamos un campo binario `my_image`, podríamos añadir antes de la línea `<div class="oe_title">`, usando:

```
<field name="my_image" widget="image" class="oe_avatar"/>
```

####Área de botones inteligentes

El área superior derecha puede tener una caja invisible donde se pueden colocar los botones. La versión 8.0 introdujo botones inteligentes, mostrados como rectángulos con un indicador estadístico que se puede seguir cuando se hace clic.

Podemos agregar la caja de botones justo después del final del DIV `oe_title`, con lo siguiente:
```
<div name="buttons" class="oe_right oe_button_box">
  <!-- Smart buttons here … -->
</div>
```

El contenedor de los botones es un `div` con la clase `oe_button_box` y también `oe_right`, para alinearlo al lado derecho del formulario. Estaremos discutiendo los botones con más detalle en una sección posterior, así que esperaremos hasta entonces para agregar botones reales en esta caja.

####Agrupando contenido en un formulario

El contenido principal del formulario debe organizarse mediante etiquetas `<group>`. La etiqueta `group` inserta dos columnas en el lienzo y, en su interior, por defecto, los campos se mostrarán con etiquetas.

Un valor de campo y una etiqueta de campo toman dos columnas, por lo que agregar campos dentro de un grupo los tendrá apilados verticalmente. Si anidamos dos elementos `<group>` dentro de un grupo superior, podremos obtener dos columnas de campos con etiquetas, lado a lado.

Continuando con nuestra vista de formulario, ahora podemos agregar el contenido principal después del cuadro de botones inteligentes:

```

<group name="group_top"> 
  <group name="group_left">



 
    <field name="date_deadline" /> 
    
<separator string="Reference" />



 
    <field name="refers_to" /> 
  
</group> 
  <group name="group_right">



 
    <field name="tag_ids" widget="many2many_tags"/> 
  
</group> 





</group>

```
Es una buena práctica asignar un nombre o `name` a las etiquetas de grupo para que sea más fácil hacer referencia a ellas más tarde para ampliar la vista (ya sea para tí u otro desarrollador). El atributo `string`también está permitido, y si se establece, se utiliza para mostrar el título de la sección.

Dentro de un grupo, un elemento `<newline>` forzará una nueva línea y el siguiente elemento se renderizará en la primera columna del grupo. Se pueden añadir títulos de sección adicionales dentro de un grupo utilizando el elemento `<separator>`.

####Tip

Prueba la opción Alternar formato de esquema de formulario **Toggle Form Layout Outline** del menú Desarrollador **Developer**: dibuja líneas alrededor de cada sección de grupo, lo que permite una mejor comprensión del diseño de formulario actual.

Podemos tener un mayor control sobre el diseño de un elemento de grupo usando los atributos `col` y `colspan`.

El atributo `col` se puede utilizar en elementos `<group>` para personalizar el número de columnas que contendrá. El valor predeterminado es `2`, pero se puede cambiar a cualquier otro número. Los números pares funcionan mejor ya que por defecto cada campo añadido toma dos columnas, para la etiqueta más el valor del campo.

Los elementos colocados dentro del grupo, incluidos los elementos `<field>`, pueden utilizar un atributo `colspan` para establecer un número específico de columnas que deben tomar. De forma predeterminada, se toma una columna.

####Libretas de notas con pestañas

Otra forma de organizar el contenido es utilizar el elemento `notebook`, que contiene varias secciones con pestañas, llamadas páginas. Estos pueden ser utilizados para mantener menos datos usados ​​fuera de la vista hasta que sea necesario, o para organizar un gran número de campos por tema.

No necesitaremos agregar esto a nuestro formulario Tarea pendiente, pero aquí hay un ejemplo que podría agregarse a un formulario de Etapas de tareas:

```
<notebook> 
  <page string="Whiteboard" name="whiteboard"> 
    <field name="docs" /> 
  </page> 
  <page> 
    <!-- Second page content --> 
  </page> 
</notebook> 
```

##Ver componentes semánticos

Hemos visto cómo organizar el contenido en un formulario, utilizando componentes estructurales como encabezado, grupo y block de notas. Ahora podemos ver más de cerca los componentes semánticos, los campos y los botones, y lo que podemos hacer con ellos.

###Campos

Los campos de vista tienen algunos atributos disponibles para ellos. La mayoría de ellos tienen valores tomados de su definición en el modelo, pero estos pueden ser anulados en la vista.

Los atributos que son genéricos y no dependen del tipo de campo son:

+ `name` identifica el nombre de la base de datos de campo.
+ `string` es el texto de la etiqueta, que se utilizará si queremos anular el texto de la etiqueta proporcionado por la definición del modelo.
+ `help` es una herramienta de texto que se muestra al colocar el puntero sobre el campo y permite anular el texto de ayuda proporcionado por la definición del modelo.
+ `placeholder` de posición es un texto de sugerencia para mostrar dentro del campo.
+ `widget` nos permite anular el widget predeterminado utilizado para el campo. Exploraremos los widgets disponibles en un momento.
+ `options` es una estructura de datos JSON con opciones adicionales para el widget y depende de lo que cada widget admite.
+ `class` son las clases CSS que se utilizarán para la representación HTML de campo.
+ `nolabel="True"` evita que se presente la etiqueta de campo automática. Sólo tiene sentido para los campos dentro de un elemento `<group>` y se utiliza a menudo junto con un elemento `<label for="...">`.
+ `invisible="True"` hace que el campo no sea visible, pero los datos se obtienen del servidor y están disponibles en el formulario.
+ `readonly="True"` hace que el campo no sea editable en el formulario.
+ `required="True"` hace que el campo sea obligatorio en el formulario.

Los atributos específicos de algunos tipos de campos son:

+ `password="True"` se utiliza para los campos de texto. Se muestra como un campo de contraseña, enmascarando los caracteres introducidos.
+ `filename` se utiliza para los campos binarios y es el nombre del campo del modelo que se usará para almacenar el nombre del archivo subido.
+ `mode` se utiliza para campos uno-a-muchos. Especifica el tipo de vista a utilizar para mostrar los registros. De forma predeterminada, es árbol o `tree`, pero también puede ser `form`, `kanban` o `graph`.

####Etiquetas para campos

El elemento `<label>` se puede utilizar para controlar mejor la presentación de una etiqueta de campo. Un caso en el que se utiliza es presentar la etiqueta sólo cuando el formulario está en modo de edición:
```
<Label for="name" class="oe_edit_only" />
```
Al hacer esto, si el campo está dentro de un elemento `<group>`, por lo general queremos también establecer `nolabel="True"` en él.

####Campos relacionales

En los campos relacionales, podemos tener algún control adicional sobre lo que el usuario puede hacer. De forma predeterminada, el usuario puede crear nuevos registros desde estos campos (también conocido como "creación rápida") y abrir el formulario de registro relacionado. Esto se puede desactivar usando el atributo de campo de `options`:
```
Options={'no_open': True, 'no_create': True}
```
El contexto y el dominio también son particularmente útiles en los campos relacionales. El contexto puede definir los valores predeterminados para los registros relacionados y el dominio puede limitar los registros seleccionables. Un ejemplo común es top, tiene la lista de registros seleccionable en un campo para depender del valor actual para otro campo del registro actual. El dominio se puede definir en el modelo, pero también se puede sobreescribir en la vista.

####Widgets de campo

Cada tipo de campo se muestra en el formulario con el widget predeterminado correspondiente. Pero los widgets alternativos adicionales están disponibles para ser usados.

Para los campos de texto, tenemos los siguientes widgets:

+ `email` se utiliza para hacer que el texto del correo electrónico sea una dirección de correo electrónico que se pueda ejecutar.
+ `url` se utiliza para formatear el texto como una URL a la que se puede hacer clic.
+ `html` se utiliza para representar el texto como contenido HTML; en el modo de edición, cuenta con un editor WYSIWYG para permitir el formato del contenido sin necesidad de utilizar la sintaxis HTML.

Para los campos numéricos, tenemos los siguientes widgets:

+ `handle` está diseñado específicamente para campos de secuencia en vistas de lista y muestra un identificador que le permite arrastrar líneas a una orden personalizada.
+ `float_time` formatea un campo flotante con cantidades de tiempo como horas y minutos.
+ `monetary`muestra un campo flotante como el monto de la moneda. Espera un campo asociado `currency_id`, pero otro nombre de campo puede ser proporcionado con `options="{'currency_field': 'currency_id'}"`.
+ `progressbar` presenta un flotante como un porcentaje de progreso y puede ser útil para los campos que representan una tasa de finalización.

Para campos relacionales y de selección, tenemos estos widgets adicionales:

+ `many2many_tags` muestra los valores como una lista de etiquetas tipo botón.
+ `seleccion` utiliza el widget de campo `selection` para un campo de varios a uno.
+ `radio` muestra las opciones del campo `selection` utilizando los botones de opción.
+ `kanban_state_selection` muestra una luz de semáforo para la lista de selección de estado kanban. El estado normal se representa en gris, hecho se representa en verde, y cualquier otro estado se representa en rojo.
+ `priority` representa el campo de selección como una lista de estrellas seleccionables. Las opciones de selección suelen ser un dígito numérico.

###Botones

Los botones admiten estos atributos:

+ `icon` es para la imagen del icono a utilizar en el botón para mostrar; A diferencia de los botones inteligentes, los iconos disponibles para los botones normales están limitados a los disponibles en `addons/web/static/src/img/icons`.
+ `string` es la etiqueta del texto del botón o el texto HTML `alt` cuando se utiliza un icono.
+ `type` es el error tipográfico de la acción a realizar. Los valores posibles son:
  - `workflow` se utiliza para activar una señal de motor de flujo de trabajo;
  - `object` se utiliza para llamar a un método Python;
  - `action` se utiliza para ejecutar una acción de ventana.
+ `name` identifica la acción específica que se debe realizar, según el tipo elegido: un nombre de señal de flujo de trabajo, un nombre de método de modelo o el ID de base de datos de la acción de ventana que se va a ejecutar. La fórmula% (xmlid) d se puede utilizar para traducir el ID XML en el ID de base de datos requerido.
+ `args` gs se utiliza cuando el tipo es objeto, para pasar parámetros adicionales al método.
+ `context` añade valores al contexto, que pueden tener efectos después de ejecutar la acción de windows o en los métodos de código Python llamados.
+ `confirm` muestra un cuadro de mensaje de confirmación, con el texto asignado a este atributo.
+ `pecial="cancel"` se utiliza en los asistentes, para cancelar y cerrar el formulario del asistente.

###  Botones inteligentes

Al diseñar la estructura del formulario, incluimos un área superior derecha para contener botones inteligentes. Ahora vamos a añadir un botón dentro de él.

Para nuestra aplicación, tendremos un botón que muestra el número total de tareas pendientes para el propietario de la tarea actual, y al hacer clic en ella, navegaremos hasta la lista de esos elementos.

Primero debemos añadir el correspondiente campo computado a `models/todo_model.py`. Agregue a la clase `TodoTask` con lo siguiente:

```
def compute_user_todo_count(self):
    for task in self:
        task.user_todo_count = task.search_count(
            [('user_id', '=', task.user_id.id)]) 

user_todo_count = fields.Integer(
    'User To-Do Count',
    compute='compute_user_todo_count')

```
A continuación, agregamos la caja del botón y el botón dentro de ella. Justo después de terminar el DIV `oe_title`, reemplaza el marcador de posición de cuadro de botones que agregamos antes, con lo siguiente:

```
<div name="buttons" class="oe_right oe_button_box">
  <button class="oe_stat_button"
    type="action" icon="fa-tasks"
    name="%(action_todo_task_button)d"
    context="{'default_user_id': user_id}"
    help="All to-dos for this user" >
    <field string="To-Dos" name="user_todo_count"
      widget="statinfo"/>
  </button>
</div>
```

Este botón muestra el número total de Tareas pendientes para la persona responsable de esta tarea, computada por el campo `user_todo_count`.

Estos son los atributos que podemos usar al agregar botones inteligentes:

+ `class="oe_stat_button"` representa un rectángulo en lugar de un botón regular.
+ `icon` establece el icono a utilizar, elegido en el conjunto de fuentes Awesome. Los iconos disponibles se pueden consultar en http://fontawesome.io.
+ `type` y `name` son el tipo de botón y el nombre de la acción a activar. Para los botones inteligentes, el tipo suele ser `action`, para una acción de ventana, y `name` será el ID de la acción a ejecutar. Espera un ID de base de datos real, por lo que tenemos que usar una fórmula para convertir un ID XML en un ID de base de datos: `"%(action-external-id) d"`. Esta acción debe abrir una vista con los registros relacionados.
+ `string` añade texto de etiqueta al botón. No lo hemos utilizado aquí porque el campo contenido ya proporciona un texto para ello.
+ `context` debe utilizarse para establecer valores predeterminados en la vista de destino, que se utilizará en los nuevos registros creados en la vista después de hacer clic en el botón.
+ `help` agrega una herramienta de ayuda que se muestra cuando el puntero del ratón está sobre el botón.

El elemento `button` en sí es un contenedor, con campos que muestran estadísticas. Estos son campos regulares que utilizan el widget `statinfo`. El campo debe ser un campo computado definido en el modelo subyacente. Aparte de campos, dentro de un botón también podemos usar texto estático, como:

```
<div>User's To-dos</div>
```

Al hacer clic en el botón, queremos ver una lista con sólo las Tareas para el usuario actualresponsable. Esto se hará mediante la acción action_`todo_task_button`, aún no implementada. Pero necesita saber el usuario responsable actual, para poder realizar el filtro. Para eso usamos el atributo del botón `context` para almacenar ese valor.

La Acción utilizada debe definirse antes del Formulario, por lo que debemos añadirla en la parte superior del archivo XML:

```
<act_window id="action_todo_task_button"
  name="To-Do Tasks"
  res_model="todo.task"
  view_mode="tree,form,calendar,graph,pivot"
  domain="[('user_id','=',default_user_id)]" />
```

Observe cómo usamos la clave de contexto default_user_id para el filtro de dominio. Esta clave en particular también establecerá el valor predeterminado en el campo user_id al crear nuevas Tareas después de seguir el enlace del botón.

##Vistas dinámicas
Los elementos de vista también admiten algunos atributos dinámicos que permiten a las vistas cambiar dinámicamente su apariencia o comportamiento dependiendo de los valores del campo. Podemos tener en eventos de cambio, la capacidad de cambiar valores en otros campos mientras edita datos en un formulario, o tener campos obligatorios o visibles sólo cuando se cumplen ciertas condiciones.

###En los eventos de cambio

El mecanismo de cambio **on change** nos permite cambiar valores en otros campos de formulario cuando se cambia un campo particular. Por ejemplo, el cambio en un campo Producto puede establecer el campo Precio con un valor predeterminado siempre que se cambie el producto.

En las versiones anteriores, los eventos de cambio se definieron en el nivel de vista, pero desde la versión 8.0 se definen directamente en la capa Modelo, sin necesidad de ninguna marcación específica en las vistas. Esto se hace creando métodos para realizar los cálculos, y usando `@api.onchange('field1', 'field2')` para enlazarlo a los campos. Estos métodos de cambio se analizan con más detalle en el Capítulo 7, *Lógica de Aplicación de ORM - Procesos de Apoyo al Negocio.

###Atributos dinámicos

El mecanismo de cambio también se encarga de la recomputación automática de los campos computados, para reaccionar inmediatamente a la entrada del usuario. Utilizando el mismo ejemplo anterior, si cambia el campo Precio cuando cambiamos el Producto, también se actualizará automáticamente un campo computado del valor total utilizando la nueva información de precios. Un par de atributos proporcionan una manera fácil de controlar la visibilidad de un elemento de interfaz de usuario particular:

+ `groups` pueden hacer visible un elemento en función de los grupos de seguridad a los que pertenece el usuario actual. Sólo los miembros de los grupos especificados lo verán. Espera una lista separada por comas de la ID de XML de grupo.
+ `states` pueden hacer que un elemento sea visible dependiendo del campo Estado del registro. Espera una lista separada por comas de los valores del Estado

Aparte de estos, también tenemos un método flexible disponible para establecer una visibilidad del elemento dependiendo de una expresión evaluada dinámicamente del lado del cliente. Este es el atributo especial `attrs`, esperando un diccionario de valores que asigna el valor del atributo `invisible` al resultado de una expresión.

Por ejemplo, para que el campo `refers_to` sea visible en todos los estados, excepto el borrador, utilice el código siguiente:

```
<field name="refers_to" attrs="{'invisible':   
  state=='draft'}" /> 
```

El atributo `invisble` está disponible en cualquier elemento, no sólo campos. Por ejemplo, podemos usarlo en páginas de block de notas y en elementos de grupo.

Los `attrs` también pueden establecer valores para otros dos atributos: `readonly` y `required`. Estos sólo tienen sentido para los campos de datos, para que no sean editables o obligatorios. Esto nos permite implementar alguna lógica básica del lado del cliente, como hacer un campo obligatorio dependiendo de otros valores de registro, como el Estado.

##Vistas de lista

En este punto, las vistas de lista deben necesitar poca introducción, pero todavía vamos a discutir los atributos que se pueden utilizar con ellos. A continuación se muestra un ejemplo de una vista de lista para nuestras Tareas pendientes:

```
<record id="todo_app.view_tree_todo_task" 
  model="ir.ui.view"> 
  <field name="model">todo.task</field> 
  <field name="arch" type="xml"> 
    <tree decoration-muted="is_done" 
      decoration-bf="state=='open'" 
      delete="false"> 
      <field name="name"/> 
      <field name="user_id"/> 
      <field name="is_done"/>
      <field name="state" invisible="1"/>
    </tree> 
  </field> 
</record> 
```

El color y la fuente del texto de fila pueden cambiar dinámicamente dependiendo de los resultados de una evaluación de expresión de Python. Esto se hace a través de los atributos `decoration-NAME`, con la expresión para evaluar basada en atributos de campo. La parte `NAME` puede ser `bf` o `it`, para fuentes en negrita y cursiva, o cualquier color contextual de texto Bootstrap: `peligro, información, silenciado, primario, éxito` o `advertencia`. La documentación Bootstrap contiene ejemplos sobre cómo se presentan: http://getbootstrap.com/css/#helper-classes-colors.

####Tip

Los atributos `colors` y `fonts`, disponibles en la versión 8.0, estaban obsoletos en la versión 9.0. Los nuevos atributos de decoración deben ser utilizados en su lugar.

Recuerde que los campos utilizados en las expresiones deben declararse en un elemento `<field>`, de modo que el cliente web sepa que esa columna debe recuperarse del servidor. Si no queremos que se muestre al usuario, debemos usar el atributo `invisible="1"` en él.

Otros atributos relevantes del elemento árbol son:

+ `default_order` permite anular el orden de clasificación predeterminado del modelo y su valor sigue el mismo formato que en el atributo de orden utilizado en las definiciones de modelo.
+ `create, delete` y `edit`, si se establece en `false` (en minúsculas) deshabilita la acción correspondiente en la vista de lista.
+ `editable` hace que los registros sean editables directamente en la vista de lista. Los valores posibles son `top` y low`, la ubicación donde se añadirán los nuevos registros.

Una vista de lista puede contener campos y botones, y la mayoría de sus atributos para formularios también son válidos aquí.

En las vistas de lista, los campos numéricos pueden mostrar valores de resumen para su columna. Para ello, agregue al campo uno de los atributos de agregación disponibles, `sum`, `avg`, `min` o `max` y asigne el texto de etiqueta para el valor de resumen. Por ejemplo:

```
<field name="amount" sum="Total Amount" />
```

##Vistas de Búsqueda

Las opciones de búsqueda disponibles se definen mediante el tipo de vista `<búsqueda>` . Podemos elegir los campos se pueden buscar automáticamente al escribir en el cuadro de búsqueda. También podemos proporcionar filtros predefinidos, activados con un clic y opciones de agrupación predefinidas que se utilizarán en vistas de lista.

Esta es una posible vista de búsqueda para las Tareas pendientes:

```
<record id="todo_app.view_filter_todo_task" 
  model="ir.ui.view"> 
  <field name="model">todo.task</field> 
  <field name="arch" type="xml"> 
    <search> 
      <field name="name"/> 
      <field name="user_id"/> 
      <filter name="filter_not_done" string="Not Done"  
        domain="[('is_done','=',False)]"/> 
      <filter name="filter_done" string="Done"  
        domain="[('is_done','!=',False)]"/> 
      <separator/> 
      <filter name="group_user" string="By User"  
        context="{'group_by': 'user_id'}"/> 
    </search> 
  </field> 
</record>
```
Podemos ver dos campos a buscar: `–name` y `user_id`. Cuando el usuario comience a escribir en el cuadro de búsqueda, un menú desplegable le sugerirá buscar en cualquiera de estos campos. Si el usuario escribe `ENTER`, la búsqueda se realizará en el primero de los campos de filtro.

Entonces tenemos dos filtros predefinidos, filtrando tareas hechas y no hechas. Estos filtros se pueden activar independientemente, y se unirán con un operador OR. Los bloques de filtros separados con un elemento `<separator/>` se unirán con un operador AND.

El tercer filtro sólo establece un grupo por contexto. Esto le indica a la vista agrupar los registros por ese campo, `user_id` en este caso.

Los elementos de campo pueden utilizar los siguientes atributos:

+ `name` identifica el campo a utilizar.
+ `string` es un texto de etiqueta que se utiliza en lugar del valor predeterminado.
+ `operador` se utiliza para cambiar el operador de la predeterminada (`=` para los campos numéricos y `ilike` para los otros tipos de campo).
+ `filter_domain` establece una expresión de dominio específica que se utilizará para la búsqueda, proporcionando una alternativa flexible al atributo operator. La cadena de texto buscada se refiere en la expresión como self. Un ejemplo trivial es: `filter_domain="[('name', 'ilike', self)]"`.
+ `grupos` hace que la búsqueda en el campo esté disponible sólo para usuarios pertenecientes a algunos grupos de seguridad. Espera una lista separada por comas de IDs XML.

Para los elementos de filtro, estos son los atributos disponibles:

+ `name` es un identificador para utilizar por herencia o para habilitarlo a través de acciones de ventana. No es obligatorio, pero es una buena práctica siempre proporcionarlo.
+ `string` es el texto de la etiqueta que se mostrará para el filtro. Necesario.
+ `domain` es la expresión de dominio que se agregará al dominio actual.
+ `context` es un diccionario de contexto que se agrega al contexto actual. Generalmente establece una clave `group_id` con el nombre del campo para agrupar registros.
+ `groups` hace que la búsqueda en el campo sólo esté disponible para una lista de grupos de seguridad (IDs XML).

##Vistas de calendario

Como su nombre indica, este tipo de vista presenta los registros de un calendario que se pueden ver durante mes, semana o días. Una vista de calendario para las Tareas pendientes podría tener este aspecto:

```
<record id="view_calendar_todo_task" model="ir.ui.view"> 
  <field name="model">todo.task</field> 
  <field name="arch" type="xml"> 
    <calendar date_start="date_deadline" color="user_id" 
      display="[name], Stage [stage_id]" > 
      <!-- Fields used for the display text --> 
      <field name="name" /> 
      <field name="stage_id" /> 
    </calendar> 
  </field> 
</record> 

```

Los atributos del calendario son:

+ `date_start` es el campo para la fecha de inicio. Obligatorio.
+ `date_end` es el campo para la fecha de finalización. Opcional.
+ `date_delay` es el campo con la duración en días, que se puede utilizar en lugar de `date_end`.
+ `all_day` proporciona el nombre de un campo booleano que se utilizará para señalar eventos de día completo. En estos eventos, se ignora la duración.
+ `color` es el campo utilizado para agrupar el color de las entradas del calendario. A cada valor distinto en este campo se le asignará un color, y todas sus entradas tendrán el mismo color.
+ `display` es el texto de visualización para cada entrada de calendario. Puede registrar valores de usuario utilizando los nombres de campo entre corchetes, como `[name]`. Estos campos deben declararse como hijos del elemento de calendario, y en el ejemplo anterior.
+ `mode` es el modo de visualización predeterminado para el calendario, ya sea `day`, `week` o `month`.

##Vistas de gráficos y pivotes

Las vistas de gráfico proporcionan una vista gráfica de los datos, en forma de gráfico. Los campos actuales disponibles en las Tareas pendientes no son buenos candidatos para un gráfico, por lo que agregamos uno para utilizarlo en dicha vista.

En la clase `TodoTask`, en el archivo `todo_ui/models/todo_model.py`, agrega:

```
effort_estimate = fields.Integer('Effort Estimate')
```

También debe añadirse al formulario Tarea pendiente, de modo que podamos añadir valores a los registros existentes y poder comprobar esta nueva vista.

Ahora vamos a añadir la vista gráfica de las Tareas pendientes:

```
<record id="view_graph_todo_task" model="ir.ui.view"> 
  <field name="model">todo.task</field> 
    <field name="arch" type="xml"> 
      <graph type="bar"> 
        <field name="stage_id" /> 
        <field name="effort_estimate" type="measure" /> 
      </graph> 
    </field> 
</record> 
```

El elemento de vista `graph` puede tener un atributo `type` que puede establecerse en `bar` (el valor predeterminado),` pie o `line`. En el caso de `bar`, el adicional `stacked="True"` se puede utilizar para hacer un gráfico de barras apiladas.

Los datos también se pueden ver en una tabla dinámica, una matriz de análisis dinámico. Para ello, tenemos la vista pivote, introducida en la versión 9.0. Las tablas pivote ya estaban disponibles en la versión 8.0, pero en 9.0, se movieron a su propio tipo de vista. Junto con esto, mejoró las características de interfaz de usuario de tablas de pivote, y optimizó la recuperación de datos de tabla dinámica en gran medida.

Para agregar también una tabla dinámica a las Tareas pendientes, utilice este código:

```
<record id="view_pivot_todo_task" model="ir.ui.view"> 
  <field name="arch" type="xml"> 
    <pivot> 
      <field name="stage_id" type="col" /> 
      <field name="user_id" /> 
      <field name="date_deadline" interval="week" /> 
      <field name="effort_estimate" type="measure" /> 
    </pivot> 
  </field> 
</record>
```
Las vistas de gráfico y pivote deben contener elementos de campo que describen el eje y las medidas a utilizar. La mayoría de los atributos disponibles son comunes a ambos tipos de vista:

+ `name` identifica el campo que se va a utilizar en el gráfico, al igual que en otras vistas
+ `type` es cómo se usará el campo, como un grupo `row` (predeterminado), un `measure` o como `col` (sólo para tablas dinámicas, úsalo para grupos de columnas)
+ `interval` es significativo para los campos de fecha y es el intervalo de tiempo utilizado para agrupar datos de tiempo por `day`, `week`, `month`, `quarter` o `año`.

Por defecto, la agregación utilizada es la suma de los valores. Esto se puede cambiar estableciendo el atributo `group_operator` en la definición de campo Python. Los valores que se pueden usar incluyen `avg`, `max` y `min`.

##Otros tipos de vista

Vale la pena señalar que no cubrimos otros tres tipos de vista que también están disponibles: kanban, gantt y diagrama.

Las vistas Kanban se tratarán en detalle en el Capítulo 9, *Vistas QWeb y Kanban*.

La vista de gantt estaba disponible hasta la versión 8.0, pero se eliminó en la versión 9.0 de la edición de la comunidad debido a las incompatibilidades de las licencias.

Por último, las vistas de diagrama se utilizan para casos muy específicos, y un módulo addon los necesitará raramente. Por si acaso, le gustaría saber que el material de referencia para los dos tipos de vista se puede encontrar en la documentación oficial, https://www.odoo.com/documentation/10.0/reference/views.html.

##Resumen

En este capítulo, aprendimos más sobre las vistas de Odoo para construir la interfaz de usuario, cubriendo los tipos de vista más importantes. En el próximo capítulo, aprenderemos más sobre cómo agregar lógica de negocios a nuestras aplicaciones.