# Capítulo 7. Lógica de aplicación de ORM - Procesos de soporte empresariales.
Con la programación API de Odoo, podemos escribir lógica compleja y asistentes para proporcionar una interacción de usuario rica para nuestras aplicaciones. En este capítulo, veremos cómo escribir código para apoyar la lógica de negocio en nuestros modelos, y también aprenderemos cómo activarlo en eventos y acciones de usuario.

Podemos realizar cálculos y validaciones en eventos, como crear o escribir en un registro, o realizar alguna lógica cuando se hace clic en un botón. Por ejemplo, hemos implementado acciones de botón para las Tareas pendientes, para alternar el indicador **Listo "Is Done"** y desactivar todas las tareas realizadas desactivándolas. 

Además, también podemos utilizar asistentes para implementar interacciones más complejas con el usuario, lo que permite solicitar entradas y proporcionar retroalimentación durante la interacción.

Comenzaremos por crear un asistente para nuestra aplicación de tareas pendientes.

## Creando un asistente

Supongamos que nuestros usuarios de la aplicación To-Do regularmente necesitan establecer los plazos y la persona responsable de un gran número de tareas. Podrían usar un asistente para ayudar con esto. Debería permitirles seleccionar las tareas a actualizar y, a continuación, elegir la fecha límite y / o el usuario responsable para establecerlas.

Los asistentes son formularios utilizados para obtener información de entrada de los usuarios y, a continuación, utilizarla para procesamiento posterior. Pueden utilizarse para tareas sencillas, como solicitar unos pocos parámetros y ejecutar un informe, o para manipulaciones de datos complejas, como el caso de uso descrito anteriormente.

Así será nuestro asistente:

![ToDoTaskCreatingWizard](file:img/7-01.jpg)

Podemos empezar creando un nuevo módulo addon para la función `todo_wizard`.

Nuestro módulo tendrá un archivo Python y un archivo XML, por lo que la descripción de `todo_wizard/__ manifest__.py` será como se muestra en el siguiente código:

```
{  'name': 'To-do Tasks Management Assistant', 
   'description': 'Mass edit your To-Do backlog.', 
   'author': 'Daniel Reis', 
   'depends': ['todo_user'], 
   'data': ['views/todo_wizard_view.xml'], } 
```

Como en los complementos anteriores, el archivo `todo_wizard/__ init__.py` es sólo una línea:

```
from . import models
```

A continuación, necesitamos describir el modelo de datos que soporta nuestro asistente.

### El modelo del asistente

Un asistente muestra una vista de formulario para el usuario, normalmente como una ventana de diálogo, con algunos campos que se rellenan. Estos serán utilizados por la lógica del asistente.

Esto se implementa utilizando la misma arquitectura de modelo/vista que para vistas regulares, pero el modelo de soporte se basa en `models.TransientModel` en lugar de `models.Model`.

Este tipo de modelo también tiene una representación de base de datos y almacena el estado allí, pero se espera que estos datos sean útiles sólo hasta que el asistente termine su trabajo. Un trabajo programado limpia regularmente los datos antiguos de las tablas de la base de datos del asistente.

El archivo `models/todo_wizard_model.py` definirá los campos que necesitamos para interactuar con el usuario: la lista de tareas a actualizar, el usuario responsable y la fecha límite para establecerlas.

Primero agregua el archivo`models/__init__.py` con la siguiente línea de código:

```
from . import todo_wizard_model 

```

Luego, crea el archivo actual `models/todo_wizard_model.py`:

```
# -*- coding: utf-8 -*- 
from odoo import models, fields, api 
 
class TodoWizard(models.TransientModel): 
    _name = 'todo.wizard' 
    _description = 'To-do Mass Assignment' 
    task_ids = fields.Many2many('todo.task', 
      string='Tasks') 
    new_deadline = fields.Date('Deadline to Set') 
    new_user_id = fields.Many2one( 
      'res.users',string='Responsible to Set') 
      
```

Vale la pena señalar que las relaciones uno-a-muchos con los modelos regulares no deben usarse en modelos transitorios. La razón de esto es que requeriría que el modelo regular tuviera la relación inversa de muchos a uno con el modelo transitorio, pero esto no está permitido, ya que podría haber la necesidad de recolectar basura a los registros de modelos regulares junto con el modelo Registros transitorios.

### El formulario de asistente

Las vistas del formulario del asistente son las mismas que para los modelos regulares, excepto en dos elementos específicos:

+ Se puede utilizar una sección `<footer>` para colocar los botones de acción
+ Un botón especial `type= "cancel"` disponible para interrumpir el asistente sin realizar ninguna acción

Este es el contenido de nuestro archivo `views/todo_wizard_view.xml`:

```
<odoo> 
  <record id="To-do Task Wizard" model="ir.ui.view"> 
    <field name="name">To-do Task Wizard</field> 
    <field name="model">todo.wizard</field> 
    <field name="arch" type="xml"> 
 
      <form> 
        <div class="oe_right"> 
          <button type="object" name="do_count_tasks" 
            string="Count" /> 
          <button type="object" name="do_populate_tasks" 
            string="Get All" /> 
        </div> 
 
        <field name="task_ids"> 
          <tree> 
            <field name="name" /> 
            <field name="user_id" /> 
            <field name="date_deadline" /> 
          </tree> 
        </field> 
 
        <group> 
          <group> <field name="new_user_id" /> </group> 
          <group> <field name="new_deadline" /> </group> 
        </group> 
 
        <footer> 
          <button type="object" name="do_mass_update" 
            string="Mass Update" class="oe_highlight" 
            attrs="{'invisible': 
            [('new_deadline','=',False),
            ('new_user_id', '=',False)] 
            }" /> 
          <button special="cancel" string="Cancel"/> 
        </footer> 
      </form> 
    </field> 
  </record> 
 
  <!-- More button Action --> 
  <act_window id="todo_app.action_todo_wizard"
    name="To-Do Tasks Wizard"
    src_model="todo.task" res_model="todo.wizard" 
    view_mode="form" target="new" multi="True" /> 
</odoo> 

```
La acción de ventana `<act_window>` que vemos en el XML añade una opción al botón **Más "More"** del formulario Tarea pendiente mediante el atributo `src_model`. El atributo `target="new"` lo hace abrir como una ventana de diálogo.

