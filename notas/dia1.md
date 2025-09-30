# Contenedores?

> Entorno aislado, dentro un host con kernel Linux, donde ejecutar procesos.

Aislado, con respecto a qué?
- Sus propias variables de ENTORNO (env)
- Su propio sistema de archivos (FILESYSTEM)
- Su propia configuración de RED... y por ende sus propias interfaces, IPs...
- Puede tener limitaciones de acceso al hierro (Recursos físicos del host)

## Para qué sirve?

Son la alternativa a las VM para desplegar/ejecutar software en un entorno.
No es que sean la alternativa... es que se las han comido! El 90% del mercado de las VMs ha muerto... en manos de contenedores.


### Instalación a hierro

        App1 + App2 + App3              PROBLEMONES DEL 15!
    -----------------------------           - Mantenimientos... 
    SO (Linux, Solaris, Windows)            - Asignación de recursos
    -----------------------------                   App1 ---> CPU 100% ---> APP1 ----------> OFFLINE
                HIERRO                                                      APP2 y APP3 ---> OFFLINE
                                            - Seguridad
                                            - Incompatibilidades en las configuraciones o en dependencias

### Instalación en VM

        APP1   |    APP2 y APP3         Resuelve los problemones de trabajar con instalaciones a hierro
    -----------------------------       Pero.... PEROOO y es un pero... a qué coste?
        SO1    |    SO2                 Me resuelven esos problemas... pero me dan otros nuevos:
    -----------------------------       
        MV1    |    MV2                 - Desperdicio de recursos (RAM, CPU, Disco)
    -----------------------------       - Configuración y mnto. más complejo que lo de arriba.
       Hipervisor                       - Peor rendimiento de las apps.
        (Citrix, VMWare, Hyper-V)
        VirtualBox, KVM...)
    ----------------------------
     SO(Linux, Solaris, Windows)
    -----------------------------
                HIERRO

### Contenedores

        APP1   |    APP2 y APP3         Me resuelve los problemas de las instalaciones a hierro...
    -----------------------------       Pero sin la parafernalia de las VMs.
        C1    |     C2                      Muchas más ventajas: ESTANDARIZACIÓN!
    -----------------------------       
       Gestor de contenedores
        Docker, Podman, 
        CRI-O, ContainerD...
    ----------------------------
              SO(Linux)
    -----------------------------
                HIERRO

Los contenedores los creamos desde imágenes de contenedor.

# Qué es una imagen de contenedor?

Es un triste fichero comprimido (tar) que contiene:
- Un sistema de archivos según estandar POSIX:
- Una serie de programas y librerías (bin, lib, usr, etc...) ya instalados y configurados en esa estructura de carpetas.
- Variables de entorno (env) con valores por defecto
- Algunos metadatos...

Ese archivo comprimido lo que hacemos es descomprimirlo y luego ejecutar los programas que vienen ahí dentro.... en un.entorno aislado del SO del host.
COSA QUE NO ES NINGÚN INVENTO NUEVO... Hace más de 30 años teníamos un SO que hacía nativamente esto: SOLARIS (ZONES)

Las imágenes de contenedor, las encontramos en REGISTROS DE REPOSITORIOS DE IMÁGENES DE CONTENEDOR:
- Docker hub (el más famoso)
- Quay.io    (el de RedHat)
- Oracle container registry
- Microsoft container registry

No hay un botón de descarga... lo hacemos mediante los gestores de contenedores.
El más usado... ha sido y no será DOCKER.
Pero Docker ha tenido un problema de licencias... y ahora mismo el más usado es PODMAN

docker              <TIPO>    <VERBO>                                           <ARGS>
                CONTAINERS    list     rm.     start.    stop       create
                IMAGES.       list.    rm.     pull.     pull
                VOLUMES       list.    rm.     create
                NETWORKS      list.    rm.     create

## Identificación de imágenes de contenedor

Con una URL:       registry/repo:tag
- registry:       docker.io, quay.io, registry.redhat.io. microsoft.com...
- repo:           library/ubuntu, nginx, http, tomcat...
- tag:            latest, 20.04, 18.04, 1.

Habitualmente no ponemos el registry... salvo casos especiales...
Los gestores de contenedores tienen configurados unos registries por defecto.
Y buscan las imágenes en ellos.

### Imágenes base de contenedor
Son imagenes (es decir, tristes archivos comprimidos .tar) que contienen:
- las 4 carpetas básicas de un SO POSIX:
- Los 20 comandos básicos (sh, cp, mv, ls, mkdir...) de POSIX
- Quizás o quizás no... otros...

