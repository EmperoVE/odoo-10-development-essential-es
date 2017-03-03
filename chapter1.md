#odoo essentials

##Capítulo 1. Iniciando con desarrollo en Odoo

Antes de sumergirnos en el desarrollo en Odoo, necesitamos armar nuestro ambiente de desarrollo y aprender las tareas de administraciión básicas para ello.

Oddo se construye utilizando el lenguaje de programación Phyton, y utiliza la base de datos PostgreSQL para almacenamiento de datos; estos son los dos principales requerimientos de un huesped Odoo. Para correr Odoo desde la fuente, necesitaremos primero instalar las librerias de Phyton de as cuales depende. El código de fuente de Odoo puede posteriormente ser descargado desde GitHub. Mientras podemos descargar un archivo ZIP o tarball, veremos que es mejor si obtenemos las fuentes utilizando la versión Git de aplicación de control.; Nos ayudará a tenerlo instalado en nuestro huesped Odoo también.

##Creando un huesped para el servidor Odoo

Se recomienda para el sistema Debian/Ubuntu para el servidor Odoo. Aún serás capaz de trabajar desde tu sistema de escritorio favorito, bien sea Windows, Mac, o Linux.

Odoo puede correr con gran variedad de sistemas operativos, entonces, por qué escoger Debian a expensas de otros sistemas operativos?: Porque Debian es considerado la plataforma de despliegue de referencia por el equipo Odoo; tiene el mejor soporte. Será más facil hallar ayuda y recursos adicionales si trabajamos con Debian/Ubuntu.

También es la plataforma con la que la mayoría de desarrolladores trabajan y donde se implementan más despliegues. Así que inevitablemente, se espera que los desarrolladores de Odoo se sientan cómodos con la plataforma Debian/Ubuntu. Incluso si tienes un pasado Windows, será importante que tengas algún conocimiento acerca de esto.

En este capítulo, aprenderás cómo armar y trabajar con Odoo hospedado en un sistema Debian, usando sólo la linea de comando. Para aquellos en casa con un sistema Windows, cubriremos cómo configurar una máquina virtual para hospedar el servidor Odoo. Como ventaja, las técnicas que aprenderás aquí también te permitirán administrar Odoo en servidores en la nube, donde tu único acceso será a través de Secure Shel ( SSH).

###Nota

Manten en mente que estas instrucciones están destinadas a organizar un nuevo sistemas para desarrollo. Si deseas probar algunos de ellos en un sistema existente, siempre toma una copia de seguridad antes de tiempo para poder restaurarlo en caso de que algo salga mal. 

##Provisión para un huesped Debian

Como se explicó antes, necesitaremos un huesped basado en Debian para nuestro servidor Odoo. Si estos son tus primeros pasos con Linux, puede gustarte notar que Ubuntu es una distribución Linux basada en Debian, así que son muy similares.

Está garantizado que Odoo trabaje con la versión actual estable de Debian o Ubuntu. Al tiempo de escribir, estos son Debian 8 "Jessie" y Ubuntu 16.04.1 LTS (Xenial Xerus). Ambos, vienen con Python 2.7, el cual es necesario para correr Odoo. Es importante señalar que Odoo no soporta Python 3 aún, así que Python 2 es requerido.

Si ya estás ejecutando Ubuntu o alguna otra distribución con basada en Debian, ¡estas listo!; Esto también puede ser utilizado como un huesped para Odoo.

Para los sitemas operativos Windows y Mac, instala Python, PostgreSQL, y todas las dependencias ; luego, ejecuta Odoo desde la fuente nativa. Sin embargo, esto puede ser un reto, así que nuestro consejo es que uses una máquina virtual que ejecute un servidor Debian o Ubuntu. Eres bienvenido a utilizar tu software de virtualización preferido para obtener un sistema de trabajo Debian en una máquina virtual.

En caso de que necesites alguna guía, aquí hay unos consejos con respecto al software de visualización. Existen varias opciones, tales como Microsoft Hyper-V (disponíble en algunas versiones de Windows recientes), Oracle VirtualBox y VMWare Workstation Player (VMWare Fusion para MAC). La VMWare Workstation Player es probablemente más fácil de utilizar y descargas fáciles de usar pueden ser halladas en https://my.vmware.com/wev/vmware/downloads.

