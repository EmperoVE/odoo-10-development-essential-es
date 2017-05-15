# Capítulo 11. Creación de características Frontend de un website

Odoo comenzó como un sistema backend, pero la necesidad de una interfaz de frontend pronto se sintió. Las primeras características del portal, basadas en la misma interfaz que el backend, no eran muy flexibles ni amigables con dispositivos móviles.

Para solucionar este vacío, la versión 8 introdujo nuevas características de sitio web, agregando un Sistema de Gestión de Contenidos (CMS) al producto. Esto nos permitiría construir frontends hermosos y eficaces sin la necesidad de integrar un CMS de terceros.

Aquí aprenderemos cómo desarrollar nuestros propios módulos complementarios orientados al usuario, aprovechando la función de sitio web proporcionada por Odoo.

## Mapa de ruta

Crearemos una página web en la que se enumerarán nuestras Tareas pendientes, lo que nos permitirá navegar a una página detallada para cada tarea existente. También queremos poder proponer nuevas Tareas pendientes a través de un formulario web.

Con esto podremos cubrir las técnicas esenciales para el desarrollo de sitios web: crear páginas dinámicas, pasar parámetros a otra página, crear formularios y manejar su validación y lógica de cálculo.

Pero primero, introduciremos los conceptos básicos del sitio web con una página web **Hola Mundo** muy sencilla.

## Nuestra primera página web

Vamos a crear un módulo de complemento para las características de nuestro sitio web. Podemos llamarlo `todo_website`. Para introducir los fundamentos del desarrollo web de Odoo, implementaremos una sencilla página web **Hola Mundo**. Imaginativo, ¿verdad?

Como de costumbre, comenzaremos a crear su archivo de manifiesto. Crea el archivo `todo_website/__ manifest__.py` con:

```
{  
  'name': 'To-Do Website', 
  'description': 'To-Do Tasks Website', 
  'author': 'Daniel Reis', 
  'depends': ['todo_kanban']}
```
Estamos construyendo sobre el módulo todo_kanban, de modo que tengamos todas las características disponibles añadidas al modelo de Tareas pendientes a lo largo del libro.

Tenga en cuenta que ahora mismo no dependemos del módulo `website`. Aunque `website` ofrece un marco útil para crear sitios web con todas las funciones, las capacidades básicas de la Web se incorporan  en el núcleo del framework. Vamos a explorarlos.

## ¡Hola Mundo!

Para proporcionar nuestra primera página web, agregaremos un objeto controlador. Podemos empezar por tener su archivo importado con el módulo:

Primero agrega un archivo `todo_website/__ init__.py` con la siguiente línea:

```
from . import controllers
```
A continuación, agregue un archivo `todo_website/controllers/__ init__.py` con la siguiente línea:

```
from . import main
```
Ahora agregue el archivo como tal para el controlador, `todo_website/controllers/main.py`, con el siguiente código:

```
# -*- coding: utf-8 -*- 
from odoo import http 
 
class Todo(http.Controller): 
 
    @http.route('/helloworld', auth='public') 
    def hello_world(self): 
        return('<h1>Hello World!</h1>')
```


El módulo `odoo.http` proporciona las funciones web relacionadas con Odoo. Nuestros controladores, responsables de la representación de páginas, deben ser objetos heredados de la clase `odoo.http.Controller`. El nombre real utilizado para la clase no es importante; Aquí elegimos utilizar `Main`.

Dentro de la clase del controlador tenemos métodos, que coinciden rutas, hace algún procesamiento, y luego devuelve un resultado; La página que se mostrará al usuario.

El decorador `odoo.http.route` es lo que une un método a una ruta de URL. Nuestro ejemplo utiliza la ruta `/hello`. Navegue hasta http://localhost:8069/hello y recibirá un mensaje de `Hello World`. En este ejemplo, el procesamiento realizado por el método es bastante trivial: simplemente devuelve una cadena de texto con el marcado HTML para el mensaje Hello World.

Probablemente haya notado que hemos añadido el argumento auth = 'public' a la ruta. Esto es necesario para que la página esté disponible para usuarios no autenticados. Si lo eliminamos, sólo los usuarios autenticados pueden ver la página. Si no hay sesión activa, en su lugar se mostrará la pantalla de inicio de sesión.

## Hola Mundo! Con una plantilla Qweb

