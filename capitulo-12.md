# Capitulo 12. API Externa - Integración con otros sistemas.


El servidor Odoo también proporciona una API externa, que es utilizada por su cliente web y también está disponible para otras aplicaciones cliente.

En este capítulo, aprenderemos cómo usar la API externa de Odoo desde nuestros propios programas cliente. Se puede utilizar cualquier lenguaje de programación, siempre y cuando tenga soporte para protocolos XML-RPC o JSON-RPC. Como ejemplo, la documentación oficial proporciona ejemplos de código para cuatro lenguajes de programación populares: Python, PHP, Ruby y Java.

Para evitar la introducción de idiomas adicionales que el lector podría no estar familiarizado con, aquí nos centraremos en los clientes basados ​​en Python, aunque las técnicas para manejar las llamadas RPC también se aplican a otros lenguajes de programación.

Describiremos cómo usar las llamadas RPC de Odoo y luego usarlas para crear una aplicación de escritorio de tareas pendientes sencilla con Python.

Finalmente, presentaremos el cliente ERPPeek. Se trata de una biblioteca cliente de Odoo, que se puede utilizar como una capa de abstracción conveniente para las llamadas RPC de Odoo y también es un cliente de línea de comandos para Odoo, que permite administrar de forma remota las instancias de Odoo.

## Configuración de un cliente Python

La API de Odoo se puede acceder externamente usando dos protocolos diferentes: XML-RPC y JSON-RPC. Cualquier programa externo capaz de implementar un cliente para uno de estos protocolos podrá interactuar con un servidor Odoo. Para evitar la introducción de lenguajes de programación adicionales, seguiremos usando Python para explorar la API externa.

Hasta ahora, hemos estado ejecutando código Python sólo en el servidor. Esta vez, vamos a utilizar Python en el lado del cliente, por lo que es posible que tenga que hacer alguna configuración adicional en su estación de trabajo.

Para seguir los ejemplos de este capítulo, necesitaras poder ejecutar archivos Python en tu computadora de trabajo. El servidor Odoo requiere Python 2, pero nuestro cliente RPC puede estar en cualquier idioma, por lo que Python 3 estará bien. Sin embargo, dado que algunos lectores pueden estar ejecutando el servidor en la misma máquina en la que están trabajando (¡hola usuarios de Ubuntu!), Será más fácil para todos seguir si nos atenemos a Python 2.

Si estas utilizando Ubuntu o una Mac, Python probablemente ya esté instalado. Abra una consola de terminal, escriba `python`, y debería recibir algo similar a lo siguiente:

```
Python 2.7.12 (default, Jul  1 2016, 15:12:24)
[GCC 5.4.0 20160609] on linux2
Type "help", "copyright",", "credits" or "license" for more information.
>>>
```

> **nota**

> Los usuarios de Windows pueden encontrar un instalador para Python y también ponerse rápidamente al día. Los paquetes oficiales de instalación se pueden encontrar en https://www.python.org/downloads/.

> Si es un usuario de Windows y tiene Odoo instalado en su máquina, es posible que se pregunte por qué no tiene un intérprete de Python, y se necesita una instalación adicional. La respuesta corta es que la instalación de Odoo tiene un intérprete de Python incrustado que no se utiliza fácilmente fuera.



## Llamadas a la API de Odoo utilizando XML-RPC

El método más sencillo para acceder al servidor es utilizar XML-RPC. Podemos usar la biblioteca xmlrpclib de la biblioteca estándar de Python para esto. Recuerde que estamos programando un cliente para conectarnos a un servidor, por lo que necesitamos una instancia de servidor Odoo en la que se ejecute para conectarse. En nuestros ejemplos, asumiremos que una instancia de servidor Odoo se ejecuta en la misma máquina (localhost), pero puede utilizar cualquier dirección IP o nombre de servidor accesible si el servidor se ejecuta en una máquina diferente.

## Abriendo una conexión XML-RPC

Vamos a tener un primer contacto con la API externa de Odoo. Inicie una consola de Python y escriba lo siguiente:

```
>>> import xmlrpclib
>>> srv = 'http://localhost:8069'
>>> common = xmlrpclib.ServerProxy('%s/xmlrpc/2/common' % srv)
>>> common.version()
{'server_version_info': [10, 0, 0, 'final', 0, ''], 'server_serie': '10.0', 'server_version': '10.0', 'protocol_version': 1}
```
Aquí, importamos la biblioteca xmlrpclib y luego configuramos una variable con la información para la dirección del servidor y el puerto de escucha. Siéntete libre de adaptarlos a tu configuración específica.