Con respecto a la imagen de Linux a utilizar, será más amigable para el usuario instalar Ubuntu Server que Debian. Si estás empezando con Linux, te recomiendo que pruebes una imagen lista para usar. TurnKey Linux proporciona imágenes preinstaladas fáciles de usar en varios formatos, incluyendo ISO. El formato ISO funcionará con cualquier programa de visualización que elijas, incluso en una máquina de metal desnudo que puedas tener. Una buena opción podría ser la imagen LAPP, que incluye Python y PostgreSQL, y se puede encontrar en http://www.turnkeylinux.org/lapp.

Una vez instalada y arrancada, deberías ser capaz de iniciar sesión en una línea de comando shell.


##Creando una cuenta de usuario para Oddo

Si has iniciado sesión usando la cuenta de super usuario `root`, tu primera tarea debe ser crear una cuenta de usuario normal para tu trabajo, ya que se considera una mala práctica trabajar como `root`. En particular, el operador de Odoo se rehusará a correr si lo inicias como `root`.

Si has inicaido sesión usando Ubuntu, probablemente no necesitaras esto, ya que el proceso de instalación debe haberte guiado para la creación de un usuario.

Primero, asegurate de que `sudo` esté instalado. Nuestro usuario de trabajo lo necesitará. Si se inició sesión como `root`, ejecuta los siguientes comandos:

```
# apt-get update && apt-get upgrade  # Install system updates
# apt-get install sudo  # Make sure 'sudo' is installed

```


El siguiente set de comandos creará un usuario `odoo`:

```
# useradd -m -g sudo -s /bin/bash odoo  # Create an 'odoo' user with sudo powers

# passwd odoo  # Ask and set a password for the new user

```
Puedes cambiar el nombre de usuario  `odoo` al que tu quieras. La opción `-m`asegura que su directorio de inicio sea creado. La opción `-g sudo` 


Ahora podemos iniciar sesión como el nuevo usuario y organizar Odoo.


##Instalando Odoo desde la fuente

Los paquetes de instalación rápida de Odoo, pueden hallarse en nigthly.odoo.com, disponíble como Windows `(.exe)`, Debian (`.deb`), CentOS (`.rpm`), y código fuente en formato tarballs (`. tar .gz`).

Como desarrolladores, preferiremos instalarlos directamente del repositorio GitHub. Esto terminará dándonos más control sobre versiones y actualizaciones.

Para mantener las cosas ordenadas, vamos a trabajar en un directorio `/odoo-dev`  dentro de nuestro directorio `home`.

###Nota

A lo largo del libro, asumiremos que `/odoo-dev` es el directorio donde tu servidor de Odoo está instalado.

Primero, asegúrate de haber iniciado sesión como el usuario creado ahora o durante el proceso de instalación, no como el usuario `root`. Asumiendo que tu usuario es `odoo`, confírmalo con el siguiente comando:

```
odoo

$ echo $HOME

/home/odoo
$ whoami

odoo

$ echo $HOME

/home/odoo
```

Ahora podemos utilizar este script. Nos muestra cómo instalar Odoo desde la fuente a un sistema Debian/Ubuntu.

Primero, intala las dependencias básicas para comenzar:

```
$ sudo apt-get update && sudo apt-get upgrade  #Install system updates

$ sudo apt-get install git  # Install Git

$ sudo apt-get install npm  # Install NodeJs and its package manager

$ sudo ln -s /usr/bin/nodejs /usr/bin/node  # call node runs nodejs

$ sudo npm install -g less less-plugin-clean-css  #Install less compiler

```

 Partiendo de la versión 9.0, el cliente web de Odoo requiere que el preprocesador `less` CSS esté instalado en el sistema para que las páginas web puedan ser renderizadas correctamente. Para instalar esto, necesitamos Node.js y npm.

Luego, necesitamos obtener el código fuente Odoo e instalar sus dependencias. El código fuente Odoo incluye un script de utilidades, dentro del directorio `odoo/setup/`, para ayudarnos a instalar las dependencias requeridas en un sistema Debian/Ubuntu:
 
```
$ mkdir ~/odoo-dev  # Create a directory to work in

$ cd ~/odoo-dev  # Go into our work directory

$ git clone https://github.com/odoo/odoo.git -b 10.0 --depth=1  # Get Odoo source code

$ ./odoo/setup/setup_dev.py setup_deps  # Installs Odoo system dependencies 

$ ./odoo/setup/setup_dev.py setup_pg  # Installs PostgreSQL & db superuser for unix user
```

