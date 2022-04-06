+++
title = "Configurar Let's Encrypt en traefik"
author = ["Daniel Menchaca Luna"]
date = 2022-04-03T00:00:20-06:00
draft = false
+++

## Resumen {#resumen}

En el siguiente articulo veremos como configurar SSL en traefik usando Let's Encrypt que aunque es fácil este requiere un par de pasos para su puesta en marcha.

Debido a que es una configuración que requiere varios pasos es posible que cometiera algun error, de ser el caso agradeceria me lo reportaran a mi twitter  <https://twitter.com/dannywolfmx/>


## Configuración de traefik.yml {#configuración-de-traefik-dot-yml}

```yaml
api:
  #Habilitar el dashboard de traefik
  dashboard: true
  debug: true
entryPoints:
  #Los entry points para http (80) y https (443)
  http:
    address: ":80"
  https:
    address: ":443"
serversTransport:
  insecureSkipVerify: true
providers:
  docker:
    endpoint: "unix:///var/run/docker.sock"
    exposedByDefault: false
  file:
    filename: /config.yml
certificatesResolvers:
  #nombre de nuestro resolver, este puede ser el nombre que quieras, por comodidas le he puestro el siguiente
  letsencrypt:
    #los datos ligados a nuestro
    acme:
      email: nuestro_email@nuestro_dominio.com
      storage: acme.json
      dnsChallenge:
        provider: linodev4
        resolvers:
          - "1.1.1.1:53"
          - "1.0.0.1:53"
```

Como notaremos tenemos indicado la existencia de un archivo llamado **acme.json**, es importante crearlo y que este tenga permiso 600

```bash
touch acme.json
chmod 600 acme.json
```

