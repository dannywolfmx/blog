+++
title = "Usar org mode en hugo"
author = ["John Doe"]
date = 2022-03-27T17:03:20-06:00
draft = false
+++

## ox-hugo (emacs) {#ox-hugo--emacs}

Pagina del proyecto: [ox-hugo - Org to Hugo exporter](https://ox-hugo.scripter.co/)

Al parecer Hugo no soporta por default los archivos de tipo ORG, por lo que debemos transpilar nuestro archivo a markdown para que hugo pueda hacer uso de este, en doom-emacs es tan facil como agregar hugo a las opciones de org mode tal que:

```lisp
(doom!
 :lang
 (org +hugo))
```

Lo anterior nos habilitara la posibilidad de usar ox-hugo sin necesidad de tocar mucho mas.

Ahora requerimos indicarle a ox-hugo donde esta nuestro proyecto de hugo, en la documentación nos brindan 2 opciones [Link documentación](https://ox-hugo.scripter.co/#before-you-export)

En mi caso encontre que la opción de agregar a mi archivo la ruta del proyecto es mas comodo

```lisp
#+title: "Usar org mode en hugo"
#+date: 2022-03-27T17:03:20-06:00
#+draft: false
#+hugo_base_dir: ../..
```

Debemos destacar que ox-hugo considera la siguiente ruta para colocar el markdown colocado

```nil
/proyecto-de-hugo/content/posts/
```

De tal forma que nuestro archivo org es recomendable colocarlo dentro de la carpeta posts, posterior a esto teniendo el archivo ORG en la ventana ejecutamos el siguiente comando de doom-emacs

```text
SPC m e H H
```

Adicional existe una opción de ox-hugo que transpila cada vez que detecta un cambio en nuestro archivo org; pero este lo encontre un poco molesto dado que emacs te tiende a pregunatar si estas seguro de editar el buffer:

```text
M-x org-mode-auto-export-mode
```


### Algunos sitios interesantes: {#algunos-sitios-interesantes}

<https://willschenk.com/articles/2019/using_org_mode_in_hugo/>

<https://lucidmanager.org/productivity/create-websites-with-org-mode-and-hugo/>
