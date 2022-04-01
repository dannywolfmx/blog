+++
title = "Configurar doom-emacs para tranpilar archivos ORG a Markdown y usarlo en Hugo"
author = ["Daniel Menchaca Luna"]
date = 2022-03-27T17:03:20-06:00
draft = false
+++

Hugo por su simpleza y rapidez resulta una opción interesante cuando deseamos tener un blog simple, pero este de forma nativa no soporta ORG, y requerimos usar un parser para esta tarea.

En los siguientes párrafos mostraré una seria de instrucciones para poder configurar nuestro doom-emacs para que cumpla con la tarea de transpilar nuestro ORG a un archivo de tipo Markdown que si pueda interpretar Hugo


## ox-hugo (emacs) {#ox-hugo--emacs}

Página del proyecto: [ox-hugo - Org to Hugo exporter](https://ox-hugo.scripter.co/)

ox-hugo es el modulo que nos ayudara a realizar la tarea de transpilación que requiere Hugo, en doom-emacs es tan fácil como agregar a nuestro ORG el flag de Hugo como sigue:

```lisp
(doom!
 :lang
 (org +hugo))
```

Posterior a esto hacemos el tipico doom sync, y doom/reload

Lo anterior nos habilitara la posibilidad de usar ox-hugo sin necesidad de tocar mucho mas.

Ahora requerimos indicarle a ox-hugo donde esta nuestro proyecto de hugo, en la documentación nos brindan 2 opciones [Link documentación](https://ox-hugo.scripter.co/#before-you-export)

En mi caso encontré que la opción de agregar a mi archivo la ruta del proyecto es mas cómodo

```org
#+title: "Usar org mode en hugo"
#+date: 2022-03-27T17:03:20-06:00
#+author: "Daniel Menchaca Luna"
#+draft: false
#+hugo_base_dir: ../..
```

Nota: este es el esquema que utilizo para que Hugo sepa el titulo y la fecha de mi post

Debemos destacar que ox-hugo considera la siguiente ruta para el markdown resultante

```nil
/proyecto-de-hugo/content/posts/
```

De tal forma que nuestro archivo ORG es recomendable colocarlo dentro de la carpeta posts, posterior a esto teniendo el archivo ORG en la ventana ejecutamos el siguiente comando de doom-emacs

```text
SPC m e H H
```

Adicional existe una opción de ox-hugo que transpila cada vez que detecta un cambio en nuestro archivo ORG; pero este lo encontré un poco molesto dado que emacs te tiende a preguntar si estas seguro de editar el buffer temporal que te genera ox-hugo cuando realiza la transpilación:

```text
M-x org-mode-auto-export-mode
```


## Algunos sitios interesantes: {#algunos-sitios-interesantes}

<https://willschenk.com/articles/2019/using_org_mode_in_hugo/>

<https://lucidmanager.org/productivity/create-websites-with-org-mode-and-hugo/>
