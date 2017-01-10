.. _doc_gui_tutorial:

Tutorial GUI (Interfaz Grafica de Usuario)
============

Introduccion
~~~~~~~~~~~~

Si hay algo que la mayoria de los programadores realmente odian, es
programar interfaces graficas de usuario (GUIs). Es aburrido, tedioso
y no ofrece retos. Varios aspectos hacen este problema peor como:

-  La alineacion de los elementos de la UI (interfaz de usuario) es
   dificil (para que se vea justo como el diseñador queria).
-  Las UIs son cambiadas constantemente debido a los problemas de
   apariencia y usabilidad que aparecen durante el testing(prueba).
-  Manejar apropiadamente el cambio de tamaño de pantalla para
   diferentes resoluciones de pantallas.
-  Animar varios componentes de pantalla, para hacerlo parecer menos
   estatico.

La programacion GUI es una de las principales causas de frustracion
de los programadores. Durante el desarrollo de Godot (y previas
iteraciones del motor), varias tecnicas y filosoficas para el
desarrollo UI fueron puestas en practica, como un modo inmediato,
contenedores, anclas, scripting, etc. Esto fue siempre hecho con el
objetivo principal de reducir el estres que los programadores tienen
que enfrentar cuando crear interfaces de usuario

Al final, el sistema UI resultante en Godot es una eficiente solucion
para este problema, y funciona mezclando juntos algunos enfoques.
Mientras que la curva de aprendizaje es un poco mas pronunciada que
en otros conjuntos de herramientas, los desarrolladores pueden crear
interfaces de usuario complejas en muy poco tiempo, al compartir las
mismas herramientas con diseñadores y animadores.


Control
~~~~~~~

El nodo basico para elementos UI es :ref:`Control <class_Control>`
(a veces llamados "Widget" o "Caja" en otras herramientas). Cada
nodo que provee funcionalidad de interfaz de usuario desciende de
el.

Cuando los controles son puestos en el arbol de escena como hijos
de otro control, sus coordenadas (posicion, tamaño) son siempre
relativas a sus padres. Esto aporta la base para editar interfaces
de usuario complejas rapidamente y de manera visual.

Entrada y dibujado
~~~~~~~~~~~~~~~~~~

Los controles reciben eventos de entrada a traves de la llamada
de retorno :ref:`Control._input_event() <class_Control__input_event>`.
Solo un control, el que tiene el foco, va a recibir los eventos
de teclado/joypad (ve :ref:`Control.set_focus_mode() <class_Control_set_focus_mode>`
y  :ref:`Control.grab_focus() <class_Control_grab_focus>`.)

Los eventos de movimiento de mouse son recibidos por el control que
esta directamente debajo del puntero de mouse. Cuando un control
recibe el evento de que se presiono un boton de mouse, todas los
eventos siguientes de movimiento son recibidos por el control
presionado hasta que el boton se suelta, aun si el puntero se mueve
fuera de los limites del control.

Como cualquier clase que hereda de :ref:`CanvasItem <class_CanvasItem>`
(Control lo hace), una llamada de retorno :ref:`CanvasItem._draw() <class_CanvasItem__draw>`
sera recibida al principio y cada vez que el control deba ser
redibujado (los programadores deben llamar :ref:`CanvasItem.update() <class_CanvasItem_update>`
para poner en cola el CanvasItem para redibujar). Si el control no
esta visible (otra propiedad CanvasItem), el control no recibe
ninguna entrada.

En general sin embargo, el programador no necesita lidiar con el
dibujado y los eventos de entrada directamente cuando se construyen
UIs, (es mas util cuando se crean controles personalizados). En su
lugar, los controles emiten diferente tipos de señales con informacion
contextual para cuando la accion ocurre. Por ejemplo, una :ref:`Button <class_Button>`
emite una señal "pressed", una :ref:`Slider <class_Slider>`emitiraun
"value_changed" cuando se arrastra, etc.

