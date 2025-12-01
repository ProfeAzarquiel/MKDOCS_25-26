# APACHE HADOOP

Hadoop es un proyecto open source que aglutina una serie de herramientas para el procesamiento distribuido de grandes conjuntos de datos a través de clústers de ordenadores utilizando modelos de programación sencillos.

<figure markdown="span">
    ![Hadoop Logo](https://upload.wikimedia.org/wikipedia/commons/0/0e/Hadoop_logo.svg){width="450"}
    <small><figcaption>Logo Apache Hadoop</figcaption></small>
</figure>

## ARQUITECTURA BÁSICA

* **COMMON UTILITIES**: conjunto de librerias y ficheros necesarios para ejecutar Haddop.
* **YARN**: gestor de recursos, se encarga de repartir los recursos disponibles en cada nodo entre las distitnas aplicaciones.
* **HDFS**: sistema de archivos distribuidos instalado en los distintos nodos del cluster y va almacenando mediante replicación los datos.
* **MapReduce**: son los procesos implementados en código por el usuario, para procesar los datos.

![Hadoop Arquitecture](https://media.geeksforgeeks.org/wp-content/uploads/20250628162844186405/Hadoop_architecture.webp)

Si queremos empezar a utilizar Hadoop y todo su ecosistema, disponemos de diversas distribuciones con toda la arquitectura, herramientas y configuración ya preparadas. Las más reseñables son:

* **EMR de AWS** (Amazon Elastic MapReduce)
* **CDP de Cloudera**: es la evolución de CDH y HDP (antiguas distribuciones de código abierto de Apache Hadoop y otros proyectos relacionados, actualmente sin soporte). Cloudera ofrecia estas distribuciones de forma gratuita, cosa que ya no sucede con CDP, pero aún se puede descargar la MV de HDP en el siguiente [enlace](https://archive.cloudera.com/hwx-sandbox/hdp/hdp-3.0.1/HDP_3.0.1_virtualbox_181205.ova).
* **Azure HDInsight** de Microsoft.
* **DataProc** de Google.

Nosotros vamos a usar una **`Apache Ambari`**, proyecto dedicado a simplificar la administración, aprovisionamiento, la gestión y la monitorización de clústeres Apache Hadoop; proporcionando una interfaz web de administración intuitiva y fácil de usar.

!!! quote "No todo el monte es Orégano :material-bell-alert:"
    Hadoop facilita el trabajo con grandes volúmenes de datos, pero montar un clúster funcional no es una cosa trivial. Existen gestores de clústers que hacen las cosas un poco más sencillas (como *Apache Ambari* o *Apache Mesos*), aunque la tendencia es utilizar una solución cloud que nos evita toda la instalación y configuración.

### **HDFS**

Es la capa de almacenamiento de Hadoop o lo que es lo mismo, un sistema de ficheros distribuido, con gran tolerancia a fallos, facil de escalar de forma incremental y capaz de almacenar grandes volúmenes de datos.

Su funcionamiento se basa en repartir los datos entre todos los nodos del clúster, dividiendo los ficheros en bloques y almacenando copias duplicadas en diferentes nodos. Por defecto cada bloque se replica en 3 nodos distintos (esto se conoce como **factor de replicación**).

Está planteado para escribir los datos una vez y leerlos muchas veces, **Write Once, Read Many** (WORM). Además, una vez escritos, los datos son inmutables; es decir, cada fichero de HDFS solo permite añadir contenido (append-only) o eliminar el fichero completo.

!!! tip "Modificación de datos en HDFS"
    Tanto **HBase** como **Hive** ofrecen una capa por encima de HDFS para dar soporte a la modificación de los datos, como en cualquier base de datos.

#### **Tipos de Nodos**

En HDFS vamos a distinguir 2 tipos de máquinas o roles:

* **Namenode**: actúa como máster y almacena todos los metadatos necesarios para construir el sistema de ficheros a partir de sus bloques. Tiene control sobre dónde están todos los bloques.

* **Datanode**: son los nodos esclavo, se limitan a almacenar los bloques que componen cada fichero.

#### **Los Bloques**

Un bloque es la cantidad mínima de datos que puede ser leída o escrita en HDFS, su tamaño predeterminado son 128 MB.

Todos los ficheros en HDFS estan divididos en bloques, por lo que si subimos un fichero de 600MB, se dividirá en 5 bloques de 128MB, que se distribuirán por todos los nodos de datos del clúster.

Además, a través del factor de replicación (por defecto 3), cada bloque se almacena varias veces en diferentes nodos. Así que finalmente, nuestro archivo de 600MB estará repartido en 15 bloques entre todos los nodos del clúster.

<figure markdown="span">
    ![Replicación HDFS](https://substackcdn.com/image/fetch/$s_!_cKl!,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F8321d7f4-8e50-4990-a3c2-169c33106c5c_582x394.png){width="400"}
    <small><figcaption>División y replicación HDFS</figcaption></small>
</figure>

### **YARN**

Es el framework que permite a Hadoop soportar varios motores de ejecución incluyendo MapReduce, y proporciona un planificador agnóstico a los trabajos que se encuentran en ejecución en el clúster. Esta mejora de Hadoop también es conocida como Hadoop 2.

Yarn separa las dos funcionalidades principales: la gestión de recursos y la planificación y monitorización de trabajos. Con esta idea, es posible tener un gestor global (Resource Manager) y un Application Master por cada aplicación.

### **MAP-REDUCE**

Hadoop MapReduce es un paradigma de procesamiento de datos caracterizado por dividirse en dos fases o pasos diferenciados: Map y Reduce. Estos subprocesos asociados a la tarea se ejecutan de manera distribuida, en diferentes nodos de procesamiento o esclavos. Para controlar y gestionar su ejecución, existe un proceso Master o Job Tracker. También es el encargado de aceptar los nuevos trabajos enviados al sistema por los clientes. Los resultados del procesamiento se pueden almacenar en el mismo sistema de almacenamiento o bien en una base de datos o sistema externo.

#### **Fases de Hadoop MapReduce**

**Map**: se ejecuta en subtareas llamadas mappers. Estos componentes son los responsables de generar pares clave-valor filtrando, agrupando, ordenando o transformando los datos originales. Los pares de datos intermedios, no se almacenan en HDFS.

**Shuffle**: puede no ser necesaria. Es el paso intermedio entre Map y reduce que ayuda a recoger los datos y ordenarlos de manera conveniente para el procesamiento. Con esta fase, se pretende agregar las ocurrencias repetidas en cada uno de los mappers.

**Reduce**: gestiona la agregación de los valores producidos por todos los mappers del sistema (o por la fase shuffle) de tipo clave-valor en función de su clave. Por último, cada reducer genera su fichero de salida de forma independiente, generalmente escrito en HDFS.

<figure markdown='span'>
    ![Fases MapReduce](https://storage.googleapis.com/algodailyrandomassets/curriculum/systems-design/map-reduce/example.png)
    <small><figcaption>Fases MapReduce</figcaption></small>
</figure>

#### **Limitaciones**

MapReduce es la implementación básica de un framework de procesamiento en paralelo para cargas big data, por lo que al final tiene ciertas limitaciones como, por ejemplo, hasta que la fase map completa su procesamiento, los reducers no empiezan a ejecutarse; o que no se puede controlar su orden de ejecución.

Es por ello, que existen alternativas como Apache Spark, Apache Hive o Pig; las cuales mantienen los principales puntos de MapReduce, pero son capaces de usar HDFS de manera más eficiente. La que resalta más entre el resto es **SPARK**, la cual veremos más adelante.

*<small>[Fuente 1: Aitor Medrano](https://aitor-medrano.github.io/iabd/)</small>*, *<small>[Fuente 2: Oscar Fernández](https://aprenderbigdata.com/hadoop/)</small>*

## INSTALACIÓN DE APACHE AMBARI SOBRE DOCKER

### ESCENARIO E IMAGEN EMPLEADA

Partimos de una máquina con Ubuntu Desktop 25.10, en la que tenemos instalado Docker 28.5.1. En ella, vamos a desplegar un entorno con cuatro contenedores, consistente en:

* **Un contenedor** ( bigtop_hostname0) para el servidor Ambari.
* **Tres contenedores** ( bigtop_hostname1, bigtop_hostname2, bigtop_hostname3) para agentes de Ambari.
* **Un volumen** compartido para el repositorio Ambari.

La imagen que usaremos `bigtop/puppet:trunk-rockylinux-8imagen`, forma parte del proyecto Apache BigTop y proporciona un marco de trabajo para la creación y prueba de proyectos relacionados con Hadoop, ya que viene preconfigurada con muchas de las dependencias necesarias para los servicios de Ambari y Hadoop. Esta imagen incluye:

* Rocky Linux 8 como sistema operativo base
* Java y herramientas de desarrollo preinstaladas
* Puppet para la gestión de la configuración
* Configuraciones de sistema optimizadas para los servicios del ecosistema Hadoop

### CONFIGURACIÓN DEL ENTORNO

#### ATAJO ÚTIL PARA DOCKER

Este es un paso opcional, pero que puede ahorrarnos mucho tiempo en el futuro. Vamos a crear una función para conectarnos más facilmente al terminal de consola de un contenedor:

    sudo su -

    nano /etc/profile

    # Añadimos la siguiente función al final del fichero
    con() {
        docker exec -it "$1" /bin/bash
    }

    # Aplicamos los cambios inmediatemente
    source /etc/profile

Tras añadir esta función, podrá acceder rápidamente a cualquier contenedor utilizando:

    con contenedor_linux

#### CREACIÓN DE CARPETAS Y FICHERO HOSTS

Creamos una carpeta *Ambari* y dentro de ella, añadiremos dos carpetas más:

    mkdir -p Ambari
    cd Ambari
    mkdir -p ambari-repo
    mkdir -p conf

Por un lado, accedemos a la carpeta **ambari-repo** y descargamos los páquetes RPM de Ambari y Bigtop:

    cd ambari-repo/

    //PARA ROCKY LINUX 8 (que es el SO que vamos a usar para nuestros nodos)
    wget -r -np -nH --cut-dirs=4 --reject 'index.html*' https://www.apache-ambari.com/dist/ambari/3.0.0/rocky8/
    wget -r -np -nH --cut-dirs=4 --reject 'index.html*' https://www.apache-ambari.com/dist/bigtop/3.3.0/rocky8/

Por otro lado, accedemos a la carpeta *conf* y creamos el fichero *hosts*, con la siguiente información:

    cd conf

    cat > hosts << EOF
    127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
    ::1         localhost localhost.localdomain localhost6 localhost6.localdomain6

    # Container hostnames
    172.20.0.2  ambari-bigtop_hostname1-1
    172.20.0.3  ambari-bigtop_hostname3-1
    172.20.0.4  ambari-bigtop_hostname0-1
    172.20.0.5  ambari-bigtop_hostname2-1
    EOF

!!! warning "ATENCIóN:"
    Los valores indicados más arriba para el fichero ***hosts***, deben personalizarse según la instalación de cada uno. Para ello confirmaremos el direccionamiento que se le ha asignado a cada contenedor con `docker network inspect ambari_net`, así como los hostnames generados con `docker ps`.

#### EL FICHERO DOCKER-COMPOSE

Utilizaremos el siguiente fichero `docker-compose.yaml` para desplegar nuestro escenario:

??? abstract "docker-compose.yaml"
        services:
        bigtop_hostname0:
            container_name: ambari_server
            command: /sbin/init
            domainname: bigtop.apache.org
            image: bigtop/puppet:trunk-rockylinux-8
            mem_limit: 8g
            mem_swappiness: 0
            ports:
            - "8080:8080"
            privileged: true
            networks:
            red_cluster:
                ipv4_address: 192.168.20.2
            volumes:
            - ./ambari-repo:/var/repo/ambari
            - ./conf/hosts:/etc/hosts

        bigtop_hostname1:
            container_name: ambari_agent1
            command: /sbin/init
            domainname: bigtop.apache.org
            image: bigtop/puppet:trunk-rockylinux-8
            mem_limit: 8g
            mem_swappiness: 0
            privileged: true
            networks:
            red_cluster:
                ipv4_address: 192.168.20.3
            volumes:
            - ./ambari-repo:/var/repo/ambari
            - ./conf/hosts:/etc/hosts

        bigtop_hostname2:
            container_name: ambari_agent2
            command: /sbin/init
            domainname: bigtop.apache.org
            image: bigtop/puppet:trunk-rockylinux-8
            mem_limit: 8g
            mem_swappiness: 0
            privileged: true
            networks:
            red_cluster:
                ipv4_address: 192.168.20.4
            volumes:
            - ./ambari-repo:/var/repo/ambari
            - ./conf/hosts:/etc/hosts

        bigtop_hostname3:
            container_name: ambari_agent3
            command: /sbin/init
            domainname: bigtop.apache.org
            image: bigtop/puppet:trunk-rockylinux-8
            mem_limit: 8g
            mem_swappiness: 0
            privileged: true
            networks:
            red_cluster:
                ipv4_address: 192.168.20.5
            volumes:
            - ./ambari-repo:/var/repo/ambari
            - ./conf/hosts:/etc/hosts

        networks:
        red_cluster:
            name: ambari_net
            driver: bridge
            ipam:
            driver: default
            config:
                - subnet: 192.168.20.0/24
                gateway: 192.168.20.1

Arrancamos el escenario con el comando:

    docker-compose up -d

Y verificamos que se han creado nuestros contenedores y que hay conectividad entre ellos:

    docker ps

    docker exec -it ambari-bigtop_hostname0-1 ping -c 4 ambari-bigtop_hostname1-1
    docker exec -it ambari-bigtop_hostname0-1 ping -c 4 ambari-bigtop_hostname2-1
    docker exec -it ambari-bigtop_hostname0-1 ping -c 4 ambari-bigtop_hostname3-1

#### CONEXIÓN SSH ENTRE LOS CONTENEDORES

Mediante la función que creamos al prinicipio, vamos conectandonos al terminal de consola de cada uno de nuestros contenedores e instalamos OPENSSH:

    con ambari-bigtop_hostname0-1
    dnf install -y sudo openssh-server openssh-clients which iproute net-tools less vim-enhanced
    ssh-keygen -t rsa -N "" -f ~/.ssh/id_rsa
    systemctl enable sshd
    systemctl start sshd
    exit

Luego desde nuestra máquina ubuntu copiamos la clave del SERVIDOR y luego la pegamos en cada uno de los AGENTES:

    docker exec -i ambari-bigtop_hostname0-1 bash -c 'cat ~/.ssh/id_rsa.pub' > id_rsa.pub

    cat id_rsa.pub | docker exec -i ambari-bigtop_hostname1-1 bash -c 'cat >> ~/.ssh/authorized_keys'
    cat id_rsa.pub | docker exec -i ambari-bigtop_hostname2-1 bash -c 'cat >> ~/.ssh/authorized_keys'
    cat id_rsa.pub | docker exec -i ambari-bigtop_hostname3-1 bash -c 'cat >> ~/.ssh/authorized_keys'

    rm id_rsa.pub

Finalmente nos volvemos a conectar al SERVIDOR y comprobamos la conectividad con los agentes:

    con ambari-bigtop_hostname0-1
    ssh -o StrictHostKeyChecking=no ambari-bigtop_hostname1-1 echo "Connection successful"
    ssh -o StrictHostKeyChecking=no ambari-bigtop_hostname2-1 echo "Connection successful"
    ssh -o StrictHostKeyChecking=no ambari-bigtop_hostname3-1 echo "Connection successful"

#### INSTALACIÓN DE SOFTWARE EXTERNO NECESARIO

Repetimos los siguientes pasos en todos los nodos del cluster:

    con ambari-bigtop_hostname0-1
    dnf install -y initscripts wget curl tar unzip git
    dnf install -y dnf-plugins-core
    dnf config-manager --set-enabled powertools
    dnf update -y
    dnf install nano
    dnf install python3
    nano /etc/yum.repos.d/Rocky-Devel.repo
    dnf repolist | grep devel

### INSTALACIÓN DE AMBARI

#### ACCESO AL REPOSITORIO LOCAL

Luego configuramos el acceso de los paquetes de estos repositorios, en todos los nodos del Cluster:

    con ambari-bigtop_hostname0-1

    dnf install -y createrepo

    cd /var/repo/ambari/
    createrepo .

    tee /etc/yum.repos.d/ambari.repo << EOF
    [ambari]
    name=Ambari Repository
    baseurl=file:///var/repo/ambari
    gpgcheck=0
    enabled=1
    EOF

#### INSTALACIÓN DE PAQUETES

Instalamos los siguientes paquetes en todos los nodos del cluster:

    con ambari-bigtop_hostname0-1

    yum install -y iproute
    yum install -y python3-distro
    yum install -y java-17-openjdk-devel
    yum install -y java-1.8.0-openjdk-devel
    yum install -y ambari-agent

    dnf clean all
    dnf makecache

Y además de lo anterior, solo en el SERVIDOR, instalamos lo siguiente:

    con ambari-bigtop_hostname0-1

    yum install -y python3-psycopg2
    yum install -y ambari-server

#### INSTALACIÓN Y CONFIGURACIÓN DE MYSQL

Seguimos conectados al SERVIDOR y configuramos lo siguiente:

    rpm -qa | grep mysql
    rpm -ev mysql-server --nodeps
    rpm -ev mysql-community-server --nodeps

    yum -y install https://dev.mysql.com/get/mysql80-community-release-el8-1.noarch.rpm

    yum -y install mysql-server
    systemctl start mysqld.service
    systemctl enable mysqld.service

Ahora configuramos en MySQL las BBDD y Usuarios que usaremos en el ecosistema Hadoop:

    mysql

    CREATE USER 'ambari'@'localhost' IDENTIFIED BY 'ambari';
    GRANT ALL PRIVILEGES ON *.* TO 'ambari'@'localhost';
    CREATE USER 'ambari'@'%' IDENTIFIED BY 'ambari';
    GRANT ALL PRIVILEGES ON *.* TO 'ambari'@'%';

    CREATE DATABASE ambari CHARACTER SET utf8 COLLATE utf8_general_ci;
    CREATE DATABASE hive;
    CREATE DATABASE ranger;
    CREATE DATABASE rangerkms;

    CREATE USER 'hive'@'%' IDENTIFIED BY 'hive';
    GRANT ALL PRIVILEGES ON hive.* TO 'hive'@'%';

    CREATE USER 'ranger'@'%' IDENTIFIED BY 'ranger';
    GRANT ALL PRIVILEGES ON *.* TO 'ranger'@'%' WITH GRANT OPTION;

    CREATE USER 'rangerkms'@'%' IDENTIFIED BY 'rangerkms';
    GRANT ALL PRIVILEGES ON rangerkms.* TO 'rangerkms'@'%';

    FLUSH PRIVILEGES;

    exit

    mysql -uambari -pambari ambari < /var/lib/ambari-server/resources/Ambari-DDL-MySQL-CREATE.sql

#### ARRANQUE DE LOS SERIVICIOS EN EL SERVIDOR

Continuamos sin salir del SERVIDOR Ambari, y pasamos a arrancar los servicios JAVA, Amberi-Server y Ambari-Agent:

    wget https://repo1.maven.org/maven2/mysql/mysql-connector-java/8.0.28/mysql-connector-java-8.0.28.jar -O /usr/share/java/mysql-connector-java.jar

    ambari-server setup --jdbc-db=mysql --jdbc-driver=/usr/share/java/mysql-connector-java.jar

    echo "server.jdbc.url=jdbc:mysql://localhost:3306/ambari?useSSL=true&verifyServerCertificate=false&enabledTLSProtocols=TLSv1.2" >> /etc/ambari-server/conf/ambari.properties

    ambari-server setup -s -j /usr/lib/jvm/java-1.8.0-openjdk --ambari-java-home /usr/lib/jvm/java-17-openjdk --database=mysql --databasehost=localhost --databaseport=3306 --databasename=ambari --databaseusername=ambari --databasepassword=ambari

    ambari-server start

    sed -i "s/hostname=.*/hostname=localhost/" /etc/ambari-agent/conf/ambari-agent.ini
    ambari-agent start

#### ARRANQUE DEL SERVICIO EN LOS AGENTES

Por último, vamos accediendo a los nodos Agente y ejecutamos lo siguiente:

    sed -i "s/hostname=.*/hostname=ambari-bigtop_hostname0-1/" /etc/ambari-agent/conf/ambari-agent.ini
    ambari-agent start

!!! bug "POSIBLE BUG"
    Es posible que nos lance un error al arrancar los Agentes, debido a que el enlace al binario de Python no se crea automáticamente. En ese caso, ejecutando los siguientes comandos debería solucionarse:

        dnf install -y python3
        ln -s /usr/bin/python3 /usr/bin/ambari-python-wrap

        yum reinstall -y ambari-agent
        ambari-agent start

Si todo ha ido bien, ya tenemos nuestro cluster Ambari operativo, y podemos acceder a él vía web, como veremos en el siguiente apartado.

## CONFIGURACIÓN DE HADOOP DESDE AMBARI

### ACCESO A LA INTERFAZ GRÁFICA

Una vez tenemos arrancados el servidor y los agentes Ambari, podemos ir a nuestro navegador y acceder a su interfaz gráfica a través de la dirección:

    https://localhost:8080

!!! tip "CREDENCIALES"
    Tanto el usuario como la contraseña para acceder la primera vez es `admin`.

### INSTALACION DE HADOOP

Cuando iniciemos sesión en Ambari, veremos que no hay nada configurado y la primera opción que nos muestra es la de crear nuestro cluster Hadoop usando un assitente de instalación, para ello pulsamos en el botón:

[<small>LAUNCH INSTALL WIZARD</small>](# .){ .md-button }

Comenzamos por poner un nombre a nuestro cluster y luego seleccionamos la versión de BIGTOP que queremos, nosotros elegimos la 3.3.0, ya que es la que hemos utilizado en nuestros nodos.

Más abajo, en esta misma página debemos marcar la opción de **Usar Repositorio Local** e indicar que distribución de SO tenemos, en nuestro caso Rocky Linux 8, esta basado en **Red Hat 8**. El resto de distribuciones las debemos borrar haciendo click en el botón **`- Remove`** que aparece en la parte izquierda de cada una de ellas.

Por último, especificamos en el campo **"Base URL"** donde se encuentra nuestro repositorio local. Para ello indicaremos el hostname de nuesto nodo SERVIDOR y que acceda usando el protocolo HTTP:

    http://ambari-bigtop_hostname0-1

[<small>NEXT</small>](# .){ .md-button .md-button--primary }

En la siguiente página, donde dice **Target Hosts** tenemos que indicar el hostname de cada nodo de nuestro cluster, separados por saltos de línea.

Luego en el apartado de **Host Registration Information** marcamos la opción de `Provide your SSH Private Key` y en el cuadro de texto debemos pegar nuestra clave SSH. Para ello, nos conectamos al nodo SERVIDOR y copiamos toda la salida del comando:

    con ambari-bigtop_hostname0-1
    cat ~/.ssh/id_rsa

La clave será algo parecido a esto:

    -----BEGIN OPENSSH PRIVATE KEY-----
    b3BlbnNzaC1rZXktdjEAAAAABG5vbmUAAAAEbm9uZQAAAAAAAAABAAABlwAAAAdzc2gtcn
    NhAAAAAwEAAQAAAYEAlzHRzdd/3usUsUuwI8n04+YzVtCfRcmfphn+zhO1iPwwkxcd4Ub6
    8SsLOIWIyR+eSoQ/2Dr27l2mk12dhcUkh/d9U8COiIlAt1QSLWPNEm5AQ+MvRgRBWADhMh
    ...
    -----END OPENSSH PRIVATE KEY-----

[<small>REGISTER AND CONFIRM</small> :material-arrow-right-thin:](# .){ .md-button .md-button--primary }

Esperamos a que finalice el proceso de registro de los nodos y si todo va bien recibiremos un mensaje que dirá `All hosts checks passed on 4 registered hosts`.

[<small>NEXT</small> :material-arrow-right-thin:](# .){ .md-button .md-button--primary }

Pasaremos a la siguiente ventana donde vamos a elegir los servicios que queremos instalar. En nuestro caso, como estamos creando un laboratorio de prácticas y nuestros recursos son limitados, seleccionaremos los más esenciales para empezar (siempre podemos instalar servicios más adelante, una vez que esté funcionando nuestro cluster desde la sección *`Stack an Versions`*).

Por lo tanto, solo mantendremos seleccionados **HDFS**, **YARN**, **MapReduce2**, **ZooKeeper** y **Ambari Metrics**. Más adelante, por ejemplo tendremos que instalar **HBase**, ya que sin él no funcionará Ambari Metrics.

[<small>NEXT</small> :material-arrow-right-thin:](# .){ .md-button .md-button--primary } :material-plus-box: [<small>PROCEED ANYWAY</small>](# .){ .md-button }

Continuaremos con la opción de **Asign Masters**, esto es personalizable según las necesidades y recursos de cada cluster, pero un ejemplo para un laboratorio similar al nuestro, podría ser el siguiente:

![Ejemplo Masters Hadoop](../img/mastersAmbari.png)

[<small>NEXT</small> :material-arrow-right-thin:](# .){ .md-button .md-button--primary }

Al igual que pasaba en el paso anterior, la configuración de **Assign Slaves and Clients** puede variar de un escenario a otro, en este caso podemos establecer algo como lo siguiente:

![Ejemplo Slaves Hadoop](../img/slavesAmbari.png)

[<small>NEXT</small> :material-arrow-right-thin:](# .){ .md-button .md-button--primary }

En la pestaña **Credentials** nos pedirá configurar usauraio y contraseña para Grafana, podemos dejarlo como `admin / admin`.

[<small>NEXT</small> :material-arrow-right-thin:](# .){ .md-button .md-button--primary }

En las siguientes pestañas **Directories**, **Accounts** y **All Configurations**; podremos hacer cambios más adelante si fuese necesario, pero por el momento vamos a dejarlos con sus valores por defecto, así que solo pulsamos en siguiente para avanzar.

[<small>DEPLOY</small> :material-arrow-right-thin:](# .){ .md-button .md-button--primary }

Comenzará el despliegue de los servicios en los nodos de nuestro cluster, esto puede llevar un tiempo según las caracteristicas de nuestra máquina

Además, el servicio *Timeline Service V2* va a fallar, pero no es un problema, así que vamos a ignorarlo por el momento. El resto de servicios, deberían instalarse con éxito, si no es así, podemos pulsar en el botón **`RETRY`:material-backup-restore:**, para volver a intentarlo. Si todo va bien, debería finalizar el despliegue con 2 o 3 warnings, pero sin errores.

!!! warning "FALTA DE RECURSOS"
    Si por algún motivo, como recursos insuficientes el despliegue se queda congelado y no responde; podemos conectarnos a la consola de nuestro nodo SERVIDOR y ejecutar el siguiente comando:
        ambari-server restart

[<small>NEXT</small> :material-arrow-right-thin:](# .){ .md-button .md-button--primary } :material-plus-box: [<small>COMPLETE</small> :material-arrow-right-thin:](# .){ .md-button }

Dejaremos unos segundos (o minutos) a que los servicios terminen de arrancar y podremos conectarnos a nuestro nodo SERVIDOR y comprobar que efectivamente tenemos habilitado el sistema de archivos de Hadoop:

    con ambari-bigtop_hostname0-1
    hadoop fs -ls /

*<small>[Fuente 1: Ambari Docker Environmet Setup](https://ambari.apache.org/docs/3.0.0/quick-start/environment-setup/docker-environment-setup/)</small>* , *<small>[Fuente 2: Ambari Installation Guide](https://ambari.apache.org/docs/3.0.0/quick-start/installation-guide)</small>* , *<small>[Fuente 3: Edward Viaene](https://youtu.be/Ao95xAGsA20)</small>*
