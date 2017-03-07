#Capítulo 2. Creación de su primera aplicación Odoo
Desarrollar en Odoo la mayor parte del tiempo significa crear nuestros propios módulos. En este capítulo, crearemos nuestra primera aplicación Odoo y aprenderemos los pasos necesarios para ponerla a disposición de Odoo e instalarla.

Inspirado por el notable proyecto http://todomvc.com/, vamos a construir una simple aplicación de tareas pendientes. Debería permitirnos agregar nuevas tareas, marcarlas como completadas y, finalmente, borrar la lista de tareas de todas las tareas ya completadas.

Comenzaremos aprendiendo los conceptos básicos del desarrollo de un flujo de trabajo: configura una nueva instancia para tu trabajo, crea e instalA un nuevo módulo y actualízalo para aplicar los cambios que realices junto con las iteraciones de desarrollo.

Odoo sigue una arquitectura similar a MVC, y pasaremos por las capas durante nuestra implementación de la aplicación de tareas pendientes:

+ La capa del **modelo**, que define la estructura de los datos de la aplicación
+ La capa de ***vista**, que describe la interfaz de usuario
+ La capa del **controlador**, que soporta la lógica de negocio de la aplicación

A continuación, aprenderemos cómo configurar la seguridad de control de acceso y, finalmente, agregaremos información sobre la descripción y la marca al módulo.

####Nota
Ten en cuenta que el concepto del término controlador mencionado aquí es diferente de los controladores de desarrollo web Odoo. Estos son puntos finales del programa que las páginas web pueden llamar para realizar acciones.

Con este enfoque, podrás aprender gradualmente sobre los bloques básicos de construcción que conforman una aplicación y experimentar el proceso iterativo de construir un módulo Odoo desde cero.

##Conceptos esenciales
Es probable que estés empezando con Odoo, así que ahora es obviamente un buen momento para explicar los módulos de Odoo y cómo se utilizan en un desarrollo Odoo.

###Descripción de aplicaciones y módulos
Es común oír hablar de los módulos y aplicaciones Odoo. Pero, ¿cuál es exactamente la diferencia entre ellos?

Los **Complementos de Módulos** son los componentes básicos para las aplicaciones Odoo. Un módulo puede agregar nuevas características a Odoo, o modificar las existentes. Es un directorio que contiene un manifiesto, o archivo descriptor, llamado `__manifest__.py`, más los archivos restantes que implementan sus características.

Las **Aplicaciones** son la forma en que se añaden las principales características a Odoo. Proporcionan los elementos básicos para un área funcional, como Contabilidad o RH, en función de qué características de módulos complementarios modifican o amplían. Debido a esto, se destacan en el menú **Apps** de Odoo.

Si su módulo es complejo y agrega funcionalidad nueva o mayor a Odoo, podrías considerar crearlo como una aplicación. Si tu módulo sólo hace cambios a la funcionalidad existente en Odoo, es probable que no sea una aplicación.

Si un módulo es una aplicación o no, se define en el manifiesto. Técnicamente no tiene ningún efecto particular sobre cómo se comporta el módulo addon. Sólo se utiliza para resaltar en la lista de **Aplicaciones**.

###Modificando y extendiendo módulos
En el ejemplo que vamos a seguir, crearemos un nuevo módulo con el menor número posible de dependencias.

Sin embargo, este no será el caso típico. Principalmente, modificaremos o extenderemos un módulo ya existente.

Como regla general, se considera una mala práctica modificar los módulos existentes al cambiar su código fuente directamente. Esto es especialmente cierto para los módulos oficiales proporcionados por Odoo. Hacerlo no te permite tener una separación clara entre el código del módulo original y las modificaciones, y esto hace que sea difícil aplicar actualizaciones ya que sobrescribirían las modificaciones.

En su lugar, debemos crear los módulos de extensión que se instalarán junto a los módulos que queremos modificar, implementando los cambios que necesitamos. De hecho, uno de los principales puntos fuertes de Odoo es el mecanismo de **herencia**, que permite módulos personalizados para extender los módulos existentes, ya sea oficialmente o desde la comunidad. La herencia es posible en todos los niveles: modelos de datos, lógica empresarial y capas de interfaz de usuario.

En este capítulo, crearemos un módulo completamente nuevo, sin extender ningún módulo existente, para enfocarnos en las diferentes partes y pasos involucrados en la creación del módulo. Vamos a tener sólo una breve mirada a cada parte ya que cada uno de ellos será estudiado con más detalle en los capítulos posteriores.

Una vez que estemos cómodos con la creación de un nuevo módulo, podemos sumergirnos en el mecanismo de herencia, que será introducido en el Capítulo 3, *Herencia - Extendiendo Aplicaciones Existentes*.

