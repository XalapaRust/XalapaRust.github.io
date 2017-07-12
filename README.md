# Sitio web del grupo de programación en Rust de Xalapa

Está construido en jekyll por "conveniencia".

## Construir en local

Vas a necesitar ruby. Y la gema bundler, que no viene por defecto...

Al parecer tabién necesitas las cabeceras C de ruby, deben estar en un paquete como `ruby-devel`.

Y no olvides tener las cabeceras de zlib... para compilar libxml2, que es reguerido por nokogiri, que es requerido por jekyll... ¿es neta ruby?

Finalmente ejecuta

```
bundle install --path vendor/bundle
```

Lo que puedes hacer ahora:
* Corre el servidor de jekyll: `bundle exec jekyll serve`,
* o compila el sitio `bundle exec jekyll build`.

## Escribir una entrada de blog

Solo créate un archivo en la carpeta `_posts/` cuyo nombre tenga la fecha en formato ISO. Checa las entradas que ya están ahí para darte una idea.

No olvides poner al comienzo las siguientes líneas:

```
---
title: Tu título
layout: default
---
```