A continuación, configuramos el acceso a los servicios públicos del servidor (que no requieren un inicio de sesión), expuestos en el `/xmlrpc/2/common` endpoint. Uno de los métodos que está disponible es `version ()`, que inspecciona la versión del servidor. Lo usamos para confirmar que podemos comunicarnos con el servidor.

Otro método público es `authenticate()`. De hecho, esto no crea una sesión, como usted puede ser llevado a creer. Este método sólo confirma que el nombre de usuario y la contraseña se aceptan y devuelve el ID de usuario que se debe utilizar en las solicitudes en lugar del nombre de usuario, como se muestra aquí:

```
>>> db = 'todo'
>>> user, pwd = 'admin', 'admin'
>>> uid = common.authenticate(db, user, pwd, {})
>>> print uid
```

Si las credenciales de inicio de sesión no son correctas, se devuelve un valor False, en lugar de un ID de usuario.

## Lectura de datos desde el servidor

Con XML-RPC, no se mantiene ninguna sesión y las credenciales de autenticación se envían con cada solicitud. Esto agrega algo de sobrecarga al protocolo, pero lo hace más fácil de usar.

A continuación, establecemos el acceso a los métodos de servidor que necesitan un inicio de sesión para acceder. Éstos se exponen en el endpoint `/xmlrpc2/object`, como se muestra a continuación:

```
>>> api = xmlrpclib.ServerProxy('%s/xmlrpc/2/object' % srv)
>>> api.execute_kw(db, uid, pwd, 'res.partner', 'search_count'[[]])
40
```

Aquí, estamos haciendo nuestro primer acceso a la API del servidor, realizando un conteo en los registros de **Partner**. Los métodos se llaman usando el método `execute_kw()` que toma los siguientes argumentos:

+ El nombre de la base de datos para conectarse a
+ El ID de usuario de la conexión
+ La contraseña de usuario
+ El nombre del identificador del modelo de destino
+ El método para llamar
+ Una lista de argumentos posicionales
+ Un diccionario opcional con argumentos de palabra clave

El ejemplo anterior llama al método `search_count` del modelo `res.partner` con un argumento de posición, [], y sin argumentos de palabra clave. El argumento posicional es un dominio de búsqueda; Ya que estamos proporcionando una lista vacía, cuenta todos los Partners.

Las acciones frecuentes son buscar y leer. Cuando se llama desde el RPC, el método `search` devuelve una lista de identificadores que coinciden con un dominio. El método `browse` no está disponible en el RPC, y leer debe ser utilizado en su lugar para dar una lista de IDs de registro y recuperar sus datos, como se muestra en el código siguiente:

```
>>> api.execute_kw(db, uid, pwd, 'res.partner', 'search', [[('country_id', '=', 'be'), ('parent_id', '!=', False)]]) 
[18, 33, 23, 22] 
>>> api.execute_kw(db, uid, pwd, 'res.partner', 'read',  [[18]],  {'fields': ['id', 'name', 'parent_id']}) 
[{'parent_id': [8, 'Agrolait'], 'id': 18, 'name': 'Edward Foster'}]
```

Ten en cuenta que para el método `read`, estamos utilizando un argumento de posición para la lista de IDs, [18] y un argumento de palabra clave, `fields`. También podemos observar que los campos relacionales many-to-one se recuperan como un par, con el ID del registro relacionado y el nombre para mostrar. Eso es algo que debe tener en cuenta al procesar los datos en su código.

La combinación de búsqueda y lectura es tan frecuente que se proporciona un método `search_read` para realizar ambas operaciones en un solo paso. El mismo resultado que los dos pasos previos se puede obtener con lo siguiente:

```
>>> api.execute_kw(db, uid, pwd, 'res.partner', 'search_read',  [[('country_id', '=', 'be'), ('parent_id', '!=', False)]],  {'fields': ['id', 'name', 'parent_id']})
```
El método `search_read` se comporta como `read`, pero espera un dominio como un primer argumento posicional en lugar de una lista de IDs. Vale la pena mencionar que el argumento `field` en `read` y `search_read` no es obligatorio. Si no se proporciona, se recuperarán todos los campos. Esto puede causar costosos cálculos de campos de función y una gran cantidad de datos que se recuperarán, pero probablemente nunca se utiliza, por lo que generalmente se recomienda proporcionar una lista explícita de campos.

## Llamando otros métodos

Todos los demás métodos de modelo se exponen a través de RPC, excepto los prefijados con un subrayado, que se consideran privados. Esto significa que podemos usar `create`, `write` y `unlink` para modificar datos en el servidor de la siguiente manera:

```
>>> api.execute_kw(db, uid, pwd, 'res.partner', 'create', [{'name': 'Packt Pub'}])
45
>>> api.execute_kw(db, uid, pwd, 'res.partner', 'write', [[45], {'name': 'Packt Publishing'}])
True
>>> api.execute_kw(db, uid, pwd, 'res.partner', 'read', [[45], ['id', 'name']])
[{'id': 45, 'name': 'Packt Publishing'}]
>>> api.execute_kw(db, uid, pwd, 'res.partner', 'unlink', [[45]])
True
```

Una limitación del protocolo XML-RPC es que no admite valores `None`. La implicación es que los métodos que no devuelven nada no serán utilizables a través de XML-RPC, ya que están implicitamente devolviendo `None`. Esta es la razón por la cual los métodos deben terminar siempre con al menos una declaración verdadera de la vuelta.

Vale la pena repetir que la API externa de Odoo puede ser utilizada por la mayoría de los lenguajes de programación. En la Documentación oficial podemos encontrar ejemplos prácticos para Ruby, PHP y Java. Está disponible en [https://www.odoo.com/documentation/10.0/api_integration.html](https://www.odoo.com/documentation/10.0/api_integration.html).

## Desarrollando una aplicación de escritorio de Notas

Hagamos algo interesante con la API de RPC. Odoo proporciona una aplicación sencilla para notas. ¿Qué pasa si los usuarios pueden administrar sus notas personales directamente desde el escritorio de su computadora? Vamos a escribir una sencilla aplicación de Python para hacer eso, como se muestra en la siguiente captura de pantalla:

![Notas](file:img/12-01.jpg)


Para mayor claridad, lo dividiremos en dos archivos: uno relacionado con las interacciones con el servidor backend, `note_api.py` y otro con la interfaz gráfica de usuario, `note_gui.py`.

## Capa de comunicación con Odoo

Crearemos una clase para configurar la conexión y almacenar su información. Debe exponer dos métodos: `get()` para recuperar datos de tareas y `set()` para crear o actualizar tareas.

Seleccione un directorio para alojar los archivos de la aplicación y cree el archivo `note_api.py`. Podemos comenzar agregando el constructor de la clase, de la siguiente manera:

```
import xmlrpclib 
class NoteAPI(): 
    def __init__(self, srv, db, user, pwd): 
        common = xmlrpclib.ServerProxy( 
            '%s/xmlrpc/2/common' % srv) 
        self.api = xmlrpclib.ServerProxy( 
            '%s/xmlrpc/2/object' % srv) 
        self.uid = common.authenticate(db, user, pwd, {}) 
        self.pwd = pwd 
        self.db = db 
        self.model = 'note.note'

```
Aquí almacenamos toda la información necesaria en el objeto creado para ejecutar llamadas en un modelo: la referencia de la API, uid, contraseña, nombre de la base de datos y el modelo a utilizar.

A continuación, definiremos un método auxiliar para ejecutar las llamadas. Se aprovecha de los datos almacenados del objeto para proporcionar una firma de función más pequeña, como se muestra a continuación:

```
def execute(self, method, arg_list, kwarg_dict=None): 
    return self.api.execute_kw( 
        self.db, self.uid, self.pwd, self.model, 
        method, arg_list, kwarg_dict or {})
```

Ahora podemos usarlo para implementar los métodos `get()` y `set()` de nivel superior.

El método `get()` aceptará una lista opcional de ID para recuperar. Si no aparece ninguno, se devolverán todos los registros:

```
def get(self, ids=None): 
    domain = [('id',' in', ids)] if ids else [] 
    fields = ['id', 'name'] 
    return self.execute('search_read', [domain, fields])
```

El método `set()` tendrá el texto de la tarea a escribir y un ID opcional como argumentos. Si no se proporciona ID, se creará un nuevo registro. Devuelve el ID del registro escrito o creado, como se muestra aquí:

```
def set(self, text, id=None): 
    if id: 
        self.execute('write', [[id], {'name': text}]) 
    else: 
        vals = {'name': text, 'user_id': self.uid} 
        id = self.execute('create', [vals]) 
    return id
```


Terminemos el archivo con un pequeño fragmento de código de prueba que se ejecutará si ejecutamos el archivo Python:

```
if __name__ == '__main__': 
    srv, db = 'http://localhost:8069', 'todo'
    user, pwd = 'admin', 'admin' 
    api = NoteAPI(srv, db, user, pwd) 
    from pprint import pprint 
    pprint(api.get())
```


Si ejecutamos el script Python, deberíamos ver el contenido de nuestras tareas pendientes impresas. Ahora que tenemos un envoltorio simple alrededor de nuestro backend Odoo, vamos a tratar con la interfaz de usuario de escritorio.

## Creación de la GUI

Nuestro objetivo aquí era aprender a escribir la interfaz entre una aplicación externa y el servidor Odoo, y esto se hizo en la sección anterior. Pero sería una vergüenza no ir el paso adicional y realmente ponerlo a disposición del usuario final.

Para mantener la configuración tan simple como sea posible, usaremos Tkinter para implementar la interfaz gráfica de usuario. Dado que forma parte de la biblioteca estándar, no requiere ninguna instalación adicional. No es nuestro objetivo explicar cómo funciona Tkinter, por lo que será breve la explicación de la misma.

Cada tarea debe tener una pequeña ventana amarilla en el escritorio. Estas ventanas tendrán un solo widget de texto. Presionando ***Ctrl + N*** se abrirá una nueva nota y presionando ***Ctrl + S*** se escribirá el contenido de la nota actual en el servidor Odoo.

Ahora, junto con el archivo `note_api.py`, crea un nuevo archivo `note_gui.py`. Primero importará los módulos de Tkinter y los widgets que usaremos, y luego la clase NoteAPI, como se muestra a continuación:

```
from Tkinter import Text, Tk 
import tkMessageBox 
from note_api import NoteAPI
```

Si aparecen errores de código mostrando `ImportError: No module named _tkinter, please install the python-tk package`, significa que se necesitan bibliotecas adicionales en su sistema. En Ubuntu necesitaría ejecutar el siguiente comando:

```
$ sudo apt-get install python-tk
```
A continuación, creamos nuestro propio widget de texto derivado del de Tkinter. Al crear una instancia, esperará una referencia de la API, que se utilizará para la acción de salvar, así como el texto y la identificación de la tarea, como se muestra a continuación:

```
class NoteText(Text): 
    def __init__(self, api, text='', id=None): 
        self.master = Tk() 
        self.id = id 
        self.api = api 
        Text.__init__(self, self.master, bg='#f9f3a9', 
                            wrap='word', undo=True)
        self.bind('<Control-n>', self.create) 
        self.bind('<Control-s>', self.save) 
        if id: 
            self.master.title('#%d' % id) 
        self.delete('1.0', 'end') 
        self.insert('1.0', text) 
        self.master.geometry('220x235') 
        self.pack(fill='both', expand=1)
```

El constructor `Tk()` crea una nueva ventana de interfaz de usuario y el widget de texto se coloca dentro de él, por lo que la creación de una nueva instancia de Nota de texto abre automáticamente una ventana de escritorio.

A continuación, implementaremos las acciones de crear y guardar. La acción `create` abre una nueva ventana vacía, pero se almacenará en el servidor sólo cuando se realice una acción de salvar. Aquí está el código correspondiente:
```
def create(self, event=None): 
    NoteText(self.api, '')
 
def save(self, event=None): 
    text = self.get('1.0', 'end') 
    self.id = self.api.set(text, self.id) 
    tkMessageBox.showinfo('Info', 'Note %d Saved.' % self.id)
```

La acción `save` se puede realizar ya sea en tareas existentes o nuevas, pero no hay necesidad de preocuparse por esto aquí ya que estos casos ya están manejados por el método `set()` de `NoteAPI`.

Finalmente, agregamos el código que recupera y crea todas las ventanas de notas cuando se inicia el programa, como se muestra en el código siguiente:
```
if __name__ == '__main__': 
    srv, db = 'http://localhost:8069', 'todo' 
    user, pwd = 'admin', 'admin' 
    api = NoteAPI(srv, db, user, pwd) 
    for note in api.get(): 
        x = NoteText(api, note['name'], note['id']) 
    x.master.mainloop()
```


El último comando ejecuta `mainloop()` en la última ventana de nota creada, para comenzar a esperar eventos de ventanas.

Esta es una aplicación muy básica, pero el punto aquí es hacer un ejemplo de formas interesantes de aprovechar la Odoo RPC API.

## Presentando el cliente ERPpeek

ERPpeek es una herramienta versátil que se puede utilizar tanto como interfaz interactiva de línea de comandos (CLI) como biblioteca de Python, con una API más conveniente que la proporcionada por xmlrpclib. Está disponible en el índice PyPi y puede instalarse con lo siguiente:

```
$ Pip install -U erppeek
```

En un sistema Unix, si lo está instalando en todo el sistema, es posible que deba agregar sudo al comando.

## La API de ERPpeek

La biblioteca ERPpeek proporciona una interfaz de programación, que envuelve xmlrpclib, que es similar a la interfaz de programación que tenemos para el código del servidor.

Nuestro punto aquí es dar una idea de lo que la biblioteca ERPpeek tiene para ofrecer, y no proporcionar una explicación completa de todas sus características.

Podemos comenzar reproduciendo nuestros primeros pasos con xmlrpclib usando la erppeek como sigue:

```
>>> import erppeek
>>> api = erppeek.Client('http://localhost:8069', 'todo','admin', 'admin')
>>> api.common.version()
>>> api.count('res.partner', [])
>>> api.search('res.partner', [('country_id', '=', 'be'),      ('parent_id','!=', False)])
>>> api.read('res.partner', [44], ['id', 'name', 'parent_id'])
```

Como puedes ver, las llamadas a la API utilizan menos argumentos y son similares a las contrapartes del servidor.

Pero ERPpeek no se detiene aquí, y también proporciona una representación para los modelos. Tenemos las siguientes dos formas alternativas de obtener una instancia para un modelo, ya sea usando el método `model()` o accediendo a él como un nombre de atributo camel case:

```
>>> m = api.model('res.partner')
>>> m = api.ResPartner
```
Ahora podemos realizar acciones en ese modelo de la siguiente manera:

```
>>> m.count([('name', 'like', 'Packt%')])
1
>>> m.search([('name', 'like', 'Packt%')])
[44]
```

También proporciona la representación de objetos del lado del cliente para los registros de la siguiente manera:

```
>>> recs = m.browse([('name', 'like', 'Packt%')])
>>> recs
<RecordList 'res.partner,[44]'>
>>> recs.name
['Packt Publishing']
```

Como puedes ver, la biblioteca de erppeek esta aun largo camino de distancia de xmlrpclib, y hace posible escribir código que puede ser reutilizado en el lado del servidor con poca o ninguna modificación.

## La CLI de ERPpeek

No sólo se puede utilizar la biblioteca erppeek como una biblioteca Python, también es una CLI que se puede utilizar para realizar acciones administrativas en el servidor. Cuando el comando odoo shell proporciona una sesión interactiva local en el servidor host, la biblioteca erppeek proporciona una sesión interactiva remota en un cliente a través de la red.

Al abrir una línea de comandos, podemos echar un vistazo a las opciones disponibles, como se muestra a continuación:

```
$ erppeek --help
```

Veamos una sesión de ejemplo como sigue:

```
$ erppeek --server='http://localhost:8069' -d todo -u admin
Usage (some commands):
models(name)                  # List models matching pattern
model(name)                   # Return a Model instance
(...)
Password for 'admin':
Logged in as 'admin'
todo >>> model('res.users').count()
3
todo >>> rec = model('res.partner').browse(43)
todo >>> rec.name
'Packt Publishing'
```
Como se puede ver, se realizó una conexión con el servidor y el contexto de ejecución proporcionó una referencia al método `model()` para obtener instancias de modelo y realizar acciones en ellas.

La instancia `erppeek.Client` utilizada para la conexión también está disponible a través de la variable cliente.

En particular, proporciona una alternativa al cliente web para administrar los módulos complementarios instalados:

+ `client.modules()`: lista los módulos disponibles o instalados
+ `client.install()`: realiza la instalación del módulo
+ `client.upgrade()`: realiza actualizaciones de módulos
+ `client.uninstall()`: desinstala los módulos
Por lo tanto, erppeek también puede proporcionar un buen servicio como una herramienta de administración remota para los servidores Odoo.


## Sumario

Nuestro objetivo para este capítulo fue aprender cómo funciona la API externa y de qué es capaz. Empezamos a explorarla usando un simple cliente XML-RPC de Python, pero la API externa se puede usar desde cualquier lenguaje de programación. De hecho, los documentos oficiales proporcionan ejemplos de código para Java, PHP y Ruby.

Existen varias bibliotecas para manejar XML-RPC o JSON-RPC, algunas genéricas y otras específicas para usar con Odoo. Tratamos de no señalar ninguna bibliotecas en particular, a excepción de erppeek, ya que no sólo es un envoltorio probado para el Odoo / OpenERP XML-RPC, sino porque también es una herramienta invaluable para la gestión de servidores remotos y la inspección.

Hasta ahora, utilizamos nuestras instancias de servidor Odoo para desarrollo y pruebas. Pero para tener un servidor de grado de producción, hay configuraciones adicionales de seguridad y optimización que deben hacerse. En el próximo capítulo, nos centraremos en ellos.