Al final, Odoo debería estar listo para utilizarse. El símbolo ~ es n atajo para nuestro directorio `home` (por ejemplo,  `/home/odoo`).
La opción `git -b 10.0` indíca a Git que descargue específicamente la rama 10.0 de Odoo. Al tiempo de la escritura, esto es redundante ya que 10.0 es la rama por defecto; sin embargo, esto puede cambiar, entonces, puede hacer el script a prueba del futuro. La opción `--depth=1` indica a Git que descargue sólo la última revisión, en vez del último historial de cambio completo, haciendo la descarga más pequeña y más veloz.

Para iniciar un servidor Odoo, solo ejecuta:

```
$ ~/odoo-dev/odoo/odoo-bin
```

###Tip

En Odoo 10, el script `odoo.py`, utilizado en versiones previas para iniciar el servidor, fue reemplazado con `odoo-bin`.

De forma predeterminada, las instancias Odoo escuchan en el puerto 8069, por lo que si apuntamos un navegador a `http: // <dirección-servidor>: 8069`, llegaremos a estas instancias. Cuando lo accedemos por primera vez, nos muestra un asistente para crear una nueva base de datos, como se muestra en la siguiente captura de pantalla:

AQUI VA UNA IMAGEN

Como desarrolladores, necesitaremos trabajar con varias bases de datos, así que es más convenientes más conveniente crearlos desde la línea de comandos, así que aprenderemos cómo hacerlo. Ahora presione ***Ctrl + C*** en el terminal para detener el servidor Odoo y volver al prompt de comando.

##Inicializando una nueva database Odoo

Para ser capaces de crear una nueva database, tu usuario debe ser un super usuario PostgreSQL. El sigiente comando crea un superusuario PostgreSQL para el usuario actual Unix.

```
$ sudo createuser --superuser $(whoami)
```
Para crear una nueva database, usa el comando `createdb`. Creeamos una database `demo`:

```
$ createdb demo
```

Para inicializar esta database con el esquema de datos Odoo, debemos ejecutar Odoo en la database vacía, usando la opción `-d`:

```
$ ~/odoo-dev/odoo/odoo-bin -d demo
```

Esto tomará un par de minutos para inicializar una database `demo`, y terminará con un mensaje de registro INFO, **Módulos cargados**.

###Nota
Ten en cuenta que puede no ser el último mensaje de registro, y puede estar en las últimas tres o cuatro líneas. Con esto, el servidor estará listo para escuchar las peticiones del cliente.

De forma predeterminada, esto inicializará la database con datos de demostración, que a menudo es útil para las databases de desarrollo. Para inicializar una database sin datos de demostración, agregue la opción `--without-demo-data = all` al comando.

Ahora abre `http: // <server-name>: 8069` con tu navegador para que se presente la pantalla de inicio de sesión. Si no conoces el nombre del servidor, escribe el comando `hostname` en el terminal para encontrarlo o el comando `ifconfig` para encontrar la dirección IP.

Si estás hospedando Odoo en una máquina virtual, es posible que debas establecer algunas configuraciones de red para poder acceder desde tu sistema huesped. La solución más simple es cambiar el tipo de red de la máquina virtual de NAT a Bridged. Con esto, en lugar de compartir la dirección IP del huesped, la máquina virtual invitada tendrá su propia dirección IP. También es posible utilizar NAT, pero eso requiere que configures el reenvío de puertos para que su sistema sepa que algunos puertos, como `8069`, deben ser manejados por la máquina virtual. En caso de que tengas problemas, esperamos que estos detalles te ayuden a encontrar información relevante en la documentación del software de virtualización elegido.

La cuenta de administrador predeterminada es `admin` con su contraseña `admin`. Al iniciar sesión, se le presenta el menú  **Apps**, que muestra las aplicaciones disponibles:

AQUI VA UNA IMAGEN

Siempre que desee detener la instancia del servidor Odoo y volver a la línea de comandos, presione ***Ctrl + C*** en el indicador de bash. Al presionar la tecla de flecha hacia arriba le llevará el comando de shell anterior, por lo que es una forma rápida de iniciar Odoo de nuevo con las mismas opciones. Las teclas ***Ctrl + C*** seguido por la tecla de flecha hacia arriba y ***Enter*** son una combinación utilizada con frecuencia para reiniciar el servidor Odoo durante el desarrollo.

##Administrar sus bases de datos

Hemos visto cómo crear e inicializar nuevas bases de datos Odoo desde la línea de comandos. Hay más comandos que vale la pena saber para administrar las bases de datos.

