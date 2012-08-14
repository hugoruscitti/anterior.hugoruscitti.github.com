---
layout: post
title: "Rediseñando los eventos de pilas-engine"
description: ""
category: 
tags: [python, pilas, sbfury]
---
{% include JB/setup %}

El sistema de eventos de [pilas-engine] necesitaba
varias mejoras. Desde que lo implementé, siempre tuve
la sensación de que se podía simplificar y mejorar
notablemente.

Por suerte hoy logré implementar los cambios que quería,
le dediqué varias horas pero valió la pena el esfuerzo, quedó
mucho mejor de lo que esperaba, y en el camino aprendí un
montón.

# ¿Por qué es importante?

Casi todos los motores de juegos tienen algún sistema de eventos,
de alguna u otra forma se te permite conocer el estado de las
teclas, del mouse etc.

Pero en [pilas-engine] la cosa es un poco mas ambiciosa, [pilas-engine]
va a ser una de las primeras experiencias de programación para
muchas personas, y actualmente hay juegos en desarrollo que van
a crecer y se van a volver mas complejos.

Por ese motivo, el sistema de eventos (el corazón de los juegos), merece
mucha atención. Tiene que ser un sistema simple, fácil de utilizar y a
la vez flexible.

# El problema

A medida que estás haciendo un juego la cosa se empieza a tornar
compleja. Comienzas a tener contadores, barras de energía, personajes, enemigos etc.

Incluso si has previsto todas estas cosas antes de empezar, no es
una buena idea diseñar con todo eso en mente. Es mejor ir paso a paso, modificando
lo existente y adaptándolo, progresando mediante prototipos simples y que se 
puedan extender.

Aquí es donde el sistema de eventos de [pilas-engine] se destaca
de otros, ¿Por qué?.

Porque los eventos se pueden usar para poner en comunicación a
varios componentes del juego, usando una solución elegante y
muy sofisticada del mundo del software llamada *patrón de diseño observador*.

# Un ejemplo: barras de energía

[pilas-engine] está pensado para hacer juegos, así que en lugar de
hablar desde un punto de vista teórico, te voy a comentar cómo
funciona el sistema de eventos en un juego real: ``Shaolin's Blind Fury``

En el juego ``Shaolin's Blind Fury`` hay una barra de energía que
nos permite ver la vitalidad de un enemigo. Esto es útil para conocer
cuanto puede vivir un enemigo mientras luchamos contra él:

![](/images/eventos/barra.jpg)

Esta barra solamente muestra la energía del enemigo que estamos
golpeando. Si nos alejamos, y luchamos contra otro enemigo, la misma
barra nos tiene que mostrar la energía del nuevo contrincante.

Imagina que no conocemos los eventos de pilas, ¿Cómo podríamos mostrar
una barra de energia?: podríamos escribir algo así:

    barra = Energia(un_enemigo)

y luego podríamos hacer que este objeto ``lea`` el atributo
numérico ``energia`` del enemigo:

    class Energia(ActorEnergia):

        def __init__(self, enemigo):
            self.enemigo = enemigo

        def actualizar(self):
            self.progreso = self.enemigo.energia


Si nuestro juego solo tuviera un enemigo, estaríamos perfecto. Pero no, lo
complicado de esta solución es lograr que la misma barra pueda
mostrar la energía de otros enemigos.

¿Cómo tendría que diseñar las cosas si mi juego tiene mas enemigos?.

La respuesta inmediata sería: "enviarle a la energía una lista de enemigos, 
en lugar de uno solo". Pero si hago eso, ¿Cómo hago para que la barra
de energía sepa el momento justo en que he logrado golpear a otro enemigo?.

Como verás, la solución inicial se va volviendo demasiado complicada, y
tenemos que hacer que la barra de energía sepa cada vez mas cosas
y reciba mas argumentos. Es demasiado amenazante, si seguimos por
este camino se va a poner demasiado complejo...


# Un enfoque distinto: menos acople

Vamos a cambiar la estrategia, usemos el nuevo sistema de eventos
de [pilas-engine]:

Claramente necesito saber "en qué momento se ha golpeado a un enemigo", así
que mi primer paso es crear un evento que represente eso:

    pilas.eventos.cuando_golpean = pilas.eventos.Evento("cuando golpean")

Ese evento, ahora me va a servir para conocer el momento exacto del
golpe.

La barra de energía necesita ``observar`` a ese evento, porque cuando
ese evento se ``emita`` voy a necesitar redibujar la energía:

    class Energia(ActorEnergia):

        def __init__(self):
            pilas.eventos.cuando_golpean.conectar(self.actualizar_energia)

        def actualizar(self):
            # ahora no hace nada...
            pass

        def actualizar_energia(self, evento):
            self.progreso = evento.quien.energia

De esta forma, la barra queda completamente libre de los enemigos, no
necesita tener una referencia o una lista, no importa. La
barra solamente será notificada cuando el evento ``cuando_golpean`` sea
``emitido`` por alguien mas.

Por último, en el código del enemigo quero emitir la señal:

    class Enemigo(Actor):
        
        def recibir_golpe(self):
            self.energia -= 10

            if self.energia < 0:
                self.morir()

            pilas.eventos.cuando_golpean.emitir(quien=self)

y listo, ahora cuando un enemigo reciba un golpe, simplemente
emitirá la señal ``cuando_golpean``. Y en nuestro caso, esa señal
es observada por la ``Energia``.

Algo interesante del ejemplo anterior, es que cuando emitimos
una señal podemos enviar los argumentos que queramos. En este caso usé el
argumento ``quien``, porque me interesa saber quién recibió el golpe para
mostrar su energía. Puedes mirar el código de la clase Energia para ver cómo estoy
leyendo ese parámetro ``quien``.

Ten en cuenta que ahora no importa cuantos enemigos tengamos en
nuestro juego. Tampoco nos limita tener una sola barra de energía, de hecho,
podríamos agregar un contador de puntajes, que nos aumente el puntaje
cada vez que golpeamos a un enemigo. ¿Cómo?, simplemente haciendo que
el puntaje sea un ``observador`` del evento ``cuando_golpean``, igual que
la energía.

# Otro ejemplo pero sin código, solo para pensar

Imagina lo simple que resulta esta comunicación y cómo nos
puede simplificar el desarrollo:

Piensa en el juego pacman. Hay un protagonista, muchas pastillas y fantasmas:

![](/images/eventos/pacman.jpg)

En un juego como pacman podríamos crear un evento llamado ``come_pastilla``,
y emitirlo cada vez que el ``pacman`` toca una pastilla.

A su vez, a este evento ``come_pastilla`` lo podrían estar observando dos
actores: un ``contador de puntaje`` que se incrementa con cada pastilla y
una ``escena``, que podría tener un contador sencillo para saber cuando
tiene que pasar al siguiente nivel.

Otro evento llamado ``muere_pacman`` podría ser observado por un actor 
``contador de vidas``, que maneje un visor de vidas al costado de la pantalla.

Y un evento como ``come_pastilla_especial`` podría hacer que todos los
fantasmas observadores de ese evento se pongan azules!



# Conclusión

El nuevo sistema de eventos de pilas es paso adelante, le
va a permitir a muchas personas lograr diseños de video-juegos
mas simples y fáciles de extender.


Personalmente, estoy contento por las posibilidades técnicas
que ofrece, y además, porque los resultados los estoy
poniendo en práctica ahora mismo con el juego ``Shaolin``.

Ojalá mi artículo te halla resultado útil, y que los eventos
de [pilas-engine] te parezcan una buena idea.

Gracias!


[pilas-engine]: http://www.pilas-engine.com.ar


