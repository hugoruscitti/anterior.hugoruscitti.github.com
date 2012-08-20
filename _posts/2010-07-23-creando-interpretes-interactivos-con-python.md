---
layout: post
title: "Creando intérpretes interactivos con python"
category: 
description: "Instrucciones para facilitar la publicación de juegos en windows."
tags: [python]
---
{% include JB/setup %}


Python, al ser un lenguaje dinámico, facilita mucho la creación de intérpretes.

En este arículo veremos algunas herramientas para que podamos crear nuestro propio intérprete de comandos.
Algo básico con readline

Readline es una biblioteca muy popular en GNU/Linux, se puede integrar a casi cualquier programa y permite crear autocompletados de comandos, historiales y busquedas.

Por ejemplo, el interprete de comandos Bash utiliza readline y por lo tanto se puede autocompletar comandos con TAB, recorrer el historial pulsado ARRIBA en el teclado y mas...

Para crear un programa interactivo de python usando readline se puede usar el módulo readline:


{% highlight python %}
import readline

nombres = ['pepe', 'pedro']

def completer(text, state):
    options = [x for x in nombres if x.startswith(text)]

    try:
        return options[state]
    except IndexError:
        return None

readline.set_completer(completer)
readline.parse_and_bind("tab: complete")

while 1:
    print ""
    nombre = raw_input("Escriba un nombre: ")
    print "Ha ingresado el nombre:", nombre

    if nombre in nombres:
        print "\t(ya existia este nombre en la lista de autocompletado)."
    else:
        nombres.append(nombre)
        print "\t(agregandolo a la lista de autocompletado)."
{% endhighlight %}


## Autocompletado inteligente con rlcompleter

Ahora bien, con lo que hemos visto ya podemos crear autocompletado de palabas personalizados. Pero si lo que buscamos es autocompletar código python, necesitarías inspeccionar objetos, módulos o bibliotecas.

Un módulo conveniente para hacer esta tarea es rlcompleter, que se encarda de hacer todo este trabajo de sugerencias por nosotros.

Esto es interesante para hacer interpretes python personalizados, por ejemplo el siguiente intéprete se puede usar para hacer pruebas sobre dos conjuntos:

{% highlight python %}
import rlcompleter
import readline
readline.parse_and_bind("tab: complete")

grupo_a = set([1, 2, 3])
grupo_b = set([3, 4, 5])

cmd = None

while cmd not in ['quit', 'exit']:
    cmd = raw_input('>> ')
    exec(cmd)
{% endhighlight %}

Inicie el script, escriba grup y luego TAB, en pantalla se tienen que mostrar las referencias que comienzan con grup. Luego, si escribe grupo_a. y luego TAB puede ver los atributos del objeto señalado por la referencia

Internamente, rlcompleter hace todo el trabajo de intronspección por nosotros.


## Atajos de teclado

Por último, ten en cuenta que readline viene con muchos atajos de teclado.

Tener en mente esto te permite usar mas eficientemente el módulo readline y cualquier aplicación que utilice la biblioteca, por ejemplo bash o ipython.


<style>
table td {
    padding-right: 2em;
}
</style>


Atajo 	          | Acción
----------------- |------------------------------------------
CTRL+f 	          | Mover el cursor un caracter a la derecha.
CTRL+b 	          | Mover el cursor un caracter a la izquierda.
CTRL+a            | Ir al principio de la linea.
CTRL+e            | Ir al final de la linea.
CTRL+w            | Borrar la última palabra que se escribió.
↑, ↓         	  | Navegar por el historial de textos ingresados.
CTRL+r            | Buscar en el historial de textos (hacia atrás).
CTRL-l            | Limpiar la pantalla (sin borrar el texto actual).
TAB 	          | Autocompletar