Ya sabemos cómo usar el comando `createdb` para crear bases de datos vacías, pero también podemos crear una nueva base de datos copiando una existente, usando la opción  `--template`.

Asegúrate de que tu instancia de Odoo está detenida y no tiene ninguna otra conexión abierta en la base de datos `demo` que acabamos de crear y, a continuación, ejecuta esto:

```
$ Createdb --template = demo demo-test
```
De hecho, cada vez que creamos una base de datos, se utiliza una plantilla. Si no se especifica ninguna, se utiliza una predeterminada llamada `template1`.

Para enumerar las bases de datos existentes en su sistema, utiliza la utilidad `psq`l de PostgreSQL con la opción `-l`:

```
$ Psql -l
```

Al ejecutarlo se listarán las dos bases de datos que hemos creado hasta ahora: `demo` y `demo-test`. La lista también mostrará la codificación utilizada en cada base de datos. El valor predeterminado es UTF-8, que es la codificación necesaria para las bases de datos Odoo.

Para eliminar una base de datos que ya no necesitas (o quieres crear nuevamente) para utilizar el comando `dropdb`:

```
$ Dropdb demo-test
```

Ahora ya sabes lo básico para trabajar con bases de datos. Para obtener más información sobre PostgreSQL, consulta la documentación oficial en http://www.postgresql.org/docs/.

###Nota

**ADVERTENCIA:**

El comando drop de la base de datos  destruirá irrevocablemente tus datos. Ten cuidado al usarlo y manten siempre copias de seguridad de bases de datos importantes antes de usar este comando.

##Una palabra sobre las versiones de productos Odoo

Al momento de la redacción de este texto, la última versión estable de Odoo es la versión 10, marcada en GitHub como rama 10.0. Esta es la versión con la que trabajaremos a lo largo del libro.

###Nota
Es importante notar que las bases de datos de Odoo son incompatibles entre las versiones principales de Odoo. Esto significa que si ejecutas un servidor Odoo 10 contra una base de datos creada para una versión principal anterior de Odoo, no funcionará.

El trabajo de migración no trivial es necesario antes de que una base de datos pueda ser usada con una versión más reciente del producto.

Lo mismo ocurre con los módulos addon: como regla general, un módulo addon desarrollado para una versión Odoo major no funcionará con otras versiones. Cuando descargue un módulo de la comunidad desde la Web, asegúrese de que esté orientado a la versión Odoo que está utilizando.

Por otra parte, se espera que las versiones principales (9.0, 10.0) reciban actualizaciones frecuentes, pero éstas deben ser en su mayoría correcciones de errores.
Se asegura que son "API estable", lo que significa que las estructuras de datos del modelo y los identificadores de elementos de vista se mantendrán estables. Esto es importante porque significa que no habrá ningún riesgo de ruptura de módulos personalizados debido a cambios incompatibles en los módulos de núcleo ascendentes.

Tenga en cuenta que la versión en la rama `master` resultará en la siguiente versión estable principal, pero hasta entonces, no es "API estable" y no debes utilizarla para crear módulos personalizados. Hacerlo es como moverse en arena movediza: no puedes estar seguro de cuándo se introducirán algunos cambios que romperán tu módulo personalizado.

##Más opciones de configuración del servidor

El servidor Odoo soporta bastantes otras opciones. Podemos comprobar todas las opciones disponibles con Más opciones de configuración del servidor
El servidor Odoo soporta bastantes otras opciones. Podemos comprobar todas las opciones disponibles con Más opciones de configuración del servidor
El servidor Odoo soporta bastantes otras opciones. Podemos comprobar todas las opciones disponibles con `--help`:

```
$ ./odoo-bin --help
```

Revisaremos algunas de las opciones más importantes en las siguientes secciones. Comencemos por ver cómo se pueden guardar las opciones actualmente activas en un archivo de configuración.

###Archivos de configuración del servidor Odoo

La mayoría de las opciones se pueden guardar en un archivo de configuración. De forma predeterminada, Odoo utilizará el archivo `.odoorc` en su directorio personal. En sistemas Linux su ubicación predeterminada está en el directorio de inicio (`$ HOME`) y en la distribución de Windows está en el mismo directorio que el ejecutable utilizado para iniciar Odoo.

###Nota

En versiones anteriores de Odoo / OpenERP, el nombre del archivo de configuración predeterminado era `.openerp-serverrc`. Para compatibilidad con versiones anteriores, Odoo 10 seguirá utilizando esto si está presente y no se encuentra ningún archivo `.odoorc`.

