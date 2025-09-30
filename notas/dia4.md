
# Trabajando con Kubernetes...

Lo que hacemos es definir/declarar un entorno de producción (basado en contenedores). Lo hacemos mediante ficheros YAML, usando un esquema propio de Kubernetes.

Vamos ir definiendo objetos (elementos/recursos) que queremos en ese entorno de producción.
Cada objeto/recursos se definen en su propio documento YAML.

Como en YAML, dentro de un fichero podemos definir múltiples objetos/recursos, podemos tener un entorno de producción completo en un único fichero YAML (en la práctica no será así... ya que esos objetos serán creados por diferentes equipos... y cada uno tendrá su propio fichero YAML).

Kubernetes viene de serie con unos 30 tipos de objetos/recursos. Mediante operadores (programas que se ejecutan en el clúster) podemos añadir más tipos de objetos/recursos (CRDs - Custom Resource Definitions).

De los de kubernetes:
- Node
- Namespace
- Secret
- Configmap
- Pod
- PersistentVolume
- PersistentVolumeClaim
---
Deployment
StatefulSet
DaemonSet
Service
Ingress
Role
ClusterRole
RoleBinding
ClusterRoleBinding
ServiceAccount
HorizontalPodAutoscaler
...

Estructura típica de un archivo YAML de Kubernetes:
```yaml
kind:          TipoDeRecurso
                    # V ^ V
apiVersion:    libreria/version

metadata:
    name:      nombre-del-recurso

spec:
    ....
```

---

# Arquitectura de un cluster de Kubernetes

     192.168.1.150:80
        http://192.168.1.101:30001
        http://192.168.1.102:30001
        http://192.168.1.103:30001
        http://192.168.1.104:30001
        http://192.168.1.105:30001                                          http://app1.miempresa.com
         |                                                                         |                
      Balanceo de carga              app1.miempresa.com                         MENCHU - PC
         |                              |    -> 192.168.1.150                      |
      192.168.1.150                  DNS Externo                              192.168.1.201
         |                              |                                          |
