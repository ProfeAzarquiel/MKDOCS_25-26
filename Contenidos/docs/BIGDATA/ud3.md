# APACHE HADOOP

Hadoop es un proyecto open source que aglutina una serie de herramientas para el procesamiento distribuido de grandes conjuntos de datos a través de clústers de ordenadores utilizando modelos de programación sencillos.

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

## INSTALACIÓN DE APACHE AMBARI SOBRE DOCKER

### ESCENARIO E IMAGEN EMPLEADA

Partimos de una máquina con Ubuntu Desktop 25.10, en la que tenemos instalado Docker 28.5.1. En ella, vamos a desplegar un entorno con cuatro contenedores, consistente en:

* **Un contenedor** ( bigtop_hostname0) para el servidor Ambari
* **Tres contenedores** ( bigtop_hostname1, bigtop_hostname2, bigtop_hostname3) para agentes de Ambari
* **Un volumen** compartido para el repositorio Ambari.

La imagen que usaremos `bigtop/puppet:trunk-rockylinux-8imagen`, forma parte del proyecto Apache BigTop y proporciona un marco de trabajo para la creación y prueba de proyectos relacionados con Hadoop, ya que viene preconfigurada con muchas de las dependencias necesarias para los servicios de Ambari y Hadoop. Esta imagen incluye:

* Rocky Linux 8 como sistema operativo base
* Java y herramientas de desarrollo preinstaladas
* Puppet para la gestión de la configuración
* Configuraciones de sistema optimizadas para los servicios del ecosistema Hadoop

### CONFIGURACIÓN DEL ENTORNO

#### ATAJO ÚTIL PARA DOCKER

Vamos a crear una función para conectarnos más facilmente a los terminales internos de cada contenedor:

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

Luego creamos el fichero *hosts* dentro de la carpeta *conf*:

    cat > conf/hosts << EOF
    127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
    ::1         localhost localhost.localdomain localhost6 localhost6.localdomain6

    # Container hostnames
    172.20.0.2  bigtop_hostname0
    172.20.0.3  bigtop_hostname1
    172.20.0.4  bigtop_hostname2
    172.20.0.5  bigtop_hostname3
    EOF

!!! warning "ATENCIóN:"
    Antes de editar el fichero `hosts`, debemos comprobar el direccionamiento que ha dado a os contenedores con "ip a", y los hostnames generados para cada contenedor. Y esos serán los valores con los que debemos personalizar el archivo.

#### EL FICHERO DOCKER-COMPOSE

Utilizaremos el siguiente fichero `docker-compose.yaml` para desplegar nuestro escenario:

??? abstract "docker-compose.yaml"
        version: '3'

        services:
        bigtop_hostname0:
            command: /sbin/init
            domainname: bigtop.apache.org
            image: bigtop/puppet:trunk-rockylinux-8
            mem_limit: 8g
            mem_swappiness: 0
            ports:
            - "8080:8080"
            privileged: true
            volumes:
            - ./ambari-repo:/var/repo/ambari
            - ./conf/hosts:/etc/hosts

        bigtop_hostname1:
            command: /sbin/init
            domainname: bigtop.apache.org
            image: bigtop/puppet:trunk-rockylinux-8
            mem_limit: 8g
            mem_swappiness: 0
            privileged: true
            volumes:
            - ./ambari-repo:/var/repo/ambari
            - ./conf/hosts:/etc/hosts

        bigtop_hostname2:
            command: /sbin/init
            domainname: bigtop.apache.org
            image: bigtop/puppet:trunk-rockylinux-8
            mem_limit: 8g
            mem_swappiness: 0
            privileged: true
            volumes:
            - ./ambari-repo:/var/repo/ambari
            - ./conf/hosts:/etc/hosts

        bigtop_hostname3:
            command: /sbin/init
            domainname: bigtop.apache.org
            image: bigtop/puppet:trunk-rockylinux-8
            mem_limit: 8g
            mem_swappiness: 0
            privileged: true
            volumes:
            - ./ambari-repo:/var/repo/ambari
            - ./conf/hosts:/etc/hosts

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

