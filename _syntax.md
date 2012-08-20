# Syntax

Comandos basicos:

    jekyll --server

    rake post title='titulo del post'
    rake page name="pages/acercade"

Imagenes:

    ![](ruta/imagen.png)

Basicos:

    *italic*, **bold**, ~~tachar~~, ``code``.

    [palabra a subrayar][id]
    [id]: http://ejemplo.com

Codigo:

    Se puede indentar, o bien:

    {% highlight python %}
    Codigo
    {% endhighlight %}

Tablas:

    col1  | col2
    ----- | -----:
    item1 | item2
    item3 | item4


    Nota: los dos puntos (:) hacen que la columna 2 esté alineada a la derecha.
