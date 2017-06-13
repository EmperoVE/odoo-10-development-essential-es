# Capítulo 2. Creación de su primera aplicación Odoo
Desarrollar en Odoo la mayor parte del tiempo significa crear nuestros propios módulos. En este capítulo, crearemos nuestra primera aplicación Odoo y aprenderemos los pasos necesarios para ponerla a disposición de Odoo e instalarla.

Inspirado por el notable proyecto http://todomvc.com/, vamos a construir una simple aplicación de tareas pendientes. Debería permitirnos agregar nuevas tareas, marcarlas como completadas y, finalmente, borrar la lista de tareas de todas las tareas ya completadas.

Comenzaremos aprendiendo los conceptos básicos del desarrollo de un flujo de trabajo: configura una nueva instancia para tu trabajo, crea e instalA un nuevo módulo y actualízalo para aplicar los cambios que realices junto con las iteraciones de desarrollo.

Odoo sigue una arquitectura similar a MVC, y pasaremos por las capas durante nuestra implementación de la aplicación de tareas pendientes:

+ La capa del **modelo**, que define la estructura de los datos de la aplicación
+ La capa de ***vista**, que describe la interfaz de usuario
+ La capa del **controlador**, que soporta la lógica de negocio de la aplicación

A continuación, aprenderemos cómo configurar la seguridad de control de acceso y, finalmente, agregaremos información sobre la descripción y la marca al módulo.

#### Nota
Ten en cuenta que el concepto del término controlador mencionado aquí es diferente de los controladores de desarrollo web Odoo. Estos son puntos finales del programa que las páginas web pueden llamar para realizar acciones.

Con este enfoque, podrás aprender gradualmente sobre los bloques básicos de construcción que conforman una aplicación y experimentar el proceso iterativo de  construir un módulo Odoo desde cero.
 
## Conceptos esenciales
Es probable que estés empezando con Odoo, así que ahora es obviamente un buen momento para explicar los módulos de Odoo y cómo se utilizan en un desarrollo Odoo.

### Descripción de aplicaciones y módulos
Es común oír hablar de los módulos y aplicaciones Odoo. Pero, ¿cuál es exactamente la diferencia entre ellos?

Los **Complementos de Módulos** son los componentes básicos para las aplicaciones Odoo. Un módulo puede agregar nuevas características a Odoo, o modificar las existentes. Es un directorio que contiene un manifiesto, o archivo descriptor, llamado `__manifest__.py`, más los archivos restantes que implementan sus características.

Las **Aplicaciones** son la forma en que se añaden las principales características a Odoo. Proporcionan los elementos básicos para un área funcional, como Contabilidad o RH, en función de qué características de módulos complementarios modifican o amplían. Debido a esto, se destacan en el menú **Apps** de Odoo.

Si su módulo es complejo y agrega funcionalidad nueva o mayor a Odoo, podrías considerar crearlo como una aplicación. Si tu módulo sólo hace cambios a la funcionalidad existente en Odoo, es probable que no sea una aplicación.

Si un módulo es una aplicación o no, se define en el manifiesto. Técnicamente no tiene ningún efecto particular sobre cómo se comporta el módulo addon. Sólo se utiliza para resaltar en la lista de **Aplicaciones**.

###  Modificando y extendiendo módulos
En el ejemplo que vamos a seguir, crearemos un nuevo módulo con el menor número posible de dependencias.

Sin embargo, este no será el caso típico. Principalmente, modificaremos o extenderemos un módulo ya existente.

Como regla general, se considera una mala práctica modificar los módulos existentes al cambiar su código fuente directamente. Esto es especialmente cierto para los módulos oficiales proporcionados por Odoo. Hacerlo no te permite tener una separación clara entre el código del módulo original y las modificaciones, y esto hace que sea difícil aplicar actualizaciones ya que sobrescribirían las modificaciones.

En su lugar, debemos crear los módulos de extensión que se instalarán junto a los módulos que queremos modificar, implementando los cambios que necesitamos. De hecho, uno de los principales puntos fuertes de Odoo es el mecanismo de **herencia**, que permite módulos personalizados para extender los módulos existentes, ya sea oficialmente o desde la comunidad. La herencia es posible en todos los niveles: modelos de datos, lógica empresarial y capas de interfaz de usuario.

En este capítulo, crearemos un módulo completamente nuevo, sin extender ningún módulo existente, para enfocarnos en las diferentes partes y pasos involucrados en la creación del módulo. Vamos a tener sólo una breve mirada a cada parte ya que cada uno de ellos será estudiado con más detalle en los capítulos posteriores.

Una vez que estemos cómodos con la creación de un nuevo módulo, podemos sumergirnos en el mecanismo de herencia, que será introducido en el Capítulo 3, *Herencia - Extendiendo Aplicaciones Existentes*.

Para obtener desarrollo productivo para Odoo debemos estar cómodos con el flujo de trabajo de desarrollo: administrar el entorno de desarrollo, aplicar cambios de código y comprobar los resultados. Esta sección le guiará a través de estos fundamentos.

### Creando el esqueleto básico del módulo
Siguiendo las instrucciones del Capítulo 1, *Iniciando con desarrollo Odoo*, deberíamos tener el servidor Odoo en `~ / odoo-dev / odoo /`. Para mantener las cosas ordenadas, crearemos un nuevo directorio junto con él para alojar nuestros módulos personalizados, en `~ / odoo-dev / custom-addons`.

Odoo incluye un comando `scaffold` para crear automáticamente un nuevo directorio de módulo, con una estructura básica ya establecida. Puedes obtener más información al respecto con el siguiente comando:
```
$ ~/odoo-dev/odoo/odoo-bin scaffold --help
```

Es posible que desees tener esto en cuenta cuando empieces a trabajar en tu próximo módulo, pero no lo usaremos ahora, ya que preferiremos crear manualmente toda la estructura de nuestro módulo.

Un módulo addon Odoo es un directorio que contiene un archivo descriptor `__manifest__.py`.

#### Nota
En versiones anteriores, este archivo descriptor se denominó `__openerp__.py`. Este nombre aún se admite pero está obsoleto.