En una instalación limpia, el archivo de configuración `.odoorc` no se crea automáticamente. Debemos usar la opción `--save` para crear el archivo de configuración predeterminado, si aún no existe, y almacenar la configuración actual de la instancia en el:

```
$ ~ / Odoo-dev / odoo / odoo-bin --save --stop-after-init #servir configuración al archivo
```

Aquí, también usamos la opción `--stop-after-init` para detener el servidor después de que termine sus acciones. Esta opción se utiliza con frecuencia cuando se ejecutan pruebas o se solicita ejecutar una actualización de módulo para comprobar si está instalada correctamente.

Ahora podemos inspeccionar lo que se guardó en este archivo de configuración predeterminado:

```
$ More ~ ​​/ .odoorc # show the configuration file
```

Esto mostrará todas las opciones de configuración disponibles con sus valores predeterminados. Su edición será efectiva la próxima vez que inicie una instancia de Odoo. Escriba `q` para salir y volver al prompt.

También podemos optar por usar un archivo de configuración específico, usando la opción `--conf = <filepath>`. Los archivos de configuración no necesitan tener todas las opciones que acabas de ver. Sólo los que realmente cambian un valor por defecto deben estar allí.

##Cambiando el puerto de escucha

La opción de comando `--xmlrpc-port = <port>` nos permite cambiar el puerto de escucha de una instancia de servidor desde el predeterminado 8069. Esto se puede usar para ejecutar más de una instancia al mismo tiempo, en la misma máquina.

Vamos a probar esto. Abre dos ventanas de terminal. En el primero, ejecuta esto:

```
$ ~ / Odoo-dev / odoo / odoo-bin --xmlrpc-port = 8070
```

Ejecuta el siguiente comando en el segundo terminal:

```
$ ~ / Odoo-dev / odoo / odoo-bin --xmlrpc-port = 8071
```

Ahí lo tienes: dos instancias Odoo en el mismo servidor de escucha en diferentes puertos! Las dos instancias pueden utilizar bases de datos iguales o diferentes, dependiendo de los parámetros de configuración utilizados. Y los dos podrían estar ejecutando las mismas o diferentes versiones de Odoo.

###La opción filtro de la base de datos

Cuando se desarrolla con Odoo, es frecuente trabajar con varias bases de datos, ya veces incluso con diferentes versiones de Odoo. Detener e iniciar diferentes instancias de servidor en el mismo puerto y cambiar entre distintas bases de datos puede provocar que las sesiones de cliente web se comporten de forma incorrecta.

El acceso a nuestra instancia utilizando una ventana del navegador que se ejecuta en modo privado puede ayudar a evitar algunos de estos problemas.

Otra buena práctica es habilitar un filtro de base de datos en la instancia del servidor para asegurarse de que sólo permite las solicitudes de la base de datos con la que queremos trabajar, ignorando todos las demás. Esto se hace con la opción `--db-filter`. Acepta una expresión regular que se utiliza como filtro para los nombres de base de datos válidos. Para que coincida con un nombre exacto, la expresión debe comenzar con un `^` y terminar con `$`.

Por ejemplo, para permitir sólo la base de datos `demo` utilizaríamos este comando:

```
$ ~ / Odoo-dev / odoo / odoo-bin --db-filter = ^ demo $
```

###Administrar los mensajes de registro del servidor

La opción `--log-level` nos permite establecer la verbosidad del registro. Esto puede ser muy útil para entender lo que está sucediendo en el servidor. Por ejemplo, para habilitar el nivel de registro de depuración, use la opción `--log-level=debug`.

Los siguientes niveles de registro pueden ser particularmente interesantes:

+  `Debug_sql` para inspeccionar consultas SQL generadas por el servidor 
+  `Debug_rp`c para detallar las peticiones recibidas por el servidor 
+  `Debug_rpc_answer` para detallar las respuestas enviadas por el servidor 

De forma predeterminada, la salida del registro se dirige a la salida estándar (la pantalla de la consola), pero se puede dirigir a un archivo de registro con la opción `--logfile=<filepath>`.

Finalmente, la opción `--dev=all` mostrará el depurador de Python (`pdb`) cuando se genera una excepción. Es útil hacer un análisis post-mortem de un error de servidor. Ten en cuenta que no tiene ningún efecto en la verbosidad del registrador. Puedes encontrar más detalles sobre los comandos del depurador de Python en https://docs.python.org/2/library/pdb.html#debugger-commands.