También puedes haber notado que `attrs` se utiliza en el botón de **actualización masiva "Mass Update"**, para añadir el toque agradable de hacerlo invisible hasta que se seleccione un nuevo plazo o un usuario responsable.

### La lógica empresarial del asistente

A continuación, tenemos que implementar las acciones a realizar en los botones de formulario. Excluyendo el botón **Cancelar "Cancel"**, tenemos tres botones de acción para implementar, pero ahora nos enfocaremos en el botón de **actualización masiva "Mass Update"**.

El método llamado por el botón es `do_mass_update` y debe definirse en el archivo `models/todo_wizard_model.py`, como se muestra en el código siguiente:

```
from odoo import exceptions 
import logging 
_logger = logging.getLogger(__name__) 
 
# ... 
# class TodoWizard(models.TransientModel): 
# ... 
 
    @api.multi 
    def do_mass_update(self): 
      self.ensure_one() 
      if not (self.new_deadline or self.new_user_id):
        raise exceptions.ValidationError('No data to update!') 
      _logger.debug('Mass update on Todo Tasks %s', 
                    self.task_ids.ids) 
      vals = {} 
      if self.new_deadline: 
        vals['date_deadline'] = self.new_deadline 
      if self.new_user_id:
        vals['user_id'] = self.new_user_id 
      # Mass write values on all selected tasks 
      if vals: 
        self.task_ids.write(vals) 
      return True 

```

Nuestro código debe manejar una instancia de asistente a la vez, por lo que usamos `self.ensure_one ()` para que quede claro. Aquí `self` representa el registro de exploración para los datos en el formulario del asistente.

El método comienza validando si se ha dado una nueva fecha límite o un usuario responsable, y si no se produce un error. A continuación, tenemos un ejemplo de cómo escribir un mensaje de depuración en el registro del servidor.

A continuación, el diccionario `vals` se construye con los valores a establecer con la actualización masiva: la nueva fecha, nuevo responsable o ambas. A continuación, el método de escritura `write` se utiliza en un conjunto de registros para realizar la actualización masiva. Esto es más eficiente que un bucle que realiza escrituras individuales en cada registro.

Es una buena práctica para los métodos de siempre devolver algo. Es por eso que devuelve el valor `True` al final. La única razón de esto es que el protocolo XML-RPC no admite valores `None`, por lo que esos métodos no se podrán utilizar mediante ese protocolo. En la práctica, es posible que no conozcas el problema porque el cliente web utiliza JSON-RPC, no XML-RPC, pero sigue siendo una buena práctica.

A continuación, vamos a tener una mirada más cercana al registro, y luego trabajaremos en la lógica detrás de los dos botones en la parte superior: Contar y obtener todo.

### Logging

Estas actualizaciones masivas podrían ser mal utilizadas, por lo que podría ser una buena idea registrar alguna información cuando se utiliza. El código anterior inicializa el `_logger` en las dos líneas antes de la clase `TodoWizard`, utilizando la biblioteca estándar de registro de Python. La variable interna Python `__name__` es para identificar los mensajes como procedentes de este módulo.

Para escribir mensajes de registro en el código del método podemos usar:

```
_logger.debug('A DEBUG message') 
_logger.info('An INFO message') 
_logger.warning('A WARNING message') 
_logger.error('An ERROR message')
```

Al pasar valores para usar en el mensaje de registro, en lugar de usar interpolación de cadena, deberíamos proporcionarlos como parámetros adicionales. Por ejemplo, en lugar de `_logger.info('Hello% s' % 'World')` deberíamos usar `_logger.info('Hello% s', 'World')`. Puedes notar que lo hicimos en el método `do_mass_update ()`.

#### Nota

Una cosa interesante a notar sobre el registro, es que las entradas del registro imprimen siempre la marca de tiempo en UTC. Esto puede ser una sorpresa para los nuevos administradores, pero se debe al hecho de que el servidor maneja internamente todas las fechas en UTC.


### Aumentando las excepciones

Cuando algo no está bien, querremos interrumpir el programa con un mensaje de error. Esto se hace levantando una excepción. Odoo ofrece algunas clases de excepción adicionales a las disponibles en Python. Estos son ejemplos para los más útiles:

```
from odoo import exceptions 
raise exceptions.Warning('Warning message') 
raise exceptions.ValidationError('Not valid message')
```

El mensaje de advertencia `Warning` también interrumpe la ejecución, pero puede sonar menos grave que un mensaje `ValidationError`. Aunque no es la mejor interfaz de usuario, aprovechamos eso en el botón de conteo **Count** para mostrar un mensaje al usuario:

```
@api.multi 
def do_count_tasks(self): 
    Task = self.env['todo.task'] 
    count = Task.search_count([('is done', '=', False)]) 
    raise exceptions.Warning(
          'There are %d active tasks.' %count) 
```

