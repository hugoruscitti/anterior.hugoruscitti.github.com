---
layout: post
title: "Haciendo un cargador de juegos python"
description: ""
category: 
tags: [python, windows]
draft: true
---
{% include JB/setup %}

Si escribes juegos usando python, seguramente te has encontrado en problemas al distribuir juegos sobre windows.

Python es un gran lenguaje, pero es algo difícil empaquetar programas para sistemas cómo Windows. Y no es una buena llevar todo el problema a los usuarios, mucho menos pedirles que instalen varias bibliotecas para que puedan probar nuestros juegos.

En este artículo veremos una alternativa para empaquetar y distribuir nuestros juegos sobre windows. Usaremos el concepto de **Cargador de juegos** y lo veremos paso a paso con varios ejemplos.


## Un adelanto para impacientes

Si quieres usar cargadores, pero no seguir paso a paso estas instrucciones, ve a la página de github de este proyecto y descarga una versión lista para utilizar:

- [Descargar los cargadores de juegos desde github][github_download]
- [Ver código de los cargadores en github][github]

[github]: https://github.com/hugoruscitti/cargador_de_juegos
[github_download]: https://github.com/hugoruscitti/cargador_de_juegos/downloads


## ¿Que es un cargador de juegos?

Un cargador de juegos es un archivo ejecutable `.exe` que incluye un intérprete de python y todas las bibliotecas necesarias para ejecutar juegos.

Este intérprete es independiente del juego, lo unico que sabe hacer es *"ejecutar un archivo .py"*, así que los cargadores se hacen una sola vez y luego se comparten.

Entonces, lo interesante de un cargador, es que podemos entregarlo a nuestros usuarios junto al código de nuestro juego y listo.

## Comenzando

Para iniciar, vamos a contar con un sistema windows recién instalado.

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



## Creando el cargador básico

Nuestro primer cargador solamente incluirá la biblioteca estándar de python y `Tkinter` para manejo de interfaz gráfica.

Construye un archivo llamado ``setup.py`` y el siguiente contenido:

{% highlight python %}
import tkMessageBox
import Tkinter
import imp

try:
    imp.load_source("__main__", "run.py")
except IOError:
    root = Tkinter.Tk()
    root.withdraw()

    MENSAJE_ERROR = "Lo siento, no se encuentra el archivo run.py"
    tkMessageBox.showerror("Error", MENSAJE_ERROR)
{% endhighlight %}

Este programa solamente va a buscar y ejecutar un archivo llamado `run.py`, y si no lo encuentra va a emitir un mensaje de error:


![](/images/cargador_de_juegos/error.png)


## Generando el archivo ejecutable

Ahora solo nos queda *compilar* nuestro cargador para que se pueda ejecutar independiente de nuestro sistema.

Tenemos que crear dos archivos.

`setup.py`:

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

Hagamos un pequeño cambio para agregar soporte a pygame.

Primero tenemos de asegurarnos de tener instalada la biblioteca pygame, para nuestra versión de python (2.6) y luego tenemos que editar el archivo cargador y colocar en la primer linea lo siguiente:

    import pygame

por último, tendríamos que ejecutar nuevamente el archivo `crear_ejecutable.bat` y revisar la carpeta `build`.


Puedes verlo con mas detalle en [github][github_pygame].

[github_pygame]: https://github.com/hugoruscitti/cargador_de_juegos/tree/master/cargador_pygame


## Versión con soporte para Cocos2D

    TODO:

## Version con soporte para pilas-engine

    TODO:
