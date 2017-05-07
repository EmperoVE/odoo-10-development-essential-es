# Capitulo 13. Lista de Verificación de despliegue - Levantar el servidor

En este capítulo, aprenderás como preparar tu ervidor odoo para su uso en un entorno de producción.

Existen muchas posibles estrategias y herramientas que pueden ser usadas para desplegar y administrar un servidor odoo de producción. Te guiaremos en uno de las formas de hacerlo.

Para la configuración del servidor seguiremos la siguiente lista de verificación:

+ Instalar las dependencias y crear el usuario dedicado dedicado para correr el servidor.
+ Instalar odoo desde el código Fuente.
+ Preparar el archivo de configuración de odoo.
+ Preparar los workers de multiprocesamiento.
+ Preparar el proxy reverso con soporte SSL.

Vamos a empezar.

## Paquetes pre-construidos disponibles

Odoo cuenta con un paquete Debian/Ubuntu disponible para su instalación, instalandolo obtendrás un servidor odoo funcional que inicia automaticamente al arranque del sistema. Este proceso de instalación es sencillo y puedes encontrar todo lo que necesario en [https://nightly.odoo.com]https://nightly.odoo.com). También puedes encontrar las compilaciones rpm para CentOS y los instaladores .exe.


Aunque esta es una forma fácil y conveniente de instalar Odoo, la mayoría de los integradores prefieren implementar y ejecutar código fuente controlado por versiones. Esto proporciona un mejor control sobre lo que se despliega y facilita la administración de cambios y arreglos una vez en producción.

## Instalando Dependencias

Cuando usas una distribución de Debian, por defecto tu inicio de sesión es `root` con poderes de administrador, y el símbolo del sistema muestra #. Al usar Ubuntu, el registro con la cuenta raíz está deshabilitado, y el usuario inicial configurado durante el proceso de instalación es un `sudoer`, lo que significa que se le permite usar el comando sudo para ejecutar comandos con privilegios de root.

En primer lugar, debemos actualizar el índice del paquete y luego realizar una actualización para asegurarnos de que todos los programas instalados estén actualizados:

```
$ sudo apt-get update

$ sudo apt-get upgrade -y
```

A continuación, vamos a instalar la base de datos PostgreSQL, y hacer que nuestro usuario un superusuario de base de datos:

```
$ sudo apt-get install postgresql -y

$ sudo su -c "createuser -s $(whoami)" postgres
```

Ejecutaremos Odoo desde el código fuente, pero antes de eso necesitamos instalar las dependencias requeridas. Estos son los paquetes Debian necesarios:

```
$ sudo apt-get install git python-pip python2.7-dev -y

$ sudo apt-get install libxml2-dev libxslt1-dev 

libevent-dev \
libsasl2-dev libldap2-dev libpq-dev 


libpng12-dev libjpeg-dev \ 
poppler-utils 


node-less node-clean-css -y

```
No debemos olvidarnos de instalar wkhtmltox, que es necesario para imprimir informes:

```
$ wget http://nightly.odoo.com/extra/wkhtmltox-0.12.1.2_linux-jessie-amd64.deb

$ sudo dpkg -i wkhtmltox-0.12.1.2_linux-jessie-amd64.deb

$ sudo apt-get -fy install
```

Las instrucciones de instalación informarán un error de dependencias faltantes, pero el último comando obliga a la instalación de esas dependencias y termina correctamente la instalación.

Ahora solo nos faltan los paquetes de Python requeridos por Odoo. Muchos de ellos también tienen paquetes para Debian / Ubuntu. El paquete oficial de instalación de Debian los usa, y puedes encontrar los nombres de los paquetes en el código fuente Odoo, en el archivo `debian/control`.

Sin embargo, estas dependencias de Python también se pueden instalar directamente desde el **Python Package Index (PyPI)**. Esto es más amigable para aquellos que prefieren instalar Odoo en `virtualenv`. La lista de paquetes requerida se encuentra en el archivo `requirements.txt` de Odoo, como es habitual en los proyectos basados ​​en Python. Podemos instalarlos con los siguientes comandos:
```
$ sudo -H pip install --upgrade pip  # Ensure pip latest version

$ wget https://raw.githubusercontent.com/odoo/odoo/10.0/requirements.txt

$ sudo -H pip install -r requirements.txt
```

Ahora que tenemos todas las dependencias instaladas, servidor de bases de datos, paquetes de sistema y paquetes de Python, podemos instalar Odoo.

## Preparación de un usuario de sistema dedicado

Una buena práctica de seguridad es ejecutar Odoo con un usuario dedicado, sin privilegios especiales en el sistema.

Necesitamos crear el sistema y los usuarios de la base de datos para eso. Podemos nombrarlos odoo-prod, por ejemplo

```
$ sudo adduser --disabled-password --gecos "Odoo" odoo

$ sudo su -c "createuser odoo" postgres

$ createdb --owner=odoo odoo-prod
```

Aquí, odoo es el nombre de usuario y odoo-prod es el nombre de la base de datos que soporta nuestra instancia de Odoo.

Ten en cuenta que estos son usuarios regulares sin privilegios de administración. Se crea automáticamente un directorio para el nuevo usuario del sistema. En este ejemplo, es `/home/odoo`, y el usuario puede referirse a su propio directorio personal con el símbolo `~`  de acceso directo. Lo usaremos para las configuraciones y archivos específicos de Odoo.

Podemos abrir una sesión como este usuario utilizando el siguiente comando:

```
$ sudo su odoo
```
El comando `exit` termina esa sesión y regresa a nuestro usuario original.

## Instalación desde el código fuente

Tarde o temprano, su servidor necesitará actualizaciones y parches. Un repositorio controlado por versiones puede ser de gran ayuda cuando llegue el momento. Utilizamos git para obtener nuestro código de un repositorio, al igual que lo hicimos para instalar el entorno de desarrollo.

A continuación, suplantaremos al usuario odoo y descargamos el código en su directorio personal:

```
$ sudo su odoo

$ git clone https://github.com/odoo/odoo.git /home/odoo/odoo-10.0 -b 10.0 
--depth=1
```


La opción -b asegura que obtengamos la rama correcta, y la opción --depth = 1 ignora el historial de cambios y recupera sólo la última revisión de código, haciendo la descarga mucho más pequeña y más rápida.

## Consejo

Git seguramente será una herramienta invaluable para administrar las versiones de tus implementaciones Odoo. Acabamos de rayar la superficie de lo que se puede hacer para administrar las versiones de código. Si no está familiarizado con Git, vale la pena aprender más sobre él. Un buen punto de partida es [http://git-scm.com/doc](http://git-scm.com/doc).

Por ahora deberíamos tener todo lo necesario para ejecutar Odoo desde la fuente. Podemos comprobar que se inicia correctamente y luego salir de la sesión del usuario dedicado:

```
$ /home/odoo/odoo-10.0/odoo-bin --help

$ exit
```


A continuación, configuraremos algunos archivos y directorios a nivel de sistema para ser usado por el servicio del sistema.

## Configuración del archivo de configuración

Si se agrega la opción `--save` al iniciar un servidor Odoo, se guarda la configuración utilizada en el archivo `~/.odoorc`. Podemos utilizar el archivo como punto de partida para nuestra configuración de servidor, que se almacenará en `/etc/odoo`, como se muestra en el siguiente código:

```
$ sudo su -c "~/odoo-10.0/odoo-bin -d odoo-prod --save --stop-after-init" odoo
```

Esto tendrá los parámetros de configuración que usará nuestra instancia del servidor.

## Consejo

El archivo de configuración anterior .openerp_serverrc sigue siendo compatible y se utiliza si se encuentra. Esto puede causar cierta confusión al configurar Odoo 10 en una máquina que también se utilizó para ejecutar versiones anteriores de Odoo. En este caso, la opción `--save` podría estar actualizando el archivo .openerp_serverrc en vez de .odoorc.

A continuación, tenemos que colocar el archivo de configuración en la ubicación esperada:

```
$ sudo mkdir /etc/odoo

$ sudo cp /home/odoo/.odoorc /etc/odoo/odoo.conf

$ sudo chown -R odoo /etc/odoo
```

También debemos crear el directorio donde el servicio Odoo almacenará sus archivos de registro. Se espera que esto esté dentro de `/var/log`:

```
$ sudo mkdir /var/log/odoo

$ sudo chown odoo /var/log/odoo

```

Ahora debemos asegurarnos de configurar algunos parámetros importantes. Estos son los valores sugeridos para los más importantes:

```
[options]
addons_path = /home/odoo/odoo-10.0/odoo/addons, /home/odoo/odoo-10.0/addons
admin_passwd = False
db_user = odoo-prod
dbfilter = ^odoo-prod$
logfile = /var/log/odoo/odoo-prod.log
proxy_mode = True
without_demo = True
workers = 3
xmlrpc_port = 8069
```
Vamos a explicarles:

+ `Addons_path` es una lista separada por comas de las rutas donde se buscarán módulos complementarios. Se lee de izquierda a derecha, con los directorios de la izquierda que tienen una prioridad más alta.
+ `Admin_passwd` es la contraseña maestra para acceder a las funciones de administración de la base de datos del cliente web. Es fundamental establecer esto con una contraseña fuerte o, mejor aún, establecerla en Falso para desactivar la función.
+ `Db_user` la instancia de la base de datos a inicializar durante la secuencia de inicio del servidor.
+ `Dbfilter` es un filtro para que las bases de datos sean accesibles. Es una expresión regex interpretada en Python. Para que el usuario no se le pida que seleccione una base de datos y para que las URL no autenticadas funcionen correctamente, debe establecerse con `^dbname$`, por ejemplo, `dbfilter=^odoo-prod$`. Es compatible con los marcadores de posición `%h` y `%d`, que se reemplazan por el nombre de host de solicitud HTTP y el nombre de subdominio.
+ `logfile` es donde se debe escribir el registro del servidor. Para los servicios del sistema, la ubicación esperada está dentro de `/var/log`. Si se deja vacío o se establece en False, la impresión de registro se imprimirá en la salida estándar.
`Proxy_mode` debe establecerse en True cuando se accede a Odoo detrás de un proxy inverso, como lo haremos.
+ `Sin_demo` se debe establecer en True en entornos de producción para que las nuevas bases de datos no tengan datos de demostración en ellos.
+ `workers` con un valor de dos o más permite el modo de multiprocesamiento. Discutiremos esto en más detalle en un momento.
+ `xmlrpc_port` es el número de puerto en el que el servidor escuchará. De forma predeterminada, se utiliza el puerto 8069.

Los siguientes parámetros también pueden ser útiles:

+ `Data_dir` es la ruta donde se almacenan los datos de sesión y los archivos adjuntos. Recuerde tener copias de seguridad de esto.

+ `xmlrpc-interface` establece las direcciones que serán escuchadas. Por defecto, escucha todos los 0.0.0.0, pero cuando se utiliza un proxy inverso, se puede establecer en 127.0.0.1 con el fin de responder sólo a las solicitudes locales.

Podemos comprobar el efecto de los ajustes realizados ejecutando el servidor con la opción `-c` o `--config` de la siguiente manera:

```
$ Sudo su -c "~ / odoo-10.0 / odoo-bin -c /etc/odoo/odoo.conf" odoo
```

Ejecutar Odoo con la configuración anterior no mostrará ninguna salida a la consola, ya que se está escribiendo en el archivo de registro definido en el archivo de configuración. Para seguir lo que está sucediendo con el servidor necesitamos abrir otra ventana de terminal, y ejecutar allí:
```
$ Sudo tail -f /var/log/odoo/odoo-prod.log
```

También es posible utilizar el archivo de configuración y aún así forzar
imprimir la salida en la consola, agregando la opción `--logfile = False`, como esto:

```
$ Sudo su -c "~ / odoo-10.0 / odoo-bin -c /etc/odoo/odoo.conf --logfile = False" odoo
```

## Workers multiprocesamiento

Se espera que una instancia de producción maneje una carga de trabajo significativa. De forma predeterminada, el servidor ejecuta un proceso y sólo puede utilizar un núcleo de CPU para su procesamiento, debido al lenguaje GIL de Python. Sin embargo, un modo multiproceso está disponible para que las solicitudes concurrentes puedan ser manejadas. La opción `workers = N` establece el número de procesos de trabajo a utilizar. Como pauta, puede intentar establecerla en **1 + 2 * P**, donde P es el número de procesadores. El mejor ajuste a utilizar debe ser ajustado para cada caso, ya que depende de la carga del servidor y qué otros servicios de carga intensiva se ejecutan en el servidor, como PostgreSQL.

Es mejor establecer trabajadores demasiado altos para la carga que demasiado bajos. El mínimo debe ser 6 debido a las conexiones paralelas utilizadas por la mayoría de los navegadores, y el máximo se limita generalmente por la cantidad de RAM en la máquina.

Hay unos pocos parámetros de configuración de `limit- *`  para sintonizar a los trabajadores. Los trabajadores son reciclados cuando alcanzan estos límites, el proceso correspondiente se detiene y se inicia uno nuevo. Esto protege al servidor de fugas de memoria y de procesos particulares que sobrecargan los recursos del servidor.

La documentación oficial ya proporciona buenos consejos sobre la afinación de los parámetros de los trabajadores, y  puedes referirte a ella Para obtener más detalles, en [https://www.odoo.com/documentation/10.0/setup/deploy.html](https://www.odoo.com/documentation/10.0/setup/deploy.html.

## Configuración como servicio del sistema

A continuación, queremos configurar Odoo como un servicio del sistema y que se inicie automáticamente cuando arranque el sistema.

En Ubuntu / Debian, el sistema `init` es responsable, para iniciar los servicios. Históricamente, Debian (y los sistemas operativos derivados) han utilizado `sysvinit` y Ubuntu ha utilizado un sistema compatible llamado `Upstart`. Recientemente, esto ha cambiado, y el sistema de `init` utilizado en la última versión es ahora `systemd`.

Esto significa que hay dos maneras diferentes de instalar un servicio del sistema, y necesitas escoger el correcto dependiendo de la versión de tu sistema operativo.

En la última versión estable de Ubuntu, 16.04, deberíamos usar `systemd`. Sin embargo, las versiones más antiguas, como 14.04, todavía se utilizan en muchos proveedores de nube, por lo que existe una buena probabilidad de que necesites usarlo.

Para comprobar si `systemd` se utiliza en su sistema, pruebe este comando:
```
$ man init
```

Esto abre la documentación del sistema init actualmente utilizado, y podrá comprobar qué se está utilizando.

## Creación de un servicio systemd

Si el sistema operativo que está utilizando es reciente, como Debian 8 o Ubuntu 16.04, debería usar `systemd` para el `init` del sistema.

Para agregar un nuevo servicio al sistema, solo necesitamos crear un archivo que lo describa. Crea un archivo `/lib/systemd/system/odoo.service` con el siguiente contenido:
```
[Unit]
Description=Odoo
After=postgresql.service
 
[Service]
Type=simple
Usuario=odoo
Group=odoo
ExecStart=/home/odoo/odoo-10.0/odoo-bin -c /etc/odoo/odoo.conf
 
[Install]
WantedBy=multi-user.target
```
A continuación, debemos registrar el nuevo servicio:

```
$ Sudo systemctl habilitar odoo.service
```

Para iniciar el nuevo servicio ejecuta el siguiente comando:

```
sudo systemctl odoo start
```

Y para verificar su status corre esto:

```
$ sudo systemctl odoo status
```
Finalmente si deses deterner el servicio puede hacerlo con este comenado:

```
$ sudo systemctl odoo stop
```

## Creación de un servicio Upstart / sysvinit

Si está utilizando un sistema operativo más antiguo, como Debian 7, Ubuntu 15.04 o incluso 14.04, es probable que su sistema sea `sysvinit` en `Upstart`. Para este propósito, ambos deben comportarse de la misma manera. Muchos servicios VPS en la nube todavía se basan en imágenes de Ubuntu 14.04, por lo que este podría ser un escenario que puede encontrar al implementar su servidor Odoo.

Muchos servicios VPS en la nube todavía se basan en imágenes de Ubuntu 14.04, por lo que este podría ser un escenario que puede encontrar al implementar su servidor Odoo.

El código fuente de Odoo incluye un script de inicio utilizado para la distribución empaquetada de Debian. Podemos utilizarlo como nuestro script `init` de servicio con pequeñas modificaciones, de la siguiente manera:

```
$ sudo cp /home/odoo/odoo-10.0/debian/init /etc/init.d/odoo

$ sudo chmod +x /etc/init.d/odoo
```

En este punto, puede que desees comprobar el contenido del script `init`. Los parámetros clave se asignan a las variables en la parte superior del archivo. Un ejemplo de esto:

```
PATH=/sbin:/bin:/usr/sbin:/usr/bin:/usr/local/bin 
DAEMON=/usr/bin/odoo 
NAME=odoo 
DESC=odoo 
CONFIG=/etc/odoo/odoo.conf 
LOGFILE=/var/log/odoo/odoo-server.log 
PIDFILE=/var/run/${NAME}.pid 
USER=odoo
```

Estas variables deben ser adecuadas y prepararemos el resto de la configuración con sus valores por defecto en mente. Pero, por supuesto, puedes cambiarlos para que se adapten mejor a tus necesidades.

La variable `USER` es el usuario del sistema en el que se ejecutará el servidor. Ya hemos creado el usuario odoo esperado.

La variable `DAEMON` es la ruta al ejecutable del servidor. Nuestro ejecutable real para iniciar Odoo está en un lugar diferente, pero podemos crear un enlace simbólico a él:

```
$ sudo ln -s /home/odoo/odoo-10.0/odoo-bin /usr/bin/odoo

$ sudo chown -h odoo /usr/bin/odoo
```

La variable `CONFIG` es el archivo de configuración que se va a usar. En una sección anterior, creamos un archivo de configuración en la ubicación predeterminada: /etc/odoo/odoo.conf.

Finalmente, la variable `LOGFILE` es el directorio donde se deben almacenar los archivos de registro. El directorio esperado es `/var/log/odoo` que creamos cuando estábamos definiendo el archivo de configuración.

Ahora debemos ser capaces de iniciar y detener nuestro servicio Odoo de la siguiente manera:

```
$ sudo /etc/init.d/odoo start

Starting odoo: ok

```

La interrupción del servicio se realiza de manera similar, como se muestra a continuación:

```
$ Sudo /etc/init.d/odoo stop

Stopping odoo: ok

```

En Ubuntu, también se puede usar el comando `service`:

```
$ sudo service odoo start


$ sudo service odoo status


$ sudo service odoo config
```


Ahora solo necesitamos hacer que este servicio se inicie automáticamente en el arranque del sistema:

```
$ Sudo update-rc.d odoo defaults
```

Después de esto, cuando reiniciamos nuestro servidor, el servicio Odoo debe iniciarse automáticamente y sin errores. Es un buen momento para comprobar que todo está funcionando como se esperaba.

## Comprobación del servicio Odoo desde la línea de comandos

En este punto, podríamos confirmar si nuestra instancia de Odoo está lista y responde a las solicitudes.

Si Odoo se está ejecutando correctamente, ahora deberíamos ser capaces de obtener una respuesta de él y no ver ningún error en el archivo de registro. Podemos comprobar dentro del servidor si Odoo está respondiendo a las peticiones HTTP utilizando el siguiente comando:

```
$ Curl http: // localhost: 8069

<Html> <head> <script> window.location = '/ web' + ubicación.hash; </ script> </ head> </ html>
```

Y para ver lo que está en el archivo de registro podemos utilizar lo siguiente:

```
$ Sudo less /var/log/odoo/odoo-server.log

```
En el caso de que estés empezando con Linux, te gustaría saber que puedes seguir lo que está sucediendo en el archivo de registro usando tail -f:

```
$ Sudo tail -f /var/log/odoo/odoo-server.log
```

## Uso de un proxy inverso

Mientras que Odoo en si mismo puede servir páginas web, se recomienda encarecidamente tener un proxy inverso delante de él. Un proxy inverso actúa como un intermediario que gestiona el tráfico entre los clientes que envían solicitudes y los servidores Odoo que responden a ellos. El uso de un proxy inverso tiene varios beneficios.

En el lado de la seguridad, puede hacer lo siguiente:

+ Manejar (y hacer cumplir) los protocolos HTTPS para cifrar el tráfico.
+ Ocultar las características de la red interna.
+ Actuar como un cortafuegos de aplicacion que limita las URL aceptadas para el procesamiento

Y en el lado del rendimiento, puede proporcionar mejoras significativas:

+ Guardar cache contenido estático, reduciendo así la carga en los servidores Odoo.
+ Comprimir contenido para acelerar los tiempos de carga.
+ Actuar como un equilibrador de carga, la distribución de carga entre varios servidores.

Apache es una opción popular para usar como un proxy inverso. Nginx es una alternativa reciente con buenos argumentos técnicos. Aquí, elegiremos usar Nginx como un proxy inverso y mostraremos cómo puede usarse para realizar las funciones de seguridad y rendimiento mencionadas aquí.

## Configuración de Nginx para proxy inverso

Primero, debemos instalar Nginx. Queremos que escuche en los puertos HTTP predeterminados, por lo que debemos asegurarnos de que ya no los hayan tomado otros servicios. La ejecución de este comando debe producir un error, de la siguiente manera:

```
$ Curl http: // localhost

Curl: (7) Error al conectar al puerto localhost 80: Conexión rechazada
```
De lo contrario, debes desactivar o eliminar dicho servicio para permitir que Nginx utilice dichos puertos. Por ejemplo, para detener un servidor Apache existente, utiliza este comando:

```
$ sudo service apache2 stop
```

O mejor aún, debe considerar eliminarlo del sistema o reconfigurarlo para escucharlo en otro puerto, por lo que los puertos HTTP y HTTPS (80 y 443) pueden ser utilizados por Nginx.

Ahora podemos instalar Nginx, que se hace de la manera esperada:

```
$ sudo apt-get install nginx
```

Para confirmar que está funcionando correctamente, debemos ver una página Bienvenido a nginx cuando visite la dirección del servidor con un navegador o usando `curl http://localhost` dentro de nuestro servidor.

Los archivos de configuración de Nginx siguen el mismo enfoque que Apache: se almacenan en `/etc/nginx/available-sites/` y se activan agregando un enlace simbólico en `/etc/nginx/enabled-sites/`. También debemos deshabilitar la configuración predeterminada proporcionada por la instalación de Nginx, de la siguiente manera:

```
$ sudo rm/etc/nginx/sites-enabled/default

$ sudo touch /etc/nginx/sites-available/odoo

$ sudo ln -s  /etc/nginx/sites-available/odoo /etc/nginx/sites-enabled/odoo
```


Utilizando un editor, como nano o vi, debemos editar nuestro archivo de configuración de Nginx de la siguiente manera:

```
$ sudo nano/etc/nginx/sites-available/odoo
```

Primero, agregamos los `upstreams`, y los servidores `backend`, Nginx redirigirá el tráfico al servidor Odoo en nuestro caso, que está escuchando en el puerto 8069, de la siguiente manera:

```
Upstream backend-odoo {
  Servidor 127.0.0.1:8069;
}
Servidor {
  ubicación / {
    Proxy_pass http: // backend-odoo;
  }
}
```

Para probar si la configuración editada es correcta, utilice el siguiente comando:

```
$ sudo nginx -t
```

Si encuentra errores, confirme que el archivo de configuración está correctamente escrito. Además, un problema común es que el HTTP predeterminado sea tomado por otro servicio, como Apache o el sitio web predeterminado de Nginx. Compruebe las instrucciones dadas antes para asegurarse de que este no es el caso, luego reinicie Nginx. Después de esto, podemos tener Nginx para recargar la nueva configuración de la siguiente manera:

```
$ sudo /etc/init.d/nginx reload

```


Ahora podemos confirmar que Nginx está redirigiendo el tráfico al servidor backend Odoo:

```
$ curl http://localhost

<Html> <head> <script> window.location = '/ web' + ubicación.hash; </ script> </ head> </ html>
```

## Aplicación de HTTPS

A continuación, debemos instalar un certificado para poder usar SSL. Para crear un certificado autofirmado, siga estos pasos:

```
$ sudo mkdir /etc/nginx/ssl && cd/etc/nginx/ssl

$ sudo openssl req -x509 -nuevo rsa: 2048 -keyout key.pem -out cert.pem -days 365 -nodos

$ sudo chmod a-wx * # hacer que los archivos sean de solo lectura


$ sudo chown www-data:root * # acceso sólo al grupo www-data
```

Esto crea un directorio `ssl/` dentro del directorio `/etc/nginx/` y crea un certificado SSL auto-firmado sin contraseña. Al ejecutar el comando openssl, se le pedirá información adicional y se generarán un certificado y archivos clave. Finalmente, la propiedad de estos archivos se da al usuario www-data utilizado para ejecutar el servidor web.

**Nota**

El uso de un certificado autofirmado puede plantear algunos riesgos de seguridad, como los ataques de hombre en el medio, e incluso puede no ser permitido por algunos navegadores. Para obtener una solución robusta, debe utilizar un certificado firmado por una autoridad de certificación reconocida. Esto es particularmente importante si está ejecutando un sitio web comercial o de comercio electrónico.

Ahora que tenemos un certificado SSL, estamos listos para configurar Nginx para usarlo.

Para hacer cumplir HTTPS, redirigiremos todo el tráfico HTTP hacia él. Reemplace la directiva de servidor que definimos anteriormente con lo siguiente:

```
Servidor {
  listen 80;
  Add_header Strict-Transport-Security max-age = 2592000;
  rewrite ^/.* $ Https://$host$request_uri? permanent;
}
```

Si volvemos a cargar la configuración de Nginx ahora y accedemos al servidor con un navegador web, veremos que la dirección `http://` se convertirá en una dirección `https://`.

Pero no devolverá ningún contenido antes de configurar correctamente el servicio HTTPS añadiendo la siguiente configuración de servidor:

```
server {
  listen 443 default;
  # ssl settings
  ssl on;
  ssl_certificate /etc/nginx/ssl/cert.pem;
  ssl_certificate_key /etc/nginx/ssl/key.pem;
  keepalive_timeout 60;
  # proxy header and settings
  Proxy_set_header Host $host;
  Proxy_set_header X-Real-IP $remote_addr;
  Proxy_set_header X-Forwarded-For $ proxy_add_x_forwarded_for;
  proxy_set_header X-Forwarded-Proto $scheme; 
  proxy_redirect off; 
  
  location / { 
    proxy_pass http://backend-odoo; 
  } 
}
```

Esto escuchará el puerto HTTPS y utilizará los archivos `/etc/nginx/ssl/ certificate` para cifrar el tráfico. También agregamos cierta información al encabezado de la solicitud para que el servicio de back-end de Odoo sepa que está siendo procesado.

Por razones de seguridad, es importante que Odoo se asegure de que el parámetro proxy_mode esté establecido en True. La razón de esto es que, cuando Nginx actúa como un proxy, toda la solicitud parecerá venir del propio servidor en lugar de la dirección IP remota. Establecer el encabezado X-Forwarded-For en el proxy y habilitar --proxy-mode resuelve eso. Pero activar el modo -proxy sin forzar este encabezado en el nivel de proxy permitiría a cualquiera falsificar su dirección remota.

Al final, la directiva de ubicación define que todas las solicitudes se pasan al backend-odoo upstream.

Vuelve a cargar la configuración y deberíamos tener nuestro servicio Odoo trabajando a través de HTTPS, como se muestra en los siguientes comandos:

```
$ sudo nginx -t

nginx: the configuration file /etc/nginx/nginx.conf syntax is ok

nginx: configuration file /etc/nginx/nginx.conf test is successful

$ sudo service nginx reload 

* Reloading nginx configuration nginx


...done.


$ curl -k https://localhost 


<html><head><script>window.location = '/web' +  location.hash;</script></head></html>
```

La última salida confirma que el cliente web Odoo está siendo distribuido a través de HTTPS.

## Consejo

Para el caso particular en el que se está usando un Odoo POSBox, necesitamos agregar una excepción para que el /pos/URL pueda acceder a él en modo HTTP. El POSBox se encuentra en la red local pero no tiene SSL habilitado. Si la interfaz POS está cargada en HTTPS, no podrá contactar al POSBox.

## Optimizaciones de Nginx

Ahora, es tiempo para algunos ajuste fino de los ajustes de Nginx. Se recomiendan para habilitar el búfer de respuesta y la compresión de datos que deberían mejorar la velocidad del sitio web. También establecemos una ubicación específica para los registros.

Las siguientes configuraciones deben agregarse dentro del servidor que escucha en el puerto 443, por ejemplo, justo después de las definiciones de proxy:

```
# Archivos de registro odoo
Access_log /var/log/nginx/odoo-access.log;
Error_log /var/log/nginx/odoo-error.log;
# Aumentar el tamaño del búfer proxy
Proxy_buffers 16 64k;
Proxy_buffer_size 128k;
# forzar Timeouts si el backend muere
Proxy_next_upstream error timeout invalid_header http_500 http_502 http_503;
# Activar la compresión de datos
Gzip on
Gzip_min_length 1100;
Gzip_buffers 4 32k;
Gzip_types text/plain text /xml text/css texto/less application/x-javascript application/xml application/json application/javascript;
Gzip_vary on;
```
También podemos activar el almacenamiento en caché de contenido estático para respuestas más rápidas a los tipos de solicitudes mencionadas en el ejemplo de código anterior y para evitar su carga en el servidor Odoo. Después de la  sección `location`, agrega la siguiente segunda sección de ubicación:

```
location~ * /web/static/ {
  # Cache de datos estáticos
  Proxy_cache_valid 200 60m;
  Proxy_buffering on;
  Expira 864000;
  Proxy_pass http: // backend-odoo;
}
```
Con esto, los datos estáticos se almacenan en caché durante 60 minutos. Las solicitudes adicionales sobre esas peticiones en ese intervalo serán respondidas directamente por Nginx desde la caché.

## Long Polling

El long polling se usa para soportar la aplicación de mensajería instantánea y, cuando se utilizan trabajadores multiprocesamiento, se maneja en un puerto independiente, que es 8072 por defecto.

Para nuestro proxy inverso, esto significa que las peticiones de long polling se deben pasar a este puerto. Para apoyar esto, necesitamos agregar un nuevo upstream a nuestra configuración de Nginx, como se muestra en el siguiente código:

```
Upstream backend-odoo-im {server 127.0.0.1:8072; }
```
A continuación, debemos agregar otra ubicación al servidor que maneja las solicitudes HTTPS, como se muestra en el código siguiente:

```
location /longpolling {proxy_pass http:// backend-odoo-im;}
```

Con estos ajustes, Nginx debe pasar estas solicitudes al puerto adecuado del servidor Odoo.

## Actualizaciones de servidor y módulos

Una vez que el servidor Odoo esté listo y funcionando, llegará el momento en que necesitará instalar actualizaciones en Odoo. Esto implica dos pasos: primero, obtener las nuevas versiones del código fuente (servidor o módulos), y segundo, instalarlas.

Si has seguido el método descrito en la sección Instalación desde el código fuente, podemos buscar y probar las nuevas versiones en el repositorio git. Se recomienda encarecidamente que haga una copia de la base de datos de producción y pruebe la actualización en ella. Si odoo-prod es su base de datos de producción, esto podría hacerse con los siguientes comandos:

```
$ dropdb odoo-stage; createdb odoo-stage


$ pg_dump odoo-prod | psql -d odoo-stage


$ sudo su odoo

$ cd ~/.local/share/Odoo/filestore/

$ cp -al odoo-prod odoo-stage

$ ~/Odoo-10.0/odoo-bin -d odoo-stage --xmlrpc-port = 8080 -c /etc/odoo/odoo.conf

$ exit

```


Si todo va bien, debería ser seguro realizar la actualización en el servicio de producción. Recuerde hacer una nota de la versión actual de la referencia Git para poder volver atrás revisando esta versión de nuevo. Mantener una copia de seguridad de la base de datos antes de realizar la actualización también es muy recomendable.

## Consejo

La copia de la base de datos se puede hacer de una manera mucho más rápida, usando el siguiente comando `createdb: $ createdb --template odoo-prod odoo-stage`

La advertencia aquí es que para que se ejecute no puede haber ninguna conexión abierta a la base de datos odoo-prod, por lo que el servidor Odoo debe detenerse para realizar la copia.

Después de esto, podemos cargar las nuevas versiones al repositorio de producción usando Git y completar la actualización, como se muestra aquí

```
$ sudo su odoo

$ cd ~/odoo-10.0

$ git pull

$ exit

$ sudo service odoo restart
```

En cuanto a la política de lanzamiento de Odoo, no se publican versiones menores. Se espera que las ramas de GitHub representen la última versión estable. Las construcciones nocturnas se consideran el último lanzamiento oficial estable.

En la frecuencia de actualización, no hay ningún motivo para actualizar con demasiada frecuencia, pero tampoco esperar un año entre actualizaciones. Realizar una actualización cada pocos meses debería estar bien. Y por lo general, un reinicio del servidor será suficiente para habilitar las actualizaciones de código, y las actualizaciones de módulos no deberían ser necesarias.

Por supuesto, si necesita una corrección de errores específica, probablemente debería realizarse una actualización anterior. También recuerde tener cuidado con las divulgaciones de errores de seguridad en los canales públicos-GitHub Issues o en la lista de correo de la comunidad. Como parte del servicio, los clientes de odoo entreprice pueden esperar notificaciones por correo electrónico tempranas de este tipo de problemas.

# Sumario

En este capítulo, aprendiste sobre los pasos adicionales para configurar y ejecutar Odoo en un servidor de producción basado en Debian. Se visitaron las configuraciones más importantes del archivo de configuración y se aprendió cómo aprovechar el modo de multiprocesamiento.

Para mejorar la seguridad y la escalabilidad, también aprendiste a usar Nginx como un proxy inverso frente a nuestros procesos de servidor Odoo.

Esto debería cubrir lo esencial de lo que se necesita para ejecutar un servidor Odoo y proporcionar un servicio estable y seguro a sus usuarios.

Para obtener más información sobre Odoo, también debe consultar la documentación oficial en [ https://www.odoo.com/documentation]( https://www.odoo.com/documentation). Algunos temas se tratan con más detalle allí, y también encontrará temas no cubiertos en este libro.

También hay otros libros publicados sobre Odoo que también puede ser útil. Pack Publishing tiene algunos en su catálogo, y en particular el Odoo Development Cookbook proporciona material más avanzado, cubriendo más temas no discutidos aquí.

Y finalmente, Odoo es un producto de código abierto con una comunidad vibrante. Involucrarse, hacer preguntas y contribuir es una gran manera no sólo para aprender, sino también para construir una red de negocios. Sobre esto no podemos dejar de mencionar la Odoo Community Association (OCA), promoviendo la colaboración y la calidad del código abierto. Puede obtener más información al respecto en [odoo-comunity.org](odoo-comunity.org).