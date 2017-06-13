# Capítulo 8. Escribiendo pruebas y depurando tu código

Una buena parte del trabajo de un desarrollador es probar y depurar código. Las pruebas automatizadas son una herramienta inestimable para construir y mantener software robusto. En este capítulo, aprenderemos cómo agregar pruebas automatizadas a nuestros módulos de complemento, para hacerlos más robustos. También se presentan las técnicas de depuración del servidor, que permiten al desarrollador inspeccionar y comprender lo que está sucediendo en su código.

## Pruebas de unidad

Las pruebas automatizadas se aceptan generalmente como una mejor práctica en software. No solo nos ayuda a asegurar que nuestro código se implemente correctamente. Más importante aún, proporciona una red de seguridad para futuras mejoras de código o reescritura.

En el caso de lenguajes de programación dinámica, como Python, ya que no hay ningún paso de compilación, los errores de sintaxis pueden pasar desapercibidos. Esto hace aún más importante que las pruebas de unidad pasen por tantas líneas de código como sea posible.

Los dos objetivos descritos pueden proporcionar una luz de guía al escribir pruebas. El primer objetivo para sus pruebas debe ser proporcionar una buena cobertura de prueba, diseñando casos de prueba que pasen por todas las líneas de código. Esto solo por lo general hará un buen progreso en el segundo objetivo - para mostrar la corrección funcional del código.

Esto solo por lo general hará un buen progreso en el segundo objetivo - para mostrar la corrección funcional del código, ya que después de esto seguramente tendrá un gran punto de partida para construir casos de prueba adicionales para casos de uso no obvio.

## Añadiendo pruebas de unidad

Las pruebas de Python se agregan a módulos addon mediante un subdirectorio `tests/`. El corredor de pruebas descubrirá automáticamente las pruebas en los subdirectorios con ese nombre en particular.

Las pruebas de nuestro complemento `todo_wizard` estarán en un archivo `test/test_wizard.py`. Necesitaremos agregar el archivo `tests/__init__.py`:

```
from . import test_wizard
```
Y este sería el esqueleto básico para el `tests/test_wizard.py`:

```
# -*- coding: utf-8 -*- 
from odoo.tests.common import TransactionCase 
 
class TestWizard(TransactionCase): 
 
    def setUp(self, *args, **kwargs): 
        super(TestWizard, self).setUp(*args, **kwargs) 
        # Add test setup code here... 
 
    def test_populate_tasks(self): 
        "Populate tasks buttons should add two tasks" 
        # Add test code
```

Odoo ofrece algunas clases para usar en las pruebas. Las pruebas de `TransactionCase` utilizan una transacción diferente para cada prueba, que se deshace automáticamente al final. También podemos usar `SingleTransactionCase`, que ejecuta todas las pruebas en una sola transacción, que se deshace sólo al final de la última prueba. Esto puede ser útil cuando deseas que el estado final de cada prueba sea el estado inicial de la siguiente prueba.

El método `setUp()` es donde preparamos los datos y las variables que se van a utilizar. Normalmente los almacenaremos como atributos de clase, de modo que estén disponibles para ser utilizados en los métodos de prueba.

Las pruebas se deben implementar como métodos de clase, como `test_populate_tasks()`. Los nombres de los métodos de casos de prueba deben comenzar con un prefijo `test_`. Se descubren automáticamente, y este prefijo es lo que identifica los métodos que implementan casos de prueba.

Los métodos se ejecutarán en orden de los nombres de las funciones de prueba. Cuando se utiliza la clase `TransactionCase`, se realizará una reversión al final de cada clase. La docstring del método se muestra cuando se ejecutan las pruebas, y debe proporcionar una breve descripción de la misma.

Estas clases de prueba son envolturas alrededor de testcases `unitcases`. Esto forma parte de la biblioteca estándar de Python y puedes consultar su documentación para obtener más detalles en https://docs.python.org/2/library/unittest.html.