También necesita ser Python importable, por lo que también debe tener un archivo `__init__.py`.

El nombre del directorio del módulo es su nombre técnico. Usaremos `todo_app` para ello. El nombre técnico debe ser un identificador Python válido: debe comenzar con una letra y sólo puede contener letras, números y el carácter de subrayado.

Los siguientes comandos crearán el directorio del módulo y crearán un archivo  `__init__.py ` vacío en él, `~ / odoo-dev / custom-addons / todo_app / __ init__.py`.

En caso de que desee hacerlo directamente desde la línea de comandos, esto es lo que debes usar:
```
$ mkdir ~/odoo-dev/custom-addons/todo_app

$ touch ~/odoo-dev/custom-addons/todo_app/__init__.py
```

A continuación, necesitamos crear el archivo de manifiesto. Debería contener sólo un diccionario Python con una docena de posibles atributos; De esto, solo se requiere el atributo de `name`. El atributo `description`, para una descripción más larga, y el atributo `author` proporcionan una mejor visibilidad y son recomendados.

Ahora debemos añadir un archivo `__manifest__.py` junto al archivo `__init__.py` con el siguiente contenido:

```

{ 
    'name': 'To-Do Application', 
    'description': 'Manage your personal
    To-Do 
    tasks.', 
    'author': 'Daniel Reis', 
    'depends': ['base'], 
    'application': True, 
}

```
El atributo `depends` puede tener una lista de otros módulos que se requieren. Odoo los instalará automáticamente cuando este módulo esté instalado. No es un atributo obligatorio, pero se aconseja tenerlo siempre. Si no se necesitan dependencias en particular, debemos depender del módulo básico `base`.

Debes tener cuidado de asegurarte de que todas las dependencias se establecen explícitamente aquí; De lo contrario, el módulo puede fallar al instalar en una base de datos limpia (debido a las dependencias que faltan) o tener errores de carga si por casualidad los otros módulos necesarios se cargan después.

Para nuestra aplicación, no necesitamos dependencias específicas, por lo que dependemos únicamente del módulo `base`.

Para ser conciso, elegimos utilizar muy pocas claves de descriptor, pero, en un escenario real, te recomendamos que también uses las claves adicionales, ya que son relevantes para la tienda de aplicaciones Odoo.

