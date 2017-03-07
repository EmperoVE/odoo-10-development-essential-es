#Capítulo 2. Creación de su primera aplicación Odoo
Desarrollar en Odoo la mayor parte del tiempo significa crear nuestros propios módulos. En este capítulo, crearemos nuestra primera aplicación Odoo y aprenderemos los pasos necesarios para ponerla a disposición de Odoo e instalarla.

Inspirado por el notable proyecto http://todomvc.com/, vamos a construir una simple aplicación de tareas pendientes. Debería permitirnos agregar nuevas tareas, marcarlas como completadas y, finalmente, borrar la lista de tareas de todas las tareas ya completadas.

Comenzaremos aprendiendo los conceptos básicos del desarrollo de un flujo de trabajo: configura una nueva instancia para tu trabajo, crea e instalA un nuevo módulo y actualízalo para aplicar los cambios que realices junto con las iteraciones de desarrollo.

Odoo sigue una arquitectura similar a MVC, y pasaremos por las capas durante nuestra implementación de la aplicación de tareas pendientes:

+ La capa del **modelo**, que define la estructura de los datos de la aplicación
+ La capa de ***vista**, que describe la interfaz de usuario
+ La capa del **controlador**, que soporta la lógica de negocio de la aplicación

A continuación, aprenderemos cómo configurar la seguridad de control de acceso y, finalmente, agregaremos información sobre la descripción y la marca al módulo.

####Nota
Ten en cuenta que el concepto del término controlador mencionado aquí es diferente de los controladores de desarrollo web Odoo. Estos son puntos finales del programa que las páginas web pueden llamar para realizar acciones.

Con este enfoque, podrás aprender gradualmente sobre los bloques básicos de construcción que conforman una aplicación y experimentar el proceso iterativo de construir un módulo Odoo desde cero.

##Conceptos esenciales
Es probable que estés empezando con Odoo, así que ahora es obviamente un buen momento para explicar los módulos de Odoo y cómo se utilizan en un desarrollo Odoo.

###Descripción de aplicaciones y módulos
Es común oír hablar de los módulos y aplicaciones Odoo. Pero, ¿cuál es exactamente la diferencia entre ellos?

Los **Complementos de Módulos** son los componentes básicos para las aplicaciones Odoo. Un módulo puede agregar nuevas características a Odoo, o modificar las existentes. Es un directorio que contiene un manifiesto, o archivo descriptor, llamado `__manifest__.py`, más los archivos restantes que implementan sus características.

Las **Aplicaciones** son la forma en que se añaden las principales características a Odoo. Proporcionan los elementos básicos para un área funcional, como Contabilidad o RH, en función de qué características de módulos complementarios modifican o amplían. Debido a esto, se destacan en el menú **Apps** de Odoo.

Si su módulo es complejo y agrega funcionalidad nueva o mayor a Odoo, podrías considerar crearlo como una aplicación. Si tu módulo sólo hace cambios a la funcionalidad existente en Odoo, es probable que no sea una aplicación.

Si un módulo es una aplicación o no, se define en el manifiesto. Técnicamente no tiene ningún efecto particular sobre cómo se comporta el módulo addon. Sólo se utiliza para resaltar en la lista de **Aplicaciones**.

###Modificando y extendiendo módulos
En el ejemplo que vamos a seguir, crearemos un nuevo módulo con el menor número posible de dependencias.

Sin embargo, este no será el caso típico. Principalmente, modificaremos o extenderemos un módulo ya existente.

Como regla general, se considera una mala práctica modificar los módulos existentes al cambiar su código fuente directamente. Esto es especialmente cierto para los módulos oficiales proporcionados por Odoo. Hacerlo no te permite tener una separación clara entre el
