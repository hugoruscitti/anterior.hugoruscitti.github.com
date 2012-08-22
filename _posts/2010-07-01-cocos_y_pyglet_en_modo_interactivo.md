---
layout: post
title: "Cocos y pyglet en modo interactivo"
description: "Cómo añadir soporte a cocos para se pueda usar interactivamente."
category: 
tags: [tip]
published: true
---


## Introducción

Python tiene una herramienta super interesante para los nuevos programadores: una consola interactiva donde se puede editar, corregir y ejecutar código de manera super sencilla.

Esto resulta de mucha utilidad cuando quieres dar clases sobre programación o simplemente dar una demostración sobre alguna biblioteca.

Lamentablemente la consola de python no se puede usar en cualquier escenario, por ejemplo si quieres dar una demostración de la biblioteca [cocos2d] desde la consola interactiva se te pueden presentar algunos problemas.

Mediante este artículo intentaré encontrarle una solución a este problema, y en el camino veremos
Una aclaración inicial

Por cierto, [cocos2d] viene con una consola interactiva que se activa con las teclas ``<CTRL+I>``.

Esa consola se ve dentro de la misma ventana de la aplicación y te permite hacer casi todo lo que veremos aquí.

La diferencia entre esa consola, y la que vamos a tratar de implementar aquí es la siguiente: en este artículo busco intergrar la consola de python tradicional, donde se puede copiar, pegar y autocompletar código.

## Un prototipo de ejemplo para descargar

La solución que veremos en este artículo me ha servido para realizar un prototipo de ejemplo.

Si quieres ir viendo el programa en funcionamiento lo puedes descargar de la siguiente dirección:

- http://examplelab.googlecode.com/hg/interactive_cocos/interactive_cocos.py

donde la imagen que utilizo para representar el objeto animado principal está aquí:

![](/images/cocos_y_pyglet_en_modo_interactivo/cara.png)


## ¿Cómo se utiliza el prototipo de consola?

Primero tienes que descargar el script de la aplicación y ejecutarlo con el siguiente comando:

    python interactive_cocos2d.py


Luego, se abrirá una pantalla con un sprite que podrás manipular:

![](/images/cocos_y_pyglet_en_modo_interactivo/screen_cara.png)

Por ejemplo, dentro de la consola python tiene dos referencias iniciales: cocos y sprite. Hagamos algo con ellas:

    sprite.rotate = 40

obteniendo:

![](/images/cocos_y_pyglet_en_modo_interactivo/rotate.png)

Esto es útil para mostrar que el objeto en cuestión tiene atributos, y que se pueden manipular de manera sencilla.

También puedes ejecutar sentencias como ``help(cocos)`` o similares.

Ahora bien, con este escenario resulta mas interesante hablar sobre una de las características mas lindas que tiene [cocos2d], el submódulo actions:

    media_vuelta = cocos.actions.RotateBy(180, 2)
    giro_completo = media_vuelta * 2

    sprite.do(giro_completo)

Eso mostrará una animación de rotación completa en 4 segundos. Y como ninguna acción detiene nuesta consola de python, podemos incluso seguir escribiendo mienstras se ejecuta la acción.

## ¿Cómo funciona?

A continuación veremos paso a paso la implementación del prototipo que se utiliza mas arriba.

## Objetivo

Quisieramos ejecutar nuestro intérprete de python tradicional en primer plano, y que la ventana de la biblioteca [cocos2d] permanezca en segundo plano.

Esto nos facilitaría jugar con la biblioteca, conocer sus funciones y manipular sprites de manera interactiva.
Un primer paso: ¿usamos hilos?

Como queremos poner en funcionamiento una ventana de [cocos2d] y al mismo tiempo tener un intérprete, podríamos usar dos hilos: Uno para el interprete que recibe los comandos del usuario y otro para la biblioteca [cocos2d].

## El problema: Hilos y OpenGL

Lo malo de este enfoque es que opengl (la biblioteca base debajo de [cocos2d]) define un contexto y espera que todas las llamadas de opengl se hagan desde el mismo hilo que ha creado el contexto. Y esto es un problema, porque en realidad para nuestro escenario lo ideal sería manipular opengl desde un hilo (la consola) y ver los cambios en otro hilo (el de cocos).