+ `summary` se muestra como un subtítulo para el módulo.
+ `version`, por defecto, es 1.0. Debe seguir reglas de versiones semánticas (vea http://semver.org/ para más detalles).
+ El Identificador de `license`, por defecto es `LGPL-3`.
+ `website` es una URL para encontrar más información sobre el módulo. Esto puede ayudar a la gente a encontrar más documentación o al rastreador de incidencias para registrar bugs y sugerencias.
+ `category` es la categoría funcional del módulo, que por defecto es `Uncategorized`. La lista de categorías existentes se encuentra en el formulario grupos de seguridad (**Settings | User | Groups**), en la lista desplegable del campo **Application**.

Estas otras teclas descriptoras también están disponibles:

+ `installable` es por defecto `True` pero se puede establecer como `False` para deshabilitar un módulo.
+ `Auto_install` si se establece en `True`, este módulo se instalará automáticamente, siempre que todas sus dependencias ya estén instaladas. Se utiliza para los módulos de pegamento.

Desde Odoo 8.0, en lugar de la clave de `description`, podemos utilizar un archivo `README.rst` o `README.md` en el directorio superior del módulo.

### Una palabra sobre las licencias
Elegir una licencia para tu trabajo es muy importante, y debes considerar cuidadosamente cuál es la mejor opción para tí y sus implicaciones. Las licencias más utilizadas para los módulos Odoo son la **Licencia Pública General Menor de GNU (LGLP¨** y la **Licencia Pública General de Affero (AGPL)**. La LGPL es más permisiva y permite el trabajo derivado comercial, sin la necesidad de compartir el código fuente correspondiente. La AGPL es una licencia de código abierto más fuerte, y requiere trabajo derivado y alojamiento de servicio para compartir su código fuente. Obten más información acerca de las licencias GNU en https://www.gnu.org/licenses/.

### Añadiendo a la ruta addons
Ahora que tenemos un nuevo módulo minimalista, queremos ponerlo a disposición de la instancia de Odoo.

Para ello, debemos asegurarnos de que el directorio que contiene el módulo está en la ruta addons, entonces actualiza la lista de módulos Odoo.

Ambas acciones se han explicado en detalle en el capítulo anterior, pero aquí, continuaremos con un breve resumen de lo que se necesita.

Posicionaremos en nuestro directorio de trabajo e iniciaremos el servidor con la configuración de ruta de addons apropiada:

```
$ cd ~/odoo-dev


$ ./odoo/odoo-bin -d todo --addons-path="custom-addons,odoo/addons" --save
```
La opción `--save` guarda las opciones que utilizaste en un archivo de configuración. Esto nos evita repetirlas cada vez que reiniciamos el servidor: solo se ejecuta `./odoo-bin` y se utilizará la última opción guardada.

Observa atentamente el registro del servidor. Debe tener una línea `INFO? Odoo: addons paths: [...]`. Debe incluir nuestro directorio de `custom-addons`.

Recuerda incluir también cualquier otro directorio de complementos que puedas estar utilizando. Por ejemplo, si también tienes un directorio `~ / odoo-dev / extra` que contiene módulos adicionales que se utilizarán, es posible que desees incluirlos también utilizando la opción `--addons-path`:
```

--addons-path = "custom-addons, extra, odoo / addons"

```


Ahora necesitamos la instancia Odoo para reconocer el nuevo módulo que acabamos de agregar.

### Instalando el nuevo módulo
En el menú superior de **Aplicaciones**, seleccione la opción **Actualizar Lista de Aplicaciones**. Esto actualizará la lista de módulos, añadiendo los módulos que se hayan agregado desde la última actualización a la lista. Recuerda que necesitamos activar el modo desarrollador para que esta opción sea visible. Esto se hace en el panel de **Configuración**, en el enlace de abajo a la derecha, debajo de la información del número de versión de Odoo.

#### Tip
Asegúrate de que tu sesión de cliente web está funcionando con la base de datos correcta. Puedes comprobarlo en la parte superior derecha: el nombre de la base de datos se muestra entre paréntesis, justo después del nombre de usuario. Una manera de aplicar la base de datos correcta es iniciar la instancia del servidor con la opción adicional `--db-filter = ^ MYDB $`.

La opción **Aplicaciones** nos muestra la lista de módulos disponibles. De forma predeterminada, muestra sólo los módulos de aplicación. Ya que hemos creado un módulo de aplicación, no necesitamos eliminar ese filtro para verlo. Escribe `todo` en la búsqueda y debes ver nuestro nuevo módulo, listo para ser instalado:

![Installed]file:img/2-01.jpg)

Ahora haZ clic en el botón **Instalar** del módulo y ¡estamos listos!

### Actualizando un módulo
El desarrollo de un módulo es un proceso iterativo, y querrás que los cambios hechos en los archivos fuente sean aplicados y hechos visibles en Odoo.

En muchos casos, esto se realiza actualizando el módulo: busca el módulo en la lista de **Aplicaciones** y una vez que ya esté instalado, tendrás disponible un botón de **Actualización**.

Sin embargo, cuando los cambios son sólo en código Python, la actualización puede no tener un efecto. En lugar de una actualización de módulo, es necesario reiniciar el servidor de aplicaciones. Dado que Odoo carga el código Python sólo una vez, cualquier cambio posterior en el código requiere que se aplique un reinicio del servidor.

En algunos casos, si los cambios de módulo estuvieran en los archivos de datos y en el código de Python, es posible que necesites ambas operaciones. Esta es una fuente común de confusión para los nuevos desarrolladores Odoo.
Pero afortunadamente, hay una mejor manera. La manera más segura y rápida de hacer que todos nuestros cambios en un módulo sean efectivos es detener y reiniciar la instancia del servidor, solicitando que nuestros módulos sean actualizados a nuestra base de datos de trabajo.

En el terminal donde se ejecuta la instancia de servidor, utiliza **Ctrl + C** para detenerla. A continuación, inicie el servidor y actualice el módulo `todo_app` mediante el siguiente comando:

```
$ ./odoo-bin -d todo -u todo_app

```


La opción `-u` (o `--update` en el forma larga) requiere la opción `-d` y acepta una lista de módulos separados por comas para actualizar. Por ejemplo, podríamos usar `-u todo_app, mail`. Cuando se actualiza un módulo, también se actualizan todos los módulos instalados que dependen de él. Esto es esencial para mantener la integridad de los mecanismos de herencia, utilizados para extender  características.

A lo largo del libro, cuando necesites aplicar el trabajo realizado en módulos, la forma más segura es reiniciar la instancia Odoo con el comando anterior. Al presionar la tecla de flecha hacia arriba, se obtiene el comando anterior que se utilizó. Por lo tanto, la mayoría de las veces, te encontrará usando la combinación de teclas _**Ctrl + C**_, arriba y _**Enter**_.

Desafortunadamente, tanto la actualización de la lista de módulos como la desinstalación de módulos son acciones que no están disponibles a través de la línea de comandos. Estos deben hacerse a través de la interfaz web en el menú de **Aplicaciones**.

### El modo de desarrollo del servidor
En Odoo 10 se introdujo una nueva opción que proporciona características amigables para los desarrolladores. Para usarla, inicia la instancia del servidor con la opción adicional `--dev = all`.
Esto permite que algunas características prácticas aceleren nuestro ciclo de desarrollo. Los más importantes son:
+ Recargar código Python automáticamente, una vez que se guarda un archivo Python, evitando un reinicio manual del servidor
+ Leer las definiciones de vista directamente desde los archivos XML, evitando actualizaciones manuales del módulo

La opción `--dev` acepta una lista de opciones separadas por comas, aunque la opción `all` será adecuada la mayor parte del tiempo. También podemos especificar el depurador que preferimos usar. De forma predeterminada, se utiliza el depurador Python, `pdb`. Algunas personas pueden preferir instalar y usar depuradores alternativos. Aquí también se admiten `ipdb` y `pudb`.

## La capa modelo
Ahora que Odoo conoce nuestro nuevo módulo, comencemos agregándole un modelo simple.

Los modelos describen objetos de negocio, como una oportunidad, ordenes de clientes o socios (cliente, proveedor, etc.). Un modelo tiene una lista de atributos y también puede definir su negocio específico.

Los modelos se implementan utilizando una clase Python derivada de una clase de plantilla Odoo. Se traducen directamente a objetos de base de datos, y Odoo se encarga de esto automáticamente al instalar o actualizar el módulo. El mecanismo responsable de esto es el **Modelo Relacional de Objetos (ORM)**.

Nuestro módulo será una aplicación muy simple para mantener las tareas pendientes. Estas tareas tendrán un solo campo de texto para la descripción y una casilla de verificación para marcarlas como completas. Más adelante deberíamos añadir un botón para limpiar la lista de tareas de las tareas completas.

### Creando el modelo de datos
Las directrices de desarrollo de Odoo establecen que los archivos Python para los modelos deben colocarse dentro de un subdirectorio `models`. Para simplificar, no lo seguiremos aquí, así que vamos a crar un archivo `todo_model.py` en el directorio principal del módulo `todo_app`.

Añade el siguiente contenido:

``` 
# -*- coding: utf-8 -*- 
from odoo import models, fields 
class TodoTask(models.Model): 
    _name = 'todo.task' 
    _description = 'To-do Task'
    name = fields.Char('Description', required=True) 
    is_done = fields.Boolean('Done?') 
    active = fields.Boolean('Active?', default=True)

```
La primera línea es un marcador especial que indica al intérprete de Python que este archivo tiene UTF-8 para que pueda esperar y manejar caracteres no ASCII. No usaremos ninguno, pero es una buena práctica tenerlo de todos modos.

La segunda línea es una instrucción de importación de código Python, haciendo disponibles los objetos  `models` y  `fields` del núcleo Odoo.

La tercera línea declara nuestro nuevo modelo. Es una clase derivada de `models.Model`.

La siguiente línea establece el atributo `_name` que define el identificador que se utilizará en Odoo para referirse a este modelo. Toma en cuenta que el nombre real de la clase Python, `TodoTask` en este caso, carece de significado para otros módulos Odoo. El valor `_name` es lo que se utilizará como identificador.

Observa que esta y las siguientes líneas tienen sangría. Si no estás familiarizado con Python, debes saber que esto es importante: la sangría define un bloque de código anidado, por lo que estas cuatro líneas deben tener estar todas igual sangría.

Luego, tenemos el atributo modelo `_description `. No es obligatorio, pero proporciona un nombre fácil de usar para los registros del modelo, que puede utilizarse para mejores mensajes de usuario.

Las tres últimas líneas definen los campos del modelo. Vale la pena señalar que `name` y `active` son nombres de campos especiales. De forma predeterminada, Odoo usará el campo de `name` como el título del registro al referenciarlo de otros modelos. El campo `active` se utiliza para inactivar los registros y, por defecto, sólo los registros activos serán mostrados. Lo utilizaremos para borrar las tareas completadas sin eliminarlas de la base de datos.

En este momento, este archivo aún no es utilizado por el módulo. Debemos decirle a Python que lo cargue con el módulo en el archivo `__init__.py`. Vamos a editarlo para agregar la siguiente línea:
```
from . Importar todo_modelo
```
¡Eso es! Para que nuestros cambios de código de Python entren en vigor, la instancia de servidor debe reiniciarse (a menos que esté utilizando el modo `--dev`).

No veremos ninguna opción de menú para acceder a este nuevo modelo ya que no los hemos añadido aún. Sin embargo, podemos inspeccionar el modelo recién creado usando el menú **Technical**. En el menú superior **Settings**, ve a **Technical | Database Structure | Models**, busca el modelo `todo.task` en la lista y haz clic en él para ver su definición:

![Settings](file:img/2-02.jpg)

Si todo va bien, se confirma que el modelo y los campos fueron creados. Si no puedes verlos aquí, intenta reiniciar el servidor con una actualización de módulo, como se describió anteriormente.

También podemos ver algunos campos adicionales que no declaramos. Estos son campos reservados que Odoo agrega automáticamente a cada modelo nuevo. Estos son los siguientes:

+ `id` es un identificador numérico único para cada registro del modelo.
+ `create_date` y `create_uid` especifican cuándo se creó el registro y quién lo creó respectivamente.
+ `write_date` y `write_uid` confirman cuándo el registro fue modificado por última vez y quien lo modificó respectivamente.
+  `__last_update` es un ayudante que en realidad no se almacena en la base de datos. Se utiliza para verificaciones de concurrencia.

### Añadiendo pruebas automatizadas
Las mejores prácticas de programación incluyen tener pruebas automatizadas para tu código. Esto es aún más importante para lenguajes dinámicos como Python. Como no hay ningún paso de compilación, no puede estar seguro de que no haya errores sintácticos hasta que el intérprete realmente  ejecute el código. Un buen editor puede ayudarnos a detectar estos problemas con antelación, pero no puede ayudarnos a asegurar que el código se ejecute como lo desean las pruebas automatizadas.

Odoo soporta dos formas de describir las pruebas: ya sea utilizando archivos de datos YAML o utilizando código Python, basado en la biblioteca `Unittest2`. Las pruebas YAML son un legado de versiones anteriores, y no se recomiendan. Preferiremos usar pruebas de Python y añadiremos un caso básico de prueba a nuestro módulo.


Los archivos de código de prueba deben tener un nombre que empiece por `test_` y se debe importar desde `tests / __ init__.py`. Pero el directorio de `test` (o submódulo Python) no se debe importar desde la parte superior del módulo  `__init__.py`, ya que se descubrirá y cargará automáticamente sólo cuando se ejecuten pruebas.

Las pruebas deben colocarse en un subdirectorio `test/`. Añade un archivo `tests / __ init__.py` con lo siguiente:
```
from . import test_todo

```
Ahora, añade el código de prueba real disponíble en el archivo  `tests/test_todo.py`:
```
# -*- coding: utf-8 -*- 
from odoo.tests.common import TransactionCase 
 
class TestTodo(TransactionCase): 
 
    def test_create(self): 
        "Create a simple Todo" 
        Todo = self.env['todo.task'] 
        task = Todo.create({'name': 'Test Task'}) 
        self.assertEqual(task.is_done, False)

```
Esto agrega un caso simple de prueba para crear una nueva tarea y verifica que el campo ** Is Done?** Tiene el valor predeterminado correcto.

Ahora queremos hacer nuestras pruebas. Esto se hace agregando la opción `--test-enable` durante la instalación del módulo:

```

$ ./odoo-bin -d todo -i todo_app --test-enable

```



El servidor Odoo buscará un subdirectorio tests/ en los módulos actualizados y los ejecutará. Si alguna de las pruebas falla, el registro del servidor te mostrará eso.

## La capa de vista

La capa de vista describe la interfaz de usuario. Las vistas se definen mediante XML, que es utilizado por el marco de cliente web para generar vistas HTML con datos.

Tenemos elementos de menú que pueden activar acciones que pueden hacer vistas. Por ejemplo, la opción de menú **Usuarios** procesa una acción también denominada **Usuarios**, que a su vez genera una serie de vistas. Existen varios tipos de vista disponibles, como las vistas de lista y formulario y las opciones de filtro también disponíbles, están definidas por un tipo particular de vista, la vista de búsqueda.

Las directrices de desarrollo de Odoo establecen que los archivos XML que definen la interfaz de usuario deben colocarse dentro de un subdirectorio `views /` subdirectorio
Comencemos a crear la interfaz de usuario para nuestra aplicación de tareas pendientes.
###Agregar elementos de menú

Ahora que tenemos un modelo para almacenar nuestros datos, debemos hacerlo disponible en la interfaz de usuario.

Para ello, debemos añadir una opción de menú para abrir el modelo `To–do Task` para que pueda utilizarse.

Cree el archivo `views / todo_menu.xml` para definir un elemento de menú y la acción realizada por él:


```

<?xml version="1.0"?> 
<odoo> 
  <!-- Action to open To-do Task list --> 
  <act_window id="action_todo_task" 
    name="To-do Task" 
    res_model="todo.task" 
    view_mode="tree,form" /> 
  <!-- Menu item to open To-do Task list --> 
  <menuitem id="menu_todo_task" 
    name="Todos" 
    action="action_todo_task" /> 
</odoo> 

```

La interfaz de usuario, incluidas las opciones y las acciones de menú, se almacena en las tablas de la base de datos. El archivo XML es un archivo de datos utilizado para cargar esas definiciones en la base de datos cuando el módulo se instala o actualiza. El código anterior es un archivo de datos Odoo, que describe dos registros para añadir a Odoo:

+ El elemento `<act_window>` define una acción de ventana del lado del cliente que abrirá el modelo `todo.task` con las vistas de árbol y formulario habilitadas, en ese orden.
+ El `<menuitem>` define un elemento de menú superior que llama a la acción `action_todo_task`, que se definió anteriormente.

Ambos elementos incluyen un atributo id. Este atributo id también llamado **XML ID**, es muy importante: se utiliza para identificar de forma única cada elemento de datos dentro del módulo, y puede ser utilizado por otros elementos para referenciarlo. En este caso, el elemento `<menuitem>` necesita hacer referencia a la acción para procesar, y necesita hacer uso de la <act_window> ID para eso. Los ID XML se tratan con mayor detalle en el Capítulo 4, *Datos del módulo*

Nuestro módulo aún no conoce el nuevo archivo de datos XML. Esto se hace agregándolo al atributo de datos en el archivo `__manifest__.py`. Este, contiene la lista de archivos a cargar por el módulo. Agregue este atributo al diccionario del manifiesto:
```

'Data': ['views / todo_menu.xml'],
```

Ahora necesitamos actualizar el módulo de nuevo para que estos cambios surtan efecto. Vaya al menú superior de **Todos** y debe ver nuestra nueva opción de menú disponible:

![Save](file:img/2-03.jpg)

Aunque no hemos definido nuestra vista de interfaz de usuario, al hacer clic en el menú **Todos** se abrirá un formulario generado automáticamente para nuestro modelo, lo que nos permitirá agregar y editar registros.

Odoo es lo suficientemente agradable como para generarlos automáticamente para que podamos empezar a trabajar con nuestro modelo de inmediato.

¡Hasta aquí todo bien! Vamos a mejorar nuestra interfaz de usuario ahora. Trata de hacer mejoras graduales como se muestra en las próximas secciones, haciendo actualizaciones de módulos frecuentes, y no tengas miedo de experimentar. También puedes intentar la opción de servidor `--dev = all`. Usándolo, las definiciones de vista se leen directamente desde los archivos XML para que tus cambios puedan estar inmediatamente disponibles para Odoo sin necesidad de una actualización de módulo.

### Tip

Si una actualización falla debido a un error de XML, no te preocupe! Comenta las últimas porciones XML editadas o elimina el archivo XML de `__manifest__.py` y repita la actualización. El servidor debe iniciarse correctamente. Ahora lee el mensaje de error en el registro del servidor con cuidado: debe señalarte dónde está el problema.

Odoo admite varios tipos de vistas, pero las tres más importantes son: `tree` (generalmente llamado vistas de lista), `form` y `search views`. Vamos a añadir un ejemplo de cada uno a nuestro módulo.
### Creando la vista de formulario

Todas las vistas se almacenan en la base de datos, en el modelo `ir.ui.view`. Para añadir una vista a un módulo, declaramos un elemento `<record>` que describe la vista en un archivo XML, que se va a cargar en la base de datos cuando se instala el módulo.

Agregue este nuevo archivo `views / todo_view.xml` para definir nuestra vista de formulario:
```
<?xml version="1.0"?> 
<odoo> 
  <record id="view_form_todo_task" model="ir.ui.view"> 
    <field name="name">To-do Task Form</field> 
    <field name="model">todo.task</field> 
    <field name="arch" type="xml"> 
     
 <form string="To-do Task"> 





        <group>
          <field name="name"/> 
          <field name="is_done"/> 
          <field name="active" readonly="1"/> 





        </group> 
      </form>



 
    </field> 
  </record> 
</odoo> 
```
Recuerde agregar este nuevo archivo a la clave de datos en el archivo de manifiesto, de lo contrario, nuestro módulo no lo sabrá y no se cargará.

Esto agregará un registro al modelo `ir.ui.view` con el identificador `view_form_todo_task`. La vista es para el modelo `todo.task` y se denomina `To-do Task Form`. El nombre es solo para información; No tiene que ser único, pero debe permitir que uno identifique fácilmente a qué registro se refiere. De hecho, el nombre puede ser totalmente omitido, en ese caso, se generará automáticamente a partir del nombre del modelo y el tipo de vista.

El atributo más importante es `arch`, y contiene la definición de la vista, resaltada en el código XML anterior. La etiqueta `<form>` define el tipo de vista y, en este caso, contiene tres campos. También agregamos un atributo al campo `active` para que sea solo de lectura.
 
### Vistas del formulario de documento empresarial

La sección anterior proporcionó una vista de formulario básica, pero podemos hacer algunas mejoras en ella. Para los modelos de documentos, Odoo tiene un estilo de presentación que imita una página en papel. Este formulario contiene dos elementos: `<header>` para contener los botones de acción y `<sheet>` para contener los campos de datos.

Ahora podemos reemplazar el `<form>` básico definido en la sección anterior por éste:
```
<header>



 
  <!-- Buttons go here--> 
 
 </header> 






<sheet>



 
    <!-- Content goes here: --> 
    <group>
      <field name="name"/> 
      <field name="is_done"/>
      <field name="active" readonly="1"/>
    </group>

  </sheet>



 
</form> 

```

### Añadiendo botones de acción

Los formularios pueden tener botones para realizar acciones. Estos botones pueden ejecutar acciones de ventana como abrir otro formulario o ejecutar funciones de Python definidas en el modelo.

Pueden colocarse en cualquier lugar dentro de un formulario, pero para los formularios de estilo de documento, el lugar recomendado para ellos es la sección `<header>`.

Para nuestra aplicación, agregaremos dos botones para ejecutar los métodos del modelo `todo.task`:
```

<header> 
  
<button name="do_toggle_done" type="object" 
    string="Toggle Done" class="oe_highlight" /> 
  <button name="do_clear_done" type="object" 
    string="Clear All Done" />



 
</header> 
```


Los atributos básicos de un botón comprenden lo siguiente:

+ `stryng` con el texto a mostrar en el botón
+ `type` de acción que realiza
+ `name` es el identificador de esa acción
+ `class` es un atributo opcional para aplicar estilos CSS, como en HTML normal

### Uso de grupos para organizar formularios

La etiqueta `<group> `te permite organizar el contenido del formulario. Colocar elementos `<group>` dentro de un elemento `<group>` crea un diseño de dos columnas dentro del grupo externo. Se aconseja que los elementos del grupo tengan un atributo de nombre para que sea más fácil para otros módulos extenderlos.

Usaremos esto para organizar mejor nuestro contenido. Cambiemos el contenido `<sheet>` de nuestro formulario para que coincida con este:

```
<sheet> 
  
<group name="group_top"> 
    <group name="group_left">



 
      <field name="name"/> 
    
</group> 
    <group name="group_right">



 
      <field name="is_done"/> 
      <field name="active" readonly="1"/> 
    
</group> 
  </group>



 
</sheet> 
```

### La vista de formulario completa

En este punto, nuestro formulario `todo.task` debe verse así:

```
<form> 
  <header> 
    <button name="do_toggle_done" type="object" 
      string="Toggle Done" class="oe_highlight" /> 
    <button name="do_clear_done" type="object" 
      string="Clear All Done" /> 
  </header> 
  <sheet> 
    <group name="group_top"> 
      <group name="group_left"> 
        <field name="name"/> 
      </group> 
      <group name="group_right"> 
        <field name="is_done"/> 
        <field name="active" readonly="1" /> 
      </group> 
    </group> 
  </sheet> 
</form> 
```
### Tip
Recuerda que para que los cambios se carguen en nuestra base de datos Odoo, se necesita una actualización del módulo. Para ver los cambios en el cliente web, el formulario debe ser recargado: haz clic de nuevo en la opción de menú que lo abre o vuelve a cargar la página del navegador (_**F5**_ en la mayoría de los navegadores).

Los botones de acción no funcionarán aún, ya que todavía necesitamos agregar su lógica de negocio. 
### Adición de vistas de lista y de búsqueda

Cuando se visualiza un modelo en modo de lista, se utiliza una vista `<tree>`. Las vistas de árbol son capaces de mostrar líneas organizadas en jerarquías, pero la mayoría de las veces, se utilizan para mostrar listas sin formato.

Podemos agregar la siguiente definición de vista `tree` a `todo_view.xml`:
```
<record id="view_tree_todo_task" model="ir.ui.view"> 
  <field name="name">To-do Task Tree</field> 
  <field name="model">todo.task</field> 
  <field name="arch" type="xml"> 
    <tree colors="decoration-muted:is_done==True"> 
      <field name="name"/> 
      <field name="is_done"/> 
    </tree> 
  </field> 
</record> 
```

Esto define una lista con sólo dos columnas: `name` y `is_done`. También añadimos un toque agradable: las líneas para las tareas hechas (`is_done == True`) se muestran en gris. Esto se hace aplicando la clase silenciada Bootstrap. Consulta http://getbootstrap.com/css/#helper-classes-colors para obtener más información sobre Bootstrap y sus colores contextuales.

En la esquina superior derecha de la lista, Odoo muestra un cuadro de búsqueda. Los campos que busca y los filtros disponibles se definen mediante una vista `<search>`.

Como antes, agregamos esto a `todo_view.xml`:
```
<record id="view_filter_todo_task" model="ir.ui.view"> 
  <field name="name">To-do Task Filter</field> 
  <field name="model">todo.task</field> 
  <field name="arch" type="xml"> 
   
 <search> 
      <field name="name"/> 
      <filter string="Not Done" 
        domain="[('is_done','=',False)]"/> 
      <filter string="Done" 
        domain="[('is_done','!=',False)]"/> 
    </search>



 
  </field> 
</record> 
```

Los elementos `<field>` definen campos que también se buscan al escribir en el cuadro de búsqueda. Los elementos `<filter>` añaden condiciones de filtro predefinidas, que se pueden alternar con un clic de usuario, definido mediante el uso de una sintaxis específica.

## La capa de lógica de negocio

Ahora vamos a añadir algo de lógica a nuestros botones. Esto se hace con código Python, utilizando los métodos de la clase de modelos Python.
### Añadiendo lógica de negocio

Debemos editar el archivo Python `todo_model.py` para agregar a la clase los métodos llamados por los botones. Primero, necesitamos importar la nueva API, así que agréguala a la declaración de importación en la parte superior del archivo Python:
```
from odoo import models, fields, api
```

La acción del botón **Toggle Done** será muy simple: solo cambia la bandera **Is Done?**. Para la lógica de los registros, utiliza el decorador `@api.multi`. Aquí, `self` representará un conjunto de registros, y entonces deberíamos hacer un bucle a través de cada registro.

Dentro de la clase TodoTask, añade esto:
```
@api.multi 
def do_toggle_done(self): 
    for task in self: 
        task.is_done = not task.is_done 
    return True
```
El código pasa por todos los registros de tarea y, para cada uno, modifica el campo `is_done`, invirtiendo su valor. El método no necesita devolver nada, pero debemos tenerlo al menos para devolver un valor `True`. La razón es que los clientes pueden utilizar XML-RPC para llamar a estos métodos y este protocolo no admite funciones de servidor devolviendo sólo un valor `None`.

Para el botón **Clear All Done**, queremos ir un poco más lejos. Debe buscar todos los registros activos que están hechos, y hacerlos inactivos. Normalmente, se espera que los botones de formulario actúen sólo en el registro seleccionado, pero en este caso, queremos que actúe también en registros distintos del actual:
```
@api.model 
def do_clear_done(self): 
    dones = self.search([('is_done', '=', True)]) 
    dones.write({'active': False}) 
    return True 
```


En los métodos decorados con `@ api.model`, la variable `self` representa el modelo sin registro en particular. Construiremos un conjunto de registros `dones` que contenga todas las tareas marcadas como terminadas. A continuación, establecemos el indicador `active` para `False` en ellos.

El método de búsqueda es un método API que devuelve los registros que cumplen algunas condiciones. Estas condiciones están escritas en un dominio, que es una lista de tripletes. Exploraremos los dominios con más detalle en el Capítulo 6, *Vistas – Diseñando la interfaz de usuario*.

El método `write` establece los valores de una vez en todos los elementos del conjunto de registros. Los valores a escribir se describen utilizando un diccionario. Usar `write here` es más eficiente que iterar a través del conjunto de registros para asignar el valor a cada uno de ellos uno por uno.
### Añadiendo de pruebas

Ahora debemos agregar pruebas para la lógica de negocio. Idealmente, queremos que cada línea de código sea cubierta por al menos un caso de prueba. En `tests / test_todo.py`, agregua unas cuantas líneas más de código al método `test_create ()`:
```
# def test_create(self): 
        # ... 
       
 # Test Toggle Done 
        task.do_toggle_done() 
        self.assertTrue(task.is_done) 
        # Test Clear Done 
        Todo.do_clear_done() 
        self.assertFalse(task.active)

```


Si ahora ejecutamos las pruebas y los métodos del modelo están correctamente escritos, no deberíamos ver ningún mensaje de error en el registro del servidor:

```
$ ./odoo-bin -d todo -i todo_app --test-enable

```
## Configurando la seguridad de acceso

Es posible que haya notado que, al cargar, nuestro módulo recibe un mensaje de advertencia en el registro del servidor:

**The model todo.task has no access rules, consider adding one.**


(**El modelo todo.task no tiene reglas de acceso, considere agregar una.**)




El mensaje es bastante claro: nuestro nuevo modelo no tiene reglas de acceso, por lo que no puede ser utilizado por nadie que no sea el superusuario de admin. Como superusuario, el admin ignora las reglas de acceso a datos, y es por eso que hemos podido utilizar el formulario sin errores. Pero debemos corregir esto antes de que otros usuarios puedan usar nuestro modelo.

Otra cuestión que todavía tenemos que abordar es que queremos que las tareas pendientes sean privadas para cada usuario. Odoo soporta reglas de acceso a nivel de fila, que usaremos para implementar eso.

### Probando la seguridad de acceso

De hecho, nuestras pruebas deben estar fallando en este momento debido a las reglas de acceso que faltan. Ellas no están porque se hacen con el usuario admin. Por lo tanto, debemos cambiarlos para que utilicen el usuario Demo en su lugar.

Para ello, debemos editar el archivo `tests / test_todo.py` para añadir un método `setUp`:
```
# class TestTodo(TransactionCase): 
 
    def setUp(self, *args, **kwargs): 
        result = super(TestTodo, self).setUp(*args, \ 
        **kwargs) 
        user_demo = self.env.ref('base.user_demo') 
        self.env= self.env(user=user_demo) 
        return result 
```


Esta primera instrucción llama al código `setUp` de la clase padre. Los siguientes cambian el entorno utilizado para ejecutar las pruebas, `self.env`, a una nueva usando el usuario `Demo`. No se necesitan más cambios en las pruebas que ya escribimos.

También debemos añadir un caso de prueba para asegurarnos de que los usuarios sólo pueden ver sus propias tareas. Para ello, primero, agregua una importación adicional en la parte superior:
```
from odoo.exceptions import AccessError 

```
A continuación, agregua un método adicional a la clase de prueba:
```
    def test_record_rule(self): 
        "Test per user record rules" 
        Todo = self.env['todo.task'] 
        task = Todo.sudo().create({'name': 'Admin Task'}) 
        with self.assertRaises(AccessError): 
            Todo.browse([task.id]).name 

```

Dado que nuestro método `env` ahora está utilizando el usuario de Demo, usamos el método `sudo ()` para cambiar el contexto al usuario admin. A continuación, lo usamos para crear una tarea que no debería ser accesible para el usuario Demo.

Al intentar acceder a los datos de esta tarea, esperamos que se genere una excepción `AccessError`.

Si ejecutamos las pruebas ahora, deberían fallar, así que nos encargamos de eso.

### Añadiendo seguridad de control de acceso

Para obtener una imagen de qué información se necesita para agregar reglas de acceso a un modelo, utiliza el cliente web y ve a **Settings | Technical | Security | Access Controls List** :

![Create](file:img/2-04.jpg)

### Nota

Aquí podemos ver la ACL de algunos modelos. Indica, por grupo de seguridad, qué acciones se permiten en los registros.

Esta información debe ser proporcionada por el módulo utilizando un archivo de datos para cargar las líneas en el modelo `ir.model.access`. Vamos a agregar acceso completo al grupo de empleados en el modelo. El empleado es el grupo básico de acceso al que casi todos pertenecen.

Esto se hace utilizando un archivo CSV denominado `security / ir.model.access.csv`. Vamos a agregarlo con el siguiente contenido:
```
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink 
acess_todo_task_group_user,todo.task.user,model_todo_task,base.group_user,1,1,1,1 

```

El nombre de archivo corresponde al modelo para cargar los datos, y la primera línea del archivo tiene los nombres de columna. Estas son las columnas proporcionadas en nuestro archivo CSV:

+ `id` es el identificador externo del registro (también conocido como XML ID). Debe ser único en nuestro módulo.
+ `name` es un título de descripción. Es sólo informativo y es mejor si se mantiene único. Los módulos oficiales normalmente usan una cadena separada por puntos con el nombre del modelo y el grupo. Siguiendo esta convención, utilizamos `todo.task.user`.
+ `model_id` es el identificador externo del modelo al que estamos dando acceso. Los modelos tienen XML IDs generados automáticamente por el ORM: para `todo.task`, el identificador es `model_todo_task`.
+ `group_id` identifica el grupo de seguridad para dar permisos. Los más importantes son proporcionados por el módulo base. El grupo Empleado es un caso así y tiene el identificador `base.group_user`.
+ Los campos `perm` marcan el acceso a garantizar `read`, `write`, ` create` o `un link` (borrar) el acceso.

No debemos olvidar añadir la referencia a este nuevo archivo en el atributo de datos del descriptor `__manifest__.py`. Debe tener un aspecto como este:
```
'data': [ 
    'security/ir.model.access.csv', 
    'views/todo_view.xml', 
    'views/todo_menu.xml', 
],

```
Como antes, actualice el módulo para que estas adiciones entren en vigor. El mensaje de advertencia debe desaparecer, y podemos confirmar que los permisos están bien iniciando sesión con el usuario `demo` (la contraseña también es `demo`). Si ejecutamos nuestras pruebas ahora solo deberían fallar el caso de prueba `test_record_rule`.
### Reglas de acceso a nivel de fila

Podemos encontrar la opción **Record Rules** en el menú **Technical**, junto con **Access Control List*.

Las reglas de registro se definen en el modelo `ir.rule`. Como de costumbre, necesitamos proporcionar un nombre distintivo. También necesitamos el modelo en el que operan y el filtro de dominio que se utilizará para la restricción de acceso. El filtro de dominio utiliza la lista usual de tuplas sintáctica utilizada en Odoo.

Por lo general, las reglas se aplican a algunos grupos de seguridad en particular. En nuestro caso, lo haremos aplicable al grupo Empleados. Si no se aplica a ningún grupo de seguridad en particular, se considera global (el campo `global` se establece automáticamente en `True`). Las reglas globales son diferentes porque imponen restricciones que las reglas no globales no pueden anular.

Para agregar la regla de registro, debemos crear un archivo `security / todo_access_rules.xml` con el siguiente contenido:
```
<?xml version="1.0" encoding="utf-8"?> 
<odoo> 
  <data noupdate="1"> 
    <record id="todo_task_user_rule" model="ir.rule"> 
      <field name="name">ToDo Tasks only for owner</field> 
      <field name="model_id" ref="model_todo_task"/> 
      <field name="domain_force">
          [('create_uid','=',user.id)] 
      </field> 
      <field name="groups" eval="
      [(4,ref('base.group_user'))]"/> 
    </record> 
  </data> 
</odoo> 

```

### Nota

Observa el atributo `noupdate = "1"`. Significa que estos datos no se actualizarán en actualizaciones de módulos. Esto le permitirá ser personalizado más adelante ya que las actualizaciones de módulos no destruirán los cambios realizados por el usuario. Pero ten en cuenta que esto también será el caso durante el desarrollo, por lo que es posible que desees establecer `noupdate = "0" ` durante el desarrollo hasta que estéss satisfecho con el archivo de datos.

En el campo de grupos, también encontrarás una expresión especial. Es un campo relacional de uno a muchos, y tienen una sintaxis especial para operar. En este caso, la tupla (4, x) indica anexar `x` a los registros, y aquí `x` es una referencia al grupo Empleados, identificado por `base.group_user`. Esta sintaxis especial de escritura de uno-a-muchos se discute con más detalle en el Capítulo 4, *Datos de Módulo*.

Como antes, debemos añadir el archivo a `__manifest__.py` antes de poder cargarlo en el módulo:
```
'data': [ 
  'security/ir.model.access.csv', 
  'security/todo_access_rules.xml', 
  'todo_view.xml', 
  'todo_menu.xml', 
], 

```

Si lo hicimos bien, podemos ejecutar las pruebas de módulo y ahora deben pasar.

## Describiendo mejor el módulo

Nuestro módulo se ve bien. ¿Por qué no añadir un icono para que se vea aún mejor? Para esto, solo necesitamos agregar al módulo un archivo `static / description / icon.png` con el icono que se va a usar.

Estaremos reutilizando el icono de la aplicación existente **Notes**, por lo que deberíamos copiar el archivo `odoo / addons / static / description / icon.png` en el directorio `addons / todo_app / static / description`.

Los siguientes comandos deben hacer ese truco para nosotros:

```
$ mkdir -p ~/odoo-dev/custom-addons/todo_app/static/description
$ cp ~/odoo-dev/odoo/addons/note/static/description/icon.png ~/odoo-dev/custom-addons/todo_app/static/description

```

Ahora, si actualizamos la lista de módulos, nuestro módulo debe mostrarse con el nuevo icono.

También podemos añadir una descripción mejor para explicar lo que hace y lo grandioso que es. Esto se puede hacer en la clave `description` del archivo `__manifest__.py`. Sin embargo, la forma preferida es agregar un archivo `README.rst` al directorio raíz del módulo.

## Resumen

Hemos creado un nuevo módulo desde el principio, cubriendo los elementos más utilizados en un módulo: modelos, los tres tipos básicos de vistas (formulario, lista y búsqueda), lógica empresarial en los métodos de modelo y seguridad de acceso.

En el proceso, nos familiarizamos con el proceso de desarrollo de módulos, que implica actualizaciones de módulos y reinicios del servidor de aplicaciones para hacer que los cambios graduales sean efectivos en Odoo.

Recuerde siempre, cuando se agregan campos del modelo, se necesita una actualización. Al cambiar el código de Python, incluyendo el archivo de manifiesto, se necesita un reinicio. Al cambiar archivos XML o CSV, se necesita una actualización; También, en caso de duda, haz lo siguiente: reinicia el servidor y actualiza los módulos.

En el siguiente capítulo, aprenderás cómo construir módulos que se apilarán en los existentes para agregar características.




