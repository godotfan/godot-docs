.. _doc_scripting:

Scripting
=========

Introducción
------------

Mucho se ha dicho sobre herramientas que permiten a los usuarios crear
juegos sin programar. Ha sido un sueño para muchos desarrolladores
independientes el crear juegos sin aprender a escribir código. Esto ha
sido así por un largo tiempo, aun dentro de compañías, donde los
desarrolladores de juegos desean tener mas control del flujo del juego
(game flow).

Muchos productos han sido presentados prometiendo un entorno sin
programación, pero el resultado es generalmente incompleto, demasiado
complejo o ineficiente comparado con el código tradicional. Como
resultado, la programación esta aquí para quedarse por un largo tiempo.
De hecho, la dirección general en los motores de juegos ha sido agregar
herramientas que reducen la cantidad de código que necesita ser escrito
para tareas especificas, para acelerar el desarrollo.

En ese sentido, Godot ha tomado algunas decisiones de diseño útiles con
ese objetivo. La primera y mas importante es el sistema de escenas. El
objetivo del mismo no es obvio al principio, pero trabaja bien mas
tarde. Esto es, descargar a los programadores de la responsabilidad de
la arquitectura del código.

Cuando se diseñan juegos usando el sistema de escenas, el proyecto
entero esta fragmentado en escenas complementarias (no individuales).
Las escenas se complementar entre si, en lugar de estar separadas.
Tendremos un montón de ejemplos sobre esto mas tarde, pero es muy
importante recordarlo.

Para aquellos con una buena base de programación, esto significa que
un patrón de diseño diferente a MVC(modelo-vista-controlador). Godot
promete eficiencia al costo de dejar los hábitos MVC, los cuales se
reemplazan por el patrón *escenas como complementos*.

Godot también utiliza el <http://c2.com/cgi/wiki?EmbedVsExtend>`__
patrones para scripting, por lo que los scripts se extienden desde
todas las clases disponibles.

GDScript
--------

:ref:`doc_gdscript` es un lenguaje de scripting de tipado dinámico
hecho a medida de Godot. Fue diseñado con los siguientes objetivos:

-  El primero y mas importante, hacerlo simple, familiar y fácil,
   tan fácil  de aprender como sea posible.
-  Hacer el código legible y libre de errores. La sintaxis es
   principalmente extraída de Python.

A los programadores generalmente les toma unos días aprenderlo, y
entre las primeras dos semanas para sentirse cómodos con el.

Como con la mayoría de los lenguajes de tipado dinámico, la mayor
productividad (el código es mas fácil de aprender, mas rápido de
escribir, no hay compilación, etc) es balanceada con una pena de
rendimiento, pero el código mas critico esta escrito en C++ en primer
lugar dentro del motor (vector ops, physics, match, indexing, etc),
haciendo que la rendimiento resultante sea mas que suficiente para
la mayoría de los juegos.

En cualquier caso, si se requiere rendimiento, secciones criticas
pueden ser reescritas en C++ y expuestas transparentemente al script.
Esto permite reemplazar una clase GDScript con una clase C++ sin
alterar el resto del juego.

Scripting de una Escena
-----------------------

Antes de continuar, por favor asegúrate de leer la referencia :ref:`doc_gdscript`
Es un lenguaje simple y la referencia es corta, no debería llevar mas
que algunos minutos darle un vistazo.

Configuración de la Escena
~~~~~~~~~~~~~~~~~~~~~~~~~~

Este tutorial comenzara programando una simple escena. Usa el botón de
agregar nodo (+) para crear la siguiente jerarquía, con los siguientes
nodos:

- Panel

  * Label
  * Button

Debería verse así en el árbol de la escena:

.. image:: /img/scriptscene.png

Y trata de que quede así en el editor 2D, para que tenga sentido:

.. image:: /img/scriptsceneimg.png

Finalmente, guarda la escena, un nombre acorde podría ser "dihola.scn"

.. _doc_scripting-adding_a_script:

Agregando un script
~~~~~~~~~~~~~~~~~~~

Selecciona el nodo del Panel, y presiona click derecho en el mouse,
luego selecciona Agregar Script:

.. image:: /img/addscript.png