Como una nota de lado, parece que podríamos haber utilizado el decorador `@api.model`, ya que este método no funciona en el conjunto de registros `self`. Pero en este caso no podemos porque el método tiene que ser llamado desde un botón.

### Acciones de ayuda en asistentes

Ahora supongamos que queremos un botón para recoger automáticamente todas las tareas pendientes para evitar que el usuario los escoge uno por uno. Ese es el punto de tener el botón **Obtener Todo "Get All"** en el formulario. El código detrás de este botón obtendrá un conjunto de registros con todas las tareas activas y lo asignará a las tareas del campo muchos-a-muchos.

Pero hay una trampa aquí. En las ventanas de diálogo, cuando se pulsa un botón, la ventana del asistente se cierra automáticamente. No enfrentamos este problema con el botón **Count** porque utiliza una excepción para mostrar su mensaje; Por lo que la acción no tiene éxito y la ventana no está cerrada.

Afortunadamente, podemos evitar este comportamiento pidiendo al cliente que vuelva a abrir el mismo asistente. Los métodos de modelo pueden devolver una acción de ventana que debe realizar el cliente web, en forma de un objeto de diccionario. Este diccionario utiliza los mismos atributos que se utilizan para definir acciones de ventana en archivos XML.

Vamos a definir una función de ayuda para el diccionario de acción de la ventana para reabrir la ventana del asistente, para que pueda reutilizarse fácilmente en varios botones:

```
@api.multi 
def _reopen_form(self): 
    self.ensure_one() 
    return {
        'type': 'ir.actions.act_window',
        'res_model': self._name,  # this model             
        'res_id': self.id,  # the current wizard record   
        'view_type': 'form', 
        'view_mode': 'form', 
        'target': 'new'} 
```

Vale la pena señalar que la acción de la ventana podría ser algo más, como saltar a un formulario de asistente diferente para solicitar la entrada de usuario adicional y que se puede utilizar para implementar asistentes de varias páginas.

Ahora, el botón **Get All** puede hacer su trabajo y mantener al usuario trabajando en el mismo asistente:

```
@api.multi 
def do_populate_tasks(self): 
    self.ensure_one()         
    Task = self.env['todo.task']
    open_tasks = Task.search([('is_done', '=', False)])
    # Fill the wizard Task list with all tasks 
    self.task_ids = all_tasks 
    # reopen wizard form on same wizard record 
    return self._reopen_form() 
```

Aquí podemos ver cómo trabajar con cualquier otro modelo disponible: primero usamos `self.env[]` para obtener una referencia al modelo, `todo.task` en este caso, y luego podemos realizar acciones en él, como `search()` para recuperar registros que cumplan algunos criterios de búsqueda.

El modelo transitorio almacena los valores en los campos de formulario del asistente y puede leerse o escribirse como cualquier otro modelo. La variable `all_tasks` se asigna al campo uno-a-muchos `task_ids`. Como puede ver, esto se hace como lo haríamos para cualquier otro tipo de campo.

## Trabajando con la API ORM

De la sección anterior, ya tenemos una idea de cómo es usar la API de ORM. A continuación veremos qué más podemos hacer con él.

### Decoradores de métodos

Durante nuestro viaje, los varios métodos que encontramos utilizaron decoradores API como `@api.multi`. Estos son importantes para el servidor para saber cómo manejar el método. Vamos a recapitular los disponibles y cuando deben ser utilizados.

El decorador `@api.multi` se utiliza para manejar conjuntos de registros con la nueva API y es el más utilizado. Aquí `self` es un conjunto de registros, y el método normalmente incluirá un bucle `for` para iterarlo.

En algunos casos, el método se escribe para esperar un singleton: un conjunto de registros que no contenga más de un registro. El decorador de `@api.one` fue descontinuado desde 9.0 y debe ser evitado. En su lugar debemos seguir utilizando `@api.multi` y añadir al código del método una línea con `self.ensure_one()`, para asegurar que es un singleton.

Como se mencionó, el decorador `@api.one` es obsoleto, pero todavía es compatible. Para completar, puede ser que valga la pena saber que envuelve el método decorado, alimentándolo de un registro a la vez, haciendo la iteración del conjunto de registros. En nuestro método `self` está garantizado para ser un singleton. Los valores de retorno de cada llamada de método individual se agregan como una lista y se devuelven.

El `@api.model` decora un método estático de nivel de clase y no utiliza datos de conjunto de registros. Para la consistencia, `self`sigue siendo un conjunto de registros, pero su contenido es irrelevante. Ten en cuenta que este tipo de método no se puede utilizar desde los botones de la interfaz de usuario.

Algunos otros decoradores tienen propósitos más específicos y se van a utilizar junto con los decoradores descritos anteriormente:

+ `@api.depends(fld1,...)` se utiliza para que las funciones de campo computadas identifiquen en qué cambios debe ser activado el (re) cálculo
+ `@api.constrains(fld1,...)` se utiliza para las funciones de validación para identificar en qué cambios debe activarse la comprobación de validación
+ `@api.onchange(fld1,...)` se utiliza para las funciones de cambio para identificar los campos en el formulario que desencadenará la acción

En particular, los métodos `onchange` pueden enviar un mensaje de advertencia a la interfaz de usuario. Por ejemplo, esto podría advertir al usuario que la cantidad de producto recién ingresada no está disponible en stock, sin impedir que el usuario continúe. Esto se hace al hacer que el método devuelva un diccionario que describe el mensaje de advertencia:

```
return { 
         'warning': { 
         'title': 'Warning!', 
         'message': 'You have been warned'} 
        } 
```

### Anulando los métodos predeterminados de ORM