Un punto importante es identificar el **provider** al cual Let's Encrypt se conectará, en la documentación podemos encontrar la lista de proveedores [Let's Encrypt - Traefik](https://doc.traefik.io/traefik/https/acme/#providers). Lamentablemente traefik no es del todo claro cuando hemos seleccionado adecuadamente nuestro proveedor, y no nos arrojara un error que nos brindé luz sobre cual es el problema. En mi caso en particular que estoy utilizando linode, traefik cuanta con 2 formas de configurar una cuenta de linode, y por desgracia demoré en darme cuenta que la mía debía ser **linodev4**

Como notaremos en la documentación, traefik por debajo utiliza LEGO un cliente para interactuar con Let's Encrypt y que nos será de utilidad para saber como configurar nuestro cliente en base al provider.


### Ejemplo de Linode V4 {#ejemplo-de-linode-v4}

[Linode (v4) :: Let’s Encrypt client and ACME library written in Go.](https://go-acme.github.io/lego/dns/linode/)

En la documentación de Lego encontraremos algo como lo siguiente:

```bash
LINODE_TOKEN=xxxxx \
lego --email myemail@example.com --dns linode --domains my.example.org run
```

Pero debido a que traefik es quien se encargará de hacer la interacción con Lego debemos ajustar el anterior comando para que traefik sea quien haga de forma interna este comando.

-   **LINODE_TOKEN:** este token lo debemos de conseguir desde nuestro panel de control dentro de linode  [Linode Manager](https://cloud.linode.com/profile/tokens), dentro del panel generaremos un nuevo **Personal Access Token** y le daremos únicamente permisos para poder leer y escribir dominios. (Mas adelante veremos donde colocar este token).
-   **email:** este ya lo hemos indicado en nuestro **traefik.yml** en el apartado de email de acme, por lo que no es necesario modificar nada.
-   **dns:** este ya lo hemos indicado en los parametros de nuestro dnsChallenge en el archivo **traefik.yml**.

En caso de que se lo pregunten, es correcto los resolvers son las IPs publicas de cloudflare para sus DNS, pero no se preocupen dado que son validos como remplazo a los de linode.


## Configuración del contenedor de traefik usando docker-compose {#configuración-del-contenedor-de-traefik-usando-docker-compose}

```nil
version: '3.4'

services:
  traefik:
    container_name: traefik
    restart: unless-stopped
    image: traefik:v2.1
    networks:
      - proxy
    ports:
      - "80:80"
      - "443:443"
    environment:
      - LINODE_TOKEN=AQUI PONDREMOS EL TOKEN QUE GENERAMOS ANTES
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock:ro
      - ./data/traefik.yml:/traefik.yml:ro
      - ./data/config.yml:/config.yml:ro
      - ./data/acme.json:/acme.json
    labels:
      - "traefik.enable=true"


      - "traefik.http.routers.traefik.entrypoints=http"
      - "traefik.http.routers.traefik.rule=Host(`traefik.example.com`)"

      - "traefik.http.middlewares.https-redirect.redirectscheme.scheme=https"
      - "traefik.http.middlewares.sslheader.headers.customrequestheaders.X-Forwarded-Proto=https"

      - "traefik.http.routers.traefik.middlewares=https-redirect"

      - "traefik.http.routers.traefik-secure.entrypoints=https"
      - "traefik.http.routers.traefik-secure.rule=Host(`traefik.example.com`)"

      #user password
      - "traefik.http.middlewares.auth.basicauth.users=user:$$apr1$$ZCdpObME$$JUZ7NMS93R/k54WEYpek80"
      - "traefik.http.routers.traefik-secure.middlewares=auth"

      - "traefik.http.routers.traefik-secure.tls=true"
      - "traefik.http.routers.traefik-secure.tls.certresolver=letsencrypt"
      - "traefik.http.routers.traefik-secure.tls.domains[0].main=example.com"
      - "traefik.http.routers.traefik-secure.tls.domains[0].sans=*.example.com"

      - "traefik.http.routers.traefik-secure.service=api@internal"


networks:
  proxy:
    external: true
```


### Volúmenes {#volúmenes}

Para entender los volúmenes de mejor forma este es mi estructura de archivos para este contenedor:

```bash
+ docker-compose.yml
- data
    | - acme.json
    | - config.yml //este no lo utilizaremos en este tutorial pero puedes crearlo para futuros usos
    | - traefik.yml
```

Como tal todos los volúmenes serán de tipo read-only **ro** menos el de acme.json debido a que traefik escribirá y leerá de este archivo para almacenar la información generada por Lego.


### Network {#network}

Definimos nuestra network llamada **proxy** en el docker-compose. No es necesario que se llame **proxy** puede ponerle el nombre que consideres mejor para tu proyecto

```nil
services:
  traefik:
    networks:
      - proxy
```

Y de igual forma dentro de nuestro servicio traefik indicamos las redes a la cual podrá estar conectado nuestro traefik, en este caso solo será a **proxy**

```nil
networks:
  - proxy
```

Creación de nuestra red llamada **proxy** en nuestra shell

```bash
docker network create proxy
```


### LINODE_TOKEN {#linode-token}

Este apartado variará dependiendo de nuestro proveedor, por lo que vale la pena consultemos en la documentación [Let's Encrypt - Traefik](https://doc.traefik.io/traefik/https/acme/) y en prestar atención a la columna **Enviroment Variables** que en mi caso solo requiere la LINODE_TOKEN, y por ejemplo las de cloudflare son CF_API_EMAIL, CF_API_KEY

```nil
services:
  traefik:
    environment:
      - LINODE_TOKEN=AQUI PONDREMOS EL TOKEN QUE GENERAMOS ANTES
```


### Ports {#ports}

Hacemos un mapeo de nuestros puertos 80 y 443 para que sean utilizados por los protocolos http y https respectivamente

```nil
    ports:
      - "80:80"
      - "443:443"
```


### Labels {#labels}


#### traefik.enable=true {#traefik-dot-enable-true}

Le indicamos a traefik que este contenedor hará uso de traefik.

¿Cual es el motivo de que traefik utilice traefik? esto es debido a que como tal quien hara uso de traefik es realmente el dashboard de traefik, el cual hemos activado con motivo de tener una mejor visualización de si nuestros servicios están corriendo de buena forma.


#### traefik.http.middleware {#traefik-dot-http-dot-middleware}

<!--list-separator-->

-  https-redirect.redirectscheme.scheme=https

    Le pedimos a traefik que haga un redirect de las request al esquema https

<!--list-separator-->

-  sslheader.headers.customrequestheaders.X-Forwarded-Proto=https

    CustomRequestHeaders revela una lista de opciones para aplicar a una request, en este caso agregaremos la X-Forwarded-Proto, este nos ayudará a identificar entre http y https  [X-Forwarded-Proto - HTTP | MDN](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/X-Forwarded-Proto)

<!--list-separator-->

-  auth.basicauth.users=user:\\[apr1\\]ZCdpObME$$JUZ7NMS93R/k54WEYpek80

    Traefik nos brinda mecanismos para solicitar un usuario y contraseña utilizando la api nativas de los navegadores, y este nos será útil para tener un control de acceso al dashboard de traefik,

    Lo siguiente es el equivalente a tener como usuario **user** y tener como contraseña **password**

    ```bash
    user:$$apr1$$ZCdpObME$$JUZ7NMS93R/k54WEYpek80
    ```

    Para generar un sistema de usuario contraseña para traefik requerimos utilizar el programa htpasswd del proyecto apache. Una vez instalado en nuestro PC podemos generar nuestro usuario y contraseña:

    ```bash
    echo $(htpasswd -nb user password) | sed -e s/\\$/\\$\\$/g
    ```

    sustituimos el **user** y **password** por el nombre que consideremos bueno

    Nota: es muy importante utilizar la sed como se muestra en el apartado anterior

    Es importante indicar que esta cadena es casi random su resultado, por lo que no esperes conseguir el mismo resultado que yo

    Ahora el resultado lo colocamos en nuestro contenedor y listo.

    Nota: Recomendaría considerar colocar este dato en una variable de entorno


#### traefik.http.routers {#traefik-dot-http-dot-routers}

<!--list-separator-->

-  traefik

    En este caso podemos hacer referencia a nuestro propio contenedor utilizando su propio indicado en el **container_name** de nuestro docker-compose.

    <!--list-separator-->

    -  entryPoints

        Este lo especificamos previamente en nuestro archivo **traefik.yml** y como se indica sera para atender las peticiones del puerto 80

    <!--list-separator-->

    -  rule

        ¿Que Host o Path atenderá este contenedor?, en este caso el dashboard sera desplegado en la pagina traefik.example.com, el cual debemos sustituir por uno que pertenezca a nuestro dominio.

        ```yaml
            labels:
              - "traefik.enable=true"


              - "traefik.http.routers.traefik.entrypoints=http"
              - "traefik.http.routers.traefik.rule=Host(`traefik.example.com`)"

              - "traefik.http.middlewares.https-redirect.redirectscheme.scheme=https"
              - "traefik.http.middlewares.sslheader.headers.customrequestheaders.X-Forwarded-Proto=https"

              - "traefik.http.routers.traefik.middlewares=https-redirect"

              - "traefik.http.routers.traefik-secure.entrypoints=https"
              - "traefik.http.routers.traefik-secure.rule=Host(`traefik.example.com`)"

              #user password
              - "traefik.http.middlewares.auth.basicauth.users=user:$$apr1$$ZCdpObME$$JUZ7NMS93R/k54WEYpek80"
              - "traefik.http.routers.traefik-secure.middlewares=auth"

              - "traefik.http.routers.traefik-secure.tls=true"
              - "traefik.http.routers.traefik-secure.tls.certresolver=letsencrypt"
              - "traefik.http.routers.traefik-secure.tls.domains[0].main=example.com"
              - "traefik.http.routers.traefik-secure.tls.domains[0].sans=*.example.com"

              - "traefik.http.routers.traefik-secure.service=api@internal"
        ```

    <!--list-separator-->

    -  middlewares

        Asignamos el middleware **https-redirect** que declaramos con anterioridad, este nos ayudara a redirigir las peticiones http al https

<!--list-separator-->

-  traefik-secure

    Con traefik podemos colocar el nombre de servicio que queramos en nuestro contenedor, no es necesario que sea el mismo que el **container_name** y utilizaremos esta característica para crear un nuevo servicio que administrara los entrypoints de tipo https.

    Algunos puntos son similares a los http por lo que omitiré su explicación

    <!--list-separator-->

    -  middlewares

        Agregamos el middleware auth a nuestro traefik seguro con motivo de que se le solicite el usuario y contraseña a los usuarios que quieran acceder al dashboard

    <!--list-separator-->

    -  tls

        Por ultimo indicamos con **true** que deseamos utilizar tls para esta conexión.

        <!--list-separator-->

        -  certresolver

            Es el nombre que indicamos en nuestro archivo **traefik.yml** en mi caso para identificarlo de mejor forma le llamé letsencrypt, pero repito puedes llamarle como gustes

        <!--list-separator-->

        -  domains[0].main

            Este sera nuestro dominio principal del cual también indicaremos los sub-dominios

        <!--list-separator-->

        -  domains[0].sans

            Los sub-dominios que deseamos incluir ssl, en mi caso deseo que todos los sub-dominios tengan ssl así que podemos usar el \* como comodín

    <!--list-separator-->

    -  service

        utilizaremos el servicio api declarado en **traefik.yml**

<!--list-separator-->

-  Puesta en marcha de traefik

    Tras lo anterior ahora ya podemos arrancar nuestro docker-compose

    Es recomendable hacer un simple **up** para visualizar si traefik tiene algun mensaje de error.

    ```nil
    docker-compose up
    ```

    Ahora debemos ser capaces de entrar a traefik.example.com