Entonces, al parecer tenemos como restricción que todo lo que actúa directamente sobre opengl tiene que ejecutarse dentro del mismo hilo que utiliza cocos.

## El modelo propuesto

Podemos tener dos hilos en ejecución, y una cola de mensajes que permite derivar todo el código a ejecutar dentro del hilo de cocos.

Este es un gráfico de la solución propuesta:

![](/images/cocos_y_pyglet_en_modo_interactivo/esquema.png)

## Creando un prototipo

El programa principal solamente tiene que poner en funcionamiento a los tres componentes juntos:

    # Cola de mensajes que se utiliza para llevar comandos al hilo.
    queue = Queue.Queue()

    # Hilo que ejecuta la funcionalidad de cocos.
    app = Application(queue)
    app.start()

    # Interprete de comandos que envia todas las lineas que se escriben
    # directamente a la cola de mensajes que consume el hilo.
    cmd = CommandLine(app, queue)
    cmd.cmdloop()

Donde queue es la cola que se utiliza para enviar o delegar todas las cadena de texto que ingresa el usuario.

## El intérprete

Para crear el intérprete que acepta los comandos del usuario he utilizado la clase Cmd del módulo cmd:

    class CommandLine(cmd.Cmd):

        def __init__(self, app, queue):
            cmd.Cmd.__init__(self, 'TAB')
            self.queue = queue
            self.app = app
            self.prompt = ">>> "

        def default(self, line):
            "Cualquier sentencia la envia a la cola de comandos que consume cocos."
            self.queue.put(line)

        def do_exit(self, line):
            self.app.kill()
            sys.exit(0)

La clase Cmd permite crear un intérprete de comandos parecido al original de python, incluido el autocompletado con la tecla TAB.

## Hilo de cocos

El hilo que muestra la ventana de [cocos2d] solo tiene que mantener una ventana de la biblioteca visible y actualizar con frecuencia el contexto que ejecuta comandos:

    class Application(threading.Thread):

        def __init__(self, queue):
            threading.Thread.__init__(self)
            self.killed = False
            self.queue = queue

        def run(self):
            cocos.director.director.init(resizable=False, width=800, height=600)
            self.scene = cocos.scene.Scene(cocos.layer.ColorLayer(240, 240, 240, 255))
            sprite = cocos.sprite.Sprite('cara.png')
            sprite.scale = 2
            sprite.x = 400
            sprite.y = 300
            self.scene.add(sprite)

            # Genera el entorno de la session, y le pasa datos que puede
            # manipular.
            self.environment = environment(queue, sprite)
            self.environment.next()

            self.scene.schedule_interval(self.environment_update, 0.01)
            cocos.director.director.run(self.scene)

        def environment_update(self, dt):
            "Mantiene en funcionamiento."
            self.environment.next()

        def kill(self):
            self.killed = True
            pyglet.app.exit()


## Un entorno donde viven las referencias

La parte mas importante del sistema es la que se encarga de ejecutar el código que escribimos en consola.

Para ello he creado un generador, que conoce la cola de donde lee los mensajes que escribimos y los pone en funcionamiento.

Y dado que se trata de un generador, todo lo que se ejecuta dentro de él se mantiene en la memoria local, representando de alguna manera el contexto o espacio de nombres que buscamos conservar.

    def environment(queue, sprite):
        """Representa la session y las variables locales de la instancia interactiva.

        Este entorno funciona como un generador, con la intension de guardar
        el estado de todas las variables locales y acceder a los comandos desde
        la cola."""

        while True:

            try:
                last = queue.get_nowait()

                if last:
                    try:
                        exec(last)
                    except:
                        print "Error al ejecutar", last

            except Queue.Empty:
                pass

            yield None

Nota: Un enfoque similar usan las aplicaciones REPL, donde incluso existe la funcionalidad de reiniciar el contexto cuando hemos hecho algo mal.

## Conclusión

Python ofrece una gran flexibilidad, y si bien creo que la solución al problema planteado se puede mejorar, actualmente puede ser de utilidad para otras personas.

Personalmente, lo encuentro útil para explicar algunos conceptos de programación mediante un enfoque práctico. Ideas como abstracción, encapsulación o polimorfismo quedan mas claras si se las puede reflejar con un ejemplo visible.

[cocos2d]: http://cocos2d.org/