Para ser más preciso, Odoo utiliza una biblioteca de extensión `unittest`, `unittest2`.

## Configurando pruebas

Debemos comenzar preparando los datos que se utilizarán en las pruebas.

Es conveniente realizar las acciones de prueba bajo un usuario específico, para probar también que el control de acceso está configurado correctamente. Esto se logra utilizando el método de modelo `sudo()`. Los conjuntos de registros llevan esa información con ellos, por lo que después de haber sido creados durante el uso de `sudo()`, las operaciones posteriores en el mismo conjunto de registros se realizarán utilizando ese mismo contexto.

Éste es el código para el método `setUp` y algunas instrucciones de importación adicionales que también son necesarias:

```
from datetime import date
from odoo.tests.common import TransactionCase
from odoo import fields

class TestWizard(TransactionCase):

    def setUp(self, *args, **kwargs):
        super(TestWizard, self).setUp(*args, **kwargs)
        # Close any open Todo tasks
        self.env['todo.task']\
            .search([('is_done', '=', False)])\
            .write({'is_done': True})
        # Demo user will be used to run tests
        demo_user = self.env.ref('base.user_demo') 
        # Create two Todo tasks to use in tests
        t0 = date.today()
        Todo = self.env['todo.task'].sudo(demo_user)
        self.todo1 = Todo.create({
            'name': 'Todo1',
            'date_deadline': fields.Date.to_string(t0)})
        self.todo2 = Todo.create({
            'name': 'Todo2'})
        # Create Wizard instance to use in tests
        Wizard = self.env['todo.wizard'].sudo(demo_user)
        self.wizard = Wizard.create({})
        
```
Para probar nuestro asistente, queremos tener exactamente abiertas dos tareas pendientes. Así que empezamos cerrando cualquier tarea pendiente existente, para que no se interpongan en nuestras pruebas, y creamos dos nuevas tareas pendientes para las pruebas, usando el usuario Demo. Finalmente creamos una nueva instancia de nuestro asistente, utilizando el usuario Demo, y asignandolo a `self.wizard`, de modo que esté disponible para los métodos de prueba.

## Probando excepciones

A veces necesitamos nuestras pruebas para verificar si se generó una excepción. Un caso común es cuando se prueba si algunas validaciones se están haciendo correctamente.

En nuestro ejemplo, el método `test_count()` utiliza una excepción de advertencia `warning` como una forma de proporcionar información al usuario. Para comprobar si se genera una excepción, colocamos el código correspondiente dentro de un bloque `whit self.assertRaises()`.

Necesitamos importar la excepción `Warning` en la parte superior del archivo:

```
from odoo.exceptions import Warning

```

Y agregua a la clase de prueba un método con otro caso de prueba:

```
def test_count(self): 
    "Test count button" 
    with self.assertRaises(Warning) as e: 
        self.wizard.do_count_tasks() 
    self.assertIn(' 2 ', str(e.exception)) 
```

Si el método `do_count_tasks()` no genera una excepción, la comprobación fallará. Si levanta esa excepción, la comprobación tiene éxito y la excepción generada se almacena en la variable `e`.

Lo usamos para inspeccionarlo. El mensaje de excepción contiene el número de tareas contadas, que esperamos que sean dos. En la sentencia final usamos `assertIn` para comprobar que el texto de excepción contiene la cadena `' 2 '`.

## Ejecutando de pruebas

Las pruebas están escritas, es hora de ejecutarlas. Para eso solo necesitamos agregar la opción `--test-enable` al comando de inicio del servidor Odoo, mientras instalamos o actualizamos (`-i` o `-u`) el módulo addon.

El comando se vería así:

```
$ ./odoo-bin -d todo --test-enable -i todo_wizard --stop-after-init 
--addons-path="..."
```

