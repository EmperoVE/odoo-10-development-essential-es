#Capítulo 4. Datos del módulo

La mayoría de las definiciones de módulos Odoo, como las interfaces de usuario y las reglas de seguridad, son en realidad registros de datos almacenados en tablas de bases de datos específicas. Los archivos XML y CSV que se encuentran en los módulos no son utilizados por las aplicaciones Odoo en tiempo de ejecución. En su lugar, son un medio para cargar esas definiciones en las tablas de la base de datos.

Debido a esto, una parte importante de los módulos de Odoo consiste en representar (serializando) esos datos utilizando archivos de modo que puedan cargarse posteriormente en una base de datos.

Los módulos también pueden tener datos predeterminados y de demostración. La representación de datos permite añadir eso a nuestros módulos. Además, la comprensión de los formatos de representación de datos de Odoo es importante para exportar e importar datos empresariales en el contexto de la implementación de un proyecto.

Antes de entrar en casos prácticos, primero exploraremos el concepto de identificador externo, que es la clave para la representación de datos Odoo.
##Comprendiendo los identificadores externos

Un **identificador externo** (también llamado XML ID) es un identificador de cadena legible por humanos que identifica de forma única un registro particular en Odoo. Son importantes al cargar datos en Odoo.

Una de las razones es que al actualizar un módulo, sus archivos de datos se cargarán nuevamente en la base de datos y debemos detectar los registros ya existentes para actualizarlos en lugar de crear nuevos registros duplicados.

Otra razón que apoya los datos interrelacionados: los registros de datos deben ser capaces de hacer referencia a otros registros de datos. El identificador de base de datos real es un número secuencial asignado automáticamente por la base de datos, durante la instalación del módulo. Los identificadores externos proporcionan una forma de hacer referencia a un registro relacionado sin necesidad de saber de antemano qué ID de base de datos se asignará, lo que nos permite definir las relaciones de datos en los archivos de datos Odoo.

Odoo se encarga de traducir los nombres de identificadores externos en los IDs de base de datos reales asignados a ellos. El mecanismo detrás de esto es bastante simple: Odoo mantiene una tabla con el mapeo entre los identificadores externos nombrados y sus IDs de base de datos numéricos correspondientes: el modelo `ir.model.data`.

Para inspeccionar los mapeos existentes, ve a la sección **Technical** del menú superior **Settings** y selecciona el elemento de menú **External Identifiers** bajo **Sequences & Identifiers**.

Por ejemplo, si visitamos la lista de **External identifiers** y lo filtramos por el módulo `todo_app`, veremos los identificadores externos generados por el módulo creado anteriormente:
AQUI VA UNA IMAGEN