---
title: Sobre los módulos locales
layout: default
---

¿Cómo usar módulos locales?
===========================

por [@Categulario](https://twitter.com/categulario)

Como antecedente de este post está este, que explica un poco sobre los módulos en rust: [Una nota sobre modulos en rust](https://medium.com/@artur.dev/modules-in-rust-68249e9894f6). Mi motivación para escribir esta entrada es amplar el conocimiento a un caso un poco más amplio que no pude resolver leyendo eso. Algunas cosas, sin embargo, las traeré de esa entrada.

Específicamente se trata de repartir código entre módulos donde unos dependen de otros, y todos dependen de bibliotecas externas.

## Partiendo el código en cachos

Mi caso era un archivo ejecutable (`cargo new --bin my-project`) en el que habría muchas cosas. Por lo menos tres traits y la función main. Se veía como esto:

```rust
// main.rs
// chingos de extern crate por aquí
// chingos de uses por aquí

struct LineCodec;

struct LineProto;
// la implementación de LineProto usa LineCodec

struct Echo;

pub fn main() {
    // Aquí uso solamente LineProto y Echo
}
```

Mi objetivo era tener el codec (`LineCodec`), el protocolo (`LineProto`) y el servicio (`Echo`) en sus propios archivos. Así que hice lo siguiente:

**codec.rs**
```rust
// los uses de LineCodec

struct LineCodec;
```

**proto.rs**
```rust
// los uses de LineProto

struct LineProto;
```

**service.rs**
```rust
// los uses de Echo

struct Echo;
```

Con esta estructura de carpetas:

```
src
├── codec.rs
├── main.rs
├── proto.rs
└── service.rs
```

### ¿Y los extern crates apá?

De alguna manera nunca pensé que debería también distribuirlos entre los archivos, los dejé todos en el `main.rs` desde el principio, y tenía razón.

## Usando lo aprendido del post

Naturalmente que esa distribución por sí misma no compila, así que integré lo aprendido en el blog aquel añadiendo algunos `pub`s por aquí y por allá:

```rust
// codec.rs
pub struct LineCodec;

// proto.rs
pub struct LineProto;

// service.rs
pub struct Echo;

// main.rs
mod proto;
mod service;
```

Claro que esto por sí mismo no funcionó, y el problema era que hasta aquí llega esa entrada de blog, así que me concentre en los errores del compilador.

## Dependencias entre módulos

En mi caso específico `LineProto` necesita `LineCodec` para funcionar, pero `LineCodec` no es necesario en `main.rs` y eso me causaba confusión. El error del compilador era el siguiente:

```rust
error[E0412]: cannot find type `LineCodec` in this scope
  --> src/proto.rs:18:32
   |
18 |     type Transport = Framed<T, LineCodec>;
   |                                ^^^^^^^^^ not found in this scope
```

y con mucha razón. Lo primero que intenté fue usar mod dentro de `proto.rs` como sigue:

```rust
mod codec;
```

lo cual resultó en el compilador quejándose:

```rust
error: cannot declare a new module at this location
 --> src/proto.rs:7:5
  |
7 | mod codec;
  |     ^^^^^
  |
note: maybe move this module `src/proto.rs` to its own directory via `src/proto/mod.rs`
 --> src/proto.rs:7:5
  |
7 | mod codec;
  |     ^^^^^
```

Algo importante fue ignorar la sugerencia de cargo, no estoy seguro de por qué la hizo pero no me hacía sentido. Yo quería usar mi distribución de archivos y no anidar carpetas todavía. Sin embargo el error daba una pista: **No puedes declarar módulos en un módulo**.

Entonces me fui a `main.rs` a declarar el módulo `codec`:

```rust
mod codec;
mod proto;
mod service;
```

Lo cual terminó en el mismo error de no encontrar `LineCodec` en el scope de `LineProto`.

## Una ayudita externa

Fue entonces cuando decidí que preguntar no era suficiente, tenía que ir a buscar en algún lugar donde ya se hubiera resuelto mi problema, lo cual me llevó al código fuente de `tokio-proto` donde el archivo `lib.rs` usa cosas de `tcp_client.rs` y viceversa.

La solución fue la siguiente:

```rust
// main.rs --------------------------------------------
mod codec;
mod proto;
mod service;

// Trae al scope LineCodec desde codec.rs y hazlo público,
// para que los submódulos lo puedan usar
pub use codec::LineCodec;

// proto.rs -------------------------------------------
// LineCodec está en el scope del módulo principal (main.rs) que
// importa a los demás, así que aquí solo tengo que decir 
// que lo estoy usando.
use LineCodec;
```

## Conclusiones

La forma en que llegué a estas conclusiones fue leyendo el código fuente de la biblioteca `tokio`. Es tan grande que tiene que resolver este caso que yo tenía que resolver. Recomiendo bastante leer código fuente de bibliotecas grandes y bonitas como esa para aprender las mejores prácticas.

## Enlaces

* [Código fuente de tokio-proto](https://github.com/tokio-rs/tokio-proto/tree/master/src)
* [lib.rs:199](https://github.com/tokio-rs/tokio-proto/blob/master/src/lib.rs#L199)
* [tcp_client.rs:6](https://github.com/tokio-rs/tokio-proto/blob/master/src/tcp_client.rs#L6)