#### INSTALACIÓN DE SOFTWARE NECESARIO EN TODOS LOS NODOS

    con ambari-bigtop_hostname0-1
    dnf install -y initscripts wget curl tar unzip git
    dnf install -y dnf-plugins-core
    dnf config-manager --set-enabled powertools
    dnf update -y
    dnf install nano
    dnf install python3
    nano /etc/yum.repos.d/Rocky-Devel.repo
    dnf repolist | grep devel

### GUÍA DE INSTALACIÓN

#### DESCARGA DE LOS REPOSITORIOS Y ACCESO DESDE LOS NODOS

En nuestra máquina Ubuntu, dentro de la carpeta **ambari-repo** que creamos al inicio, descargamos los páquetes RPM de Ambari y Bigtop:

    cd ambari-repo/

    //PARA ROCKY LINUX 8
    wget -r -np -nH --cut-dirs=4 --reject 'index.html*' https://www.apache-ambari.com/dist/ambari/3.0.0/rocky8/
    wget -r -np -nH --cut-dirs=4 --reject 'index.html*' https://www.apache-ambari.com/dist/bigtop/3.3.0/rocky8/

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

    yum install -y python3-distro
    yum install -y java-17-openjdk-devel
    yum install -y java-1.8.0-openjdk-devel
    yum install -y ambari-agent

    dnf clean all
    dnf makecache

Además de lo anterior, solo en el SERVIDOR, isntalamos lo siguiente:

    con ambari-bigtop_hostname0-1

    yum install -y python3-psycopg2
    yum install -y ambari-server

#### CONFIGURACIÓN DE LA BBDD MYSQL

Seguimos conectados al SERVIDOR y configuramos lo siguiente:

    rpm -qa | grep mysql
    rpm -ev <package-name> --nodeps

    yum -y install https://dev.mysql.com/get/mysql80-community-release-el8-1.noarch.rpm

    yum -y install mysql-server
    systemctl start mysqld.service
    systemctl enable mysqld.service

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

Todavía continuamos conectados al SERVIDOR y ejecutamos:

    wget https://repo1.maven.org/maven2/mysql/mysql-connector-java/8.0.28/mysql-connector-java-8.0.28.jar -O /usr/share/java/mysql-connector-java.jar

    ambari-server setup --jdbc-db=mysql --jdbc-driver=/usr/share/java/mysql-connector-java.jar

    echo "server.jdbc.url=jdbc:mysql://localhost:3306/ambari?useSSL=true&verifyServerCertificate=false&enabledTLSProtocols=TLSv1.2" >> /etc/ambari-server/conf/ambari.properties

    ambari-server setup -s -j /usr/lib/jvm/java-1.8.0-openjdk --ambari-java-home /usr/lib/jvm/java-17-openjdk --database=mysql --databasehost=localhost --databaseport=3306 --databasename=ambari --databaseusername=ambari --databasepassword=ambari

    ambari-server start

    sed -i "s/hostname=.*/hostname=localhost/" /etc/ambari-agent/conf/ambari-agent.ini
    ambari-agent start

#### ARRANQUE DEL SERVICIO EN CADA AGENTE

Por último, iniciamos en cada nodo agente el servicio agent:

    sed -i "s/hostname=.*/hostname=ambari-bigtop_hostname0-1/" /etc/ambari-agent/conf/ambari-agent.ini
    ambari-agent start

!!! bug "POSIBLE FALLO"
    En caso de error al arrancar los agentes, lanzar los siguientes comandos y probar de nuevo:

        dnf install -y python3
        ln -s  /usr/bin/python3 /usr/bin/ambari-python-wrap

        yum reinstall -y ambari-agent
        ambari-agent start

#### ACCESO A LA INTERFAZ GRÁFICA

Una vez tenemos arrancado el servidor y los agentes, podemos ir a nuestro navegador y acceder a la interfaz gráfica a través de la dirección:

    https://localhost:8080

!!! tip "CREDENCIALES"
    Tanto el usuario como la contraseña para acceder la primera vez es `admin`.