Son imágenes que usan los equipos que publican imágenes de contenedor para crear sus propias imágenes.
Si voy a generar una imagen de nginx... Tomo un archivo comprimido ya de esos... le pego encima mis 4 archivos del nginx... y a comprimir de nuevo..

Entre ellas encontramos: ubuntu, debian, alpine, fedora, centos...
    En ubuntu: carpetas + 4 comandos + bash + apt + apt-get + dpkg
    En fedora: carpetas + 4 comandos + bash + yum + dnf + rpm
    En alpine: carpetas + 4 comandos + ash + apk


### Qué leches es eso del tag:
Es un texto arbitrario que ponen los que crean la imagen para de alguna forma identificarla.
Suele incluir:
- La versión del software que viene instalado en la imagen.
- En ocasiones la imagen base sobre la que está construida (ubuntu, debian, alpine, fedora...)
- Software adicional que viene dentro y pueda ser relevante


Hay 2 tipos de tags:
- Fijos   
- Variables


nginx:latest  <- Tag variable. Apuntará a la última imagen/versión de nginx que hayan publicado.
nginx:stable  <- Tags variables. Apuntan a la última imagen/versión de nginx que sea estable
nginx:nightly <- Tags variables. Apuntan a la última imagen/versión de nginx que esté en desarrollo
nginx:1       <- Tag variable. Apuntará a la última imagen/versión de nginx 1.x.x
nginx:1.29.   <- Tag variable. Apuntará a la última imagen/versión de nginx 1.29.x
nginx:1.29.1  <- Tag fijo. Siempre apuntará a la misma imagen/versión de nginx

Y hay que tener cuidao al elegir!

Cada equipo que publica imágenes usa los tags que le sa la gana... ás o menos están estandarizados... pero por convención. Casi todo el mundo usa versionados... y latest(aunque no es obligatorio)


nginx:1       <- Tag variable. Apuntará a la última imagen/versión de nginx 1.x.x
        Hoy puede apuntar a la 1.29.1 y mañana a la 1.34.3...
            Viniendo nueva funcionalidad QUE NO USO... y que puede traer nuevos bugs.

nginx:1.29.   <- Tag variable. Apuntará a la última imagen/versión de nginx 1.29.x
        ES GUAY. Siempre apunta a la versión que trae la funcionalidad que uso (1.29)... pero con la mayor cantidad de bugs arreglados posible (1.29.x)
            1.29.1----> 1.29.2

nginx:1.29.1  <- Tag fijo. Siempre apuntará a la misma imagen/versión de nginx
        ES DEMASIADO CONSERVADORA.. NO ESTA MAL.


## Esquema semántico de versionado (SemVer)

Los programas que siguen esta forma de versionado, usan 3 números separados por puntos:
va.b.c

            ¿Cuándo se incrementa cada número?
MAJOR   a       Breaking changes: Cambios que rompen compatibilidad con versiones anteriores
                        Oracle 11 -> Oracle 12... no me garantizan que lo que antes tenia en 11, me funcione en 12.
                        Quizás necesite hacer cambios... o no.
MINOR   b       Nueva funcionalidad
                Funcionalidad marcada como obsoleta (deprecated)
                    + Opcionalmente, pueden venir bugs arreglados
PATCH   c       Con arreglos de bugs: Bugfixes

# Qué es Linux? 

Un kernel de SO.
Un SO que esté corriendo un kernel Linux.
Windows corre un kernel Linux? Y tanto que si... de forma nativa... en Características de Windows (En el disco de instalación de Microsoft) WLS2 (Windows Subsystem for Linux)

GNU/Linux: Ubuntu, Debian, Fedora, CentOS, RedHat, SUSE...
Android:   
Windows 10/11: WSL2 (Windows Subsystem for Linux)


# Windows? 

Familia de SO de Microsoft: Windows 3, 95, XP, 10, 11, 2019 Server, 2022 Server...

# Qué era Unix?

UNIX ERA UN SISTEMA OPERATIVO de la americana de telecomunicaciones AT&T (lab bell) en los 70.
Problemón... llegó a haber más de 400 versiones de UNIX... y empezaron a presentar incompatibilidades entre ellas.
Montarón entonces (por separado) 2 estandares para controlar esto: 
- POSIX (IEEE) -> Portable Operating System Interface
- SUS (The Single Unix Specification) -> The Open Group
Empezando los 2000... AT&T paró el desarrollo de UNIX y vendió los derechos a The Open Group.

# Qué es Unix?