Para obtener desarrollo productivo para Odoo debemos estar cómodos con el flujo de trabajo de desarrollo: administrar el entorno de desarrollo, aplicar cambios de código y comprobar los resultados. Esta sección le guiará a través de estos fundamentos.

###Creando el esqueleto básico del módulo
Siguiendo las instrucciones del Capítulo 1, *Iniciando con desarrollo Odoo*, deberíamos tener el servidor Odoo en `~ / odoo-dev / odoo /`. Para mantener las cosas ordenadas, crearemos un nuevo directorio junto con él para alojar nuestros módulos personalizados, en `~ / odoo-dev / custom-addons`.

Odoo incluye un comando `scaffold` para crear automáticamente un nuevo directorio de módulo, con una estructura básica ya establecida. Puedes obtener más información al respecto con el siguiente comando:
```
$ ~/odoo-dev/odoo/odoo-bin scaffold --help
```

Es posible que desees tener esto en cuenta cuando empieces a trabajar en tu próximo módulo, pero no lo usaremos ahora, ya que preferiremos crear manualmente toda la estructura de nuestro módulo.

Un módulo addon Odoo es un directorio que contiene un archivo descriptor `__manifest__.py`.

####Nota
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

###Una palabra sobre las licencias
Elegir una licencia para tu trabajo es muy importante, y debes considerar cuidadosamente cuál es la mejor opción para tí y sus implicaciones. Las licencias más utilizadas para los módulos Odoo son la **Licencia Pública General Menor de GNU (LGLP¨** y la **Licencia Pública General de Affero (AGPL)**. La LGPL es más permisiva y permite el trabajo derivado comercial, sin la necesidad de compartir el código fuente correspondiente. La AGPL es una licencia de código abierto más fuerte, y requiere trabajo derivado y alojamiento de servicio para compartir su código fuente. Obten más información acerca de las licencias GNU en https://www.gnu.org/licenses/.

###Añadiendo a la ruta addons
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

###Instalando el nuevo módulo
En el menú superior de **Aplicaciones**, seleccione la opción **Actualizar Lista de Aplicaciones**. Esto actualizará la lista de módulos, añadiendo los módulos que se hayan agregado desde la última actualización a la lista. Recuerda que necesitamos activar el modo desarrollador para que esta opción sea visible. Esto se hace en el panel de **Configuración**, en el enlace de abajo a la derecha, debajo de la información del número de versión de Odoo.

####Tip
Asegúrate de que tu sesión de cliente web está funcionando con la base de datos correcta. Puedes comprobarlo en la parte superior derecha: el nombre de la base de datos se muestra entre paréntesis, justo después del nombre de usuario. Una manera de aplicar la base de datos correcta es iniciar la instancia del servidor con la opción adicional `--db-filter = ^ MYDB $`.

La opción **Aplicaciones** nos muestra la lista de módulos disponibles. De forma predeterminada, muestra sólo los módulos de aplicación. Ya que hemos creado un módulo de aplicación, no necesitamos eliminar ese filtro para verlo. Escribe `todo` en la búsqueda y debes ver nuestro nuevo módulo, listo para ser instalado:

IMAGEN

Ahora haZ clic en el botón **Instalar** del módulo y ¡estamos listos!

###Actualizando un módulo
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

A lo largo del libro, cuando necesites aplicar el trabajo realizado en módulos, la forma más segura es reiniciar la instancia Odoo con el comando anterior. Al presionar la tecla de flecha hacia arriba, se obtiene el comando anterior que se utilizó. Por lo tanto, la mayoría de las veces, te encontrará usando la combinación de teclas Ctrl + C, arriba y Enter.

Desafortunadamente, tanto la actualización de la lista de módulos como la desinstalación de módulos son acciones que no están disponibles a través de la línea de comandos. Estos deben hacerse a través de la interfaz web en el menú de **Aplicaciones**.

###El modo de desarrollo del servidor
En Odoo 10 se introdujo una nueva opción que proporciona características amigables para los desarrolladores. Para usarla, inicia la instancia del servidor con la opción adicional `--dev = all`.
Esto permite que algunas características prácticas aceleren nuestro ciclo de desarrollo. Los más importantes son:
+ Recargar código Python automáticamente, una vez que se guarda un archivo Python, evitando un reinicio manual del servidor
+ Leer las definiciones de vista directamente desde los archivos XML, evitando actualizaciones manuales del módulo

La opción `--dev` acepta una lista de opciones separadas por comas, aunque la opción `all` será adecuada la mayor parte del tiempo. También podemos especificar el depurador que preferimos usar. De forma predeterminada, se utiliza el depurador Python, `pdb`. Algunas personas pueden preferir instalar y usar depuradores alternativos. Aquí también se admiten `ipdb` y `pudb`.



