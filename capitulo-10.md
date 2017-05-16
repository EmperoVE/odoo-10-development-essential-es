#Capítulo 10. Creando reportes de QWeb

Los informes son una característica invaluable para las aplicaciones empresariales. El motor incorporado de los informes de QWeb, disponible desde la versión 8.0, es el motor de informe por defecto. Los informes se diseñan utilizando plantillas QWeb para producir documentos HTML que luego se pueden convertir en formato PDF.

Los motores de informe incorporados de Odoo han experimentado cambios significativos. Antes de que los informes de la versión 7.0 estuvieran basados ​​en la biblioteca ReportLab y utilizaran una sintaxis de marcado específico, RML. En la versión 7.0, el motor de informes Webkit se incluyó en el núcleo, permitiendo que los informes se diseñen utilizando HTML normal. Finalmente, en la versión 8.0 este concepto fue tomado un poco más, y las plantillas de QWeb se convirtieron en el concepto principal detrás del motor de informes incorporado.

Esto significa que podemos aprovechar convenientemente lo que hemos aprendido sobre QWeb y aplicarlo para crear informes de negocios. En este capítulo, estaremos agregando un informe a nuestra aplicación de **Tareas Pendientes "To Do"** y revisaremos las técnicas más importantes que se utilizarán con los informes de QWeb, incluidos los cálculos de informes, como los totales, los formatos de traducción y de impresión de papel.

Pero antes de empezar, debemos asegurarnos de que hemos instalado la versión recomendada de la utilidad utilizada para convertir HTML en documentos PDF.


##Instalando wkhtmltopdf

Para generar correctamente los informes, es necesario instalar la versión recomendada de la biblioteca `wkhtmltopdf. Su nombre significa **Webkit HTML a PDF**. Odoo lo usa para convertir una página HTML renderizada en un documento PDF.

Se sabe que las versiones anteriores de la biblioteca `wkhtmltopdf` tienen problemas, como no imprimir encabezados y pie de página, por lo que debemos ser exigentes con la versión que se va a usar. Para la versión 9.0, en el momento de escribir la versión recomendada es 0.12.1. Desafortunadamente, las probabilidades son que la versión empaquetada proporcionada para su sistema host, Debian/Ubuntu u otro, no sea adecuada. Por lo tanto, debemos descargar e instalar el paquete recomendado para nuestro SO y la arquitectura del CPU. Los enlaces de descarga se pueden encontrar en http://wkhtmltopdf.org o http://download.gna.org/wkhtmltopdf.

Primero debemos asegurarnos de que no tenemos una versión incorrecta ya instalada en nuestro sistema:

```
$ wkhtmltopdf --version

```

Si lo anterior informa una versión diferente a la que queremos, debemos desinstalarlo. En un sistema Debian/Ubuntu podemos usar:

```
$ sudo apt-get remove --purge wkhtmltopdf
```

a continuación necesitamos descargar el paquete apropiado para nuestro sistema e instalarlo. Comprueba el nombre de archivo correcto en http://download.gna.org/wkhtmltopdf/0.12/0.12.1. Para Ubuntu 14.04 LTS (Trusty) 64 bits, el comando de descarga sería así:

```
$ wget http://download.gna.org/wkhtmltopdf/0.12/0.12.1/wkhtmltox-0.12.1_linux-trusty-amd64.deb -O /tmp/wkhtml.deb
```

Luego debemos instalarlo. La instalación de un archivo `deb` local no instala automáticamente las dependencias, por lo que se necesitará un segundo paso para hacerlo y completar la instalación:

```

$ sudo dpkg -i wkhtml.deb





$ sudo apt-get -f install 

```

Ahora podemos comprobar si la biblioteca `wkhtmltopdf` está correctamente instalada y confirmar que es el número de versión que queremos:


```

$ wkhtmltopdf --version





wkhtmltopdf 0.12.1 (with patched qt)

```

Después de esto, la secuencia de inicio del servidor Odoo no mostrará el mensaje de información **Necesitas un archivo Wkhtmltopdf para imprimir una versión en pdf del reporte "You need Wkhtmltopdf to print a pdf version of the report's"**.

##Creando informes empresariales

Por lo general, implementaríamos el informe en nuestro módulo de complemento de la aplicación de tareas pendientes. Pero para propósitos de aprendizaje, crearemos un nuevo módulo addon solo para nuestro informe.

Nuestro informe se verá así:

![Report](file:///home/dticucv/Escritorio/OEBPS/Image00027.jpg)

Nombraremos este nuevo módulo addon `todo_report`. Lo primero que debe hacer es crear un archivo `__init__.py` vacío y el archivo de manifiesto `__manifest__.py`: 

```
{  
  'name': 'To-Do Report', 
  'description': 'Report for To-Do tasks.', 
  'author': 'Daniel Reis', 
  'depends': ['todo_kanban'], 
  'data': ['reports/todo_report.xml'] } 

```

El archivo `reports/todo_report.xml` puede comenzar declarando el nuevo informe de la siguiente manera: 

```
<?xml version="1.0"?> 
<odoo> 
  <report id="action_todo_task_report" 
    string="To-do Tasks" 
    model="todo.task" 
    report_type="qweb-pdf" 
    name="todo_report.report_todo_task_template" 
  /> 
</odoo> 

```

La etiqueta `<report>` es un acceso directo para escribir datos al modelo `ir.actions.report.xml`, que es un tipo particular de acción del cliente. Sus datos están disponibles en las opciones de menú **Configuración | Técnico | Informes " Settings | Technical | Reports "**.

####Tip

Durante el diseño del informe, es posible que prefieras dejar `report_type="qweb-html"`, y cambiarlo de nuevo al archivo `qweb-pdf` una vez terminado. Esto hará que sea más rápido generar y más fácil inspeccionar el resultado HTML de la plantilla de OWeb.

Después de instalarlo, la vista de formulario de tareas pendientes mostrará un botón **Imprimir "Print"** en la parte superior, a la izquierda del botón **Más "More"**, que contiene esta opción para ejecutar el informe.

No funcionará en este momento, ya que todavía no hemos definido el informe. Este será un informe QWeb, por lo que utilizará una plantilla QWeb. El atributo `name` identifica la plantilla que se va a utilizar. A diferencia de otras referencias de identificador, se requiere el prefijo del módulo en el atributo `name`. Debemos usar la referencia completa `<module_name>.<identifier_name>`.

##Plantillas de informes QWeb

Los informes generalmente seguirán un esqueleto básico, como se muestra a continuación. Esto se puede agregar al archivo `reports/todo_report.xml`, justo después del elemento `<report>`.

```
<template id="report_todo_task_template"> 
  <t t-call="report.html_container"> 
    <t t-call="report.external_layout"> 
      <div class="page"> 
        <!-- Report page content --> 
      </div> 
    </t> 
  </t> 
</template> 
```

Los elementos más importantes aquí son las directivas `t-call` que utilizan estructuras de informe estándar. La plantilla report.html_container realiza la configuración básica para admitir un documento HTML. La plantilla `report.external_layout` maneja el encabezado y pie de página del informe, utilizando la configuración correspondiente de la empresa adecuada. Como alternativa, podemos usar la plantilla `report.internal_layout`, que utiliza sólo un encabezado básico.

Ahora tenemos en su lugar, el esqueleto básico para nuestro módulo y vista de informe. Ten en cuenta que, dado que los informes son sólo plantillas QWeb, se puede aplicar la herencia, al igual que en las otras vistas. Las plantillas QWeb utilizadas en los informes se pueden ampliar utilizando las vistas heredadas normales con expresiones **XPATH**.

##Presentando datos en informes

A diferencia de las vistas Kanban, las plantillas de QWeb en los informes se representan en el lado del servidor y utilizan una implementación Python QWeb. Podemos ver esto como dos implementaciones de la misma especificación, y hay algunas diferencias que necesitamos tener en cuenta.

Para empezar, las expresiones QWeb se evalúan utilizando la sintaxis de Python, no JavaScript. Para las expresiones más simples, puede haber poca o ninguna diferencia, pero las operaciones más complejas probablemente serán diferentes.

La forma en que se evalúan las expresiones también es diferente. Para los informes, tenemos disponibles las siguientes variables:

+ `docs` es una colección iterable con los registros a imprimir
+ `doc_ids es una lista de los IDs de los registros a imprimir
+ `doc_model` identifica el modelo de los registros, `todo.task` por ejemplo
+ `time is` es una referencia a la biblioteca de tiempo de Python
+ `user` es el registro para el usuario que ejecuta el informe
+ `res_company` es el registro de la empresa del usuario actual

El contenido del informe está escrito en HTML, los valores de los campos pueden referenciarse mediante el atributo `t-field` y puede complementarse con el atributo `t-field-options` para utilizar un widget específico para representar el contenido del campo.

Ahora podemos empezar a diseñar el contenido de la página de nuestro informe:

```
<!-- Report page content  
<div class="row bg-primary"> 
  <div class="col-xs-3"> 
    <span class="glyphicon glyphicon-pushpin" /> 
      What 
  </div> 
  <div class="col-xs-2">Who</div> 
  <div class="col-xs-1">When</div> 
  <div class="col-xs-3">Where</div> 
  <div class="col-xs-3">Watchers</div> 
</div> 
 
<t t-foreach="docs" t-as="o"> 
  <div class="row"> 

    <!-- Data Row Content --> 




  </div> 
</t> 

```

El diseño del contenido puede utilizar el sistema de cuadrícula HTML de Bootstrap de Twitter. En pocas palabras, Bootstrap tiene un diseño de cuadrícula con 12 columnas. Se puede añadir una nueva fila usando `<div class="row">`. Dentro de una fila, tenemos celdas, cada una de ellas abarcando un cierto número de columnas, que debe ocupar las 12 columnas. Cada celda se puede definir con la fila `<div class="col-xs-N">`, donde N es el número de columnas que abarca.

####Nota

Una referencia completa para Bootstrap, que describe estos y otros elementos de estilo, se puede encontrar en http://getbootstrap.com.

Aquí estamos agregando una fila de encabezado con títulos, y luego tenemos un bucle `t-foreach`, ​​iterando a través de cada registro, y representando una fila para cada uno.

Dado que la renderización se realiza en el lado del servidor, los registros son objetos y podemos usar la notación de puntos para acceder a los campos de los registros de datos relacionados. Esto hace que sea fácil de seguir a través de campos relacionales para acceder a sus datos. Ten en cuenta que esto no es posible en el Qweb renderado del lado del cliente, vistas, como las vistas kanban del cliente web.

Este es el XML para el contenido de las filas de registros:

```
<div class="col-xs-3"> 
  <h4><span t-field="o.name" /></h4> 
</div> 
<div class="col-xs-2"> 
  <span t-field="o.user_id" /> 
</div> 
<div class="col-xs-1"> 
  <span t-field="o.date_deadline" /> 
    <span t-field="o.amount_cost" 
      t-field-options='{ 
        "widget": "monetary", 
        "display_currency": "o.currency_id"}'/> 
</div> 
<div class="col-xs-3"> 
  <div t-field="res_company.partner_id" 
    t-field-options='{ 
      "widget": "contact", 
      "fields": ["address", "name", "phone", "fax"], 
                "no_marker": true}' /> 
</div> 
<div class="col-xs-3"> 
  <!-- Render followers --> 
</div> 
```

Como podemos ver, los campos se pueden utilizar con opciones adicionales. Estos son muy similares al atributo `options` utilizado en las vistas de formulario, como se muestra en el Capítulo 6, *Vistas - Diseñando la interfaz de usuario*, utilizado con un `widget` adicional para configurar el widget para que se utilice para representar el campo.

Un ejemplo es el widget monetario, usado arriba, al lado de la fecha límite.

Un ejemplo más sofisticado es el widget `contact`, utilizado para dar formato a las direcciones. Utilizamos la dirección de la empresa, `res_company.partner_id`, ya que tiene algunos datos predeterminados y podemos ver inmediatamente la dirección procesada. Pero tendría más sentido usar la dirección del usuario asignado, `o.user_id.partner_id`. De forma predeterminada, el widget `contact` muestra direcciones con algunos pictogramas, como un icono de teléfono. La opción `no_marker="true"` que usamos los deshabilita.

##Renderizando imágenes

La última columna de nuestro informe contará con la lista de seguidores, con los avatares. Usaremos el componente Bootstrap `media-list` y un bucle a través de los seguidores para renderizar cada uno de ellos:

```
<!-- Render followers --> 
<ul class="media-list"> 
  <t t-foreach="o.message_follower_ids" t-as="f"> 
    <li t-if="f.partner_id.image_small"  
      class="media-left"> 
      <img class="media-object" 
        t-att-src="'data:image/png;base64,%s' %  
          f.partner_id.image_small" 
        style="max-height: 24px;" /> 
      <span class="media-body"  
        t-field="f.partner_id.name" /> 
    </li> 
  </t> 
</ul> 
```
El contenido de los campos binarios se proporciona en una representación `base64`. El elemento `<img>` puede aceptar directamente este tipo de datos para el atributo `src`. Así podemos utilizar la directiva Qweb `t-att-src` para generar dinámicamente cada una de las imágenes.

##Total de resumen y totales corrientes

Una necesidad común en los informes es proporcionar totales. Esto se puede hacer usando expresiones de Python para calcular esos totales.

Después de la etiqueta de cierre de `<t t-foreach>`, añadiremos una fila final con los totales:

```
<!-- Totals --> 
<div class="row"> 
  <div class="col-xs-3"> 
    Count: <t t-esc="len(docs)" /> 
  </div> 
  <div class="col-xs-2" /> 
  <div class="col-xs-1"> 
    Total:  
   <t t-esc="sum([o.amount_cost for o in docs])" /> 
  </div> 
  <div class="col-xs-3" /> 
  <div class="col-xs-3" /> 
</div> 
```

La instrucción Python `len()` se utiliza para contar el número de elementos de una colección. Los totales se pueden calcular utilizando el valor `sum()` sobre una lista de valores. En el ejemplo anterior, utilizamos una comprensión de lista para producir una lista de valores fuera del conjunto de registros `docs`. Puedes pensar en la lista de comprensiones como un bucle `for` incorporado.

A veces queremos realizar algunos cálculos a medida que vamos junto con el informe. Por ejemplo, un total de ejecución, con el total hasta el registro actual. Esto se puede implementar con t-set para definir una variable de acumulación y luego actualizarla en cada fila.

Para ilustrar esto, podemos calcular el número acumulado de seguidores. Debemos empezar por inicializar la variable, justo antes del bucle `t-foreach` en el conjunto de registros `docs`, usando:

```
<t t-set="follower_count" t-value="0" /> 
```
Y luego, dentro del bucle, añade el número de seguidores del registro a la variable. Elegiremos hacerlo justo después de presentar la lista de seguidores, e imprimiremos también el total actual en cada línea:

```
<!-- Running total--> 
  <t t-set="follower_count" 
    t-value="follower_count + len(o.message_follower_ids)" /> 
  Accumulated # <t t-esc="follower_count" /> 

```

##Definiendo formatos de papel
En este punto nuestro informe se ve bien en HTML, pero no se imprime bien en una página PDF. Podríamos obtener mejores resultados usando una página horizontal. Así que tenemos que añadir este formato de papel.

En la parte superior del archivo XML, agregua este registro:

```
<record id="paperformat_euro_landscape" 
        model="report.paperformat"> 
  <field name="name">European A4 Landscape</field> 
  <field name="default" eval="True" /> 
  <field name="format">A4</field> 
  <field name="page_height">0</field> 
  <field name="page_width">0</field> 
  <field name="orientation">Landscape</field> 
  <field name="margin_top">40</field> 
  <field name="margin_bottom">23</field> 
  <field name="margin_left">7</field> 
  <field name="margin_right">7</field> 
  <field name="header_line" eval="False" /> 
  <field name="header_spacing">35</field> 
  <field name="dpi">90</field> 
</record> 
```

Es una copia del formato A4 europeo, definido en el archivo `report_paperformat.xml`, efinido en `addons/report/data`, pero cambiando la orientación de Vertical a Horizontal. Los formatos de papel definidos se pueden ver desde el cliente web a través del menú **Configuración | Técnico | Informes | Formato del papel "Settings | Technical | Reports | Paper Format"**.

Ahora podemos usarlo en nuestro informe. El formato de papel predeterminado se define en la configuración de la empresa, pero también podemos especificar el formato de papel que utilizará un informe específico. Esto se hace utilizando un atributo `paperfomat` en la acción del informe.

Vamos a editar la acción utilizada para abrir nuestro informe, para agregar este atributo:

```
<report id="action_todo_task_report" 
  string="To-do Tasks" 
  model="todo.task" 
  report_type="qweb-pdf" 
  name="todo_report.report_todo_task_template" 
  paperformat="paperformat_euro_landscape" 
/> 
```
####Tip

El atributo `paperformat` en la etiqueta `<report>` se agregó en la versión 9.0. Para 8.0 necesitamos usar un elemento `<record>` para agregar una acción de informe con un valor `paperformat`.

##Habilitando la traducción de idiomas en los informes

Para habilitar las traducciones de un informe, debe ser llamado desde una plantilla, usando un elemento `<t t-call>` con un atributo `t-lang`.

El atributo `t-lang` debe evaluarse a un código de idioma, como `es` o `en_US`. Necesita el nombre del campo donde se puede encontrar el idioma a utilizar. Este será con frecuencia el idioma del socio al que se enviará el documento, almacenado en el campo `partner_id.lang`. En nuestro caso, no tenemos un campo de socio, pero podemos utilizar el usuario responsable, y la preferencia de idioma correspondiente está en `user_id.lang`.

La función espera un nombre de plantilla, y lo procesará y traducirá. Esto significa que necesitamos definir el contenido de la página de nuestro informe en una plantilla separada, como se muestra a continuación:

```
<report id="action_todo_task_report_translated" 
  string="Translated To-do Tasks" 
  model="todo.task" 
  report_type="qweb-pdf" 
  name="todo_report.report_todo_task_translated" 
  paperformat="paperformat_euro_landscape" 
/> 
 
<template id="report_todo_task_translated"> 
  <t t-call="todo_report.report_todo_task_template" 
    t-lang="user.lang" > 
    <t t-set="docs" 
      t-value="docs" /> 
    </t> 
  </t> 
</template> 

```

##Informes basados ​​en SQL personalizado

El informe que construimos se basó en un conjunto de registros regulares. Pero en algunos casos necesitamos transformar o agregar datos de una manera que no sea fácil al procesar datos sobre la marcha, como al procesar el informe.

Un enfoque para esto es escribir una consulta SQL para construir el conjunto de datos que necesitamos, exponer esos resultados a través de un modelo especial y tener nuestro trabajo de informe basado en un conjunto de registros.

Para ello, crearemos un archivo `reports/todo_task_report.py` con este código:

```
# -*- coding: utf-8 -*- 
from odoo import models, fields 
 
class TodoReport(models.Model): 
    _name = 'todo.task.report' 
    _description = 'To-do Report'
    _sql = """
           CREATE OR REPLACE VIEW todo_task_report AS
           SELECT *
           FROM todo_task
           WHERE active = True
           """ 
    name = fields.Char('Description') 
    is_done = fields.Boolean('Done?') 
    active = fields.Boolean('Active?') 
    user_id = fields.Many2one('res.users', 'Responsible') 
    date_deadline = fields.Date('Deadline') 
 
    

```

Para que este archivo se cargue necesitamos agregar una línea `from . import reports` a la parte superior del archivo `__init__.py` y `from . import todo_task_report` al archivo `reports/__init__.py`.

El atributo `sql` se utiliza para anular la creación automática de la tabla de la base de datos, proporcionando un SQL para eso. Queremos que cree una vista de base de datos para proporcionar los datos necesarios para el informe. Nuestra consulta SQL es bastante simple, pero el punto es que podríamos usar cualquier consulta SQL válida para nuestra vista.

También mapeamos los campos que necesitamos con tipos de campo ORM, para que estén disponibles en conjuntos de registros generados en este modelo.

A continuación, podemos agregar un nuevo informe basado en este modelo, `reports/todo_model_report.xml`:

```
<odoo> 
 
<report id="action_todo_model_report" 
  string="To-do Special Report" 
  model="todo.task" 
  report_type="qweb-html" 
  name="todo_report.report_todo_task_special" 
/> 
 
<template id="report_todo_task_special"> 
  <t t-call="report.html_container"> 
    <t t-call="report.external_layout"> 
      <div class="page"> 
 
        <!-- Report page content --> 
        <table class="table table-striped"> 
          <tr> 
            <th>Title</th> 
            <th>Owner</th> 
            <th>Deadline</th> 
          </tr> 
          <t t-foreach="docs" t-as="o"> 
            <tr> 
              <td class="col-xs-6"> 
                <span t-field="o.name" /> 
              </td> 
              <td class="col-xs-3"> 
                <span t-field="o.user_id" /> 
              </td> 
              <td class="col-xs-3"> 
                <span t-field="o.date_deadline" /> 
              </td> 
            </tr> 
          </t> 
        </table> 
 
      </div> 
    </t> 
  </t> 
</template> 
 
</odoo> 

```

Para casos aún más complejos, podemos utilizar una solución diferente: un asistente. Para esto debemos crear un modelo transitorio con líneas relacionadas, donde el encabezado incluya parámetros de informe, introducidos por el usuario, y las líneas tendrán los datos generados que usará el informe. Estas líneas se generan mediante un método modelo que puede contener cualquier lógica que podamos necesitar. Se recomienda encarecidamente inspirarse en un informe similar existente.

##Resúmen

En el capítulo anterior aprendimos sobre QWeb y cómo usarlo para diseñar una vista Kanban. En este capítulo aprendimos sobre el motor de informes QWeb y sobre las técnicas más importantes para generar informes con el lenguaje de plantillas QWeb.

En el próximo capítulo, seguiremos trabajando con QWeb, esta vez para construir páginas web. También aprenderemos a escribir controladores web, proporcionando funciones más completas a nuestras páginas web.