Mini tutorial de controles personalizados
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Antes de ir mas profundo, crear un control personalizado sera una
buena forma de entender como funcionan los controles, ya que no son
tan complejos como pueden parecer.

Adicionalmente, aunque Godot viene con docenas de controles para
diferentes propositos, sucede a menudo que es simplemente mas
sencillo obtener la funcionalidad específica creando uno nuevo.

Para comenzar, crea una escena con un solo nodo. El nodo es del tipo
"Control" y tiene cierta area de la pantalla en el editor 2D, como
esto:

.. image:: /img/singlecontrol.png

Agregale un script a ese nodo, con el siguiente codigo:

::

    extends Control

    var pulsado=false

    func _draw():

        var r = Rect2( Vector2(), get_size() )
        if (pulsado):
            draw_rect(r, Color(1,0,0) )
        else:
            draw_rect(r, Color(0,0,1) )

    func _input_event(ev):

        if (ev.type==InputEvent.MOUSE_BUTTON and ev.pressed):
            pulsado=true
            update()

Luego corre la escena. Cuando el rectangulo es clickeado/pulsado, ira
de azul a rojo. Esa sinergia entre los eventos y el dibujo es
basicamente como funcionan internamente la mayoria de los controles.

.. image:: /img/ctrl_normal.png

.. image:: /img/ctrl_tapped.png

Complejidad de la UI
~~~~~~~~~~~~~

Como mencionamos antes, Godot incluye docenas de controles listos para
usarse en una interface. Esos controles estan divididos en dos
categorias. La primera es un pequeño grupo de controles que funcionan
bien para crear la mayoria de las interfaces de usuario. La segunda
(y la mayoria de los controles son de este tipo) estan destinadas a
interfases de usuario complejas y el skinning(aplicar un forro) a
traves de estilos. Una descripcion es presentada a continuacion para
ayudar a entender cual debe ser usada en que caso.

Controles UI simplificados
~~~~~~~~~~~~~~~~~~~~~~~~~~

Este conjunto de controles es suficiente para la mayoria de los
juegos, donde interacciones complejas o formas de presentar la
informacion no son necesarios. Pueden ser "skineados" facilmente
con texturas regulares.

-  :ref:`Label <class_Label>`: Nodo usado para mostrar texto
-  :ref:`TextureFrame <class_TextureFrame>`: Muestra una sola
   textura, que puede ser escalada o mantenia fija.
-  :ref:`TextureButton <class_TextureButton>`: Muestra un
   simple boton con textura para los estados como pressed, hover,
   disabled, etc.
-  :ref:`TextureProgress <class_TextureProgress>`: Muestra una
   sola barra de progreso con textura.

Adicionalmente, el reposicionado de controles es mas eficientemente
hecho con anclas en este caso (ve el tutorial :ref:`doc_size_and_anchors`
para mas informacion)

De cualquier forma, sucedera seguido que aun para juegos simples,
comportamientos de UI mas complejos son requeridos. Un ejemplo de
esto es una lista de elementos con scrolling (desplazamiento) (por ejemplo
para una tabla de puntuaciones altas), la cual necesita un
:ref:`ScrollContainer <class_ScrollContainer>` y un :ref:`VBoxContainer <class_VBoxContainer>`.
Este tipo de controles mas avanzados puede ser mezclado con los
regulares sin problema (son todos controles de todas formas).

Controles de UI complejos
~~~~~~~~~~~~~~~~~~~~~~~~~

El resto de los controles (y hay docenas de ellos!) estan destinados
para otro tipo de escenario, los mas comunes:

-  Juegos que requieren UIs complejas, como RPGs (juegos de rol),
   MMOs (juegos online masivos), strategy (estrategia), sims
   (simulacion), etc.
-  Crear herramientas de desarrollo personalizadas para acelerar
   la creacion de contenido.
-  Crear Plugins de Editor de Godot, para extender la funcionalidad
   del motor.

Reposicionar controles para este tipo de interfaces es mas comunmente
hecho con contenedores (ve el tutorial :ref:`doc_size_and_anchors` para
mas informacion).