+--------+------------------------------+------------------------------------------+---red de mi empresa (192.168.1.0/24)
|
+- 192.168.1.101-NodoA (master: ControlPlane)
||                Linux
||                   Netfilter
||                        10.10.20.2:3307 -> 10.10.10.20:3306
||                        10.10.20.3:8888 -> 10.10.10.19:80, 10.10.10.21:80
||                        10.10.20.4:80   -> 10.10.10.22:80
||                        192.168.1.101:30001 -> 10.10.20.4:80
||                kubelet
||                crio | containerd
||......10.10.10.2 - apiServer1
||......10.10.10.3 - controllerManager1
||......10.10.10.4 - coredns1
||                        bbdd -> 10.10.20.2
||                        wordpress -> 10.10.20.3
||                        proxy-reverso -> 10.10.20.4
||......10.10.10.5 - kubeproxyA
||......10.10.10.6 - etcd1
||
|+- 192.168.1.102-NodoB (master: ControlPlane)
||                Linux
||                   Netfilter
||                        10.10.20.2:3307 -> 10.10.10.20:3306
||                        10.10.20.3:8888 -> 10.10.10.19:80, 10.10.10.21:80
||                        10.10.20.4:80   -> 10.10.10.22:80
||                        192.168.1.102:30001 -> 10.10.20.4:80
||                kubelet
||                crio | containerd
||......10.10.10.7 - apiServer2
||......10.10.10.8 - controllerManager1
||......10.10.10.9 - coredns2
||                        bbdd -> 10.10.20.2
||                        wordpress -> 10.10.20.3
||                        proxy-reverso -> 10.10.20.4
||......10.10.10.10 - kubeproxyB
||......10.10.10.11 - etcd2
||
|+- 192.168.1.103-NodoC (master: ControlPlane)
||                Linux
||                   Netfilter
||                        10.10.20.2:3307 -> 10.10.10.20:3306
||                        10.10.20.3:8888 -> 10.10.10.19:80, 10.10.10.21:80
||                        10.10.20.4:80   -> 10.10.10.22:80
||                        192.168.1.103:30001 -> 10.10.20.4:80
||                kubelet
||                crio | containerd
||......10.10.10.12 - controllerManager2
||......10.10.10.13 - scheduler1
||......10.10.10.14 - kubeproxyC
||......10.10.10.15 - etcd3
||
|+- 192.168.1.201-Nodo1 (workers)
||                Linux
||                   Netfilter
||                        10.10.20.2:3307 -> 10.10.10.20:3306
||                        10.10.20.3:8888 -> 10.10.10.19:80, 10.10.10.21:80
||                        10.10.20.4:80   -> 10.10.10.22:80
||                        192.168.1.201:30001 -> 10.10.20.4:80
||                kubelet
||                crio | containerd
||......10.10.10.16 - kubeproxyD
||......10.10.10.22 - pod-proxy-reverso
||                      - contenedor-proxy-reverso
||                           proceso nginx (Abre puerto 80 en la IP del pod)
||                                  REGLAS DE PROXY REVERSO:
||                                     app1.miempresa.com -> wordpress:8888   < INGRESS = Regla de proxy reverso
||......10.10.10.19 - pod-apache
||                      - contenedor-apache
||                           proceso apache (Abre puerto 80 en la IP del pod)
||                                   10.10.10.19:80 (para el apache)
||                                      Fichero de configuración... donde necesite apuntar al mariadb:
||                                         HOST: bbdd          , PUERTO: 3306       FUNCIONARIA? √ 
||                                              Problemas:
||                                                  Si el pod del mariadb se cae.. y hay que recrearlo en otro nodo...
||                                                          cambiará su IP
||                                                  Conozco a priori la IP del mariadb?  NO
||
+- 192.168.1.202-Nodo2 (workers)
||                Linux
||                   Netfilter
||                        10.10.20.2:3307 -> 10.10.10.20:3306 ¿Quién da de alta estas reglas?
||                        10.10.20.3:8888 -> 10.10.10.19:80, 10.10.10.21:80
||                        10.10.20.4:80   -> 10.10.10.22:80
||                        192.168.1.202:30001 -> 10.10.20.4:80     # Esto es lo que hace el NodePort, que no hace el clusterIP
||                kubelet
||                  v
||                crio | containerd
||......10.10.10.17 - kubeproxyE
||......10.10.10.20 - pod-mariadb
||                      - contenedor-mariadb
||                           proceso mariadb (Abre puerto 3306 en la IP del pod)
||                                   10.10.10.20:3306 (para el mariadb)
||......10.10.10.21 - pod-apache-2
||                      - contenedor-apache
||                           proceso apache (Abre puerto 80 en la IP del pod)
||                                   10.10.10.21:80 (para el apache)
||                                      Fichero de configuración... donde necesite apuntar al mariadb:
||                                         HOST: bbdd          , PUERTO: 3307       FUNCIONARIA? √ 
||
|+-----RED VIRTUAL DE KUBERNETES (10.10.0.0/16)------------------------------
|


IP FIJA: 10.10.20.2 ----> fqdn (nombre resoluble por dns)

