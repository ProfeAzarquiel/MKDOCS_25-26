# UNIDAD 4: INTRODUCCIÓN A DOCKER

## CONCEPTO Y CARACTERÍSTICAS

### ¿QUE ES DOCKER?

* Tecnología de virtualización ligera de software libre, también conocida como virtualización por contenedores.
* Son entornos de ejecución aislados, con su propio sistema de ficheros, su propia configuración de red y que acceden directamente a los recursos del anfitrión (memoria, procesador...)
* No tienen kernel propio, utilizan el del anfitrión.
* Su uso principal es para el despliegue de aplicaciones.

![Logo Docker](https://upload.wikimedia.org/wikipedia/commons/thumb/4/4e/Docker_%28container_engine%29_logo.svg/1024px-Docker_%28container_engine%29_logo.svg.png)

### ELEMENTOS BÁSICOS DE DOCKER

* **IMAGEN**: aplicación empaquetada, es el sistema de ficheros de nuestro contenedor
* **CONTENEDOR**: una imagen en ejecución
* Los contenedores pueden conectarse a diferentes **REDES VIRTUALES** y tienen sus propios **VOLÚMENES** o sistemas de almacenamiento.

### ARQUITECTURA DE DOCKER

* **DEMONIO**: es el más importante (Docker Engine) estará siempre en ejecución y es el que gestiona los contenedores
* **CLIENTE**: linea de comandos que nos permite decirle al demonio que queremos hacer
* **REGISTRO**: repositorio de imágenes local (existe tambien uno público, Docker Hub, de donde podemos descargar)
* **DOCKER SWARN**: software orquestador de contenedores de Docker. Sirve para gestionar y automatizar la operativa con conjuntos de contenedores ejecutados en clusteres de servidores. El más conocido es **KUBERNETES**.
![Arquitectura Docker](https://miro.medium.com/v2/resize:fit:1400/1*09i6gCc0tBhSsXToKA7Cnw.png)

Algunas **alternativas a Docker** son:

* Podman (RedHat)
* Containerd (proyecto independiente basado en Docker)
* CRI-0 (RedHat, pensada para trabajar con Kubernetes)
* Pouch (AliBaba).

## INSTALACIÓN DE DOCKER

* [Documentación oficial](https://docs.docker.com/engine/install/)
* [Requerimientos mínimos Ubuntu](https://docs.docker.com/engine/install/ubuntu/)
* Docker solo corre sobre distribuciones Linux, para Windows y MAC, se puede usar instalando el software **Docker Desktop**, que en realidad crea una máquina virtual Linux para ejecutar el demonio Docker sobre ella. Sin embargo el cliente si corre sobre el SO anfitrión.

* Existen varios métodos de instalación, en nuestro caso usaremos los repositorios apt de de Docker:

    1. Actualizamos los repositorios e instalamos los paquetes necesarios para instalar desde repositorios HTTPS:

            sudo apt-get update
            sudo apt-get install ca-certificates curl gnupg

    2. Añadir la clave GPG oficial de Docker:

            sudo install -m 0755 -d /etc/apt/keyrings
            curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
            sudo chmod a+r /etc/apt/keyrings/docker.gpg

    3. Añadimos el repositorio oficial de Docker:

            echo \
            "deb [arch="$(dpkg --print-architecture)" signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
            "$(. /etc/os-release && echo "$VERSION_CODENAME")" stable" | \
            sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

    4. Instalamos Docker Engine:

            sudo apt-get update
            sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

* Para manejar Docker con un usuario sin privilegio, debemos añadir el usuario al grupo `docker`, para ello:

        sudo usermod -aG docker $USER

* Debes volver a iniciar una terminal con el usuario, o ejecutar el siguiente comando:

        su - $USER

* Y finalmente, comprobamos la versión con la que vamos a trabajar:

        docker --version

* Si queremos más información de las versiones de los componentes con los que vamos a trabajar, ejecutamos:

        docker version

* Para más información del sistema que hemos instalado podemos ejecutar:

        docker info

## EJECUCIÓN DE CONTENEDORES

* **HOLA MUNDO:**
* Todos los subcomandos referidos a contenedores, estan dentro del bloque "docker container", pero por abreviación, se omite "container" y solo se escribe "docker". Por ejemplo: **docker container run = docker run**

* Para ver todos los subcomandos con referidos a contenedores, podemos ejecutar:

        docker container

* Como mínimo, para crear un contenedor debemos indicarle una imagen. Esa imagen debe estar en nuestro REGISTRO (repositorio) local, si no es así, la buscara en el REGISTRO público (**Docker Hub**) y la descargará. Por ejemplo, existe la imagen pública "hello-world" que ejecuta un programa Hello World en el contenedor.

* Si no especificamos un nombre de contenedor al crearlo, la herramienta le asigna un nombre aleatorio.

* Todas las imágenes tienen definido un comando por defecto que ejecutarán al crear un contenedor con ellas. Por ejemplo, en el caso de las imagenes de sistemas operativos, como ubuntu, por defecto ejecutan el comando  **`bash`**.

* **COMANDOS DE GESTIÓN BÁSICA:**
* Crear y ejecutar un contenedor:

        docker run hello-world

* Crear un contenedor sin ejecutarlo:

```bash
docker create hello-world
```

* Iniciar un contenedor ya creado:

```bash
docker start -a micontenedor1
# La opción -a sirve para ver la salida que genera la ejecución de ese contenedor, en este caso el mensaje "Hello form Docker!"
```

* Pausar la ejecución de un contenedor:

```bash
docker pause micontenedor
```

* Finalizar la ejecución de un contenedor:

```bash
docker stop micontenedor
```

* Reiniciar la ejecución de un contenedor:

```bash
docker restart micontenedor
```

* Renombrar un contenedor ya creado:

```bash
docker rename micontenedor nuevo_nombre
```

* Borrar un contenedor:

```bash
    #Mediante su ID:
    docker rm 64f44087771a
    # Mediante su nombre
    docker rm micontenedor1

    # Para forzar el borrado de un contenedor en ejecución, se usaría la opción -f
    docker rm -f micontenedor1
```

* Enlazar con la entrada/salida de un contenedor interactivo en ejecución:

```bash
docker attach micontenedor
```

* Mostrar contenedores en ejecución:

```bash
docker ps
```

* Mostrar contenedores creados:

```bash
docker ps -a
```

* Mostrar imágenes de nuestro REGISTRO local:

```bash
docker images
```

* Descargar una imagen del REGISTRO público:

```bash
#Por ejemplo, descargar la imagen del SO Ubuntu
docker pull ubuntu
```

* Visualizar los pasos que se dan cuando creamos o ejecutamos contenedores:

```bash
#Dejamos lanzado este comando y se nos mostrarán logs detallados de todos las acciones que ejecutemos despues:
docker events
```

* **OPCIONES COMUNES SOBRE LOS COMANDOS BÁSICOS:**
* Indicar nombre del contenedor al crearlo:

```bash
docker run --name contenedor1 ubuntu
```

* Para especificar que comando o comandos queremos que ejecute nuestro contenedor, se indican despues de la imagen:

```bash
docker run --name contenedor1 ubuntu echo "Hola Mundo"
```

* Indicar hostname del contenedor al crearlo:

```bash
#Aunque no es muy común utilizarlo, ya que los contenedores ofrecen un servicio y por lo general no nos conectamos a ellos. Es decir, no suelen tratarse como máquinas virtuales tradicionales a las que solemos conectarnos para realizar distintas tareas
docker -h contenedor_ubuntu
```

* Crear una sesión de terminal interactivo al crear un contenedor:

```bash
docker run -it --name contenedor2 -h cont2 ubuntu bash
```

* Borrar el contenedor despues de que termine su ejecución:

```bash
docker run -it --rm --name contenedor3 -h cont3 ubuntu top
```

* **CONTENEDORES DEMONIO:** ejecutan procesos indefinidamente y de forma desatendida (no veremos su salida). Ejemplo:

```bash
# La opción -d indica que la ejecución es desatendida, es decir, no veremos su salida. Y la opción -c, permite especificar entre comillas, un listado de comandos para ejecutar:
docker run -d --name micontenedor ubuntu bash -c "while true; do echo hello world; sleep 1; done"

# Para ver la salida generada por un contenedor demonio
docker logs micontenedor

#Y si queremos verlo de forma continuada, podemos especificar
docker logs -f micontenedor

# Para detener un contenedor demonio
docker stop micontenedor

# Una vez detenido, podemos eliminarlo si queremos
docker rm micontenedor
```

* **CREACIÓN DE VARIABLES DE ENTORNO ASOCIADAS A UN CONTENEDOR:** Las variables de entorno son del tipo "clave:valor", y podemos crearlas al arrancar un contenedor. En el siguiente ejemplo, creamos la variable "USUARIO" y le asignamos el valor "prueba":

```bash
#Crea un contenedor interactivo (-it) con la imagen de ubuntu y define una variable de entorno (-e) para ese contenedor:
docker run -it --name micontenedor -e USUARIO=prueba ubuntu
```

* Para comprobar que realmente se ha creado la variable, dentro del BASH del contenedor escribimos **`env`** y veremos todas las variables d entorno.
* Las variables de entorno, suele utilizarse mucho para definir ciertos parámetros de configuración de las aplicaciones que se desplegamos mediante contenedores.

* **COMANDOS DE GESTIÓN MÁS AVANZADOS:**
Ejecutar un comando dentro de un contenedor en ejecución:

```bash
docker  exec micontenedor ls
#o
docker exec micontenedor cat datos.txt
```

El comando anterior es muy utilizado para levantar sesiones interactivas de consola en contenedores ya creados, por ejemplo:

```bash
# Abrir una sesión interactiva en un contenedor MariaDB existente, ejecutando bash y pasándole usuario y contraseña para autenticarse
docker exec -it some-mariadb bash -c 'mysql -u root -p$MARIADB_ROOT_PASSWORD'
```

Copiar ficheros del Host al Contenedor o viceversa:

```bash
#Para copiar del host al contenedor, se especifica el nombre del contenedor y el directorio donde lo queremos copiar
docker cp mifichero.txt micontenedor:/

#Para copiar del contenedor al host, debemos especificar el directorio destino, en este caso "." para indicar el directorio actual
docker cp micontenedor:/hora.txt .
```

Visualizar los procesos que se estan ejecutando dentro de un contenedor:

```bash
docker top micontenedor
```

Obtener información detallada de un contenedor:

```bash
# Muestra en formato JSON el identificador, puertos abiertos, almacenamiento, tamaño, configuración de red, comandos, varibles de entorno... 
docker inspect micontenedor
```

Podemos formatear la salida del comando anterior, para que solo muestre ciertos parámetros:

```bash
# Para mostrar el ID del contenedor
docker inspect --format='{{.Id}}' micontenedor

#Para mostrar todas las variables de entorno:
docker inspect --format='{{range .Config.Env}}{{println .}}{{end}}' micontenedor

#Para mostrar la dirección o direcciones IPs que tiene el contenedor:
docker inspect --format='{{range .NetworkSettings.Networks}}{{IPAddress}}{{end}}' micontenedor
```

Etiquetar contenedores, para filtrarlos o agruparlos

```bash
# USO ETIQUETADO

```

Limitar recursos utilizados por contenedores:

```bash
# La cantidad de CPU --cpus

# La cantidad de memoria --memory
```

Visualizar estadísticas de uso de recursos de un contenedor

```bash
# ESTADISTICAS

```

Modificar en tiempo real las limitaciones de recursos de un contenedor en ejecución

```bash
# MODIFICACIÓN LIMITACIONES

```

### DESPLEGAR UN SERVIDOR WEB APACHE CON DOCKER

Generalmente, nunca se va a utilizar la dirección IP del contenedor para acceder a los servicios que despliega, ya que esta es dinámica y puede ir cambiando dependiendo de que acciones realicemos en el contenedor.

Además, la dirección IP de un contenedor solamente es accesible desde el anfitrión. No sería alcanzable desde otros equipos de la red.

Por lo tanto, en este caso para acceder al servidor Web que nos proporcionará este contenedor, vamos a `mapear puertos`. Es decir, con la opción `-p` indicamos que cuando accedamos al puerto 8080 del host, esa petición se reenvíe al puerto 80 del contenedor, independientemente de la dirección IP.

Sería como si definieramos una regla de NAT en el Firewall del anfitrión.

```bash
# httpd:2.4 es la imagen oficial del Servidor Web APACHE
docker run -d --name my-apache-app -p 8080:80 httpd:2.4
```

Para ver las redirecciones o nateos que se están haciendo en nuestro contenedor:

```bash
# Muestra tanto IPv4 como IPv6
docker port my-apache-app
```

Para conectarnos al servidor web bastará con ir al navegador del host y acceder a `https://localhost:8080`

### DESPLEGAR UN SERVIDOR DE BBDD CON DOCKER

## GESTIÓN DE IMAGENES EN DOCKER

Las imágenes son plantillas de solo lectura que define el sistema de ficheros que va a tener el contenedor. También establece el comando por defecto que ejecutará ese contenedor, siempre que no se indique otro.

Por defecto en docker se usa el registro público de Docker Hub para descargar imágenes a nuestro registro local.

La nomenclatura para identificar una imagen es `usuario/nombre:etiqueta`. Las imágenes oficiales de docker vienen sin nombre de usuario para diferenciarlas.

Si no idicamos etiqueta al usar una imagen, por defecto se utiliza la etiqueta "latest" que se refiere a la última versión disponible de esa imagen.

Además de la version, las etiquetas pueden indicar la tecnología interna que se usa para servir esa aplicación y el SO base que usa esa imagen.

* **¿Como se organizan las imágenes?**
El sistema de ficheros (FS) de una imágen se llama sistema de ficheros de unión, ya que es el resultado de la unión de varias capas.

Cada capa representa una serie de directorios, que al juntarlos todos obtenemos nuestro FS final.

Las imagenes suelen partir de una capa base, que contiene los directorios y archivos básicos para su funcionamiento. Y luego cada una de las otras capas define un conjunto de diferencias respecto a la capa anterior (ficheros añadidos, modificados o eliminados).

![Formato capas Docker](https://miro.medium.com/v2/resize:fit:1138/1*B5qyIxFVIDvFOcx7t1ogsQ.png)

Si tengo varias imágenes que tengan la misma capa base, solo se almacenará una vez en nuestra máquina y será compartida por todas esas imágenes. Esto podemos verlo con el comando:

```bash
# Nos da la información del almacenamiento utilizado por docker
docker system df -v
```

Esto por ejemplo tambien ocurre cuando creamos contenedores con diferentes versiones de una misma imagen, muchas de sus capas, son compartidas.

Cuando creamos un contenedor, usa el FS de la imagen en `solo lectura`. Por tanto, cuando instalamos o modificamos algo en un contenedor, esto se hace en una nueva `capa de lectura/escritura` que se genera al crearlo, para así no modificar las capas de la imagen. De este modo, si creamos varios contenedores a partir de la misma imagen, todos ellos compartirar el mismo FS.

Esto hace que al crear un contenedor, su tamaño sea mínimo Esto podemos verlo con el comando:

```bash
# La opción -s nos muestra el tamaño del contenedor
# indicando el tamaño de la capa R/W del contenedor y
# el tamaño (virtual) del FS de la imagen usada
docker ps -a -s
```

Cuando borramos un contenedor, esta capa R/W desaparece, se perderían los datos de ese contenedor.

### COMANDOS DE GESTIÓN DE IMÁGENES

* Buscar imágnes en Docker Hub:

```bash
docker search nginx

# Si queremos ver los posibles filtros
docker search --help
```

* Para borrar una imagen que ya no vayamos a usar:

```bash
docker rmi hello-world
```

!!! warning "ATENCIÓN:"
    No podremos borrar una imagen de nuestro repositorio local, mientras exista algún contenedor creado a partir de ella.

* Ver la información detallada de una imagen, como su identificador, puertos que utiliza, arquitectura, SO, variables de entorno, el comando por defecto, las distintas capas...

```bash
docker inspect nginx:stable
```

### DOCKER HUB

[Enlace Docker Hub](https://hub.docker.com/)

Su filosofía es bastante parecida a GitHub, ya que denominan a las imágenes como repositorios y usan practicamente los mismos comandos que este (pull, push, commit...)

Existe contenido confiable marcado con 3 tipos de insignias, como `imagen oficial de docker` (las mantiene el equipo de docker), `proveedores verificados` (suelen ser fabricantes reconocidos) y `proyectos de código libre esponsorizados`.

Podemos entrar a un repositorio par ver toda su información, etiquetas, versiones, como utilizarlo... Hay imagenes de SO, de servicios (apache, maraidb...), de lenguajes de programación o aplicaciones comerciales (Wordpress...)

Es imporante tener en cuenta la arquitectura en la que estamos trabajando, ya que dependiendo de eso deberemos buscar una imagen u otra.

## ALMACENAMIENTO DE DATOS EN DOCKER

Para no perder la información generada por los contenedores una vez sean borrados (por defecto se guarda en la capa R/W del contenedor), existen formas de almacenarla en el equipo anfitrión.

Un **VOLUMEN** es un directorio creado y gestionados únicamente por Docker, que se va a montar en un directorio interno del contenedor. Es decir, cuando guardemos información en ese directorio montado, se estaremos escribiendo en el directorio de nuestro Host que corresponde a ese volumen. Es decir, esa información no se perdería al borrar el contenedor.

El directorio del contenedor donde se montan es `/var/lib/docker/volumes/`.

Los volúmenes se pueden montar simultaneamente en diferentes contenedores (almacenamiento compartido). Es decir, lo que escribe un contenedor, puede ser leido más tarde por otro contenedor.

Como la información persiste al borrar el contenedor, podemos usar esa información para que sea utilizada por otro contenedor que creemos en el futuro. Además, gracias a esto son muy útiles para realizar copias de seguridad o migrar datos.

Hay dos posibles formas de asociar un volumen a un contenedor, ambas hacen exactamente lo mismo pero tienen diferente nomenclatura:

=== "Volumen con `-v`"

    ```bash
        -v miweb:/usr/local/apache2/htdocs:ro
    ```
=== "Volumen con `--mount`"

    ```bash
        --mount type=volume,src=miweb,dst=/usr/local/apache2/htdocs,ro
    ```

Otra opción es **BIND MOUNT**, que es un directorio o fichero propiedad del usuario, que podemos montar en un directorio interno del contenedor.

Se referencian por su ruta completa, aunque podemos crear un punto de montaje para más comodidad.

En este caso, no se gestionan desde el cliente Docker. Y se puede cambiar su contenido desde el anfitrión.

Son útiles cuando queremos compartir archivos de configuración o código fuente en nuestro contenedor. Y al igual que pasaba con los volúmenes, podemos asociarlos a un contenedor de dos formas distintas:

=== "Bind Mount con `-v`"

    ```bash
        -v /opt/web:/usr/local/apache2/htdocs:ro
    ```

=== "Bind Mount con `--mount`"

    ```bash
        --mount type=bind,src=/opt/web,dst=/usr/local/apache2/htdocs,ro
    ```

![Volumenes y BindMount Docker](https://iamachs.com/images/posts/docker/part-5-understanding-docker-storage-and-volumes/docker-storage.png){width="350"}

* **Ejercicio con Volúmenes:**
Todos los subcomandos de acciones con Volúmenes, estan bajo el bloque `docker volume`. Por ejemplo:

```bash
# Para crear un volumen:
docker volume create mivolumen

# Para borrar un volumen:
docker volume rm mivolumen

# Mostrar los volumenes creados:
docker volume ls

# Mostrar información detallada de un volumen que hayamos creado:
docker volume inspect mivolumen
```

Para asociar un contenedor a un volumen que hayamos creado, un comando de ejemplo sería el siguiente. Aunque igualmente, si indicamos un volumen que no se haya creado previamente, docker lo creará en este momento:

```bash

```

Podemos comprobar el acceso al servidor web usando el navegador web o desde la consola, usando el comando `curl`:

```bash
curl http://localhost:8080
```

* **Ejercicios con Bind Mount:**

En este caso, no necesitamos crear volúmenes, si no que vamos a crear un directorio en el sistema de archivos del Host, donde añadiremos nuestro fichero `index.html`. Para ello, desde la consola del host:

```bash
mkdir miweb
cd miweb
echo "<h1>Hola Mundo</h1>" > index.html
```

A continuación, ya podemos montar ese directorio en un contenedor:

```bash
docker run -d --name my-apache-app --mount type=bind,src=/home/usuario/miweb,dst=/usr/local/apache2/htdocs -p 8080:80 httpd:2.4

# O si lo preferimos, sería lo mismo que:
docker run -d --name my-apache-app -v /home/usuario/miweb:/usr/local/apache2/htdocs -p 8080:80 httpd:2.4
```

En este caso, con bind mount, puedo modificar mi fichero HTML desde el anfitrión y al estar montado, esos cambios se verán directamente reflejados en mi servidor web.

### DESPLEGAR EL SERVICIO NEXTCLOUD EN UN CONTENEDOR CON ALMACENAMIENTO PERSISTENTE

Lo primero es ir a la página de documentación de la imagen de Nextcloud en [Docker Hub](https://hub.docker.com/_/nextcloud).

Allí buscamos el apartado `Persistent data` para saber cual es el directorio que debo montar en un volumen para no perder los datos que nos interesan generados por esta aplicación. En este caso sería `/var/www/html`.

Por lo tanto, si usamos volumenes el comando quedaría algo así:

```bash
docker run -d -p 8080:80 -v nextcloud:/var/www/html --name contenedor_nextcloud nextcloud:28.0.1
```

### DESPLEGAR EL SERVICIO MARIADB EN UN CONTENEDOR CON ALMACENAMIENTO PERSISTENTE

De nuevo, vamos a la página de documentación de la imagen que vamos a usar en [Docker Hub](https://hub.docker.com/_/mariadb).

En el caso de MariaDB, la información sobre alamcenamiento persistente, aparece bajo el título `Where to Store Data`. Ahí podemos leer que el directorio a montar es `/var/lib/mysql`

En este caso vamos a hacerlo con Bind Mount, y quedaría así:

```bash
docker run --name some-mariadb -v /opt/mariadb:/var/lib/mysql -e MARIADB_ROOT_PASSWORD=my-secret-pw -d mariadb:10.5
```

Si el directorio `/opt/mariadb` no existe en nuestro host, con la opción -v se creará en ese momento.

### OTROS USOS DEL ALMACENAMIENTO PERSISTENTE

- Almacenamiento compartido entre distintos contenedores.

* Por ejemplo, tener un servidor web en un contenedor, cuyo `index.html` deba ser actualizarse automáticamente leyendo de un repositorio GitHub o similar. Podemos tener un segundo contenedor, que modifique el código de ese HTML.

* Otro ejemplo sería probar como funciona un mismo código en diferentes versiones del lenguaje de programación (vease el caso de PHP que hay isntrucciones que cambian su comportamiento entre la versión 5 y la 7, o entre la 7 y la 8).

## REDES EN DOCKER

Existen **3 tipos** principales de redes en Docker:

1. **Bridge:** es la que usaremos generalmente, genera una red privada virtual, asignando direccionamiento IP propio al contenedor y proporciona reclas SNAT (salida a internet) y DNAT (acceso desde el exterior al contenedor).
2. **Host:** el contenedor no tendría dirección IP propia, si no que se ejecuta como un proceso más en el Host, saliendo con la IP de este por el puerto que le indiquemos.
3. **None:** para contenedores que no queremos que tengan conexión a internet.

Hay que diferenciar entre la red Bridge que crea Docker por defecto y las redes Bridge que puede crear el usuario. La **red por defecto** se llama **bridge** y va a crear un switch virtual (Linux Bridge) llamado **docker0** con el direccionamiento **172.17.0.0/16** a la que estarán conectados todos los contenedores que creemos (mientras no indiquemos lo contrario). Al anfitrión se le asigna la **172.17.0.1**, que será la puerta de enlace de nuestros contenedores.

![Esquema Bridge Docker](https://godleon.github.io/blog/images/docker/docker-bridge-network-custom.png){width="600"}

Sin embargo, cuando creamos una **red definida por el usuario**, podemos personalizar que contenedores que contenedores queremos que se conecten a esa red, además tendremos un servicio de DNS que permitirá la interconexión de contenedores mediante sus nombres.
Tambien podemos conectar y desconectar contenedores en caliente a redes definidas por el usuario.

Los subcomandos para gestionar las redes se engloban bajo el comando `docker network`.

---

* **EJERCICIO CONEXION RED HOST:**

> Listamos las redes disponibles en docker, y vemos que por defecto están los tres tipos que hemos indicado más arriba.
>
>Luego vamos a crear un contenedor con un servidor web NGINX usando la red Host. De modo que si accedemos directamente al localhost, podremos ver nuestro servidor, ya que el contenedor no tiene IP propia, si no que usa la del HOST ejecutandose como un proceso más sobre un puerto específico:
>
```bash
docker network ls

docker run -d --network host --name myNGINX nginx

curl http://localhost
```

---

* **EJERCICIO CONEXION RED BRIDGE POR DEFECTO:**

>En la siguiente práctica creamos un contenedor interactivo con la distribución linux Alpine mapeando el puerto 8080 del host al 80 del contenedor.
>
>Luego comprobamos en el terminal del contenedor la dirección ip asignada, la puerta de enlace y las entradas del servicio DNS.
>
>Luego en el terminal del host, podemos comprobar que ha creado una conexión con la IP que usará el contenedor como Gateway.

```bash
docker run -it -p 8080:80 --name CONT1 alpine ash

/$ ip a
/$ ip r
/$ cat /etc/resolv.conf

ip a    #En el anfitrión
```
>
>Aquí podemos instalar un servidor apache dentro de Alpine y levantar el servicio. Luego desde el anfitrión accedemos al localhost con el puerto 8080 y vemos que nos muestra nuestro servidor.
>
>Tambien desde el host, podemos comprobar las reglas iptables que se han creado para gestionar la comunicación de nuestros contenedores (bajo el nombre MASQUERADE y DNAT).

```bash
/$ apk add apache2
/$ http -D foreground

curl http://localhost:8080      #En el anfitrión
sudo iptables -L -n -t nat      #En el anfitrión
```

---

* **EJERCICIO CONEXION RED DEFINIDA POR EL USUARIO:**

>Para este ejercicio creamos primero una red definida por el usuario, sin indicar ningun parámetro, para ver que valores le da docker por defecto. Que como vemos, será el siguiente valor a la 17, que es el bridge por defecto.
>
>Listamos nuestras redes en docker, para ver que la ha creado y luego consultamos la información detallada de la red que acabamos de crear.
>
>Tambien podemos comprobar en el HOST que se ha creado un nuevo interfaz de red que es el que hará la función de gateway para esta red.
>
>Por último, borramos esta red de prueba, ya que generalmente especificaremos el direcciónamiento que queremos.

```bash
docker network create red1

docker network ls

docker network inspect red1

ip a

docker network rm red1
```
>
> Ahora, creamos una segunda red pero personalizando los valores que queramos, y luego comprobaremos que se ha creado correctamente:
>
```bash
docker network create --subnet 192.168.0.0/24 --gateway 192.168.0.1 red2

docker network ls

docker network inspect red2

ip a
```

* **PRÁCTICA: CONTENEDOR WORDPRESS CONECTACO CON CONTENEDOR MARIADB:**
>
>Lo primero es revisar la documentación de las imágenes de [`mariadb`](https://hub.docker.com/_/mariadb) y de [`wordpress`](https://hub.docker.com/_/wordpress) en `Docker Hub`, para saber en que ruta debemos montar los volúmenes y que variables de entorno necesitamos crear para que se comuniquen. Nos quedará algo así:

```bash
docker network create --subnet 192.168.1.0/24 --gateway 192.168.1.1 red_wp

docker run -d --name server_mariadb --network red_wp -v vol_mariadb:/var/lib/mysql -e MARIADB_DATABASE=bd_wp -e MARIADB_USER=user_wp -e MARIADB_PASSWORD=user123 -e MARIADB_ROOT_PASSWORD=admin123 mariadb
                
docker run -d --name server_wp --network red_wp -v vol_wordpress:/var/www/html/ -e WORDPRESS_DB_HOST=server_mariadb -e WORDPRESS_DB_USER=user_wp -e WORDPRESS_DB_PASSWORD=user123 -e WORDPRESS_DB_NAME=bd_wp -p 8085:80 wordpress
```

> Si te das cuenta la **variable de entorno** `WORDPRESS_DB_HOST` la hemos inicializado al nombre del servidor de base de datos. Como están conectada a la misma red bridge definida por el usuario, el contenedor WordPress al intentar acceder al nombre `servidor_mariadb` estará accediendo al contenedor de la base de datos.
>
>Como vamos a **acceder desde el exterior** al servidor web, por lo que hemos mapeado los puertos con la opción `-p`. Sin embargo, en el contenedor de la base de datos no es necesario mapear los puertos porque no vamos a acceder desde el exterior a él. Pero el contenedor `servidor_wp` si que podrá acceder al puerto 3306/tcp del `servidor_mariadb` sin problemas al estar conectados a la misma red.
>
>Nos registramos en Wordpress con un correo ficticio y creamos algún post. Luego ejecutamos los siguientes comandos para comprobar que la información se ha guardado en la BBDD:

```bash
docker exec -it server_mariadb bash

mariadb -u root -p admin123

SHOW TABLES;
SELEC * FROM wp_posts
```

## ESCENARIOS MULTICONTENEDOR

Cuando queremos levantar aplicaciones o servicios compuestos por varios microservicios (contenedores), normalmente se hace uso de **DOCKER COMPOSE**.

Compose es una especificación de Docker que permite declrar en un único fichero con formato **.YAML** o **.YML**, todas las características del escenario que queremos desplegar; como contenedores, volúmenes, redes, orden de arranque, variables de entorno, etc.

Actualmente, Docker Compose V2 ya se encuentra integrado en el cliente Docker, por lo que podemos ejecutarlo con `docker compose`

### EL FICHERO DOCKER-COMPOSE.YAML

En él definimos un escenario compuesto por varios servicios (`services`). Y dentro de cada servicio, especificamos todos los detalles que van a definir ese servicio.

Los parámetros `version` y `name` son opcionales. El primero es meramente informativo y si no especificamos nombre del escenario, cogerá el del directorio en el que se encuentra nuestro fichero .YAML.

Características de un servicio:

* Cada servicio está representado por un nombre de servicio y dentro de él cuelgan todas sus características.
* Nombre del contenedor asociado `container_name`
* Imagen usada `image`
* Política de reinicio cuando hay un fallo o error `restart`
* Comando a ejecutar cuando creamos el contenedor `command`
* Variables de entorno `environment`
* Dependencias de arranque y apagado con otros servicios `depends_on`
* Mapeo de puertos `ports`
* Volúmenes de almacenamiento `volumes`
* Configuración de red `networks`

Por defecto, al ejecutar un escenario compose se crear una red definida por el usuario, que permite la conexión entre todos los contenedores que se han generado,aceptando conexiones por nombre de contenedor o por nombre de servicio. Es por esto, que generalmente no suelen definirse las redes en los ficheros .YAML:

Para arrancar un escenario, vamos al directorio donde está nuestro `docker-compose.yaml` y ejecutamos **docker compose up -d**.

Para eliminar un escenario, sería con el comando **docker compose down -v**. ¡OJO! Esto eliminará tambien los volumenes asociados al escenario, si quisieramos mantenerlos, podemos quitar la opción `-v`.

### USO DE PARÁMETROS EN DOCKER COMPOSE

Podemos crear plantillas para el despliegue de aplicaciones o servicios mediante Docker Compose, y arrancarlas con diferentes valores, según la finalidad o necesidades que tengamos en los distintos entornos en que podemos ejecutar nuestro escenario.

Para ello, en lugar de dar valores estáticos a las características definidas en el fichero .YAML, podemos usar variables usando la nomenclatura `${NOMBRE_VARIABLE}`. Por ejemplo:

```yaml
version: '3.1'
services:
db:
    container_name: servidor_mysql
    image: mariadb:${VERSION_MDB}
    restart: always
    environment:
      MARIADB_DATABASE: ${BASEDEDATOS}
      MARIADB_USER: ${USUARIO}
      MARIADB_PASSWORD: ${PASS}
      MARIADB_ROOT_PASSWORD: ${PASS_ROOT}
    volumes:
      - mariadb_data:/var/lib/mysql
volumes:
    mariadb_data:
```

Los valores de esas variables se definen en un fichero aparte de tipo `CLAVE=valor`, que por convenio podemos nombrar como `env_nombre_entorno`. Por ejemplo:

```bash
VERSION_MDB=latest
PUERTO=8080
USUARIO="prueba"
PASS="asdasd"
PASS_ROOT="asdasd"
BASEDEDATOS="wordpress"
```

Es decir, en el directorio donde se encuantre nuestro fichero .YAML crearemos tantos ficheros como posibles entornos de ejecución hayamos previsto. Por ejemplo, "*env_desarrollo*" y "*env_produccion*".

Cuando queramos que nuestra plantilla arranque con los valores de un entorno específico, simplemente tendremos que renombrar ese fichero como **`.env`**, por ejemplo: *`mv env_desarrollo .env`*; y luego arrancar el escenario normalmente con *`docker compose up -d`*.

Podemos comprobar el valor de las variables de entorno con las que ha arrancado nuestro contenedor con:

```bash
docker compose exec db env
```

!!! warning "IMPORTANTE"
    No olvidar que cuando queramos cambiar de entorno primero debemos deshacer el renombrado anterior con *`mv .env env_desarrollo`*; y luego renombrar el entorno que queremos arrancar ahora *`mv env_produccion .env`*.

### DESPLIEGUE REAL CON DOCKER COMPOSE

Vamos a ver un ejemplo práctico real de Docker Compose, donde desplegaremos una aplicación de código abierto para gestión de reuniones online, similar al servicio Google Meet.

Si accedemos al repositorio oficial de Jitsi en Github [Jitsi Meet](https://github.com/jitsi/docker-jitsi-meet), tenemos toda la documentación necesaria para desplegar la aplicación con Docker, incluyendo el fichero `docker-compse.yaml`.

Si seguimos las intrucciones ahí indicadas:

1. Descargamos los ficheros del repositorio, lo descomprimimos, accedemos al directorio y vemos lo que contiene:

```bash

wget $(curl -s https://api.github.com/repos/jitsi/docker-jitsi-meet/releases/latest | grep 'zip' | cut -d\" -f4)
unzip stable-9258
cd jitsi-docker-jitsi-meet-c92026a/
ls
```

2. Utilizamos el fichero de variables globales para configurar la aplicación y ejecutamos el script para generar las contraseñas:

```bash
cp env.example .env
 $ ./gen-passwords.sh
```

3. Luego dentro del fichero **.env**, descomentamos el parámetro `PUBLIC_URL` y modificamos su valor con la dirección que queramos usar:

```bash
nano .env
PUBLIC_URL=https://localhost:8443
```

4. Creamos los directorios donde vamos a guardar la información de la aplicación (Bind Mount en este caso):

```bash
mkdir -p ~/.jitsi-meet-cfg/{web,transcripts,prosody/config,prosody/prosody-plugins-custom,jicofo,jvb,jigasi,jibri}
```

5. Por último, levantamos el escenario y accedemos a la URL desde un navegador web:

```bash
docker compose up -d
```

![Interfaz Web Jitsi](https://plataforma.josedomingo.org/pledin/cursos/docker2024/contenido/modulo6/img/jitsi.png){width="700"}

Si revisamos los contenedores que se han creado, veremos que Jitsi esta compuesto por 4 servicios:
Se crean 4 contenedores que corresponde a cuatro componentes de Jitsi:

* **web**: Jitsi Meet web UI, aplicación web servida por nginx.
* **prosody**: Prosody, servidor XMPP.
* **jicofo**: [Jicofo], el componente JItsi COnference FOcus.
* **jvb**: Jitsi Videobridge, el enrutador de vídeo.

## CREACIÓN DE IMÁGENES EN DOCKER

Hay dos mecánismos básicos de construcción:

* **A partir de un contenedor** con `docker commit`
* **Automatizar su construcción** (recomendada), usando un fichero `Dockerfile` y el comando `docker build`.

El **Dockerfile** es como una receta que contiene todos los comandos para construir la imágen. Además, se puede distribuir, versionar y cambiar la imagen base de manera sencilla.

Tambien hay dos posibles formas de distribución de nuestras imágenes:

* **En un fichero**: `docker load` / `docker save`
* **En un registro** (recomendada): `docker push` / `docker pull`

![Creación de Imágenes](https://plataforma.josedomingo.org/pledin/cursos/docker2024/contenido/modulo7/img/build.png){width="500"}

> **EJERCICIO PRÁCTICO**: Creación de una imágen a partir de un contenedor.
>Vamos a crear un contenedor a partir de una imagen base del SO Debian.

```bash
docker  run -it --name contenedor1 debian 
```

>Realizamos las modificaciones que queramos sobre el contenedor, por ejemplo, actualizar, instalar un servidor web y modificar el fichero *index.html*:

```bash
apt update
apt install apache2 -y
echo "<h1>Curso Docker</h1>" > /var/www/html/index.html
exit
```

>Creamos una nueva imagen partiendo de ese contenedor usando `docker commit`.

```bash
 docker commit --change='CMD apachectl -D FOREGROUND' contenedor1 josedom24/myapache2:v1
```

>Si quiero subir mi imagen a **Docker Hub**, es muy importante ponerle delante nuestro nombre de usuario de Docker Hub, si no, no me dejará. El formato sería *nombre_usuario*/*nombre_imagen*:*versión* (si no indicamos versión, asignará la etiqueta *latest*)
>
>El contenedor creado, va a ejecutar el comando por defecto de la imagen base, en este caso Debian, al ser un SO ejecuta `bash` por defecto. Pero podemos personalizarlos con la opción `--change` y la instruccion `CMD`. Por ejemplo, en este caso queremos que ejecute el servidor web por defecto, y e comando sería `apachectl -D FOREGROUND`.

### EL FICHERO DOCKERFILE

Esta compuesto por una serie de instrucciones que se ejecutan de forma secuencial para crear de manera automatizada imágenes Docker.

La primera linea de este fichero es `# syntax=docker/dockerfile:1`.

Hay multitud de instrucciones que pueden en un DockerFile, las más utilizadas son:

* **FROM**: especifica la imagen base.
* **MAINTAINER**: indica los datos del autor de la imagen.
* **LABEL**: añade metadatos a la imagen del tipo `clave:valor`.
* **RUN**: ejecuta un comando que modifica el sistema de ficheros de la imagen base.
* **COPY**: copia ficheros desde mi equipo a la imagen, estos deben estar en la misma carpeta que el *Dockerfile*.
* **ADD**: similar a "COPY", pero permite descargar archivos desde una URL o extraer automáticamente archivos `tar.gz` en el destino especificado.
* **EXPOSE**: indica los puertos abiertos que usará esta imagen.
* **CMD**: establede el comando por defecto que se ejecuta al crear un contenedor con esta imagen.
* **WORKDIR**: establece el directorio de trabajo dentro de la imagen. Las siguientes instrucciones se ejecutarán en esa ruta.
* **ENV**: establece las variables de entorno que se podrán usar dentro del contenedor.
* **ARG**: define parámetros que se pueden personalizar a la hora de hacer el `docker build`.
* **ENTRYPOINT**: prácticamente igual que CMD, pero no puede modificarse al crear un contenedor.
* **VOLUME**: crea un punto de montaje en el directorio especificado y lo marca como volumen.

Los comandos RUN, COPY y ADD, **crean nuevas capas** en la imagen, ya que modifican el sistema de ficheros de la imagen base.

Un ejemplo de DockerFile sería:

        FROM debian:stable-slim
        RUN apt-get update  && apt-get install -y  apache2 
        WORKDIR /var/www/html
        COPY index.html .
        CMD apache2ctl -D FOREGROUND

Ejecutando el comando **`docker build`** en el mismo directorio del Dockerfile, crearemos la imagen definida en este. Por ejemplo:

        docker build -t profe_web/milinux:v1 .

!!! tip "IMPORTANTE"

    Nunca usar la etiqueta **latest** para crear imágenes, ya que esta apunta a la última versión de una imagen. Lo ideal es indicar versiones específicas de nuestra imagen.

Cuando usamos el comando *docker build* para construir imágenes, es común que aparezcan en nuestro repositorio local imágenes con nombre y etiqueta `<none>`. Son registros intermedios que se generan y que podriamos borrar mediante el comando `docker image prune`.

!!! danger "IMPORTANTE"
    La opción **PRUNE** sirve para eliminar objetos docker de forma másiva, ya sean imágenes, contenedores, redes, volúmenes... Es por ello, que debemos usarla con cautela para no borrar cosas que no queramos.

### DISTRIBUCIÓN DE IMÁGENES DOCKER

El comando `docker save` nos permite coger una imagen de nuestro repositorio local y comprimirla en un fichero `.TAR`. Y del mismo modo, con `docker load -i` puedo recuperar la imagen comprimida en ese *.TAR*.

No obstante, no es la práctica más recomendada para distribuir imágenes. Ya que lo ideal es subirla al registro público de **Docker Hub** (es necesario estar registrado).

Desde el terminal podemos hacerlo con:

        docker login
        docker push profe_web/milinux:v1

Una vez subidas, puedo acceder a ellas desde cualquier sitio mediante `docker pull`.

### PARAMETRIZAR EL FICHERO DOCKERFILE

Podemos crear variables dentro del Dockerfile, para que a la hora de crear las imágenes el usuario pueda indicar valores específicos para ellas. Para ello usamos el comando `ARG` y un ejemplo sería el siguiente:

        ARG PHP_VERSION=8.2-apache
        FROM php:${PHP_VERSION}
        ARG APP_VERSION=desarrollo
        ENV VERSION=${APP_VERSION}
        COPY app /var/www/html/
        EXPOSE 80

De este modo, si al crear la imagen no se especifica nada, las variables cogerán el valor por defecto especificado dentro del comando *ARG*:

    docker build  -t josedom24/app_php:v1 .

Y en caso de que el usuario quiera personalizar sus valores, podría indicarlo en la cosntrucción de la siguiente manera:

    docker build --build-arg PHP_VERSION=7.4-apache --build-arg APP_VERSION=produccion -t josedom24/app_php:v2 .

### DOCKERIZAR APLICACIONES PYTHON

En este ejemplo tenemos una aplicación web desarrollada en Python mediante el módulo Flask. Que va a servir el contenido por el puerto 3000.

        from flask import Flask, render_template, abort, redirect, request
        import datetime
        import socket

        app = Flask(__name__)

        @app.route('/',methods=["GET","POST"])
        def inicio():
            hoy=datetime.datetime.now()
            return render_template("inicio.html",hoy=hoy,server=socket.gethostname())
        
        if __name__ == '__main__':
            app.run('0.0.0.0',3000,debug=True)

Un posible **dockerfile** para crear la imagen de mi aplicación web, sería el siguiente:

        FROM debian:12
        RUN apt-get update && apt-get install -y python3-pip  && apt-get clean && rm -rf /var/lib/apt/lists/*
        WORKDIR /usr/share/app
        COPY app .
        RUN pip3 install --no-cache-dir --break-system-packages -r requirements.txt
        EXPOSE 3000
        CMD python3 app.py

Para crear un contenedor a partir de la imagen que hemos creado:

        docker run -d -p 80:3000 --name appweb profe_web/miappweb:v1

Otra opción, sería usar una imagen base que ya contenga el paquete PIP, y el *dockerfile* quedaría así:

        FROM python:3.12.1-bookworm
        WORKDIR /usr/share/app
        COPY app .
        RUN pip install --no-cache-dir -r requirements.txt
        EXPOSE 3000
        CMD python app.py