Hemos aprendido sobre los métodos estándar proporcionados por la API, ¡pero sus usos no terminan ahí! También podemos ampliarlos para agregar un comportamiento personalizado a nuestros modelos.

El caso más común es extender los métodos `create()` y `write()`. Esto se puede utilizar para agregar la lógica a ser activada cada vez que se ejecutan estas acciones. Colocando nuestra lógica en la sección apropiada del método personalizado, podemos hacer que el código se ejecute antes o después de que se ejecuten las operaciones principales.  

Utilizando el modelo `TodoTask` como ejemplo, podemos hacer un `create()` personalizado, que se vería así:

```
@api.model 
def create(self, vals): 
    # Code before create: can use the `vals` dict 
    new_record = super(TodoTask, self).create(vals) 
    # Code after create: can use the `new_record` created 
    return new_record 

```

Un `write()` personalizado debería seguir esta estructura:

```
@api.multi 
def write(self, vals): 
    # Code before write: can use `self`, with the old values 
    super(TodoTask, self).write(vals) 
    # Code after write: can use `self`, with the updated values 
    return True 

```
Estos son ejemplos comunes de extensión, pero por supuesto cualquier método estándar disponible para un modelo puede ser heredado de una manera similar para agregar nuestra lógica personalizada a ella.

Estas técnicas abren muchas posibilidades, pero recuerda que también hay otras herramientas disponibles que pueden ser más adecuadas para tareas específicas comunes:

+ Para tener un valor de campo calculado basado en otro, debemos usar campos calculados. Un ejemplo de esto es calcular un total de cabecera cuando se cambian los valores de las líneas.
+ Para que los valores por defecto de campo se calculen dinámicamente, podemos usar un campo predeterminado enlazado a una función en lugar de un valor fijo.
+ Para que los valores se establezcan en otros campos cuando se cambia un campo, podemos usarlo en las funciones de cambio. Un ejemplo de esto es cuando se selecciona un cliente, estableciendo su moneda como la moneda del documento, que posteriormente puede ser modificada manualmente por el usuario. Tenga en cuenta que el cambio sólo funciona en la interacción de vista de formulario y no en las llamadas de escritura directa.
+ Para las validaciones, debemos utilizar funciones de restricción decoradas con `@api.constraints(fld1, fld2, ...)`. Estos son como campos calculados pero, en lugar de calcular valores, se espera que generen errores.

### Métodos para llamadas a clientes web y RPC

Hemos visto los métodos de modelo más importantes usados ​​para generar conjuntos de registros y cómo escribir en ellos. Pero hay algunos métodos de modelo más disponibles para acciones más específicas, como se muestra aquí:

+ `read([fields])` es similar al método `browse`, pero en lugar de un conjunto de registros, devuelve una lista de filas de datos con los campos dados como argumento. Cada fila es un diccionario. Proporciona una representación serializada de los datos que se pueden enviar a través de protocolos RPC y está destinado a ser utilizado por los programas cliente y no en la lógica del servidor.
+ `search_read([domain], [fields], offset=0, limit=None, order=None)` realiza una operación de búsqueda seguida de una lectura en la lista de registros resultante. Se detina para ser utilizado por los clientes de RPC y les ahorra el viaje redondo adicional necesario al hacer una búsqueda `search` seguida de una lectura `read` en los resultados.
+ `load([fields], [data])` se utiliza para importar datos adquiridos de un archivo CSV. El primer argumento es la lista de campos a importar y se mapea directamente a una fila superior de CSV. El segundo argumento es una lista de registros, donde cada registro es una lista de valores de cadena para analizar e importar, y se asigna directamente a las filas y columnas de datos CSV. Implementa las características de importación de datos CSV descritas en el Capítulo 4, *Datos de Módulo*, como el soporte de identificadores externos. Se utiliza por la característica de **importación "Import"** de cliente web. Sustituye el método import_data obsoleto.
+ `export_data([fields], raw_data=False)` es utilizado por la función de ** exportación "Expor"**del cliente web. Devuelve un diccionario con una clave de datos que contiene los datos; Una lista de filas. Los nombres de campo pueden utilizar los sufijos `.id` y `/id` utilizados en los archivos CSV y los datos están en un formato compatible con un archivo CSV importable. El argumento `raw_data` opcional permite que los valores de datos se exporten con sus tipos Python, en lugar de la representación de cadena utilizada en CSV.

Los siguiente métodos son utilizados principalmente por el cliente web para renderizar la interfaz de ususario y realizar interacción básica:

+ `name_get()`: Devuelve una lista de tuplas (`ID`, `name`) con el texto que representa cada registro. Se utiliza por defecto para calcular el valor `display_name`, proporcionando la representación de texto de campos de relación. Se puede extender para implementar representaciones de visualización personalizadas, como mostrar el código de registro y el nombre en lugar de sólo el nombre.
+ `name_search(name='', args=None, operator='ilike', limit=100)` devuelve una lista de tuplas (`ID, name`) donde el nombre de visualización coincide con el texto del argumento `name`. Se utiliza en la interfaz de usuario al escribir en un campo de relación para producir la lista con los registros sugeridos que coinciden con el texto escrito. Por ejemplo, se utiliza para implementar la búsqueda de productos tanto por nombre como por referencia, al escribir en un campo para seleccionar un producto.
+ `name_create(name)` crea un nuevo registro con sólo el nombre del título que se va a utilizar para ello. Se utiliza en la interfaz de usuario para la función de "creación rápida", donde puedes crear rápidamente un registro relacionado proporcionando su nombre. Puede extenderse para proporcionar valores predeterminados específicos para los nuevos registros creados a través de esta función.
+ `default_get([fields])` devuelve un diccionario con los valores por defecto para crear un nuevo registro. Los valores predeterminados pueden depender de variables como el usuario actual o el contexto de la sesión.
+ `fields_get()` se utiliza para describir las definiciones de campo del modelo, como se ve en la opción **Ver Campos "Viem Fields"** del menú del desarrollador.
+ `fields_view_get()` es utilizado por el cliente web para recuperar la estructura de la vista de interfaz de usuario a procesar. Se le puede dar el ID de la vista como un argumento o el tipo de vista que queremos usando `view_type='form'`. Por ejemplo, puedes probar esto: `rset.fields_view_get(view_type='tree')`.

