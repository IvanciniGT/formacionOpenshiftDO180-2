# Contenedores

## Imágenes de Contenedores
## Comunicaciones / Redes virtuales
## Volúmenes:
- Compartir info entre contenedores
- Persistencia de datos tras la eliminación de contenedores
- Inyección de ficheros y carpetas al filesystem del contenedor (configuraciones)

# Kubernetes

Nos permite usar un lenguaje DECLARATIVO para explicitar un entorno de producción (basado en contenedores).

A Kubernetes le vamos configurando cosas que queremos tener en nuestro entorno de producción. Eso lo hacemos cargando objetos de configuración.
Kubernetes trae como 30 tipos de objetos de configuración diferentes.... Mediante plugins (operadores) podemos ampliar el número de tipos de objetos disponibles (CRD=Custom Resource Definition)

- Namespace:            Son grupos lógicos de recursos/objetos que gestiono bajo una cuenta de usuario, 
                        con limitaciones (potenciales)
- Pod:                   Conjunto de contenedores que:
                        - Corren en el mismo host
                          - Comparten configuración de red
                          - Pueden compartir volúmenes de almacenamiento local
                        - Se gestionan juntos (y escalan juntos)
- ConfigMap / Secret:   Permiten almacenar información de configuración y secretos de forma segura, parámetros.
                        Cada parámetro iba identificado por una clave única.
- Service
  - ClusterIP           IP de balanceo + entrada en el DNS interno de Kubernetes
  - NodePort            ClusterIP + NAT a nivel de cada host apuntando a esa IP (en un puerto > 30000)
  - LoadBalancer        NodePort + Gestión automática de un balanceador de carga externo compatible con Kubernetes
- Ingress               Reglas de enrutamiento para servicios dentro del cluster que configuramos en el proxy inverso
                        (IngressController)

Ingress Controller: Proxy reverso

## Plano de control de Kubernetes

Estos son los programas que componen Kubernetes. Kubernetes no es un programa.. Son muchos.
Los que no instalamos como contenedores:
- kubelet               Habla con el gestor de contenedores de cada host
Los que si instalamos como contenedores:
- api-server            La puerta de entrada al cluster. Todo cliente se comunica con el cluster a través de él.
- controller-manager    El cerebro principal.. el que monitoriza, gestiona la BBDD interna...solicita que se creen pods...
- scheduler             Se encarga de programar los pods en los nodos del cluster.
- etcd                  Base de datos distribuida que almacena la configuración del cluster.
- coreDns               Proporciona servicios de DNS para el cluster.
- kubeProxy             Dar de alta reglas en netfilter de cada host... para los servicios.




$ kubectl VERBO TIPO_OBJETO props

TIPO_DE_OBJETO: Node, Pod, Service,  Namespace, ConfigMap, Secret, Ingress...
VERB: get               listado
      describe          detalles de uno
      delete            eliminar

$ kubectl apply -f FICHERO.yaml

---

# Deployment

> Plantilla de pod + Número inicial de réplicas

# Statefulset

> Plantilla de pod + Número inicial de réplicas + ....

# DaemonSet

> Plantilla de pod de la que kubernetes genera una réplica en cada nodo del cluster.

---

> Ejemplo wordpress

    Apache + PHP + WordPress
    MariaDB




    MariaDB 1 \              <  apache 1    \
    MariaDB 2  >   Service   <  apache 2     >  Service (ClusterIP) >  Ingress (regla de proxy inverso)    <    Menchu
    MariaBD 3 /              <  apache 3    /


    Si menchu guarda un dato en la BBDD, como está en cluster Activo/Activo, ese dato se guardará en 2 de las 3 réplicas de la BBDD. Y cada BBDD tiene sus propios archivos de datos.

    Habitualmente una BBDD guarda los datos de una tabla en un único fichero o en varios? En uno.
    (A no ser que la tabla esté particionada horizontalmente, pero eso es otra historia)
    Si tuviera un fichero compartido entre todas las instancias de MariaDB... podrían todas ellas, estar simultáneamente escribiendo en el mismo fichero? No.

    Qué pasa con los apaches? Menchu sube un PDF... los metadatos (título, tamaño... etc) se guardan en la BBDD, pero el fichero PDF se guarda en el filesystem del contenedor apache.
    Ese archivo se tiene que guardar en cada apache o en uno... como va esto?
    O lo guardo en todos, o lo guardo en un sistema de ficheros compartido entre todos los apaches. El hecho es que si menchu acceder luego a otro apache, tiene que ser capaz de encontrar el fichero PDF que subió.
    Puede ocurrir el caso que varios apaches traten de guardar o modificar el mismo fichero a la vez? NO
    Wordpress impide eso. El archivo no se guarda con el ombre que lo suben... se guarda con un identificador único, que se genera.
    Si luego suben otro archivo con el mismo nombre, se guarda con otro identificador único.
    Si se modifica ese archivo, se guarda una nueva versión del mismo con otro identificador único... y se borra el viejo.
    Los wordpress no van a actuar nunca sobre el mismo fichero.... pero eso por cómo está diseñado el wordpress.

    Si se cae un apache, como es el nuevo apache que tengo que levantar? Como son todos los apaches entre si? IDENTICOS
    Si se cae un mariaDB, como es el nuevo mariaDB que tengo que levantar? Como son todos los mariaDB entre si? DIFERENTES
    Si se cae el mariadb2... donde tenía guardados una colección de datos X, puedo levantar otra instancia del mariadb3? o del mariadb1? NO... lo que necesito es levantar/crear otro mariadb2...     que es el que tiene los datos que he perdido... los del mariadb1 y los del mariadb3 ya están disponibles... los que me faltan son los del mariadb2, que es el que se ha caído.

    Los mariadb necesitan cada uno su propio volumen de almacenamiento persistente.
    Los apaches pueden compartir un único volumen de almacenamiento persistente.... no entran en conflicto.
    No tengo 3 wordpress ni 2 tratanndo de escribir en el mismo fichero a la vez.
    Cosa que las BBDD si querrían hacer.

    Por cierto:
    - iSCSI: Permite montar una lun (un volumen) en varios equipos en modo lectura/escritura a la vez. SI ... depende.
                Bloque
    - NFS:   Permite montar un volumen en varios equipos en modo lectura/escritura a la vez. SI
                Fichero
    Querría yo que si crease un volumen para las BBDD se pudiera entregar a varios mariadb a la vez? Ni que fiuera por error? NO.

    Tipos de almacenamiento:
    - Bloque (DISCO)
    - Fichero (NFS)
    - Objeto (S3)

