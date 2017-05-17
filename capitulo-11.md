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

El uso de cadenas de Python para construir HTML se tornará aburrido muy rápido. Las plantillas QWeb hacen un trabajo mucho mejor en eso. Así que vamos a mejorar nuestra página web de **Hola Mundo** para usar una plantilla en su lugar.

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
La representación de la plantilla se proporciona mediante `request`, a través de su función `render()`.

### Consejo

> Observa que agregamos `**kwargs` a los argumentos del método. Con esto, si cualquier parámetro adicional proporcionado por la petición HTTP, como una consulta de cadenas de texto o parámetros POST, puede ser capturado por el diccionario kwargs. Esto hace que nuestro método sea más robusto, ya que proporcionar parámetros inesperados no causará errores.

## Ampliación de funciones web

Extensibilidad es algo que esperamos en todas las características de Odoo, y las características de la web no son una    excepción. Y de hecho podemos extender los controladores y plantillas existentes. Por ejemplo, extenderemos nuestra página web de **Hola Mundo** para que tome un parámetro con el nombre para saludar: usando la `URL/hello?name=John` devolvería un saludo **Hello John!** .

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

El método extendido `hello()` ahora tiene un parámetro de nombre. Los parámetros pueden obtener sus valores de segmentos de la URL de la ruta, de los parámetros de consulta de una cadena o de los parámetros POST. En este caso, la ruta no tiene variable extraíble (lo demostraremos en un momento), y como estamos manejando peticiones GET, no POST, el valor del parámetro name será extraído de la consulta de cadena en la URL. Una URL de prueba podría ser `http://localhost:8069/hello?name=John`.

Dentro del método `hello()` ejecutamos el método heredado para obtener su respuesta, y luego podemos modificarlo de acuerdo a nuestras necesidades. El patrón común para los métodos de controlador es que terminen con una instrucción para renderizar una plantilla. En nuestro caso:

```
return request.render('todo_website.hello')  

```

Esto genera un objeto `http.Response`, pero la representación real se retrasa hasta el final del despacho.

Esto significa que el método de herencia todavía puede cambiar la plantilla QWeb y el contexto a utilizar para la representación. Podríamos cambiar la plantilla modificando `response.template`, pero no lo necesitaremos. Preferimos modificar `response.qcontext` para agregar la clave `name al contexto de renderizado.

No olvides agregar el nuevo archivo Python en `todo_website/controllers/__init__.py:`

```
from . import main 
from . import extend 
```

Ahora necesitamos modificar la plantilla `QWeb`, Para que haga uso de esta información adicional. Agrega lo que sigue a `todo/website/views/todo_extend.xml`:

```
<odoo> 
  <template id="hello_extended"  
    name="Extended Hello World" 
    
inherit_id="todo_website.hello"> 
    <xpath expr="//h1" position="replace"> 
      <h1> 
        Hello <t t-esc="name or 'Someone'" />! 
      </h1> 
    </xpath> 
  </template>
</odoo> 
```
Las plantillas de páginas web son documentos XML, al igual que los otros tipos de vista Odoo, y podemos usar xpath para localizar elementos y luego manipularlos, tal como podríamos con los otros tipos de vista. La plantilla heredada se identifica en el elemento `<template>` por el atributo `inherited_id`.

No debemos olvidar declarar este archivo de datos adicional en nuestro manifiesto, `todo_website/__ manifest__.py`:

```
'data': [
   'Views/todo_web.xml',
  
'views/todo_extend.xml'
],
```
Después de esto, accediendo a `http://localhost:8069/hello?name=John debe mostrarnos un mensaje **Hello John!**.

También podemos proporcionar parámetros a través de segmentos de URL. Por ejemplo, podríamos obtener exactamente el mismo resultado de la URL `http://localhost:8069/hello/John usando esta implementación alternativa:

```
class TodoExtended(Todo):
    
