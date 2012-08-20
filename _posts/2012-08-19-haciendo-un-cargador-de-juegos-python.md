---
layout: post
title: "Haciendo un cargador de juegos python"
category: 
description: "Instrucciones para facilitar la publicación de juegos en windows."
tags: [python, windows]
---
{% include JB/setup %}

Si escribes juegos usando python, seguramente te has encontrado en problemas al distribuir juegos sobre windows.

Python es un gran lenguaje, pero cuando se utiliza con varias bibliotecas
sobre Windows termina siendo algo difícil de transportar de un equipo a otro, y si quieres
presentar tus juegos a muchas personas eso termina convirtiendose en un problema.

En este artículo veremos una alternativa para empaquetar y distribuir nuestros juegos sobre windows de
manera bastante sencilla, crearemos varios cargadores de juegos para bibliotecas como [pygame], [cocos2d] y
[pilas-engine].


## Un adelanto para impacientes

Si quieres usar cargadores, pero no seguir paso a paso estas instrucciones, ve a la página de github de este proyecto y descarga las versiones listas para utilizar:

- [Descargar los cargadores de juegos desde github][github_download]
- [Ver código de los cargadores en github][github]

[github]: https://github.com/hugoruscitti/cargador_de_juegos
[github_download]: https://github.com/hugoruscitti/cargador_de_juegos/downloads

## ¿Que es un cargador de juegos?

Un cargador de juegos es un archivo ejecutable `.exe` que incluye un intérprete de python y todas las bibliotecas necesarias para ejecutar juegos.

Este intérprete es independiente del juego, lo unico que sabe hacer es *"ejecutar un archivo .py"*, así que los cargadores se hacen una sola vez y luego se comparten, ya sea entre proyectos o entre programadores.

Entonces, lo interesante de un cargador, es que podemos entregarlo a nuestros usuarios junto al código de
nuestro juego y listo, van a poder jugar sin necesidad de configurar o instalar nada mas...

## Comenzando

Para iniciar, vamos a comenzar con un sistema windows que no tiene python ni otras
bibliotecas instaladas, solo para comenzar desde el principio.

Mi recomendación es que utilices [virtual box], así todo tu entorno permanece independiente de las pruebas que realicemos.

[virtual box]: https://www.virtualbox.org/

![](/images/cargador_de_juegos/desktop.png)


# Paso a paso

Primero debes instalado python 2.6.6, la ultima versión binaria para windows está en:

<http://www.python.org/download/releases/2.6.6/>

Asegurate de instalar python en el directorio `c:\Python26`:

![](/images/cargador_de_juegos/instala_python.png)

Luego tendríamos que instalar `cx-freeze` para la versión `2.6`. El sitio de descargas es:

<http://cx-freeze.sourceforge.net>


Ten en cuenta que aquí estoy usando python *2.6*, aunque las mismas instrucciones podrían
funcionar con versiones mas nuevas también.


## Creando el cargador básico

Nuestro primer cargador solamente incluirá la biblioteca estándar de python y `Tkinter` para manejo de interfaz gráfica.

Construye un archivo llamado ``cargador.py`` y el siguiente contenido:

{% highlight python %}
import tkMessageBox
import Tkinter
import imp
import sys
import os

window = Tkinter.Tk()
window.wm_withdraw()

if not os.path.exists('run.py'):
    tkMessageBox.showerror("Error", "No se encuentra el archivo run.py")
    sys.exit(1)

try:
    imp.load_source("__main__", "run.py")
except Exception, e:

    tkMessageBox.showerror("Error", e)
{% endhighlight %}

Este programa solamente va a buscar y ejecutar un archivo llamado `run.py`, y si no lo encuentra va a emitir un mensaje de error:


![](/images/cargador_de_juegos/error.png)


## Generando el archivo ejecutable

El siguiente paso es *compilar* nuestro cargador para que se pueda ejecutar de manera independiente.

Tenemos que crear dos archivos, `setup.py`:

{% highlight python %}
from cx_Freeze import setup
from cx_Freeze import Executable

exe = Executable(
        script="cargador.py",
        base="Win32GUI",
)

setup(
    name="Cargador",
    version="0.1",
    description="Un cargador de juegos",
    executables=[exe],
)
{% endhighlight %}


y `crear_ejecutable.bat`:

    c:\Python26\python.exe setup.py build
    pause


tendríamos que tener un directorio similar al siguiente:

![](/images/cargador_de_juegos/archivos.png)

