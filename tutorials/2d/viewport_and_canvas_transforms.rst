.. _doc_viewport_and_canvas_transforms:

Viewport y transformaciones canvas
==================================

Introducción
------------

Este tutorial esta creado con un tema que es algo confuso para la mayoría
de los usuarios, explica todas las transformaciones 2D que suceden en
los nodos desde el momento que dibujan su contenido localmente hasta el
momento que son dibujados en la pantalla.

Transformaciones Canvas
-----------------------

Como mencionamos en el tutorial previo, :ref:`doc_canvas_layers`, todos
los nodos CanvasItem (recuerda que los nodos basados en Node2D y Control
usan CanvasItem como su raíz común) van a residir en una *Capa Canvas*.
Toda capa canvas tiene transformación (traslación, rotación, escala,
etc.) que puede ser accedida como una :ref:`Matrix32 <class_Matrix32>`.

Como fue explicado en el tutorial previo, los nodos son dibujados por
defecto en el Layer 0, en el canvas incorporado. Para poner nodos en
diferentes capas, puedes usar un nodo :ref:`CanvasLayer <class_CanvasLayer>.


Transformaciones globales de canvas
-----------------------------------

Los Viewports también tienen una transformación Global Canvas (que
también es una :ref:`Matrix32 <class_Matrix32>`). Esta es la
transformación maestra y afecta a todas las transformaciones individuales
*Canvas Layer*. Generalmente esta transformación no es de mucha utilidad,
pero es usada en el CanvasItem Editor en el editor Godot.

Transformación de estiramiento
------------------------------

Finalmente, los viewports tienen *Stretch Transform*, la cual es
usado cuando se cambia de tamaño o se estira la pantalla. Esta
transformación es usada internamente (como se describe en
:ref:`doc_multiple_resolutions`), pero puede también ser ajustada
manualmente en cada viewport.

Los eventos de entrada recibidos en la llamada de retorno
:ref:`MainLoop._input_event() <class_MainLoop__input_event>`
son multiplicados por esta transformación, pero carece de las
superiores. Para convertir coordenadas InputEvent a coordenadas
locales CanvasItem, la funcion :ref:`CanvasItem.make_input_local() <class_CanvasItem_make_input_local>`
fue agregada por comodidad.

Orden de transformación
-----------------------

Para que una coordenada en las propiedades locales CanvasItem se vuelva
una coordenada de pantalla, deben aplicarse las siguientes cadenas de transformaciones:

.. image:: /img/viewport_transforms2.png

Funciones de transformación
---------------------------

Obtener cada transformación puede ser logrado con las siguientes
funciones:

+----------------------------------+--------------------------------------------------------------------------------------+
| Tipo                             | Transformacion                                                                       |
+==================================+======================================================================================+
| CanvasItem                       | :ref:`CanvasItem.get_global_transform() <class_CanvasItem_get_global_transform>`     |
+----------------------------------+--------------------------------------------------------------------------------------+
| CanvasLayer                      | :ref:`CanvasItem.get_canvas_transform() <class_CanvasItem_get_canvas_transform>`     |
+----------------------------------+--------------------------------------------------------------------------------------+
| CanvasLayer+GlobalCanvas+Stretch | :ref:`CanvasItem.get_viewport_transform() <class_CanvasItem_get_viewport_transform>` |
+----------------------------------+--------------------------------------------------------------------------------------+

Finalmente pues, para convertir coordenadas CanvasItem locales a
coordenadas de pantalla, solo multiplica en el siguiente orden:

::

    var coord_pant = get_viewport_transform() + ( get_global_transform() + pos_local )

Ten en mente, sin embargo, que en general no es deseable trabajar con
coordenadas de pantalla. El enfoque recomendado es simplemente trabajar
con coordenadas Canvas (``CanvasItem.get_global_transform()``), para
permitir que el cambio de tamaño automático de resolución de pantalla
funcione adecuadamente.


Alimentando eventos de entrada personalizados
---------------------------------------------

Es a menudo deseable alimentar eventos de entrada personalizados al
árbol de escena. Con el conocimiento anterior, para lograr esto
correctamente, debe ser hecho de la siguiente manera:


::

    var pos_local = Vector2(10,20) # local a Control/Node2D
    var ie = InputEvent()
    ie.type = InputEvent.MOUSE_BUTTON
    ie.button_index = BUTTON_LEFT
    ie.pos = get_viewport_transform() + (get_global_transform() + pos_local)
    get_tree().input_event(ie)
