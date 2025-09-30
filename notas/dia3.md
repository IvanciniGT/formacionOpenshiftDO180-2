
# Contenedor

- Imagen de contenedor
- Volúmenes:
  - Para qué sirve?
    - Persistencia de datos
    - Compartir datos entre contenedores 
    - Inyectar archivos y carpetas al Filesystem del contenedor (configuración)

# Kubernetes

Gestor de Gestores de Contenedores.
Definir entornos de producción (HA, Escalabilidad, Seguridad) de forma DECLARATIVA!
Y Kubernetes CREA ese entorno y lo OPERA 24x7.


En kubernetes lo único que hacemos es cargarle configuraciones ... de cosas... mediante esos ficheros yaml.
Y hay más de 20 cosas (casi 30) que podemos definir en un kubernetes:

- Node         En kubernetes, un nodo es una máquina, virtual o física, que forma parte del clúster de Kubernetes y que puede ejecutar a instancias del cluster procesos en contenedores.
  Nosotros no definimos nodos. Los nodos los iremos añadiendo al cluster con ayuda de una herramienta: kubeadm
  Es de las pocas cosas que no definimos mediante ficheros yaml. Aunque tienen un fichero YAML de definición, que a veces me interesa ver.
- Namespace    Es un espacio de nombres únicos.
            Para qué sirve? Para crear particiones lógicas dentro de un mismo cluster.
            Particiones para qué? Para lo que quieras: Separar entornos (dev, pre, prod), separar proyectos, separar equipos... Separar despliegues de apps diferentes.
            En muchas ocasiones ambas cosas a la vez.
            Los namespace pueden tener limitaciones de recursos (cpu, memoria, almacenamiento, número de pods, etc etc etc). A los usuarios de un cluster, les puedo limitar el trabajo a un namespace concreto.
            CASI TODOS (no todos... 90%) de los objetos de kubernetes deben estar dentro de un namespace.
            Cada objeto que cree en el cluster tiene un NAME, un nombre, que lo identifica.
            Cada objeto lo creo en un ESPACIO DE NOMBRES (NAMESPACE). En ese espacio de nombres, el nombre del objeto debe ser único, sirve de identificador.
            ES DECIR:
            - No puedo tener objetos (del mismo tipo) con el mismo nombre en el mismo namespace.
            - Si puedo tener objetos (de distinto tipo) con el mismo nombre en el namespace.
            - Si puedo tener objetos (del mismo tipo) con el mismo nombre en diferentes namespaces.
            Los namespace son GRUPOS DE RECURSOS, que administro juntos... e independientemente del resto de grupos de recursos del cluster. 

            Cómo se define un namespace: DENTRO de un archivo YAML.
- Pod
- ---
- ConfigMap
- Secret
- Deployment
- DaemonSet
- StatefulSet
- PersistentVolume
- PersistentVolumeClaim
- StorageClass
---- 
- Job
- CronJob
---- 
- Service
- Ingress
- NetworkPolicy
---- 
- HorizontalPodAutoscaler
--- 
- ClusterRole
- ClusterRoleBinding
- LimitRange
- ResourceQuota
- Role
- RoleBinding
- ServiceAccount
- Y alguno más...


Eso en un kubernetes pelao... recien salido de la caja. Pero resulta que kubernetes es extensible, y en un cluster podemos instalar operadores, que añaden más tipos de objetos a nuestro kubernetes.
En Openshift por ejemplo... recién salido de la caja, ya tenemos Casi 150 tipos de objetos más.
Si además instalo los operadores recomendados por Red Hat, ya tengo más de 300 tipos de objetos diferentes.

---

# Entornos gráficos y Kubernetes:

Kubernetes NO TIENE ENTORNO GRAFICO PREINSTALADO POR DEFECTO.
Si hay un entorno gráfico oficial, que podemos inatalar a posteriori, que se llama DASHBOARD.

Openshift trae su propio entorno gráfico, que es mucho más potente que el dashboard oficial de kubernetes.

Ambos uno y otro los tenemos de adorno. NO NOS GUSTAN LOS ENTORNOS GRAFICOS... NO QUEREMOS ENTORNOS GRAFICOS.
NO HAGO NADA EN UN ENTORNO GRAFICO (si acaso entrar a ver cómo está todo... pero no toco NADA.

Los entornos gráficos tienen un problema: Las operaciones que hago en ellos no quedan registradas en ningín sitio... NI SON AUTOMATIZABLES.

Algo que tengo metido en un fichero, puedo aplicarlo en 5 clusters.. reaplicarlo, ver que cambió desde la última vez que lo apliqué, versionarlo en un git... etc etc etc.
Lo que rellene a manita ¿?¿?¿?¿?

NADIE USA ENTORNOS GRAFICOS EN EL MUNDO KUBERNETES.
















Las metodologías (formas de trabajo), los lenguajes de programación, las herramientas (desarrollo, pruebas, sistemas), las arquitecturas... todo ha ido (y va... y seguirá) evolucionando en PARALELO, junto con los problemas.


JAVA 1.7, 1.8 ... JSPs(servlets),  Mega Servidores de Apps (Weblogic, Websphere, JBoss, Glassfish...)
Y trabajábamos con metodologías cascada / (espiral, v).

    Toma de requisitos
        Análisis y Diseño
            Implementación
                Pruebas
                    Despliegue
                        Mantenimiento

---

Sistema de Animalitos Fermín del 2007!

Menchu
Necesito pienso para el perro
http://www.animalitosfermin.com
                                html    <---  Weblogic
                                                WebApp  ---->   BBDD
                                                 .war
                                                 .ear
                                                MONOLITO
Sistema de Animalitos Fermín
                                                300 programas .. mierdiprogramas
App Nativa Android
App Nativa iOS
Web                                             GESTION DEL PIENSO ------>. BBDD
Alexa, Siri, OK Google                  REST(JSON)
IVR                                             LOGIN
TV                                              PAGOS
App desktop más potente                         DETALLE DE LOS ANIMALITOS
                                                MICROSERVICIOS