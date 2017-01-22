.. _doc_scripting_continued:

Scripting (continuación)
=====================

Procesando
----------

Varias acciones en godot son disparadas por callbacks o funciones
virtuales, por lo que no hay necesidad de escribir código de chequeo
que corre todo el tiempo. Además, mucho puede ser hecho con animation
players (reproductores de animación).

Sin embargo, es aun un caso muy común tener un script procesando en
cada frame. Hay dos tipos de procesamiento, procesamiento idle(inactivo)
y procesamiento fixed(fijo).

El procesamiento Idle es activado con la funcion :ref:`Node.set_process() <class_Node_set_process>`
Una vez activado, el callback :ref:`Node._process() <class_Node__process>`
podrá ser llamado en cada frame(cuadro). Ejemplo:

::

    func _ready():
        set_process(true)

    func _process(delta):
        # hacer algo...

El parámetro delta describe el tiempo que paso (en segundos, como
numero de punto flotante) desde la llamada previa a la funcion
_process(). El procesamiento fijo es similar, pero solo se necesita
para sincronización con el motor de física.

Una forma simple de probar esto es crear una escena con un solo nodo
Label, con el siguiente script:

::

    extends Label

    var accum=0

    func _ready():
        set_process(true)

    func _process(delta):
        accum += delta
        set_text(str(accum))

Lo que mostrara un contador aumentando cada segundo.

Grupos
------

Los nodos pueden ser agregados a grupos (tantos como se desee por
nodo). Esta es una característica simple pero efectiva para
organizar escenas grandes. Hay dos formas de hacer esto, la primera
es por la UI, con el botón Grupos en la pestaña Nodo.

.. image:: /img/groups.png

Y la segunda desde el código. Un ejemplo útil podría ser, por ejemplo,
marcar escenas que son enemigos.

::

    func _ready():
        add_to_group("enemigos")

De esta forma, si el jugador, entrando sigilosamente a la base secreta,
es descubierto, todos los enemigos pueden ser notificados sobre la
alarma activada, usando :ref:`SceneTree.call_group() <class_SceneTree_call_group>`:

::

    func _on_discovered():
        get_tree().call_group(0, "guardias", "jugador_fue_descubierto")

El código superior llama la función "jugador_fue_descubierto" en cada
miembro del grupo "guardias".

Opcionalmente, es posible obtener la lista completa de nodos "guardias"
llamando a :ref:`SceneTree.get_nodes_in_group() <class_SceneTree_get_nodes_in_group>`:

::

    var guardias = get_tree().get_nodes_in_group("guardias")

Luego agregaremos mas sobre :ref:`SceneTree <class_SceneTree>`

Notificaciones
-------------

Godot utiliza un sistema de notificaciones. Usualmente no son
necesarias desde scripts, debido a que es demasiado bajo nivel y
las funciones virtuales están disponibles para la mayoría de ellas.
Es solo que es bueno saber que existen. Simplemente agrega una
funcion :ref:`Object._notification() <class_Object__notification>`
en tu script:

::

    func notificacion (what):
        if (what == NOTIFICATION_READY):
            print("Esto es lo mismo que sobrescribir _ready()...")
        elif (what == NOTIFICATION_PROCESS):
            var delta = get_process_time()
            print("Esto es lo mismo que sobrescribir _process()...")

La documentación de cada clase en :ref:`Class Reference <toc-class-ref>`
muestra las notificaciones que puede recibir. Sin embargo, nuevamente,
para la mayoría de los casos los scripts proveen funciones mas simples
Sobreescribibles.

Funciones Sobreescribibles
--------------------------

Como mencionamos antes, es mejor usar estas funciones. Los nodos
proveen muchas funciones sobreescribibles útiles, las cuales se
describen a continuación:

::

    func _enter_tree():
        # Cuando el nodo entre en la _Scene Tree_. se vuelve activa
        # y esta función se llama. Los nodos hijos aun no entraron
        # en la escena activa. En general, es mejor usar _ready()
        # para la mayoría de los casos.
        pass

    func _ready():
        # Esta función es llamada luego de _enter_tree, pero se
        # asegura que todos los nodos hijos también hayan entrado
        # a _Scene Tree_, y se han vuelto activos.
        pass

    func _exit_tree():
        # Cuando el nodo sale de _Scene Tree_. esta funcion es
        # llamada. Los nodos hijos han salido todos de _Scene Tree_
        # en este punto y todos están activos.
        pass

    func _process(delta):
        # Cuando set_process() esta habilitado, esta función es
        # llamada en cada frame.
        pass

    func _fixed_process(delta):
        # Cuando set_fixed_process() esta habilitado, esto es
        # llamado en cada frame de física.
        pass

    func _paused():
        # Se llama cuando el juego esta en pausa, Luego de esta
        # llamada, el nodo no recibirá mas callbacks de proceso.
        pass

    func _unpaused():
        # Llamada cuando el juego se reanuda.
        pass

Creando nodos
-------------

Para crear nodos desde código, solo llama el método .new(), (al igual
que para cualquier otra clase basada en tipo de dato). Ejemplo:

::

    var s
    func _ready():
        s = Sprite.new() # crear un nuevo sprite!
        add_child(s) # lo agrega como hijo de este nodo

Para borrar el nodo, sea dentro o fuera de la escena, free() debe
ser usado:

::

    func _someaction():
        s.free() # inmediatamente remueve el nodo de la escena y
                 # lo libera

Cuando un nodo es liberado, también son liberados todos los nodos hijos.
Por este motivo, borrar nodos manualmente es mucho mas simple de lo que
parece. Solo libera el nodo base y todo lo demás en el sub árbol se
irá con el.

Sin embargo, puede suceder muy seguido que queramos borrar un nodo que
esta actualmente "blocked"(bloqueado), esto significa, el nodo esta
emitiendo una señal o llamado a función. Esto resultara en que el juego
se cuelgue. Correr Godot en el debugger (depurador) a menudo va a
capturar este caso y advertirte sobre el.

La forma mas segura de borrar un nodo es usando
:ref:`Node.queue_free() <class_Node_queue_free>`
en su lugar. Esto borrara el nodo mientras esta inactivo, de forma
segura.

::

    func _someaction():
        s.queue_free() # remueve el nodo y lo borra mientras nada esta
        sucediendo.

Instanciando escenas
--------------------

Instancias una escena desde código es bastante fácil y se hace en dos
pasos. El primero es cargar la escena desde el disco.

::

    var scene = load("res://myscene.scn") # cargara cuando el script es
    instanciado

Precargar es mas conveniente a veces, ya que sucede en tiempo de
parse (análisis gramatical).

::

    var scene = preload("res://myscene.scn") # será cargado cuando el
                                             # script es "parseado"

Pero 'escena' todavía no es un nodo que contiene sub nodos. Esta
empaquetado en un recurso especial llamado :ref:`PackedScene <class_PackedScene>`.
Para crear el nodo en si, la función
:ref:`PackedScene.instance() <class_PackedScene_instance>`
debe ser llamada. Esta regresara el árbol de nodos que puede ser
agregado a la escena activa:

::

    var node = scene.instance()
    add_child(node)

La ventaja de este proceso en dos pasos es que una escena empaquetada
puede mantenerse cargada y listo para usar, por lo que puede ser usada
para crear tantas instancias como se quiera. Esto es especialmente
útil, por ejemplo, para instanciar varios enemigos, armas, etc. de
forma rápida en la escena activa.