### El comando shell

Python tiene una interfaz de línea de comandos que es una gran manera de explorar su sintaxis. Del mismo modo, Odoo también tiene una característica equivalente, donde podemos interactivamente probar comandos para ver cómo funcionan. Ese es el comando `shell`.

Para usarlo, ejecuta Odoo con el comando `shell` y la base de datos para usar, como se muestra aquí:


```
$ ./odoo-bin shell -d todo

```


Deberías ver la habitual secuencia de inicio del servidor en el terminal hasta que pare en un prompt Python `>>>` esperando tu entrada. Aquí, `self` representará el registro para el usuario `Administratoṛ`, el que puedes confirmar escribiendo lo siguiente:


```

>>> self





res.users(1,)





>>> self._name





'res.users'





>>> self.name





u'Administrator'

```

En la sesión anterior, hacemos una cierta inspección sobre nuestro medio ambiente. El `self` representa un conjunto de registros `res.users` que contiene sólo el registro con el ID `1`. También podemos confirmar el nombre de modelo del conjunto de registros que inspecciona `self._name` y obtener el valor del campo de registro `name`, confirmando que es el usuario `Administrator`.

Al igual que con Python, puedes salir del prompt utilizando _**Ctrl + D**_. Esto también cerrará el proceso del servidor y regresará al prompt del sistema shell.

#### Tip

La característica de shell se agregó en la versión 9.0. Para la versión 8.0 hay un módulo back-portted de la comunidad para agregarlo. Una vez descargado y incluido en la ruta addons, no es necesario realizar ninguna instalación adicional. Se puede descargar desde https://www.odoo.com/apps/modules/8.0/shell/.

### El entorno del servidor

El servidor shell proporciona una referencia `self` idéntica a la que encontrarías dentro de un método del modelo Users, `res.users`.

Como hemos visto, `self`es un conjunto de registros. Los **Conjuntos de Registros "Recordsets"** llevan consigo una información de entorno, incluido el usuario que navega por los datos y la información de contexto adicional, como el idioma y la zona horaria. Esta información es importante y el indicador o zona horaria.

Podemos comenzar a inspeccionar nuestro entorno actual con:


```

>>> self.env





<openerp.api.Environment object at 0xb3f4f52c>
```

El entorno de ejecución en `self.env` tiene los siguientes atributos disponibles:

+ `env.cr` es el cursor de la base de datos que se está utilizando
+ `env.uid` es el ID para el usuario de sesión
+ `env.user` es el registro para el usuario actual
+ `env.context` es un diccionario inmutable con un contexto de sesión

El entorno también proporciona acceso al registro donde están disponibles todos los modelos instalados. Por ejemplo, `self.env['res.partner']` devuelve una referencia al modelo Socios. Podemos usar `search()` o `browse()` para recuperar los conjuntos de registros:

```

>>> self.env['res.partner'].search([('name', 'like', 'Ag')])





res.partner(7, 51)


```

En este ejemplo, un conjunto de registros para el modelo `res.partner` contiene dos registros, con IDs `7` y `51`.


### Modificando del entorno de ejecución

El ambiente es inmutable, y por lo tanto no se puede modificar. Pero podemos crear un entorno modificado y luego ejecutar acciones con él.

Estos métodos se pueden utilizar para eso:

+ `env.sudo(user)` se proporciona con un registro de usuario, y devuelve un entorno con ese usuario. Si no se proporciona ningún usuario, se utilizará el superusuario `Administrator`, que permite ejecutar consultas específicas sin pasar por las reglas de seguridad.
+ `env.with_context(diccionary)` sustituye el contexto por uno nuevo.
+ `env.with_context(key=value,...)` modifica los valores actuales de configuración de contexto para algunas de sus claves.

Además, tenemos la función `env.ref()`, tomando una cadena con un identificador externo y devuelve un registro para ella, como se muestra aquí:

```

>>> self.env.ref('base.user_root')





res.users(1,)



```

### Transacciones y SQL de bajo nivel

Las operaciones de escritura de base de datos se ejecutan en el contexto de una transacción de base de datos. Por lo general, no tenemos que preocuparnos de esto ya que el servidor se encarga de que mientras se ejecuta métodos de modelo.

Pero en algunos casos, podemos necesitar un control más fino sobre la transacción. Esto se puede hacer a través del cursor de la base de datos `self.env.cr`, como se muestra aquí:

+ `self.env.cr.commit()` compromete las operaciones de escritura en búfer de la transacción
+ `self.env.savepoint()` establece un punto de almacenamiento de transacciones para retroceder a
+ `self.env.rollback()` cancela las operaciones de escritura de la transacción desde el último punto de salvación, o todas si no se creó ningún punto de salvación

#### Tip

En una sesión shell, la manipulación de datos no se hará efectiva en la base de datos hasta que utilices `self.env.cr.commit()`.

