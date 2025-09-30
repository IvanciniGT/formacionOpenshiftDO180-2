
# Contenedor

Entorno aislado dentro de un sistema operativo que ruede un kernel Linux, donde corren procesos.

Aislado?

- Sus propio sistema de archivos
- Sus propias variables de entorno
- Su propia configuración de red
- Puede tener limitaciones de acceso a los recursos del hierro

Se crean desde Imágenes de contenedor, que se identifican mediante una URL:
   > registry/repo:tag

Tags: (incluyen información de versión del producto, librerías, imagen base, etc)
- variables
- fijos

# Imágenes base:

Alpine, Ubuntu, Debian, CentOS, Fedora, etc.

- 4 carpetas (POSIX)
- 4 comandos
- Otro software: curl, bash...
- Gestores de paquetes (apt, yum, dnf, apk)

## Gestores de contenedores

Que me ayudan a descarga imágenes de contenedor y a crear y operar contenedores:
- Podman
- Docker
- CRI-O
- Containerd

## Cual es la gran diferencia entre un contenedor y una VM?

En los contenedores no tenemos un SO ni puede tenerse.. No puedo montar un kernel.
Se comparte el kernel del host.

# Qué puedo hacer para que un puerto de un contenedor sea accesible desde fuera de mi máquina?

NAT: Redirección de puertos en el host

# De qué iba eso de los volúmenes?

Puntos de montaje que hacemos en el FileSystem del contenedor, que apuntan a ubicaciones de almacenamiento fuera del contenedor (HOST, Cabina, NFS...)

## Para qué sirven?

- Compartir datos entre contenedores
- Persistencia de datos tras la eliminación del contenedor
- Inyectar ficheros / carpetas con archivos (configuraciones)

---

ElasticSearch
- Monitorizar logs de apps
    
    C1-200 tomcats                     Logstash --->  ElasticSearch <- Kibana
            v
        ---> Log <<<< CREO UN VOLUMEN PARA ESTO... pero... necesito PERSISTENCIA? NO
            ^          Lo creo con soporte en RAM. Monto un trozo de la RAM (128Kbs) como una carpeta en disco
            |          Al tomcat le configuro 2 archivos de log rotados de 64kbs
    CA  FileBeat ------------------>

---

Software:

Aplicación
Demonio
    Servicio
Comando
Script
Driver
Librería


En un contenedor puedo ejecutar casi cualquier cosa.

---

# Entornos de producción:

Es un entorno muy especial... dentro de la infra de mi empresa, tendré muchos entornos: producción, preproducción, desarrollo, testing...
Pero de ellos, el de producción tiene algunas características que no tienen el resto de entornos:

- Alta disponibilidad:
    Tratar de conseguir que el sistema funcione sin interrupciones, incluso en caso de fallos de hardware o software.

    Se mide en 9s.
    Disponibilidad 90%          - 36 días de inactividad al año         |   €
    Disponibilidad 99%          - 3.65 días de inactividad al año       |   €€
    Disponibilidad 99.9%        - 8.76 horas de inactividad al año      |   €€€€€€
    Disponibilidad 99.99%       - 52.56 minutos de inactividad al año   |   €€€€€€€€€€€€
                                                                        v

    Como consigo esa alta disponibilidad? O como trato de conseguirla: REDUNDANCIA!

- Escalabilidad:
    Capacidad de ajustar mi infraestructura y recursos a las necesidades cambiantes de la aplicación.

    App1: App de un hospital.. pa' que los médicos metan datos de los pacientes.
    Dia 1:       100 usuarios
    Dia 100:     100 usuarios         MO NECESITO ESCALAR
    Dia 1000:    100 usuarios
    
    App2: 
    Dia 1:       100 usuarios
    Dia 100:     1000 usuarios        ESCALABILIDAD VERTICAL: Más máquina! más CPU, más RAM, más disco...
    Dia 1000:    10000 usuarios   

    App3: INTERNET
    Dia n:       100 usuarios
    Dia n+1:     1000000 usuarios
    Dia n+2:     0 usuarios
    Dia n+3:     100000000 usuarios
    Dia n+4:     10 usuarios

    Lo podemos poner en horas:

    Soy la wb del telepi:
    00:00         usuario? 0 que estoy cerrao
    6:00          usuario? 0 que empiezo a currar
    11:00         usuario? 10
    14:00         usuario? 10000                ESCALABILIDAD HORIZONTAL: Más máquinas!
    17:00         usuario? 100
    20:30         Madrid vs Barça? usuario? 10000000000
    23:30         usuario? 0

    Clouds: Me permiten adquirir recursos bajo demanda y pagar solo por lo que uso. Y liberarlos cuando ya no los necesito.

- Seguridad. Es algo que debemos tener en cuenta en todo entorno.
  Cierto es que al de producción le ponemos un poquito más de atención.

Kubernetes

Servidor1                               REGLAS DE FIREWALL                          REGLAS DE FIREWALL
 docker
    App1    C1(TOMCAT1 http://ip_servidor1:8080)  < 
    App2.   C2
    App17.  C17
Servidor2
  docker
    App1    C2(TOMCAT2 http://ip_servidor2:8080)  <  BALANCEO DE CARGA http://ip_balanceo:8090  <  PROXY REVERSO     | <   PROXY <   Cliente
Servidor3                                                                                                                         MENCHU :(
  docker
    App1    C3(TOMCAT3 http://ip_servidor3:8080)  <
Servidor4
  docker
    App1    C4(TOMCAT4 http://ip_servidor4:8080)  <

VOLUMEN COMPARTIDO (NFS, GlusterFS, CephFS...) para que la app comparta datos entre los distintos nodos

Qué tiene que ver esto con el docker y los contenedores?

# KUBERNETES

Es una herramienta que me permite configurar y definir (EN UN FICHERO YAML) un entorno de producción... Y además monta y opera ese entorno por mí. Como anécdota usa contenedores para ejecutar los procesos de las aplicaciones.
De hecho, gracias a la estandarización que existe en el mundo de los contenedores, Kubernetes puede hacer eso.

Servidores: Node
Reglas de firewall: Network Policies
Balanceo de carga: Service
Proxy reverso: Ingress Controller
Volumen compartido: Persistent Volume Claim , Persistent Volume


VMWARE - Tanzu -> VSphere
OpenShift (RedHat) -> Clouds (AWS, Azure, IBM Cloud, on-premise)

---

En kubernetes no gestionamos CONTENEDORES, gestionamos PODS

> POD?

Conjunto de contenedores (quizás solo uno) que comparten:
- Configuración de Red (misma IPs, 127.0.0.1: localhost)
  - Puedo comunicarme entre contenedores del mismo pod por localhost
- Tengo garantizado que se ejecutan en el mismo nodo / host
  - Pueden compartir almacenamiento (volúmenes) a nivel de host (Disco del host, RAM del host...)
- Escalan juntos

---

Quiero montar un wordpress:
- Apache + php   + WORDPRESS
  Nginx
- MySQL / MariaDB

Cuántos contenedores querría yo?
- Opción A: 1 contenedor con Apache + php + MYSQL
- Opción B: 3 contenedores: Apache , MySQL, php
- Opción C: 2 contenedores: Apache + php, MySQL

La opción A ni de coña. Por qué?
- Los quiero escalar por separado
- Si actualizo el Mysql, tengo que actualizar/parar también el Apache?
- O al revés... si actualizo el Apache, tengo que actualizar/parar también el Mysql?
- Si uno de los dos falla, se me cae todo el servicio


- Opción C: 2 contenedores: 
      - Apache + php
      - MySQL

Si lo llevo a Kubernetes, cuántos pods?
1 pod con los 2 contenedores. Esto no vale, por lo mismo que arriba... Si tengo un sol pod, escalan juntos, y no quiero eso.
2 pods: 
    - 1 pod con el contenedor del Apache + php, 
    - 1 pod con el contenedor del MySQL
  
Quiero ir monitorizando los ficheros de log de esos apaches... Quiero llevarlos a un ElasticSearch.
Eso lo hará un Filebeat. Ese filebeat, lo ejecuto en el mismo contenedor del Apache o en otro?
Todos los programas por definición los montamos en contenedores independientes. 
Cada programa su contenedor. Se trata de aislar... de que si uno falla, no arrastre todo el sistema.

Ahora... 1 pod o 2 pods?
2 pods... quiero que escalen JUNTOS.
Cada apache, debe llevar pegao al culo su propio filebeat.

---

Estas cosas se las vamos a explicar a Kubernetes.

Kubernetes AUTOMATIZA el entorno de producción: El montarlo y operarlo.
Qué es automatizar?
Es montar una máquina o configurar una (programándola, con programas) para que haga una tarea que antes hacía un humano.

Puedo AUTOMATIZAR EL LAVADO DE LA ROPA:
LAVADORA! que hace el trabajo que antes hacía un humano.
Incluso puedo ajustar el comportamiento de la lavadora (con programas: PROGRAMA DE ROPA DELICADA, FRÍA, DEPORTE) para que haga lo que yo quiero.

En nuestro caso, esas automatizaciones las haremos mediante COMPUTADORAS... esas son nuestras máquinas.
Y en ellas vamos a crear PROGRAMAS, que hagan lo que antes hacía un humano.
Por ejemplo, puedo automatizar tareas de ADMINISTRACIÓN DE SISTEMAS... de hecho lo llevamos haciendo décadas: script .sh .ps1, playbook de ansible, etc.

ESAS COSAS SON PROGRAMAS.

Dicho de otra forma... cuando automatizo dejo de ser ejecutor/operador, y paso a ser PROGRAMADOR.

Nuestro trabajo ya no va a ser operar sistemas. Mi trabajo pasa a ser crear un programa que le diga a kubernetes como tiene que operar el sistema. ESTO ESCUECE!... está bien!
LO BUENO!

Cuando creamos programas, muchas veces usamos LENGUAJES DE PROGRAMACIÓN.
Y puedo usar esos lenguajes de programación de muchas formas... los horteras de los desarrolladores llaman a esto: PARADIGMAS DE PROGRAMACIÓN:
- Imperativo
- Funcional
- Programación orientada a objetos
Digo que ese es solo un nombre hortera.. porque en los lenguajes naturales (los idiomas que hablamos) también existen paradigmas... aunque nos les llamamos así:

> FELIPE, IF (SI) hay algo que no sea una silla debajo la ventana,   CONDICIONAL
>   FELIPE QUITALO.                                                     ORACIÓN IMPERATIVA
> IF SI... no hay una silla debajo de la ventana,   CONDICIONAL
>     IF not silla GOTO IKEA!
>     Y compra silla! 
    > Felipe, pon una silla debajo de la ventana.                        ORACIÓN IMPERATIVA


> Felipe, debajo de la ventana tiene que haber una silla. Es tu responsabilidad.    Estoy dando una orden? NO
                                                                                    > ORACIÓN DECLARATIVA
                                                                                    De hecho, el cómo conseguir eso, lo dejo a tu criterio.
                                                                                    Lo delego!
                                                                                    Cuando hablo de forma declarativa, me centro en el qué quiero conseguir, no en el cómo conseguirlo.

Estamos muy acostumbrados al lenguaje imperativo al programar. Y ES UNA MIERDA DEL 15. Pierdo de vista lo que quiero conseguir, y me centro en el CÓMO (el procediiento a seguir) para conseguirlo.

Por ejemplo toda la comunicación con una terminal es imperativa.


Hay muchos sistemas informáticos que están empezando a dejar de lado el paradigma imperativo, y están adoptando el declarativo.
Y LO ESTÁN PETANDO!!!! 
- Ansible
- Kubernetes
- Terraform
- Docker Compose


Cluster Activo/Pasivo

    Nodo1 App1 ON           VIPA -----
    Nodo2 App1 OFF   <----

    En kubernetes ese nodo 2 no lo tengo precreado.
    Ese nodo2 es un contenedor! y se el primero falla, creo uno nuevo, le pincho los mismos datos que al otro y pa'rriba.


Weblogic 1
    app1
Weblogic 2
    app1
Weblogic 3
    app1
Weblogic 4
    app1


Weblogic1
    app1
Weblogic2
    app1



OPENSHIFT : Plataforma orientada a DESARROLLADORES, donde ellos puedan montar SU INFRA !
Y el dpto de operaciones entero a la puta calle!

Quiero un MariaDB con 2 cores y 2Gbs de ram y 5 gbs de disco. OPENSHIFT OK ... 1 minutos.. ya lo tienes!
Que quiero monitorizar mi sistema!
Y al segundo tiene allí cuadros de mando y logs y estadísticas y de todo.


# DEVOPS

Una cultura, un movimiento en pro de la AUTOMATIZACION de qué? De todo lo que está entre el DEV---> OPS
Y básicamente es quitar de en medio de la cadena de suministro a los tíos de operaciones!

Un desarrollador hace commit en el git ---> y en 5 minutos la app está en producción... sin mediación humana!
CONTINUOUS DEPLOYMENT: Despliegue continuo

Eso si... para llegar a eso... una de dos.. 
[- O ESTOY MUY VOLAO!]
- O TENGO MUCHA CONFIANZA EN EL DESARROLLO <--- qué me da esa confianza? PRUEBAS AUTOMATIZADAS
En un entorno de pruebas: CONTINUOUS INTEGRATION: Integración continua


Alguno de operaciones necesito... antes tenía a 200 -> 10


EXALOGIC
EXADATA