Sólo se probarán los módulos instalados o actualizados. Si algunas dependencias necesitan ser instaladas, sus pruebas también se ejecutarán. Si desea evitar esto, puede instalar el módulo para probar la forma usual y, a continuación, ejecutar las pruebas mientras realiza una actualización (`-u`) del módulo para probar.

# Acerca de las pruebas de YAML

Odoo también soporta un segundo tipo de pruebas, descritas usando archivos de datos YAML. Originalmente todas las pruebas usaban YAML, hasta que más recientemente se introdujeron las pruebas basadas en pruebas `unittest`. Si bien ambos son compatibles, y muchos complementos principales aún incluyen pruebas YAML, la documentación oficial actualmente no menciona las pruebas YAML. La última documentación disponible está disponible en https://doc.odoo.com/v6.0/contribute/15_guidelines/coding_guidelines_testing/.

Los desarrolladores con un fondo Python probablemente se sentirán más a gusto con `unittest`, ya que es una característica estándar de Python, mientras que las pruebas YAML están diseñadas con convenciones específicas de Odoo. La tendencia es claramente preferir `unittest` sobre YAML, y se espera que el soporte de YAML sea eliminado en futuras versiones.

Por estas razones, no haremos una cobertura en profundidad de las pruebas de YAML. Podría ser útil tener algún entendimiento básico sobre cómo funcionan.

Las pruebas YAML son archivos de datos, similares a CSV y XML. De hecho, el formato YAML estaba destinado a ser un formato de datos más compacto que se puede utilizar en lugar de XML. A diferencia de las pruebas Python, donde las pruebas deben estar en un subdirectorio `test/`, los archivos de prueba YAML pueden estar en cualquier parte dentro del módulo de complemento. Pero frecuentemente estarán dentro de un subdirectorio `test/` o `tests/`. Y aunque las pruebas de Python se descubren automáticamente, las pruebas de YAML deben declararse en el archivo de manifiesto `__manifest__.py`. Esto se hace con la clave `test`, similar a la clave `data` que ya conocemos.

En Odoo 10 pruebas YAML ya no se utilizan, pero aquí es un ejemplo, desde el `02_order_to_invoice.yml` en el módulo addon `point_of_sale`:

```
- 
  I click on the "Make Payment" wizard to pay the PoS order 
- 
  !record {model: pos.make.payment, id: pos_make_payment_2, context: '{"active_id": ref("pos_order_pos1"), "active_ids": [ref("pos_order_pos1")]}' }: 
    amount: !eval > 
        (450*2 + 300*3*1.05)*0.95 
- 
  I click on the validate button to register the payment. 
- 
  !python {model: pos.make.payment}: | 
    self.check(cr, uid, [ref('pos_make_payment_2')], context={'active_id': ref('pos_order_pos1')} ) 
```

Las líneas que comienzan con un `!` Son etiquetas YAML, equivalentes a los elementos de la etiqueta que encontramos en los archivos XML. En el código anterior podemos ver una etiqueta `¡ record`, equivalente a la etiqueta XML `<record>`, y una etiqueta `!pyython`, que nos permite ejecutar código Python en un modelo, `pos.make.payment` en el ejemplo.

Como puedes ver, las pruebas YAML utilizan una sintaxis específica de Odoo que necesita aprendizaje. En comparación, las pruebas de Python utilizan el framework `unittest` existente, solo añadiendo clases envolventes específicas de Odoo como `TransactionCase`.

## Herramientas de desarrollo

Hay algunas técnicas que el desarrollador debe aprender para ayudar en su trabajo. En el Capítulo 1, *Iniciando con el desarrollo Odoo*, ya hemos introducido la interfaz de usuario del modo de desarrollador **Developer Mode**. También tenemos disponible una opción de servidor que proporciona algunas características amigables para desarrolladores. Lo describiremos más detalladamente a continuación. Después de eso vamos a discutir otro tema relevante para los desarrolladores: cómo depurar el código del lado del servidor.


### Opciones de desarrollo del servidor

El servidor Odoo proporciona la opción `--dev` para permitir que algunas funciones del desarrollador aceleren nuestro ciclo de desarrollo, como por ejemplo:

+ Introducir el depurador cuando se encuentre una excepción en un módulo de complemento
+ Recargar código Python automáticamente, una vez que se guarda un archivo Python, evitando un reinicio manual del servidor
+ Lee las definiciones de vista directamente desde los archivos XML, evitando las actualizaciones manuales de módulos

La opción `--dev` acepta una lista de opciones separada por comas, aunque la opción todo será adecuada la mayor parte del tiempo. También podemos especificar el depurador que preferimos usar. De forma predeterminada, se utiliza el depurador Python, `pdb`. Algunas personas pueden preferir instalar y usar depuradores alternativos. Aquí también se admiten `ipdb` y `pudb`.


#### Nota

Antes de Odoo 10 teníamos en lugar de la opción `--debug`, permitiendo abrir el depurador en una excepción de módulo de complemento.

Cuando se trabaja con código Python, el servidor debe reiniciarse cada vez que se cambia el código, para que se vuelva a cargar. La opción de línea de comandos `--dev` hace que la recarga: cuando el servidor detecta que se cambia un archivo Python, repite automáticamente la secuencia de carga del servidor, haciendo que el cambio de código sea efectivo de inmediato.

Para usarlo simplemente agregue la opción `--dev=all` al comando del servidor:


```
$ ./odoo-bin -d todo --dev=al
```

Para que esto funcione el paquete Python `watchdog` es necesario, y debe instalarse como se muestra aquí:

```
$ pip install watchdog

```
#### Nota

Ten en cuenta que esto es útil sólo para los cambios de código Python y arquitecturas de vista en archivos XML. Para otros cambios, como la estructura de datos del modelo, se necesita una actualización del módulo y la recarga no es suficiente.


### Depuración

Todos sabemos que una buena parte del trabajo de un desarrollador es depurar código. Para hacer esto a menudo hacemos uso de un editor de código que puede establecer puntos de interrupción y ejecutar nuestro programa paso a paso.

Si estás utilizando Microsoft Windows como tu estación de trabajo de desarrollo, la configuración de un entorno capaz de ejecutar código Odoo desde el origen es una tarea no trivial. También el hecho de que Odoo es un servidor que espera las llamadas del cliente, y sólo entonces actúa sobre ellos, lo hace muy diferente de depurar en comparación con los programas del lado del cliente.

#### El depurador Python

Si bien puede parecer un poco intimidante para los recién llegados, el enfoque más pragmático para depurar Odoo es utilizar el depurador integrado de Python, `pdb`. También introduciremos extensiones que proporcionarán una interfaz de usuario más rica, similar a lo que normalmente proporcionan los IDEs sofisticados.

Para usar el depurador, el mejor enfoque es insertar un punto de interrupción en el código que queremos inspeccionar, normalmente un método de modelo. Esto se hace insertando la siguiente línea en el lugar deseado:

```
import pdb; pdb.set_trace() 

```

Ahora reinicia el servidor para que se cargue el código modificado. Tan pronto como la ejecución del programa llegue a esa línea, se mostrará un prompt de Python (`pdb`) en la ventana de terminal donde se está ejecutando el servidor, esperando por nuestra entrada.

#### Nota

La opción `--dev` no es necesaria para usar puntos de interrupción de depurador Python manualmente configurados.

Este prompt funciona como un shell de Python, donde puede ejecutar cualquier expresión o comando en el contexto de ejecución actual. Esto significa que las variables actuales pueden ser inspeccionadas e incluso modificadas. Estos son los comandos de acceso directo más importantes disponibles:

+ `h` (ayuda) muestra un resumen de los comandos de pdb disponibles
+ `p` (imprimir) evalúa e imprime una expresión
+ `pp` (impresión bonita) es útil para imprimir estructuras de datos como diccionarios o listas
+ `l` (lista) enumera el código alrededor de la instrucción siguiente a ejecutar
+ `n`(siguiente) pasos hacia la siguiente instrucción
+ `s`(paso) pasos en la instrucción actual
+ `c`(continuar) continúa la ejecución normalmente
+ `u` (arriba) sube la pila de ejecución
+ `d` (hacia abajo) se mueven hacia abajo en la pila de ejecución

El servidor Odoo también admite la opción `dev=all`. Si se activa, cuando se produce una excepción, el servidor entra en un modo *post mortem* en la línea correspondiente. Este es un prompt `pdb`, como el descrito anteriormente, que nos permite inspeccionar el estado del programa en el momento en que se encontró el error.

Mientras que `pdb` tiene la ventaja de estar disponible fuera de la caja, puede ser bastante concisa, y existen algunas opciones más cómodas.

#### Un ejemplo de sesión de depuración

Veamos cómo se ve una simple sesión de depuración. Podemos comenzar añadiendo punto de interrupción del depurador en la primera línea del método del asistente `do_populate_tasks`:

```
def do_populate_tasks(self):
    import pdb; pdb.set_trace()
    self.ensure_one()
    # ...
```

Ahora reinicia el servidor, abre un formulario **Asistente para tareas pendientes "To-do Tasks Wizard"** y haz clic en el botón **Obtener todo "Get All"**. Esto activará el método del asistente `do_populate_tasks` en el servidor y el cliente web permanecerá en un estado de carga **Loading...**, esperando la respuesta del servidor. Observando la ventana de terminal donde se ejecuta el servidor, verás algo similar a esto:

```
> /home/daniel/odoo-dev/custom-addons/todo_wizard/models/todo_wizard_model.py(54)do_populate_tasks()
-> self.ensure_one()
(Pdb)
```

Este es el prompt del depurador de pdb y las dos primeras líneas te proporcionan información sobre dónde se encuentra en la ejecución del código Python. La primera línea informa el archivo, el número de línea y el nombre de la función en la que te encuentras, y la segunda línea es la siguiente línea de código que se va a ejecutar.

Durante una sesión de depuración, los mensajes del registro del servidor pueden deslizarse. Estos no perjudicarán nuestra depuración, pero pueden perturbarnos. Podemos evitarlo reduciendo la verbosidad de los mensajes de registro. La mayoría de las veces estos mensajes de registro serán del módulo `werkzeug`. Podemos silenciarlos usando la opción `--log-handler=werkzeug:CRITICAL`. Si esto no es suficiente, podemos bajar el nivel de registro general, usando `--log-level=warn`.

Si escribimos `h` ahora, veremos una referencia rápida de los comandos disponibles. Al escribir `l` se muestra la línea de código actual y las líneas de código circundantes.

Al escribir `n` se ejecutará la línea de código actual y se moverá a la siguiente. Si simplemente pulsamos *Enter*, el comando anterior se repetirá. Si haces eso tres veces deberíamos estar en la declaración de retorno del método.

Podemos inspeccionar el contenido de cualquier variable, como las `open_tasks` utilizadas en este método y escribiendo `p open_tasks` o `print open_tasks` mostrará la representación de esa variable. Cualquier expresión de Python está permitida, incluso las asignaciones de variables. Por ejemplo, para mostrar una lista más amigable con los nombres de tareas que podríamos usar:

```
(pdb) p open_tasks.mapped('name')
```

Al ejecutar la línea de retorno, usando `n` una vez más, se mostrarán los valores retornados de la función. Algo como esto:

```
--Return--
> /home/daniel/odoo-dev/custom-addons/todo_wizard/models/todo_wizard_model.py(59)do_populate_tasks()->{'res_id': 14, 'res_model': 'todo.wizard', 'target': 'new', 'type': 'ir.actions.act_window', ...}
-> return self._reopen_form()
```

La sesión de depuración continuará en las líneas de código de la persona que llama, pero podemos terminarla y continuar la ejecución normal escribiendo `c`.

#### Depuradores de Python alternativos

Mientras que `pdb` tiene la ventaja de estar disponible fuera de la caja, puede ser bastante concisa, y algunas opciones más cómodas existen.

El depurador de hierro de Python, `ipdb`, es una opción popular que utiliza los mismos comandos que `pdb`, pero añade mejoras como la finalización de tabulaciones y el resaltado de sintaxis, para un uso más cómodo. Se puede instalar con:

```

$ sudo pip install ipdb






```

Y se agrega un punto de interrupción con la línea:

```
import ipdb; ipdb.set_trace() 
```

Otro depurador alternativo es `pudb`. También soporta los mismos comandos que `pdb` y funciona en terminales sólo de texto, pero utiliza una pantalla gráfica similar a lo que puedes encontrar en un depurador IDE. La información útil, como las variables en el contexto actual y sus valores, está fácilmente disponible en la pantalla en sus propias ventanas:

![debugger](file:img/9-01.jpg)

Se puede instalar a través del gestor de paquetes del sistema o a través de `pip`, como se muestra aquí:

```

$ sudo apt-get install python-pudb  # using OS packages





$ sudo pip install pudb  # using pip, possibly in a virtualenv

```

La adición de un punto de interrupción `pudb` se hace justo de la manera que esperarías:

```
import pudb; pudb.set_trace() 
```

#### Imprmiendo mensajes y registro

A veces sólo necesitamos inspeccionar los valores de algunas variables o comprobar si se están ejecutando algunos bloques de código. Una instrucción `print` de Python puede hacer el trabajo perfectamente sin detener el flujo de ejecución. Al ejecutar el servidor en una ventana de terminal, el texto impreso se mostrará en la salida estándar. Pero no se almacenará en el registro del servidor si se está escribiendo en un archivo.

Otra opción a tener en cuenta es establecer los mensajes de registro de nivel de depuración en puntos sensibles de nuestro código si creemos que podríamos necesitarlos para investigar los problemas en una instancia implementada. Sólo sería necesario elevar ese nivel de registro de servidor para depurar e inspeccionar los archivos de registro.

### Inspeccioando procesos en ejecución

También hay algunos trucos que nos permiten inspeccionar un proceso Odoo en ejecución.

Para eso primero necesitamos encontrar el ID de proceso correspondiente (PID). Para encontrar el PID, ejecuta otra ventana de terminal y escribe:

```
$ ps ax | grep odoo-bin
```
La primera columna en la salida es el PID para ese proceso. Toma una nota en el PID para el proceso de inspección, ya que lo necesitaremos a continuación.

Ahora queremos enviar una señal al proceso. El comando utilizado para hacer eso es matar "kill". De forma predeterminada envía una señal para finalizar un proceso, pero también puede enviar otras señales más amigables.

Conociendo el PID para nuestro proceso de servidor Odoo en ejecución, podemos imprimir las trazas del código que se está ejecutando actualmente usando:

```

$ kill -3 <PID>




```

Si observamos la ventana de terminal o el archivo de registro donde se está escribiendo la salida del servidor, veremos la información de los varios subprocesos que se están ejecutando y los trazados detallados de la pila en la línea de código que están ejecutando.

También podemos ver un volcado de las estadísticas de caché/memoria usando:

```

$ kill -USR1 <PID>





```

## Resumen

Las pruebas automatizadas son una práctica valiosa, tanto para las aplicaciones empresariales en general, como para garantizar la robustez del código en un lenguaje de programación dinámico, como Python.

Aprendimos los principios básicos de cómo agregar y ejecutar pruebas para un módulo addon. También discutimos algunas técnicas para ayudarnos a depurar nuestro código.

En el próximo capítulo, profundizaremos en la capa de vistas, y discutiremos las vistas kanban.