Con el método cursor `execute()`, podemos ejecutar SQL directamente en la base de datos. Se necesita una cadena con la instrucción SQL para ejecutar y un segundo argumento opcional con una tupla o lista de valores para usar como parámetros para el SQL. Estos valores se utilizarán donde se encuentren los marcadores de posición `%s`.

#### Nota

#### ¡Precaución!

Con `cr.execute()` debemos resistir a añadir directamente los valores de los parámetros a la cadena de consulta. Este es un riesgo de seguridad bien conocido que puede ser explotado a través de ataques de inyección de SQL. Siempre usa marcadores de posición `%s`y el segundo parámetro para pasar valores.

Si está utilizando una consulta `SELECT`, los registros se deben buscar. La función `fetchall()` recupera todas las filas como una lista de tuplas, y `dictfetchall()` las recupera como una lista de diccionarios, como se muestra en el siguiente ejemplo:


```

>>> self.env.cr.execute("SELECT id, login FROM res_users WHERE 





login=%s OR id=%s", ('demo', 1))





>>> self.env.cr.fetchall() 





[(4, u'demo'), (1, u'admin')]



También es posible ejecutar instrucciones de **Data Manipulation Language (DML)** como `UPDATE` e `INSERT`. Dado que el servidor mantiene cachés de datos, pueden volverse inconsistentes con los datos reales de la base de datos. Debido a esto, mientras se utiliza DML sin procesar, las caches se deben borrar después usando `self.env.invalidate_all()`.

#### Nota

#### ¡Precaución!

Ejecutar SQL directamente en la base de datos puede dar lugar a datos inconsistentes. Debes usarlo sólo si está seguro de lo que está haciendo.

## Trabajando con conjuntos de registros

Ahora exploraremos cómo funciona el ORM y nos enteramos de las operaciones más comunes que se realizan con ella. Usaremos el prompt proporcionado por el comando `shell` para explorar interactivamente cómo funcionan los recordsets.


### Consultando modelos

Con `self`, sólo podemos acceder al conjunto de registros del método. Pero la referencia de entorno `self.env` nos permite acceder a cualquier otro modelo. Por ejemplo, `self.env['res.partner']` devuelve una referencia al modelo Partners (que en realidad es un conjunto de registros vacío). Podemos usar `search()` o `browse()` en él para generar conjuntos de registros.

El método `search()` toma una expresión de dominio y devuelve un conjunto de registros con los registros que coinciden con esas condiciones. Un dominio vacío `[]` devolverá todos los registros. Para obtener más detalles sobre las expresiones de dominio, consulta el Capítulo 6, *Vistas - Diseñando la interfaz de usuario*. Si el modelo tiene el campo especial `active`, por defecto sólo se considerarán los registros con `active=True`.

Algunos argumentos opcionales de palabras clave están disponibles, como se muestra aquí:

+ `order` es una cadena que se utilizará como la cláusula `ORDER BY` en la consulta de la base de datos. Esto suele ser una lista de nombres de campos separados por comas.
+ limit` establece un número máximo de registros para recuperar.
+ `offset` ignora los primeros resultados `n`; Se puede utilizar con `límit` para consultar bloques de registros a la vez.

A veces sólo necesitamos saber el número de registros que cumplen ciertas condiciones. Para ello podemos usar `search_count()`, que devuelve el recuento de registros en lugar de un conjunto de registros. Se ahorra el costo de recuperar una lista de registros sólo para contarlos, por lo que es mucho más eficiente cuando aún no tenemos un conjunto de registros y sólo queremos contar el número de registros.

El método `browse()` toma una lista de IDs o una sola ID y devuelve un conjunto de registros con esos registros. Esto puede ser conveniente para los casos en los que ya conocemos los ID de los registros que queremos.

Algunos ejemplos de uso de esto se muestran aquí:


```

>>> self.env['res.partner'].search([('name', 'like', 'Ag')])





res.partner(7, 51)





>>> self.env['res.partner'].browse([7, 51])






res.partner(7, 51)



```

### Singletons

El caso especial de un conjunto de registros con sólo un registro se llama un conjunto de registros **singleton**. Los singletons siguen siendo un conjunto de registros y se pueden utilizar donde se espera un conjunto de registros.

Pero a diferencia de los conjuntos de registros de elementos múltiples, los singletons pueden acceder a sus campos usando la notación de puntos, como se muestra aquí:

```
>>> print self.name 
Administrator
```


En el siguiente ejemplo, podemos ver que el mismo conjunto de registros `self` singleton también se comporta como un conjunto de registros, y podemos iterarlo. Tiene sólo un registro, por lo que solo se imprime un nombre:

```
>>> for rec in self: 
      print rec.name 
Administrator
```

Intentar acceder a valores de campo en conjuntos de registros con más de un registro generará error, por lo que este puede ser un problema en los casos en los que no estamos seguros si estamos trabajando con un conjunto de registros singleton. En los métodos diseñados para trabajar sólo con singleton, podemos comprobar esto usando `self.ensure_one()` al principio. Se planteará un error si `self` no es singleton.


#### Tip

Ten en cuenta que un registro vacío también es un singleton.

### Escribir en los registros

Los conjuntos de registros implementan el patrón de registro activo. Esto significa que podemos asignar valores a ellos, y estos cambios se harán persistentes en la base de datos. Esta es una manera intuitiva y conveniente de manipular datos, como se muestra aquí:

```

>>> admin = self.env['res.users'].browse(1)





>>> print admin.name





Administrator





>>> admin.name = 'Superuser'





>>> print admin.name





Superuser


```

Los conjuntos de registros también tienen tres métodos para actuar en sus datos: `create()`, `write()` y `unlink()`.