El dialogo de creación de script aparecerá. Este dialogo permite
seleccionar el lenguaje, nombre de clase, etc. GDScript no usa nombres
de clase en los archivos de script, por lo que este campo no es
editable. El script debería heredar de "Panel" (ya que su función es
extender el nodo, que es de tipo Panel, esto se llena automáticamente
de todas formas).

Selecciona el nombre de archivo para el script (si ya salvaste la
escena previamente, uno se generara automáticamente como dihola.gd)
y presiona "Crear":

.. image:: /img/scriptcreate.png

Una vez hecho, el script se creara y se agregara al nodo. Puedes verlo
tanto como el icono en el nodo, como en la propiedad script:

.. image:: /img/scriptadded.png

Para editar el script, presionar arriba del icono debería hacerlo (
aunque, la UI(interfaz de usuario) te llevara directamente a la
ventana de edicion de Script). Asique, aquí esta la plantilla del
script:

.. image:: /img/script_template.png

No hay mucho allí. La función "_ready()" es llamada cuando el nodo (y
todos sus hijos) entran en la escena activa. (Recuerda, no es un
constructor, el constructor es "_init()" ).

El rol del script
~~~~~~~~~~~~~~~~~

Un script básicamente agrega un comportamiento al nodo. Es usado para
controlar las funciones del nodo así como otros nodos (hijos, padres,
primos, etc). El alcance local del script es el nodo (como en cualquier
herencia) y las funciones virtuales del nodo son capturadas por el
script.

.. image:: /img/brainslug.jpg

Manipulando una señal
~~~~~~~~~~~~~~~~~~~~~

Las señales son usadas principalmente en los nodos GUI(interfaz grafica
de usuario) (aunque otros nodos también las tienen). Las señales se
emiten cuando una acción especifica sucede, y pueden estar conectadas
a cualquier otra función en cualquier de cualquier instancia de script.
En este paso, la señal "pressed" del botón será conectada a una función
personalizada.

En la pestaña "Nodo" puedes ver las señales disponibles para el nodo
seleccionado:

.. image:: /img/button_connections.png

Pero este ejemplo no lo usara. No queremos hacer las cosas *demasiado*
fáciles. Asique por favor, cierra esa pantalla!

En cualquier caso, a esta altura es claro que estamos interesados en
la señal "pressed"(presionado), asique en lugar de hacerlo con la
interfaz visual, la conexión será hecha por código.

Para esto, existe una función que es probablemente la que los
programadores de Godot usaran mas, esta es :ref:`Node.get_node() <class_Node_get_node>`.
Esta función usa caminos para traer nodos en el árbol actual o en
cualquier parte de la escena, relativa al nodo que posee el script.

Para traer el botón, lo siguiente debe ser utilizado:

::

    get_node("Button")

Entonces, a continuación, un callback(llamada de retorno) será
agregado cuando el botón sea presionado, que cambiara el texto de la
etiqueta:

::

    func _on_button_pressed():
        get_node("Label").set_text("HELLO!")

Finalmente, la señal "pressed" sera conectada al callback en _ready(),
usando :ref:`Object.connect() <class_Object_connect>`.

::

    func _ready():
        get_node("Button").connect("pressed",self,"_on_button_pressed")

El script final debería verse así:

::

    extends Panel

    # member variables here, example:

    # var a=2
    # var b="textvar"

    func _on_button_pressed():
        get_node("Label").set_text("HOLA!")

    func _ready():
        get_node("Button").connect("pressed",self,"_on_button_pressed")

Correr la escena debería tener el resultado esperado cuando se presiona
el botón:

.. image:: /img/scripthello.png

**Nota:** Ya que es un error común en este tutorial, clarifiquemos
nuevamente que get_node(camino) funciona regresando el hijo *inmediato*
del nodo que es controlado por el script (en este caso, *Panel*), por
lo que *Button* debe ser un hijo de *Panel* para que el código anterior
funcione. Para darle mas contexto a esta aclaración, si *Button*
fuese hijo de *Label*, el código para obtenerlo sería:

::

    # not for this case
    # but just in case
    get_node("Label/Button")

Y, también, trata de recordar que los nodos son referenciados por
nombre, no por tipo.