UNIX NO ES UN SISTEMA OPERATIVO.
Es cualquier sistema operativo que cumpla los estandares POSIX y SUS.

HP ----> HPUX (UNIX®)
IBM ----> AIX (UNIX®)
ORACLE -> Solaris (UNIX®)
APPLE --> macOS (UNIX®)

Linux... no es un SO UNIX® ... de hecho ni siquiera es un SO... es un kernel.

BSD386 -> FreeBSD, OpenBSD, NetBSD, MacOS
GNU ----> Tenían de todo.. hasta juegos.
            UN SO no es un programa... son miles de programas.juegos(chess, gnome, gedit, bash, gcc, make, vim...)
            Solo hubo un programa que no valieron para montar: el kernel.
Encabronao con la situación de no tener un SO GUAY gratis, llego Linus Torvals y montó el kernel Linux.
Y pasó lo que tenía que pasar: GNU + Linux = GNU/Linux
---


- Forma de empaquetar una app! IMAGEN DE CONTENEDOR... u otras cosas
- Cerrado
- No afecta al Sistema Operativo... o si...
- Entorno personalizado con un SO que no tiene que ser el del host.. y sus librerías.
  Como si fuera una MV dentro del SO. 


---
Dentro de POSIX:
- Una estructura de carpetas:
      /
          bin/
          boot/
          dev/
          etc/
          home/
          lib/
          media/
          mnt/
          opt/
          proc/
          root/
          run/
          sbin/
          srv/
          sys/
          tmp/
          usr/
          var/
- Permisos de archivos y carpetas
- Usuarios y grupos
- Señales y procesos
- Comandos básicos de comunicación con el kernel: sh, cp, mv, ls, mkdir






















---





# Openshift. Redhat DO180 -> 280

Realmente es un curso de Kubernetes.

## Kubernetes? Openshift?

- Orquestador de contenedores
- Gestor de GESTORES de contenedores


---

El contenedor, al arrancar, pilla IP... de dónde?

                                                        
                                                          MenchuPC
                                                            |
                                                        172.31.15.17
                                                            |
   +--------------------------------------------------------+-Red de Amazon
   |
 172.31.15.203:9999 -> nginx(172.17.0.3:80)
   |
 IvanPC --172.17.0.1------------------+---------------Red por defecto de Docker 172.17.0.0/16
   |                                  |
 127.0.0.1 (localhost)             172.17.0.3:80
   |                                 nginx
   loopback (127.0.0.0/8)

# Al descargar la imagen del contenedor...

HOST
 /
    bin/
    etc/
    var/
        lib/
            docker/
                images/
                    nginx:1.29/   <--------------   Engañar a un proceso, haciéndole creer que una carpeta X
                                                    es la raíz del sistema de archivos es algo que llevamos 
                                                    haciendo 40-50 años.

                                                    chroot
                                                    Esa carpeta es INALTERABLE! y se comparte entre contenedores.. lo cual es super-eficiente.
                        bin/
                        var/
                        etc/
                            nginx/
                                nginx.conf
                        tmp/
                        home/
                        var/
                             www/
                                 html/
                                     index.html
                containers/
                    <ID del contenedor>/
                        etc/
                            nginx/
                                nginx.conf          MIO
                        var/
                            log/
                                nginx/
                                    access.log     
                                    error.log      
    tmp/
    home/

# PREGUNTA:

Los contenedores tienen persistencia de los datos? CLARO!
Y cuando se apagan NO SE VAN!

La persistencia es la misma que en las VMs o en los SO a hierro:
- Si paro una vm y la arranco, pierdo datos? NO
- Si apago fisicamente una máquina y lo vuelvo a encender, pierdo datos? NO
- Si paro un contenedor y lo vuelvo a arrancar, pierdo datos? NO

Si borro una VM.. pierdo datos? SI
Si borro una máquina física... pierdo datos? SI
Si borro un contenedor... pierdo datos? SI
Claro... la diferencia es que al trabajar con contenedores, borramos contenedores como churros... un contenedor, lo podemos borrar 30 veces al día! HABLO DEL ENTORNO DE PRODUCCIÓN !
Y entonces, necesito buscar una forma de guardar los datos, tras la muerte del contenedor.
Alternativas: 
- mount al host         |
- mount carpeta nfs      > VOLUMEN
- mount iscsi           |

KUBERNETES:
Es una herramienta que controla/opera/administra un entorno de PRODUCCIÓN basado en contenedores.

VSPHERE:
Es una herramienta que me permite controlar/operar/administrar un entorno de PRODUCCIÓN basado en VMs.