El uso de cadenas de Python para construir HTML se tonará aburrido muy rápido. Las plantillas QWeb hacen un trabajo mucho mejor en eso. Así que vamos a mejorar nuestra página web de **Hola Mundo** para usar una plantilla en su lugar.

Las plantillas de QWeb se agregan a través de archivos de datos XML, y técnicamente son un tipo de vista, junto con vistas de formulario o árbol. En realidad se almacenan en el mismo modelo, ir.ui.view.

Como de costumbre, los archivos de datos que se cargan deben declararse en el archivo de manifiesto, por lo que debes editar el archivo `todo_website/__ manifest__.py` para agregar la clave:

```
'data': ['views/todo_templates.xml'],
```
A continuación, agrega el archivo de data, `views/todo_web.xml, con el siguiente contenido:

```
<odoo> 
  <template id="hello" name="Hello Template"> 
    <h1>Hello World !</h1> 
  </template> 
</odoo>
```

### Nota


> El elemento `<template>` es en realidad un acceso directo para declarar un `<record>` para el modelo `ir.ui.view`, usando `type = "qweb"`, y una plantilla `<t>` dentro de él.

Ahora necesitamos que nuestro método de controlador use esta plantilla:

```
from odoo.http import request
# ...
@http.route('/hello', auth='public')
def hello(self, **kwargs):
    return request.render('todo_website.hello')
```
La representación de la plantilla se proporciona mediante `requeest`, a través de su función `render()`.

### Consejo

> Observa que agregamos `**kwargs` a los argumentos del método. Con esto, si cualquier parámetro adicional proporcionado por la petición HTTP, como una consulta de cadenas de texto o parámetros POST, puede ser capturado por el diccionario kwargs. Esto hace que nuestro método sea más robusto, ya que proporcionar parámetros inesperados no causará errores.

## Ampliación de funciones web

Extensibilidad es algo que esperamos en todas las características de Odoo, y las características de la web no son una    excepción. Y de hecho podemos extender los controladores y plantillas existentes. Por ejemplo, extenderemos nuestra página web de **Hola Mundo** para que tome un parámetro con el nombre para saludar: usando la `URL/hello? Name=John` devolvería un saludo **Hello John!** .

La extensión se hace generalmente desde un módulo distinto, pero funciona de igual manera dentro del mismo módulo. Para mantener las cosas concisas y sencillas, lo haremos sin crear un nuevo módulo.

Vamos añadir un nuevo archivo `todo_website/controllers/extend.py` con el siguiente código:

```
# -*- coding: utf-8 -*- 
from odoo import http 
from odoo.addons.todo_website.controllers.main import Todo 
 
class TodoExtended(Todo): 
    @http.route() 
    def hello(self, name=None, **kwargs): 
        response = super(TodoExtended, self).hello() 
        response.qcontext['name'] = name 
        return response
```
Aquí podemos ver lo que necesitamos hacer para extender un controlador.

Primero usamos una importación de Python para obtener una referencia a la clase de controlador que queremos extender. Comparados con los modelos, tienen un registro central, proporcionado por el objeto `env`, donde se puede obtener una referencia a cualquier clase de modelo, sin necesidad de conocer el módulo y el archivo que los implementa. Con los controladores no tenemos eso, y necesitamos conocer el módulo y el archivo que implementa el controlador que queremos ampliar.

A continuación tenemos que (re)definir el método desde el controlador que se está extendiendo. Tiene que estar decorado con al menos el simple `@http.route()` para que su ruta se mantenga activa. Opcionalmente, podemos proporcionar parámetros a `route()`, y luego estaremos reemplazando y redefiniendo sus rutas.

El método extendido `hello()` ahora tiene un parámetro de nombre. Los parámetros pueden obtener sus valores de segmentos de la URL de la ruta, de los parámetros de consulta de una cadena o de los parámetros POST. En este caso, la ruta no tiene variable extraíble (lo demostraremos en un momento), y como estamos manejando peticiones GET, no POST, el valor del parámetro name será extraído de la consulta de cadena en la URL. Una URL de prueba podría ser `http://localhost:8069/hello?Name=John`.

Dentro del método hello () ejecutamos el método heredado para obtener su respuesta, y luego podemos modificarlo de acuerdo a nuestras necesidades. El patrón común para los métodos de controlador es que terminen con una instrucción para renderizar una plantilla. En nuestro caso: