# Sitio web del grupo de programación en Rust de Xalapa

Está construido en jekyll por "conveniencia".

## Construir en local

Vas a necesitar ruby.

* Instala `jekyll` y `bundler`: `bundle install jekyll bundler`
	- vas a querer bundler en tu `PATH`
* Instala `github-pages` en local: `bundle install --path vendor/bundle`
* Corre el servidor de jekyll: `bundle exec jekyll serve`,
* o compila el sitio `bundle exec jekyll build`.