Ahora tienes que ejecutar el archivo `crear_ejecutable.bat`.

En pantalla debería aparecer todo el proceso de generación del ejecutable:

![](/images/cargador_de_juegos/compilando.png)


Al final de la compilación, se habrá generado una carpeta con el
ejecutable del cargador:

![](/images/cargador_de_juegos/build.png)


## Probando el cargador

Ahora la carpeta `build` tendrá todos los archivos para incluir, excepto el de nuestro programa.

Hagamos un programa sencillo, algo llamado `run.py`:

{% highlight python %}
import tkMessageBox
import Tkinter

root = Tkinter.Tk()
root.withdraw()
tkMessageBox.showinfo("Hola", "Bienvenido al primer programa de ejemplo")
{% endhighlight %}

Listo, ahora solo hay que colocarlo en la carpeta `build` y ejecutar el nuevo archivo `cargador.exe`:

![](/images/cargador_de_juegos/exito.png)


## Versión 2: añadiendo soporte para pygame

Hasta ahora, tenemos lo principal del cargador de juegos. Aunque no incluye muchas bibliotecas que utilizamos para realizar videojuegos.

Hagamos un pequeño cambio para agregar soporte a [pygame].

Primero tenemos de asegurarnos de tener instalada la biblioteca [pygame], para nuestra versión de python (2.6) y luego tenemos que editar el archivo cargador y colocar en la primer linea lo siguiente:

    import pygame

por último, tendríamos que ejecutar nuevamente el archivo `crear_ejecutable.bat` y revisar la carpeta `build`.


Puedes verlo con mas detalle en [github][github_pygame].

[github_pygame]: https://github.com/hugoruscitti/cargador_de_juegos/tree/master/cargador_pygame

[pygame]: http://www.pygame.org

## Versión 3: con soporte para Cocos2D

Para crear un cargador especial de [cocos2d] necesitamos instalar [pyglet]
y luego [cocos2d]:

![](/images/cargador_de_juegos/pyglet.png)

Una vez concluido el proceso de instalación, tendríamos que volver
a editar el archivo cargador e incluir a [cocos2d]:


    import cocos2d


Y listo, ahora solo queda ejecutar ``crear_ejecutable.bat`` y distribuir nuestro cargador:

![](/images/cargador_de_juegos/cocos2d.png)


[cocos2d]: http://cocos2d.org
[pyglet]: http://www.pyglet.org


Puedes verlo con mas detalle en [github][github_cocos2d].

[github_cocos2d]: https://github.com/hugoruscitti/cargador_de_juegos/tree/master/cargador_cocos2d


## Version con soporte para pilas-engine

Siguiendo las [instrucciones de instalación][install_pilas] para [pilas-engine] sobre windows, instalamos
pyqt4, box2D y luego [pilas-engine].

Es importante tener en cuenta que box2D necesita un [pequeño cambio][patch_box2d] como nos indicó barajas en el foro de losersjuegos.

[patch_box2d]: http://www.losersjuegos.com.ar/foro/viewtopic.php?f=26&t=1526#p6985

[pilas-engine]: http://www.pilas-engine.com.ar

[install_pilas]: http://pilas-engine.com.ar/doc/tutoriales/instalacion/windows_xp.rst

Una vez concluido el proceso de instalación, tendríamos que volver
a editar el archivo ``cargador.py`` e incluir a [pilas-engine]:

    import pilas

A diferencia de los anteriores, [pilas-engine] necesita varios archivos de
recursos para funcionar correctamente. Vé al directorio donde se encuentra
el código fuente de [pilas-engine] en tu equipo, y copia el directorio ``data``
completamente dentro del directorio ``build`` que contiene al cargador (y 
el archivo ``run.py``)

Ahora sí, ejecutando el cargador obtendremos:

![](/images/cargador_de_juegos/pilas1.png)

y cuando pulsemos el botón "OK" aparecerá:

![](/images/cargador_de_juegos/pilas2.png)

Puedes ver un ejemplo de cargador [pilas-engine] en [github][github_pilas].

[github_pilas]: https://github.com/hugoruscitti/cargador_de_juegos/tree/master/cargador_pilas


## Finalizando

Los cargadores de juegos son una gran oportunidad para distribuir juegos y hacerlos mucho mas accesibles a los usuarios.

Espero que este documento te resulte útil, y recuerda que tenemos un repositorio en [github] con la última versión de los cargadores. Estás invitado a participar del desarrollo.