Todo esto vamos a tener que configurarlo en kubernetes.

---

# Volúmenes en Kubernetes

Para qué sirven los volúmenes:
- Compartir datos entre contenedores                                        EMPTY_DIR: 
                                                                            Crea una carpeta a nivel del host inicialmente 
                                                                            vacía, que puede compartir entre los contenedores del pod.
- Inyectar configuraciones/Archivos/carpetas al filesystem del contenedor   CONFIGMAPS y SECRETS
                                                                            HOSTPATH:
                                                                                Monta una carpeta/archivo del host en el filesystem del contenedor.. uso muy particular:
                                                                                MONITORIZACION: Mete la carpeta /proc del host en el contenedor.
- Persistencia de datos                                                     CIENTO Y LA MADRE:
                                                                            nfs, iscsi, ceph, glusterfs, portworx, rook/ceph, longhorn, para cabinas especiales (HUAWEI, EMC2, NETAPP, etc), aws, gcp, azure, 
En Kubernetes los volúmenes se definen a nivel del POD... pero luego se montan a nivel del contenedor.


---

Antiguamente, teníamos 2 formas de operar:
1. Que el administrador diera de alta 40 volúmenes... y los dejase ahí disponibles, para cuando los pidiera un usuario (pvc)
    Mucho trabajo
    Mal dimensionamiento
2. Que el usuario lanzase la pvc y en paralelo una petición al CAU... para que le dieran de alta el volumen.
    Mucho tiempo
    Mucho papeleo
    Mucho trabajo para el CAU

Que hacemos hoy en día: MONTAR UN PROVISIONADOR DE ALMACENAMIENTO DINÁMICO.
Eso es un programa que está monitorizando 24x7 las pvc que se lanzan al cluster.
Cuando una PVC es lanzada, ese programa mira sus datos.
Estos programas se asocian a un storageClass (una clase de almacenamiento).
Por ejemplo, tengo un programa (provisionador) para volúmenes de tipo 'rapidito-redundante'
Y tendré otro para volúmenes de tipo 'lento-barato'.. y otro para 'redundante-encriptado'... etc etc etc.

Cuando se lanza una pvc de un tipo concreto, el provisionador de ese tipo concreto:
- Crea en el backend de almacenamiento el volumen que se ha pedido (con las características CONCRETAS que se han pedido)
- Registra en Kubernetes ese volumen (pv)
- Asocia ese volumen (pv) a la pvc que se ha lanzado, para que no haya opción de liarse el Kubernetes.

De estos programas hay cientos.
AWS me da los suyos.
Azure me da los suyos.
GCP me da los suyos.
Servidores NFS... con subcarpetas.
Cabinas de almacenamiento comerciales: HUAWEI DORADO V6... viene con provisionador dinámico (drivers) para Kubernetes.

Yo (administrador del cluster) instalo el/los provisionador/es que quiero.

Habitualmente en mi cluster tendré 3/4 tipos de almacenamiento diferentes... cada uno con su provisionador dinámico:
Eso son los storage Class.
Al crear una pvc puedo poner en storageClass lo que me de la real gana: `menchu`
Y siempre que haya un volumen cuyo storageClass sea `menchu`, lo vincula.

Pero!!!!! Kubernetes da la posibilidad de pre-registrar storageClass y asociales a un provisionador concreto.