El método `create()` toma un diccionario para asignar los campos a los valores y devuelve el registro creado. Los valores predeterminados se aplican automáticamente como se espera, lo que se muestra aquí:

```

>>> Partner = self.env['res.partner']





>>> new = Partner.create({'name': 'ACME', 'is_company': True})





>>> print new





res.partner(72,)


```

El método `unlink()` borra los registros en los conjuntos de registros, como se muestra aquí:

```

>>> rec = Partner.search([('name', '=', 'ACME')])





>>> rec.unlink()





True

```

El método `write()` toma un diccionario para asignar campos a valores. Éstos se actualizan en todos los elementos del conjunto de registros y no se devuelve nada, como se muestra aquí:

```
>>> Partner.write({'comment': 'Hello!'})
```

El uso del patrón de registro activo tiene algunas limitaciones; Actualiza sólo un campo a la vez. Por otro lado, el método `write()` puede actualizar varios campos de varios registros al mismo tiempo utilizando una sola instrucción de base de datos. Estas diferencias deben tenerse en cuenta para los casos en que el rendimiento puede ser un problema.

También vale la pena mencionar `copy()` para duplicar un registro existente; Toma eso como un argumento opcional y un diccionario con los valores para escribir en el nuevo registro. Por ejemplo, para crear una nueva copia de usuario desde el usuario de demostración:

```

>>> demo = self.env.ref('base.user_demo')





>>> new = demo.copy({'name': 'Daniel', 'login': 'dr', 'email':''})

```

#### Nota

Recuerda que los campos con el atributo `copy=False` no se copiarán.


### Trabajando con el tiempo y las fechas

Por razones históricas, los conjuntos de registros ORM controlan los valores `date` y `datetime` utilizando sus representaciones de cadenas, en lugar de los objetos `Date` y `Datetime` reales de Python. En la base de datos se almacenan en los campos de fecha, pero las fechas se almacenan en la hora UTC.

+ `odoo.tools.DEFAULT_SERVER_DATE_FORMAT`
+ `odoo.tools.DEFAULT_SERVER_DATETIME_FORMAT`

Corresponden a `%Y-%m-%d and %Y-%m-%d %H:%M:%S` respectivamente.

Para ayudar a manejar fechas, `fields.Date` y `fields.Datetime` proporcionan pocas funciones. Por ejemplo:

```

>>> from odoo import fields





>>> fields.Datetime.now()





'2014-12-08 23:36:09'





>>> fields.Datetime.from_string('2014-12-08 23:36:09')





datetime.datetime(2014, 12, 8, 23, 36, 9)

```

Las fechas y horas son manejadas y almacenadas por el servidor en un formato UTC, que no es consciente de la zona horaria y puede ser diferente de la zona horaria en la que el usuario está trabajando. Debido a esto, podemos hacer uso de algunas otras funciones para ayudarnos a lidiar con esto:

+ `fields.Date.today()` devuelve una cadena con la fecha actual en el formato esperado por el servidor y utilizando UTC como referencia. Esto es adecuado para calcular los valores predeterminados.
+ `fields.Datetime.now()` devuelve una cadena con el datetime actual en el formato esperado por el servidor utilizando UTC como referencia. Esto es adecuado para calcular los valores predeterminados.
+ `fields.Date.context_today(record, timestamp=None)` devuelve una cadena con la fecha actual en el contexto de la sesión. El valor de la zona horaria se toma del contexto del registro, y el parámetro opcional a utilizar es datetime en lugar de la hora actual.
+ `fields.Datetime.context_timestamp(record, timestamp)` convierte un datetime ingenuo (sin zona horaria) en un datetime de zona horaria. La zona horaria se extrae del contexto del registro, de ahí el nombre de la función.

Para facilitar la conversión entre formatos, ambos objetos  `fields.Date` y `fields.Datetime` proporcionan estas funciones:

+ `from_string(value)` convierte una cadena en un objeto date o datetime
+ `to_string(value)` convierte un objeto date o datetime en una cadena en el formato esperado por el servidor

### Operaciones en conjuntos de registros

Los conjuntos de registros admiten operaciones adicionales en ellos. Podemos comprobar si un registro está incluido o no en un conjunto de registros. Si `x` es un conjunto de registros singleton y `my_recordset` es un conjunto de registros que contiene muchos registros, podemos utilizar:

+ `X im my_recordset`
+ `X not en my_recordset`

También están disponibles las siguientes operaciones:

+ `recordset.ids` devuelve la lista con los ID de los elementos del conjunto de registros
+ `recordset.ensure_one()` comprueba si es un registro único (singleton); Si no es así, se genera una excepción `ValueError`
+ `recordset.filtered(func)` devuelve un conjunto de registros filtrado
+ `recordset.mapped(func)` devuelve una lista de valores mapeados
+ `recordset.sorted(func)` devuelve un conjunto de registros ordenado

Estos son algunos ejemplos de uso para estas funciones:

```

>>> rs0 = self.env['res.partner'].search([])





>>> len(rs0)  # how many records?





40





>>> starts_A = lambda r: r.name.startswith('A')





>>> rs1 = rs0.filtered(starts_A)





>>> print rs1





res.partner(8, 7, 19, 30, 3)





>>> rs2 = rs1.filtered('is_company')





>>> print rs2





res.partner(8, 7)





>>> rs2.mapped('name')





[u'Agrolait', u'ASUSTeK']





>>> rs2.mapped(lambda r: (r.id, r.name))





[(8, u'Agrolait'), (7, u'ASUSTeK')]





>> rs2.sorted(key=lambda r: r.id, reverse=True)





res.partner(8, 7)

```

### Manipulando conjuntos de registros

