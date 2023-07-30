---
title: "Retomando El Blog"
date: 2023-07-30T00:27:18-06:00
draft: false
---

Ha pasado un tiempo desde que escribí una entrada a esté blog, y me gustaria retomarlo, por lo que el primer paso para hacerlo es empezar con esta entrada.

## ¿Qué he estado haciendo?

Por lo que veo mi última entrada está relacinada a un proyecto que tuve el año pasado, por lo que quizás lo mejor sea empezar desde el día de hoy hasta ese día.

### Sistema de cotizaciones

Link al proyecto que a la fecha de este blog aun no lo tendré publico: [Cotizador](https://github.com/dannywolfmx/cotizador)

Desde hace varios años he tenido la inquietud de crear un sistema de cotizaciones, debido a que el software de terceros que utilizo hoy en día tiene carencias que considero con un sistema propio podría solucionar. Es por ello que en estos últimos meses me he puesto manos a la obra y por fin estoy trabajando en este proyecto.

#### ¿Qué tecnologías estoy utilizando?

Para este proyecto estoy utilizando las siguientes tecnologías:

- [Go](https://golang.org/): Lenguaje de programación.
- [SQLITE](https://www.sqlite.org/index.html): Base de datos.
- [SQLX](https://jmoiron.github.io/sqlx/): Librería para trabajar con bases de datos en Go.
- [Svelte](https://svelte.dev/): Framework para crear interfaces de usuario.
- [UnoCSS](https://unocss.dev/): Framework CSS que en pocas palabras es una alternativa a TailwindCSS.
- [Axios](https://axios-http.com/): Librería para realizar peticiones HTTP.
- [VSCode](https://code.visualstudio.com/): Editor de código.
- [Git](https://git-scm.com/): Control de versiones.
- [Github copilot](https://copilot.github.com/): Inteligencia artificial para programar (Que ya hablaremos de ella más a fondo en otra entrada)

#### ¿Qué he logrado hasta el momento?

El camino ha sido un tanto complejo, me lo veia venir, pero la caracteristica libre que queria seguir con este proyecto personal ha causado que tenga que hacer y rehacer muchas cosas. Por ejemplo, actualmente me encuentro migrando un estado funcional de la aplicación que usaba GORM (un ORM) para en su lugar usar SQLX que basicamente es un acercamiento a usar sql puro en Go con la diferencia de que te brinda algunas funciones utiles para trabajar con las estructuras de Go.

Gracias a esto me he forzado a entender por fin como hacer una base de datos real, debido a que en el pasado dependía más de usar Go en lugar de SQL para hacer las cosas. Por ejemplo usaba un for para hacer insert, pero ahora puedo usar un bulk insert que es mucho más eficiente.

### Image to ascii

Link al proyecto: [Image to ascii](https://github.com/dannywolfmx/image-to-ascii)

Este proyecto tambien es un proyecto personal, con la diferencia que fue creado unicamente para aprender varios temas que tenia pendintes, los cuales son el como funcionan las imagenes (En este caso gif) y como interactuar con la terminal de forma un poco más cruda.

El resultado ha sido bastante bueno, ya que logré un sistema que en poco tiempo logró mostrar Gif de forma eficiente y facil en terminal.

### Chat voice

Link al proyecto: [Chat voice](https://github.com/dannywolfmx/twitch-chat-voice)

Originalmente mi intención era crear una aplicación para tener en mi celular y que leyera con un TTS (Text to speech) los mensajes de chat de Twitch, pero tras varias iteraciones terminé dejando el proyecto unicamente estable para escritorio.

Este fue un proyecto un tanto más complejo, debido a que Go no se caracteriza por tener librerias populares para crear GUI. Pasé por Gio, Fyne y al final me terminé quedando con Wails, que es basicamente usar webkit para crear una aplicación de escritorio con Go.

Este proyecto tengo intenciones de volverlo a hacer pero usando el navegador como base, dado que este viene con TTS incorporado y me puede venir bien para tener mejor control sobre como se lee el texto.

## ¿Qué sigue?

Al momento me interesa continuar con el blog, pero siempre resulta complejo tener la cabeza para pensar en los temas, por lo que estaré ideando una lista con ideas potenciales a tratar.