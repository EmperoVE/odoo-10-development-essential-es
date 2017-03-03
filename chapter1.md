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






