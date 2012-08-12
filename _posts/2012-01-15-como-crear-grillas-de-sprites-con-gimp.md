---
layout: post
title: "Como crear grillas de sprites con Gimp"
description: ""
category: 
tags: [juegos, gimp]
draft: true
author: Hugo
---
{% include JB/setup %}

En este artículo veremos cómo crear una grilla de
imágenes para colocar en un videojuego.

Usaremos [gimp] junto a unos complementos que escribí
hace varios meses. Y por último, vamos
a poner esa grilla en acción usando el motor [pilas]

# Conceptos iniciales

Básicamente, una grilla se utiliza en los videojuegos 2D
para representar todos los cuadros de una animación:

![image](/images/como-crear-grillas-gimp/special.jpg)


Por ejemplo, en la imagen anterior tenemos 5 cuadros
de animación en la misma imagen (tomado el juego
[sbfury] de losersjuegos).

En un juego tradicional, podrías tener muchos cuadros
de animación, así que las grillas ayudan a reducir
la cantidad de imágenes drásticamente. Además, facilitan
muchísimo la programación.

Además, las grillas no solo sirven para animaciones
de personajes, también pueden ser utilizadas para todo
tipo de imágenes. 

# Herramientas

Vamos a usar dos herramientas, una es [gimp], un editor
de imágenes muy popular en Gnu/Linux, y un conjunto
de plugins:

![](/images/como-crear-grillas-gimp/gimp.jpg)


# Instalar los plugins

Vamos a instalar unas extensiones para [gimp]. En la
consola de tu equipo escribí los siguientes comandos:

    cd .Gimp-2.6/plug-ins/

    wget http://www.losersjuegos.com.ar/_media/software/Gimp/spritetool.zip

    unzip spritetool.zip

Listo, ahora solo hay que reiniciar [gimp].

Ten en cuenta que hay una descripción mucho mas detallada
de estos complementos en la [página de losersjuegos](http://www.losersjuegos.com.ar/software/Gimp)

    


# Comencemos

Vamos a tomar una imagen con varios cuadros de animación
y lo vamos a convertir en una grilla.

Tomemos la imagen de *hannias*, uno de los enemigos del
juego [sbfury]:

![](/images/como-crear-grillas-gimp/hannias.jpg)

Este imagen tiene varios cuadros de animación, pero están
desorganizados, no los podemos incorporar en el juego
directamente, necesitamos organizarlo.

Usando [gimp], vamos a copiar y pegar cada una de los cuadros
que nos interesan en una nueva imagen. Aquí podrías usar
la opción *pegar en nueva imagen*.

# Capas

Ten en cuenta que al momento de pegar, [gimp] **no hace** una nueva
capa. Tienes que pulsar el botón ``Nueva capa`` para que
esto ocurra.

Por ejemplo, al pegar la ventana de capas me muestra la selección:

![](/images/como-crear-grillas-gimp/pegar.jpg)

Pero al momento de pulsar el icono ``nueva capa`` aparecen
las dos capas:

![](/images/como-crear-grillas-gimp/pegar_en_capa.jpg)

Es importante tener cada cuadro en su propia capa, esto es lo que
nos va a permitir alinear todos los cuadros de animación y hacerlo
de manera prolija.

La ventana de capas de [gimp] te permitirá organizar las capas, ocultarlas
o incluso modificar la transparencia (bastante útil a la hora de
alinear los cuadros).

Por ejemplo, acá cambié la transparencia de una de las capas para
lograr la alineación que quería:

![](/images/como-crear-grillas-gimp/alpha.jpg)

Ten en cuenta que hay dos atajos de teclas
super útiles:

 - La tecla ``TAB`` oculta todas las ventanas auxiliares de manera temporal.
 - La tecla ``m`` te permite seleccionar la herramienta para trabajar con capas, así que luego de pulsar ``m`` vas a poder mover las capas con el teclado, usando las flechas direccionales.


## Arreglo de capas

El siguiente paso es hacer que las capas ocupen el área de la
imagen. Esto se puede hacer pulsando el botón derecho sobre el panel
de capas y seleccionando ``capa al tamaño de la imagen``. Esto
se tiene que hacer en cada una de las capas.

## Generando la grilla

Ahora si, podemos activar el plugin para convertir nuestras
capas en una grilla.

Pulsa el botón derecho sobre la ventana de la imagen y seleccionar
``Python-fu`` → ``SpriteTool`` → ``Create Sheet``.

La imagen se va a convertir en un grilla lista para usar:


![](/images/como-crear-grillas-gimp/final.jpg)


Haciendo una prueba con pilas
-----------------------------

Para probar la grilla podemos guardar la imagen cómo un
archivo ``png`` y hacer una prueba usando pilas-engine_:

Vamos a suponer que nuestro archivo se llama ``animacion.png``, el
código que nos nos permite ver la animación sería:

    import pilas
    pilas.iniciar()

    grilla = pilas.imagenes.cargar_grilla('animacion.png', 3)

    animacion = pilas.actores.Animacion(grilla, ciclica=True)
    
Esta es una imagen de la sesión interactiva de pilas:

![](/images/como-crear-grillas-gimp/prueba.jpg)


Conclusión
----------

Amigarse con el manejo de grillas en un videojuego es muy
valioso, hay un montón de trabajo fino que necesitas
hacer, incluso si solamente te dedicas a la programación.

Ojalá te animes a tomarle la mano a [gimp], es un gran
programa y se aprende rápido.


[gimp]: http://www.gimp.org/
[pilas]: http://www.pilas-engine.com.ar
[sbfury]: https://github.com/hugoruscitti/sbfury 
