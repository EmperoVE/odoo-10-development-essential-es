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


Puedes cambiar el nombre de usuario  `odoo` al que tu quieras. La opción `-m`asegura que su directorio de inicio sea creado. La opción `-g sudo` 


Ahora podemos iniciar sesión como el nuevo usuario y organizar Odoo.


##Instalando Odoo desde la fuente

Los paquetes de instalación rápida de Odoo, pueden hallarse en nigthly.odoo.com, disponíble como Windows `(.exe)`, Debian (`.deb`), CentOS (`.rpm`), y código fuente en formato tarballs (`. tar .gz`).

Como desarrolladores, preferiremos instalarlos directamente del repositorio GitHub. Esto terminará dándonos más control sobre versiones y actualizaciones.

Para mantener las cosas ordenadas, vamos a trabajar en un directorio `/odoo-dev`  dentro de nuestro directorio `home`.

###Nota

A lo largo del libro, asumiremos que `/odoo-dev` es el directorio donde tu servidor de Odoo está instalado.

Primero, asegúrate de haber iniciado sesión como el usuario creado ahora o durante el proceso de instalación, no como el usuario `root`. Asumiendo que tu usuario es `odoo`, confírmalo con el siguiente comando:




