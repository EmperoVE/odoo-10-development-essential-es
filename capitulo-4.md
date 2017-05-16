# Capítulo 4. Datos del módulo

La mayoría de las definiciones de módulos Odoo, como las interfaces de usuario y las reglas de seguridad, son en realidad registros de datos almacenados en tablas de bases de datos específicas. Los archivos XML y CSV que se encuentran en los módulos no son utilizados por las aplicaciones Odoo en tiempo de ejecución. En su lugar, son un medio para cargar esas definiciones en las tablas de la base de datos.

Debido a esto, una parte importante de los módulos de Odoo consiste en representar (serializando) esos datos utilizando archivos de modo que puedan cargarse posteriormente en una base de datos.

Los módulos también pueden tener datos predeterminados y de demostración. La representación de datos permite añadir eso a nuestros módulos. Además, la comprensión de los formatos de representación de datos de Odoo es importante para exportar e importar datos empresariales en el contexto de la implementación de un proyecto.

Antes de entrar en casos prácticos, primero exploraremos el concepto de identificador externo, que es la clave para la representación de datos Odoo.

## Comprendiendo los identificadores externos

Un **identificador externo** (también llamado XML ID) es un identificador de cadena legible por humanos que identifica de forma única un registro particular en Odoo. Son importantes al cargar datos en Odoo.

Una de las razones es que al actualizar un módulo, sus archivos de datos se cargarán nuevamente en la base de datos y debemos detectar los registros ya existentes para actualizarlos en lugar de crear nuevos registros duplicados.

Otra razón que apoya los datos interrelacionados: los registros de datos deben ser capaces de hacer referencia a otros registros de datos. El identificador de base de datos real es un número secuencial asignado automáticamente por la base de datos, durante la instalación del módulo. Los identificadores externos proporcionan una forma de hacer referencia a un registro relacionado sin necesidad de saber de antemano qué ID de base de datos se asignará, lo que nos permite definir las relaciones de datos en los archivos de datos Odoo.

Odoo se encarga de traducir los nombres de identificadores externos en los IDs de base de datos reales asignados a ellos. El mecanismo detrás de esto es bastante simple: Odoo mantiene una tabla con el mapeo entre los identificadores externos nombrados y sus IDs de base de datos numéricos correspondientes: el modelo `ir.model.data`.

Para inspeccionar los mapeos existentes, ve a la sección **Technical** del menú superior **Settings** y selecciona el elemento de menú **External Identifiers** bajo **Sequences & Identifiers**.

Por ejemplo, si visitamos la lista de **External identifiers** y lo filtramos por el módulo `todo_app`, veremos los identificadores externos generados por el módulo creado anteriormente:

![Externalidentifiers](file:///home/dticucv/Escritorio/OEBPS/Image00013.jpg)

Podemos ver que los identificadores externos tienen una etiqueta **ID Completa**. Observa cómo se compone del nombre del módulo y el nombre del identificador unido por un punto, por ejemplo, `todo_app.action_todo_task`.

Los identificadores externos deben ser únicos dentro de un módulo Odoo, por lo que no hay riesgo de que dos módulos entren en conflicto porque accidentalmente eligieron el mismo nombre de identificador. Para construir un identificador único global Odoo une el nombre de los módulos con el nombre del identificador externo real. Esto es lo que puede ver en el campo **ID Completo**.

Cuando se utiliza un identificador externo en un archivo de datos, puedes elegir utilizar el identificador completo o simplemente el nombre del identificador externo. Por lo general, es más sencillo usar el nombre de identificador externo, pero el identificador completo nos permite referenciar registros de datos de otros módulos. Al hacerlo, asegúrate de que esos módulos están incluidos en las dependencias del módulo, para asegurarse de que esos registros estén cargados antes de los nuestros.

En la parte superior de la lista, tenemos el identificador completo `todo_app.action_todo_task`. Esta es la acción de menú que creamos para el módulo, al que también se hace referencia en el ítem de menú correspondiente. Al hacer clic en él, vamos a la vista de formulario con sus detalles; El identificador externo `action_todo_task` en el módulo `todo_app` mapea un ID de registro específico en el modelo `ir.actions.act_window`, `72` en este caso:

![Todo](file:///home/dticucv/Escritorio/OEBPS/Image00014.jpg)

Además de proporcionar una forma para que los registros hagan referencia fácilmente a otros registros, los identificadores externos también te permiten evitar la duplicación de datos en las importaciones repetidas. Si el identificador externo ya está presente, se actualizará el registro existente; No es necesario crear un nuevo registro. Es por esto que en las actualizaciones de módulos posteriores, los registros cargados previamente se actualizan en vez de duplicarse.

### Hallando identificadores externos

Al preparar archivos de definición y demostración de datos para los módulos, con frecuencia necesitamos buscar identificadores externos existentes que sean necesarios para las referencias.

Podemos usar el menú de **External Identifiers** mostrado anteriormente, pero el menú **Developer** puede proporcionar un método más conveniente para esto. Como puedes recordar del Capítulo 1, *Iniciando con desarrollo de Odoo*, el menú **Developer** se activa en el panel de **Settings**, en una opción en la parte inferior derecha.

Para encontrar el identificador externo de un registro de datos, en la vista de formulario correspondiente, seleccione la opción **View Metadata** en el menú **Developer**. Esto mostrará un diálogo con el ID de base de datos del registro y el identificador externo (también conocido como XML ID).

Como ejemplo, para buscar el ID de `demo user`, podemos navegar a la vista de formulario, en **Settings | Users** y seleccione la opción **View Metadata** y esto se mostrará:

![Metadata](file:///home/dticucv/Escritorio/OEBPS/Image00015.jpg)

Para hallar el identificador externo para elementos de vista, como formulario, árbol, búsqueda o acción, el menú **Developer** también es una buena fuente de ayuda. Para ello, podemos utilizar la opción **Manage Views** o abrir la información para la vista deseada mediante la opción **Edit <view type>**. A continuación, selecciona la opción **View metadata**.

## Exportando e importando data

Comenzaremos a explorar cómo exportar e importar datos desde la interfaz de usuario de Odoo, ya partir de ahí, pasaremos a los detalles más técnicos sobre cómo utilizar los archivos de datos en nuestros módulos de complemento.

### Exportando data

La exportación de datos es una característica estándar disponible en cualquier vista de lista. Para usarlo, primero debemos seleccionar las filas para exportar seleccionando las casillas de verificación correspondientes en el extremo izquierdo y luego seleccionar la opción **Export** del botón **More**.

He aquí un ejemplo, usando las tareas de tareas pendientes creadas recientemente:

![Todocreate](file:///home/dticucv/Escritorio/OEBPS/Image00016.jpg)

También podemos marcar la casilla de verificación en el encabezado de la columna. Comprobará todos los registros a la vez y exportará todos los registros que coincidan con los criterios de búsqueda actuales.

### Nota

En versiones anteriores de Odoo, sólo los registros vistos en la pantalla (la página actual) se exportarían realmente. Desde Odoo 9, esta casilla de verificación cambió y marcar el checkbox en el encabezado exportará todos los registros que coincidan con el filtro actual, no sólo los que se muestran actualmente. Esto es muy útil para exportar grandes conjuntos de registros que no encajan en la pantalla.

La opción **Export** nos lleva a un formulario de diálogo, donde podemos elegir qué exportar. La opción **Import-Compatible Export** (Importar exportación compatible) asegura que el archivo exportado se puede importar de nuevo a Odoo. Necesitaremos usar esto.

El formato de exportación puede ser CSV o Excel. Preferiremos un archivo CSV para obtener una mejor comprensión del formato de exportación. A continuación, seleccionamos las columnas que queremos exportar y hacemos clic en el botón **Export to File**. Esto iniciará la descarga de un archivo con los datos exportados:

![Minion](file:///home/dticucv/Escritorio/OEBPS/Image00016.jpg)

Si seguimos estas instrucciones y seleccionamos los campos mostrados en la captura de pantalla anterior, deberíamos terminar con un archivo de texto CSV similar a este:

```
"id","name","user_id/id","date_deadline","is_done" 
"todo_user.todo_task_a","Install Odoo","base.user_root","2015-01-30","False" 
"__export__.todo_task_9","Create my first module","base.user_root","","False"
```


#### Nota

Observa que Odoo exportó automáticamente una columna de id adicional. Este es un identificador externo asignado a cada registro. Si no se ha asignado ninguna todavía, se genera automáticamente una nueva utilizando `__export__` en lugar de un nombre de módulo real. Los nuevos identificadores se asignan solamente a los registros que no tienen ya uno, y de allí en adelante, se mantienen ligados al mismo expediente. Esto significa que las exportaciones posteriores preservarán los mismos identificadores externos.

### Importando datos

Primero tenemos que asegurarnos de que la función de importación esté habilitada. Ya que Odoo 9 está habilitado de forma predeterminada. Si no, la opción está disponible en el menú principal **Settings**, opción **General Settings**. Bajo la sección **Import | Export** hay una casilla de verificación **Allow users to import data from CSV/XLS/XLSX/ODS files** que debe ser habilitada. 

#### Nota

Esta característica es proporcionada por el complemento **Initial Setup Tools** (`base_setup` es el nombre técnico). El efecto real de la casilla **Allow users to import**... es instalar o desinstalar `base_setup`.

Con esta opción habilitada las vistas de lista muestran una opción **Import** junto al botón **Create** en la parte superior de la lista.

Vamos a realizar una edición masiva en nuestros datos de tareas pendientes primero. Abre el archivo CSV que acabamos de descargar en una hoja de cálculo o un editor de texto y cambia algunos valores. Además, agregua algunas nuevas filas, dejando la columna de ID en blanco.

Como se mencionó anteriormente, la primera columna, id, proporciona un identificador único para cada fila. Esto permite actualizar los registros ya existentes en vez de duplicarlos cuando importamos los datos de nuevo a Odoo. Para las nuevas filas añadidas al archivo CSV, podemos elegir entre proporcionar un identificador externo de nuestra elección, o dejar en blanco la columna id, se creará un nuevo registro para ellos.

Después de guardar los cambios en el archivo CSV, haz clic en la opción **Import** (junto al botón **Create**) y se nos presentará el asistente de importación.

Allí, debemos seleccionar la ubicación del archivo CSV en el disco y hacer clic en **Validate** para comprobar su exactitud. Ya que que el archivo a importar se basa en una exportación Odoo, lo más probable es que sea válido:

![Minion](file:///home/dticucv/Escritorio/OEBPS/Image00017.jpg)

Ahora podemos hacer clic en **Import**, y ahí lo tienes; Nuestras modificaciones y nuevos registros deberían haber sido cargados en Odoo.

### Archivos relacionados en archivos de datos CSV

En el ejemplo anterior, el usuario responsable de cada tarea es un registro relacionado en el modelo de usuario, con una relación de varios a uno (o una clave externa). El nombre de columna utilizado para ello fue `id_usuario / id` y los valores de campo eran identificadores externos para los registros relacionados, tales como  `base.user_root` para el usuario administrador.
 
#### Tip
El uso de IDs de base de datos sólo se recomienda si estás exportando e importando desde / hasta la misma base de datos. Por lo general, preferirás utilizar identificadores externos.

Las columnas de la relación deben tener `/ id` añadido a su nombre si utilizan identificadores externos o `/.id` si utilizan IDs (numéricos) de la base de datos. Alternativamente, se puede usar un colon (:) en lugar de una barra para el mismo efecto.

Del mismo modo, también se apoyan las relaciones de muchos a muchos. Un ejemplo de una relación muchos-a-muchos es el que existe entre usuarios y grupos: cada usuario puede estar en muchos grupos y cada grupo puede tener muchos usuarios. El nombre de columna para este tipo de campo debe tener `/ id` añadido. Los valores de campo aceptan una lista separada por comas de identificadores externos, rodeada de comillas dobles.

Por ejemplo, los seguidores de tareas pendientes tienen una relación muchos-a-muchos entre las tareas por realizar y los socios. Su nombre de columna debe ser `follower_ids / id`, y un valor de campo con dos seguidores podría tener este aspecto:
```
"__export__.res_partner_1,__export__.res_partner_2"
```
Finalmente, las relaciones uno-a-muchos también se pueden importar a través de un archivo CSV. El ejemplo típico para este tipo de relación es un documento *cabecera* de documento con varias *líneas*. Observa que una relación de uno a muchos es siempre la inversa de una relación de muchos a uno. Cada *cabecera* del documento puede tener muchas *líneas* e inversamente cada línea tiene una *cabecera*.

Podemos ver un ejemplo de tal relación en el modelo de empresa (la vista de formulario está disponible en el menú **Settings**): cada empresa puede tener varias cuentas bancarias, cada una con sus propios detalles; Inversamente, cada registro de cuenta bancaria pertenece y tiene una relación de muchos a uno con una sola compañía.

Podemos importar empresas junto con sus cuentas bancarias en un solo archivo. Aquí hay un ejemplo donde cargamos una empresa con tres bancos:
```
id,name,bank_ids/id,bank_ids/acc_number,bank_ids/state 
base.main_company,YourCompany,__export__.res_partner_bank_4,123456789,bank 
,,__export__.res_partner_bank_5,135792468,bank 
,,__export__.res_partner_bank_6,1122334455,bank

```

Podemos ver que las dos primeras columnas, `id` y `name`, tienen valores en la primera línea y están vacíos en las dos líneas siguientes. Tienen datos para el registro de la cabeza, que es la empresa.

Las otras tres columnas están todas prefijadas con `bank_ids /` y tienen valores en todas las tres líneas. Tienen datos para las tres líneas relacionadas para las cuentas bancarias de la compañía. La primera línea tiene datos de la empresa y el primer banco, y las dos líneas siguientes tienen datos sólo para la compañía adicional y los bancos.

Estos son aspectos esenciales mientras se trabaja con la exportación e importación de la GUI. Es útil configurar datos en nuevas instancias de Odoo o preparar archivos de datos que se incluirán en módulos Odoo. A continuación, aprenderemos más sobre el uso de archivos de datos en módulos.

## Datos de modulo

Los módulos utilizan archivos de datos para cargar sus configuraciones en la base de datos, datos predeterminados y datos de demostración. Esto se puede hacer utilizando archivos CSV y XML. Para completar, también se puede usar el formato de archivo YAML, pero es muy raro que se use para cargar datos; Por lo tanto, no lo estaremos discutiendo.

Los archivos CSV utilizados por los módulos son exactamente los mismos que los que hemos visto y utilizado para la función de importación. Cuando se usan en módulos, una restricción adicional es que el nombre de archivo debe coincidir con el nombre del modelo al que se cargarán los datos, de modo que el sistema pueda inferir el modelo en el que deben importarse los datos.

Un uso común de los archivos CSV de datos es para acceder a definiciones de seguridad, cargadas en el modelo `ir.model.access`. Usualmente usan archivos CSV que se llaman `ir.model.access.csv`.

### Datos demostrativos

Los módulos addon de Odoo pueden instalar datos de demostración, y se considera una buena práctica hacerlo. Esto es útil para proporcionar ejemplos de uso para un módulo y conjuntos de datos que se utilizarán en las pruebas. Los datos de demostración de un módulo se declaran utilizando el atributo de demostración del archivo de manifiesto `__manifest__.py`. Al igual que el atributo de datos, es una lista de nombres de archivo con las correspondientes rutas relativas dentro del módulo.

Es hora de agregar algunos datos de demostración a nuestro módulo `todo_user`. Podemos comenzar exportando algunos datos de las tareas pendientes, como se explicó en la sección anterior. La convención es colocar los archivos de datos en un subdirectorio `data/`. Así que debemos guardar estos archivos de datos en el módulo addon `todo_user` como `data / todo.task.csv`. Dado que estos datos serán propiedad de nuestro módulo, deberíamos editar los valores de ID para eliminar el prefijo `__export__` de los identificadores.

Por ejemplo, nuestro archivo de datos `todo.task.csv` podría tener este aspecto:
```
id,name,user_id/id,date_deadline 
todo_task_a,"Install Odoo","base.user_root","2015-01-30" 
todo_task_b","Create dev database","base.user_root","" 
```
no debemos olvidar añadir este archivo de datos al atributo `demo` `__manifest__.py`:
```
'demo': ['data/todo.task.csv'], 
```
La próxima vez que actualizemos el módulo, siempre y cuando se instale con los datos de demostración habilitados, se importará el contenido del archivo. Ten en cuenta que estos datos se reimportarán siempre que se realice una actualización de módulo.

Los archivos XML también se utilizan para cargar los datos del módulo. Vamos a aprender más acerca de lo que los archivos de datos XML pueden hacer que los archivos CSV no pueden.

## Archivo de datos XML
Mientras que los archivos CSV proporcionan un formato simple y compacto para representar datos, los archivos XML son más potentes y dan más control sobre el proceso de carga. Sus nombres de archivo no son necesarios para coincidir con el modelo que se va a cargar. Esto se debe a que el formato XML es mucho más rico y esa información es proporcionada por los elementos XML dentro del archivo.

Ya hemos utilizado archivos de datos XML en los capítulos anteriores. Los componentes de la interfaz de usuario, tales como vistas y elementos de menú, son de hecho registros almacenados en modelos de sistema. Los archivos XML en los módulos son los medios utilizados para cargar estos registros en el servidor.

Para mostrar esto, agregaremos un segundo archivo de datos al módulo `todo_user`, `data / todo_data.xml`, con el siguiente contenido:
```
<?xml version="1.0"?> 
<odoo> 
  <!-- Data to load --> 
  <record model="todo.task" id="todo_task_c"> 
    <field name="name">Reinstall Odoo</field> 
    <field name="user_id" ref="base.user_root" /> 
    <field name="date_deadline">2015-01-30</field> 
    <field name="is_done" eval="False" /> 
  </record> 
</odoo> 
```
Este XML es equivalente al archivo de datos CSV que acabamos de ver en la sección anterior.

Los archivos de datos XML tienen un elemento superior `<odoo>`, dentro del cual podemos tener varios elementos `<record>` que corresponden a las filas de datos CSV.

#### Nota

El elemento superior `<odoo>` en archivos de datos se introdujo en la versión 9.0 y reemplaza a la etiqueta anterior `<openerp>`. La sección `<data>` dentro del elemento superior todavía es compatible, pero ahora es opcional. De hecho, ahora `<odoo>` y `<data>` son equivalentes, por lo que podríamos usar uno como elementos principales para nuestros archivos de datos XML.

Un elemento `<record>` tiene dos atributos obligatorios a saber, `modelo` e `id` (el identificador externo para el registro), y contiene una etiqueta `<field>` para cada campo en el que escribir.

Ten en cuenta que la notación de barras en los nombres de campo no está disponible aquí; No podemos usar `<field name = "user_id / id">`. En su lugar, el atributo `ref` special se utiliza para hacer referencia a identificadores externos. Discutiremos los valores de los campos relacionados-a-muchos en un momento.

### El atributo data noupdate

Cuando se repite la carga de datos, los registros cargados de la ejecución anterior se reescriben. Es importante tener en cuenta que esto significa que la actualización de un módulo sobrescribirá los cambios manuales que se pudieron haber realizado dentro de la base de datos. En particular, si las vistas se modificaron con las personalizaciones, estos cambios se perderán con la próxima actualización del módulo. El procedimiento correcto consiste en crear vistas heredadas para los cambios que necesitamos, como se describe en el Capítulo 3, *Herencia - Extendiendo las aplicaciones existentes*.

Este comportamiento de re-importación es el predeterminado, pero se puede cambiar, de modo que cuando se actualiza un módulo, algunos registros de archivos de datos quedan intactos. Esto se hace mediante el atributo `noupdate = "1"` del elemento `<odoo>` o `<data>`. Estos registros se crearán cuando se instale el módulo addon, pero en las actualizaciones de módulos posteriores no se les hará nada.

Esto te permite asegurar que las personalizaciones hechas manualmente se mantengan a salvo de las actualizaciones de módulos. A menudo se utiliza con reglas de acceso a registros, lo que les permite adaptarse a las necesidades específicas de la implementación.

Es posible tener más de una sección `<data>` en el mismo archivo XML. Podemos aprovechar esto para separar los datos para importar sólo uno, con `noupdate = "1"`, y los datos para ser reimportados en cada actualización, con `noupdate = "0"`.

El indicador `noupdate` se almacena en la información de **External Identifier** para cada registro. Es posible editarlo manualmente directamente usando el formulario al Identifier** disponible en el menú **Technical**, usando la casilla **Non Updatable**.
####Tip

El atributo `noupdate` puede ser complicado al desarrollar módulos, ya que los cambios realizados en los datos posteriormente serán ignorados. Una solución es, en lugar de actualizar el módulo con la opción `-u`, vuelve a instalarlo con la opción `-i`. Al reinstalar desde la línea de comandos utilizando la opción `-i`, se ignoran los indicadores de `noupdate` en los registros de datos.

### Definiendo registros en XML

Cada elemento `<record>` tiene dos atributos básicos, `id` y `model`, y contiene elementos `<field>` que asignan valores a cada columna. Como se mencionó anteriormente, el atributo `id` corresponde al identificador externo del registro y al atributo `model` al modelo de destino en el que se escribirá el registro. Los elementos <field> tienen diferentes formas de asignar valores. Echemos un vistazo a ellos en detalle.

#### Configurando de valores de campo

El elemento `<record>` define un registro de datos y contiene elementos `<field>` para establecer valores en cada campo.

El atributo `name` del elemento de campo identifica el campo que se va a escribir.

El valor a escribir es el contenido del elemento: el texto entre la etiqueta de apertura y cierre del campo. Para dates y datetimes, las cadenas con `"YYYY-mm-dd"` y `"AAAA-mm-dd HH: MM: SS"` se convertirán correctamente. Sin embargo, para los campos Booleanos cualquier valor no vacío se convertirá en `True` y los valores `"0"` y `"False"` se convertirán en `False`.

#### Tip

La forma en que los valores Booleanos `false` se leen de los archivos de datos se mejora en Odoo 10. En las versiones anteriores, los valores no vacíos, incluidos `"0"` y `"Falso"`, se convirtieron en `True`. Serecomendó para los booleanos que usaban el atributo `eval` descrito a continuación.

#### Configurando valores mediante expresiones

Una alternativa más elaborada para definir un valor de campo es el atributo `eval`. Evalúa una expresión de Python y asigna el valor resultante al campo.

La expresión se evalúa en un contexto que, además de Python incorporado, también tiene algunos identificadores adicionales disponibles. Echemos un vistazo a ellos.

Para manejar fechas, están disponibles los siguientes módulos: `time, datetime, timedelta y relativedelta`. Ellos permiten calcular valores de fecha, algo que se utiliza con frecuencia en los datos de demostración y prueba, de modo que las fechas utilizadas están cerca de la fecha de instalación del módulo. Por ejemplo, para establecer un valor para ayer, utilizaremos esto:

```
<field name="date_deadline" 
  eval="(datetime.now() + timedelta(-1)).strftime('%Y-%m-%d')" />
```


También está disponible en el contexto de evaluación la función `ref ()`, que se utiliza para traducir un identificador externo en el ID de base de datos correspondiente. Esto se puede usar para establecer valores para campos relacionales. Como ejemplo, lo hemos utilizado antes para establecer el valor de `user_id`:

```
<field name="user_id" eval="ref('base.group_user')" /> 
```

#### Valores de ajuste para campos de relación

Acabamos de ver cómo establecer un valor en un campo de relación many-to-one, como `user_id`, usando el atributo `eval` con una función `ref ()`. Pero hay una manera más simple.

El elemento `<field>` también tiene un atributo `ref` para establecer el valor de un campo many-to-one, utilizando un identificador externo. Con esto, podemos fijar el valor para `user_id` usando apenas esto:
```

<Field name = "user_id" ref = "base.user_demo" />
```

Para los campos one-to-many y many-to-many, se espera una lista de IDs relacionados, por lo que se necesita una sintaxis diferente; Odoo proporciona una sintaxis especial para escribir en este tipo de campos.

El siguiente ejemplo, tomado de la aplicación oficial Fleet, reemplaza la lista de registros relacionados de un campo `tag_ids`:
```
<field name="tag_ids" 
  eval="[(6,0,    
    [ref('vehicle_tag_leasing'), 
    ref('fleet.vehicle_tag_compact'), 
    ref('fleet.vehicle_tag_senior')] 
  )]" /> 

```

Para escribir en un campo to-many, usamos una lista de triples. Cada triple es un comando de escritura que hace cosas diferentes según el código usado:

+ (0,_ ,{'field': value}) crea un nuevo regristro y lo enlaza con este 
+ (1,id,{'field': value}) Actualiza los valores en un registro previamente enlazado
+ (2,id,_) Desenlaza y elimina un registro relacionado 
+ (3,id,_) Desenlaza pero no elimina un registro relacionado 
+ (4,id,_) Enlaza un registro ya existente
+ (5,_,_) Desenlaza pero no elimina todos los registros relacionados
+ (6,_,[ids]) reemplaza la lista de registros enlazados con la lista proporcionada

El símbolo de subrayado utilizado en la lista anterior representa valores irrelevantes, normalmente se rellenan con `0` o `False`.

### Atajos para los modelos de uso frecuente

Si volvemos al Capítulo 2, *Construyendo tu Primera Aplicación Odoo*, encontraremos elementos distintos de `<record>`, como `<act_window>` y `<menuitem>`, en los archivos XML.

Estos son accesos directos convenientes para los modelos de uso frecuente que también se pueden cargar utilizando elementos `<record>` normales. Ellos cargan datos en modelos de base que soportan la interfaz de usuario y se explorarán con más detalle más adelante, específicamente en el Capítulo 6, *Vistas - Diseñando la interfaz de usuario*.

Como referencia, los siguientes elementos de acceso directo están disponibles con los modelos correspondientes en los que cargan datos:

+ `<Act_window>` es el modelo de acción de la ventana, `ir.actions.act_window`
+ `<Menuitem>` es el modelo de elementos de menú, `ir.ui.menu`
+ `<Report>` es el modelo de acción del informe, `ir.actions.report.xml`
+ `<Plantilla>` es para las plantillas QWeb almacenadas en el modelo `ir.ui.view`
+ `<Url>` es el modelo de acción de URL, `ir.actions.act_url`

### Otras acciones en archivos de datos XML

Hasta ahora, hemos visto cómo agregar o actualizar datos usando archivos XML. Pero los archivos XML también le permiten realizar otros tipos de acciones que a veces son necesarias para configurar los datos. En particular, pueden eliminar datos, ejecutar métodos de modelo arbitrarios y activar eventos de flujo de trabajo.
 
#### Eliminando registros

Para eliminar un registro de datos, usamos el elemento `<delete>`, proporcionándolo con un ID o un dominio de búsqueda para encontrar el registro de destino. Por ejemplo, el uso de un dominio de búsqueda para encontrar el registro que desea eliminar se ve así:

```
<delete 
  model="ir.rule" 
  search="
  [('id','=',ref('todo_app.todo_task_user_rule'))]" 
/> 
```

Puesto que en este caso conocemos el ID específico a eliminar, podríamos haberlo utilizado directamente para el mismo efecto:

```
<delete model="ir.rule" id="todo_app.todo_task_user_rule" /> 
```

#### Funciones desencadenantes y flujos de trabajo

Un archivo XML también puede ejecutar métodos durante su proceso de carga a través del elemento `<function>`. Esto se puede utilizar para configurar datos de demostración y de prueba. Por ejemplo, la aplicación CRM lo utiliza para configurar datos de demostración:

```
<function  
   model="crm.lead"  
   name="action_set_lost" 
   eval="[ref('crm_case_7'), ref('crm_case_9') 
         , ref('crm_case_11'), ref('crm_case_12')] 
         , {'install_mode': True}" /> 

```

Esto llama al método `action_set_lost` del modelo `crm.lead`, pasando dos argumentos a través del atributo `eval`. El primero es la lista de IDs para trabajar, y el siguiente es el contexto a utilizar.

Otra forma en que los archivos de datos XML pueden realizar acciones es desencadenando flujos de trabajo Odoo a través del elemento `<workflow>`. Los flujos de trabajo pueden, por ejemplo, cambiar el estado de una orden de ventas o  convertirlo en una factura. La aplicación `sale` ya no utiliza flujos de trabajo, pero este ejemplo todavía se puede encontrar en los datos de demostración:
```
<workflow model="sale.order" 
  ref="sale_order_4" 
  action="order_confirm" /> 
```


El atributo `model` es autoexplicativo por ahora, y `ref` identifica la instancia de flujo de trabajo sobre la que estamos actuando. `action` es la señal de flujo de trabajo enviada a esta instancia de flujo de trabajo.

## Resumen

Haz aprendido todos los aspectos esenciales sobre la serialización de datos y adquirido una mejor comprensión de los aspectos XML que vimos en los capítulos anteriores. También pasamos un tiempo entendiendo los identificadores externos, un concepto central de manejo de datos en general y configuraciones de módulos en particular. Los archivos de datos XML se explicaron en detalle. Aprendiste acerca de las varias opciones disponibles para establecer valores en campos y también para realizar acciones, como eliminar registros y métodos de modelo de llamada. También se explican los archivos CSV y las funciones de importación / exportación de datos. Estas son herramientas valiosas para la configuración inicial de Odoo o para la edición masiva de datos.

En el próximo capítulo, exploraremos cómo construir modelos Odoo en detalle y aprenderemos más acerca de cómo construir sus interfaces de usuario.