@ Http.route (['/ hello', '/hello/<name>])
def hello(self, name=None, **kwargs): 
        response = super(TodoExtended, self).hello() 
        response.qcontext['name'] = name 
        return response
```

Como puedes ver, las rutas pueden contener **placeholders** correspondientes a los parámetros que se van a extraer y luego pasar al método. Los **placeholders** también pueden especificar un convertidor para implementar una asignación de tipo específica. Por ejemplo, `<int: user_id>` extraería el parámetro `user_id` como un valor entero.

Los convertidores son una característica proporcionada por la biblioteca werkzeug, utilizada por Odoo, y la mayoría de los disponibles se pueden encontrar en la documentación de la biblioteca `werkzeug`, en [http://werkzeug.pocoo.org/docs/routing/](http://werkzeug.pocoo.org/docs/routing/).

Odoo añade un conversor específico y particularmente útil: extraer un registro de modelo. Por ejemplo `@http.route('/hello/<model("res.users"):user>)` extrae el parámetro user como un objeto de registro para el modelo `res.users`.

## ¡HolaCMS!

Vamos a hacer esto aún más interesante, y crear nuestro propio CMS simple. Para esto podemos hacer que la ruta espere un nombre de plantilla (una página) en la URL y luego simplemente renderizarla. Podríamos entonces crear dinámicamente páginas web y hacerlas servir por nuestro CMS.

Resulta que esto es bastante fácil de hacer:

```
@http.route('/hellocms/<page>', auth='public')
def hello (auto, página, ** kwargs):
     return http.request.render(page)
```
Ahora, abre `http://localhost:8069/hellocms/todo_website.hello` en tu navegador web y verás nuestra página de **Hello World!**

De hecho, el sitio web incorporado ofrece funciones de CMS, incluyendo una implementación más robusta de lo anterior, en la ruta del endpoint `/page`.

### Nota

En la jerga werkzeug el endpoint es un alias de la ruta, y representado por su parte estática (sin los placeholders). Para nuestro ejemplo simple de CMS, el endpoint fue `/hellocms`.

La mayor parte del tiempo queremos que nuestras páginas se integren en el sitio web de Odoo. Así que para el resto de este capítulo en nuestros ejemplos vamos a trabajar con el módulo `website`.

## Creando sitios web

Las páginas dadas por los ejemplos anteriores no están integradas con el módulo `website`: no tenemos pie de página, menú, etc. El módulo `website` de Odoo proporciona convenientemente todas estas características para que no tengamos que preocuparnos por ellas.

Para usarlo, debemos comenzar instalando el módulo `website` en nuestra instancia de trabajo y luego agregarlo como una dependencia a nuestro módulo. La clave `depends` en __manifest__.py debe tener el siguiente aspecto:

```
'depends': ['todo_kanban', 
'website'
], 
```

Para utilizar el sitio web, también debemos modificar el controlador y la plantilla.

El controlador necesita un argumento adicional `website=True` en la ruta:

```
@http.route('/hello', auth='public', website=True) 
def hello(self, **kwargs): 
    return request.render('todo_website.hello')
```     
Y la plantilla debe insertarse dentro del diseño general del sitio web:

```
<template id="hello" name="Hello World"> 
  
<t t-call="website.layout">
    <h1>Hello World!</h1> 
</t>
</template> 
```
Con esto, el ejemplo **Hola Mundo** que utilizamos antes debe ahora ser mostrado dentro de una página del sitio web de Odoo.

## Adición de elementos CSS y JavaScript

Nuestras páginas web pueden necesitar algunos recursos adicionales de CSS o JavaScript. Este aspecto de las páginas web es gestionado por el sitio web, por lo que necesitamos una forma de decirle que también utilice nuestros archivos.

Vamos a añadir un poco de CSS para agregar un efecto de tachado simple para las tareas realizadas. Para ello, crea el archivo `todo_website/static/src/css/index.css` con este contenido:

```
.todo-app-done { 
    text-decoration: line-through; 
} 
```
Luego tenemos que incluirlo en las páginas del sitio web. Esto se hace agregándolos en la plantilla `website.assets_frontend` responsable de cargar los activos específicos del sitio web. Edite el archivo de datos todo_website / views / todo_templates.xml, para ampliar esa plantilla:

<Odoo>
  <Template id = "assets_frontend"
    Name = "todo_website_assets"
   
 Inherit_id = "website.assets_frontend">
    <Xpath expr = "." Position = "inside">
      <Link rel = "stylesheet" type = "text / css"
        Href = "/ todo_website / static / src / css / index.css" />
    </ Xpath>



 
  </ Template>
</ Odoo>
Pronto estaremos usando esta nueva clase de estilo todo-app-done. Por supuesto, los activos de JavaScript también se pueden agregar usando un enfoque similar.

El controlador de lista de tareas pendientes

Ahora que hemos pasado por lo básico, vamos a trabajar en nuestra lista de tareas Todo. Tendremos una URL de / todo mostrándonos una página web con una lista de Tareas Todo.

Para ello, necesitamos un método controlador, la preparación de los datos a presentar, y una plantilla QWeb para presentar esa lista al usuario.

Edite el archivo todo_website / controllers / main.py para agregar este método:

#class Main (http.Controller):
   
    @ Http.route ('/ todo', auth = 'usuario', sitio web = Verdadero)
    Índice def (self, ** kwargs):
        TodoTask = request.env ['todo.task']
        Tasks = TodoTask.search ([])
        Return request.render (
            'Todo_website.index', {'tareas': tareas})
El controlador recupera los datos que se van a utilizar y los pone a disposición de la plantilla renderizada. En este caso, el controlador requiere una sesión autenticada, ya que la ruta tiene el atributo auth = 'user'. Incluso si ese es el valor predeterminado, es una buena práctica declarar explícitamente que se requiere una sesión de usuario.

Con esto, la declaración Todo Task search () se ejecutará con el usuario de la sesión actual.