Seguramente queremos añadir, eliminar o reemplazar los elementos en estos campos relacionados, y esto nos lleva a la pregunta: ¿cómo se pueden manipular los conjuntos de registros?

Los conjuntos de registros son inmutables, lo que significa que sus valores no pueden modificarse directamente. En su lugar, modificar un conjunto de registros significa componer un nuevo conjunto de registros basado en los existentes.

Una forma de hacerlo es utilizando las operaciones de conjunto compatibles:

+ `rs1 | rs2` es la operación de conjunto de **unión** y da como resultado un conjunto de registros con todos los elementos de ambos conjuntos de registros.
+ `rs1 + rs2` es la operación de conjunto de **adiciones**, para concatenar ambos conjuntos de registros en uno. Puede resultar en un conjunto con registros duplicados.
+ `rs1 & rs2` es la operación de conjunto de **intersección** y resulta en un conjunto de registros con sólo los elementos presentes en ambos conjuntos de registros.
+ `rs1 - rs2` es la operación de conjunto de **diferencias** y resulta en un conjunto de registros con los elementos rs1 no presentes en rs2

La notación de corte también se puede utilizar, como se muestra en estos ejemplos:

+ `rs[0]` y `rs[-1]` recuperan el primer elemento y el último elemento, respectivamente.
+ `rs[1:]` da como resultado una copia del conjunto de registros sin el primer elemento. Esto produce los mismos registros que `rs - rs[0]` pero conserva su orden.

#### Nota

En Odoo 10, la manipulación del conjunto de registros conserva el orden. Esto es diferente a las versiones anteriores de Odoo, donde la manipulación del conjunto de registros no estaba garantizada para conservar el orden, aunque se sabe que la adición y el corte mantienen el orden de los registros.

Podemos utilizar estas operaciones para cambiar un conjunto de registros mediante la eliminación o la adición de elementos. Aquí hay unos ejemplos:

+ `self.task_ids |= task1` agrega el registro `task1`, si no en el conjunto de registros
+ `self.task_ids -= task1` elimina el registro específico `task1`, si está presente en el conjunto de registros
+ `self.task_ids = self.task_ids[:-1]` elimina el último registro

Los campos relacionales contienen valores de conjunto de registros. Campos muchos-a-uno pueden contener un conjunto de registros singleton y campos a-muchos contienen conjuntos de registros con cualquier número de registros. Establecemos valores en ellos mediante una sentencia de asignación regular, o utilizando los métodos `create()` y `write()` con un diccionario de valores. En este último caso, se utiliza una sintaxis especial para modificar muchos campos. Es el mismo que se utiliza en los registros XML para proporcionar valores para los campos relacionales y se describe en el capítulo 4, *Datos del módulo*, en la sección *Configurando valores para los campos de relación*.

Como ejemplo, la sintaxis `write()` equivalente a los tres ejemplos anteriores de asignación es:

+ `self.write([(4, task1.id, None)])` agrega el registro `task1`
+ `self.write([(3, task1.id, None)])` elimina `task1` del conjunto de registros
+ `self.write([(3, self.task_ids [-1].id, False)])` elimina el último registro

### Usando campos relacionales

Como vimos anteriormente, los modelos pueden tener campos relacionales: **muchos-a-uno**, **uno-a-muchos** y **muchos-a-muchos**. Estos tipos de campo tienen conjuntos de registros como valores.

En el caso de muchos-a-uno, el valor puede ser un singleton o un conjunto de registros vacío. En ambos casos, podemos acceder directamente a sus valores de campo. Por ejemplo, las siguientes instrucciones son correctas y seguras:

```

>>> self.company_id





res.company(1,)





>>> self.company_id.name





u'YourCompany'





>>> self.company_id.currency_id





res.currency(1,)






>>> self.company_id.currency_id.name





u'EUR'

```

Convenientemente, un conjunto de registros vacío también se comporta como singleton, y el acceso a sus campos no devuelve un error, pero simplemente devuelve `False`. Debido a esto, podemos recorrer registros usando la notación de puntos sin preocuparnos por errores de valores vacíos, como se muestra aquí:

```

>>> self.company_id.country_id





res.country()





>>> self.company_id.country_id.name





False

```

### Trabajando con campos relacionales

Mientras se utiliza el patrón de registro activo, se pueden asignar conjuntos de registros a los campos relacionales.

Para campos muchos-a-uno, el valor asignado debe ser un registro único (un conjunto de registros singleton).

Para los campos a-muchos, su valor también se puede asignar con un conjunto de registros, reemplazando la lista de registros vinculados, si los hay, por uno nuevo. Aquí se permite un conjunto de registros con cualquier tamaño.

Al utilizar los métodos `create()` o `write()`, donde los valores se asignan mediante diccionarios, no se pueden asignar campos relacionales a los valores del conjunto de registros. Debe utilizarse el ID o la lista de identificadores correspondientes.

Por ejemplo, en lugar de `self.write({'user_id': self.env. user })`, deberíamos usar `self.write({'user_id': self.env. user.id })`.

## Resumen

En los capítulos anteriores, vimos cómo construir modelos y diseñar vistas. Aquí fuimos un poco más lejos, aprendiendo a implementar la lógica de negocios y usar conjuntos de registros para manipular datos de modelos.

También vimos cómo la lógica de negocio puede interactuar con la interfaz de usuario y aprendido a crear asistentes que se comunican con el usuario y servir como una plataforma para lanzar procesos avanzados.

En el próximo capítulo, aprenderemos sobre la adición de pruebas automatizadas para nuestro módulo addon, y algunas técnicas de depuración.