SERVICE:
    ClusterIP: IP FIJA DE BALANCEO + entrada en el DNS INTERNO DEL CLUSTER, apuntando a esa IP FIJA
    NodePort:  ClusterIP + NAT a nivel de cada host en un puerto > 30000 apuntando a la IP Fija del ClusterIP
    LoadBalancer: NodePort + gestión automatizada de un balanceador de carga externo SIEMPRE QUE SEA COMPATIBLE
    CON KUBERNETES!!!!

        Cuando contratamos un cluster a un cloud: AWS, GCP, Azure... siempre nos dan(€€€€) un balanceador de carga compatible con kubernetes.
        El problema es cuando instalamos kubernetes en nuestras máquinas (on-premise)... en este caso, el balanceador de carga que es compatible con kubernetes es MetalLB (https://metallb.universe.tf/)


    En un cluster TIPO / ESTANDAR / EL QUE SEA de kubernetes, cuántos (% o valor absoluto) servicios de cada tipo tendremos?

                                CANTIDAD
        ClusterIP:                 Todos menos 1
        NodePort:                  0%
        LoadBalancer:              1(2...3)          <- PROXY REVERSO, por ejemplo un nginx, ha proxy , apache, envoy...
                                                        Estos proxies reversos en el mundo de kubernetes se llaman INGRESS CONTROLLERS

...
                                                    donde?
    kubectl ----> apiServer -----> ControllerManager -----> Scheduler
        crea un pod                                  <--- nodo2
                                                     -----> Kubelet (del nodo2)
                                                            crea el contenedor ----> containerd/crio
                                                            <---- contenedor creado
                                                     <---- contenedor creado
                             <---- pod creado
        <---- pod creado


NETFILTER: Componente del KERNEL de Linux que se gestionar TODO paquete que entra o sale de cualquier interfaz de red de la máquina.
    IPTABLES: LO único que hace iptables es dar una interfaz de usuario para gestionar las reglas del netfilter.

# Kubelet.

Demonio que se instala a hierro en cada nodo del cluster.... y se queda corriendo en segundo plano.
Es el responsable de hablar con el gestor de contenedores (containerd, crio) para arrancar y parar contenedores, ver los logs, ejecutar comandos en ellos...

Pero.. éste no es el único programa que hay que instalar...
La mayor parte de los programas de Kubernetes (de su "plan de control") se instalan en los nodos "master" (nodos de control)
como contenedores(pods) dentro del cluster.
- ApiServer (Es el que recibe las peticiones de los clientes)
    kubectl         ---> apiserver
    ui  (dashboard) ---> apiserver
- ControllerManager (Es el cerebro de Kubernetes)
- Scheduler (Es el que decide en qué nodo se ejecuta cada pod)
  - Quiero ejecutar una app que necesita una GPU muy potente.
      App1 necesita GPU ---> El scheduler busca un nodo que tenga GPU y que esté libre
        AFINIDADES
  - Hay máquinas que queremos tener de uso restringido (tengo unas máquinas con GPUs)
      App2  ----> El scheduler podría asignarlo a un nodo que tenga GPU sin necesidad. 
        TOLERACIONES/TINTES
  - Máquinas que tengan unos HDD muy rápidos, cpus más potentes
    O CPUs de cierta arquitectura (ARM, x86)
  - Tengo 5 nodos... y quioero desplegar 3 copias de la app1... Me da igual la forma en que se 
    repartan entre los nodos?
        Nodo1 <- App1(copia 1), App1(copia 2), App1(copia 3)
        Nodo2-Nodo5 <- vacíos
- ETCD (Base de datos donde se guardan las definiciones de los objetos del cluster y su estado)
- CoreDns (Servidor de DNS para comunicaciones internas del cluster)
- KubeProxy ... De los demás tenemos 2 réplicas en el cluster(instaladas en 2 nodos master)
            Es quién se encarga de gestionar las reglas del netfilter (iptables) en cada nodo del cluster
            para que las peticiones a los servicios (services) se redirijan a los pods 
                 



----

ETCD, igual que muchas BBDD o indexadores (ElasticSearch), sistemas de mensajería (Kafka, RabbitMQ) funcionan en clústeres, de números impares (3, 5, 7...) para evitar empates en las votaciones.

                    T1.       T2
    mariadb1        DATO1     DATO2            < DATO1
     v
    mariadb2*       DATO1     DATO3
     v
    mariadb3        DATO2     DATO3            < DATO2


No solo por Alta disponibilidad
Sino también para mejorar el rendimiento (más réplicas, más peticiones atendidas) (ESCALABILIDAD)

    Al multiplicar la infra x3, tengo una mejora de rendimiento máxima teórica del 50%

    Con 1 máquina en 1 ud de tiempo guardado 1 dato
                    En 2 ud de tiempo guardo 2 datos

    Con 3 máquinas en 2 ud de tiempo guardo 3 datos

    Y es frustrante.... ya que además ese es el límite máximo teórico.
    Cuando meta las comunicaciones entre las máquinas, y empiece a sincronizar tareas entre ellas...





## Comunicaciones dentro de un cluster (y fuera del cluster)

Route: Objeto de OpenShift (Kubernetes) que permite definir rutas de acceso a las aplicaciones desplegadas en el cluster desde fuera del cluster.

       Ingress + Configuración automática de DNS Externo... basada en subdominios
                    Dirección del cluster: micluster.miempresa.com
                    Dirección de la app1: app1.micluster.miempresa.com
                    Dirección de la app2: app2.micluster.miempresa.com
                    Dirección de la app3: app3.micluster.miempresa.com


INGRESS Es una regla de proxy reverso, en una sintaxis que estandariza KUBERNETES
INGRESS CONTROLLER ES:
- Un proxy reverso (nginx, apache, envoy...)
- Un programa que automáticamente genera reglas en la sintaxis de ese proxy reverso
  a partir de las reglas definidas en los objetos INGRESS de kubernetes.