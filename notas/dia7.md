

# Volúmenes

- Compartir datos entre contenedores                                        EMPTY_DIR
- Inyectar configuraciones/Archivos/carpetas al filesystem del contenedor   CONFIGMAP y SECRET
                                                                            HOSTPATH
- Persistir datos entre reinicios de contenedores                           NFS, ISCSI, Ceph,
                                                                            GlusterFS, Portworx, Rook/Ceph, Longhorn, para cabinas especiales (HUAWEI, EMC2, NETAPP, etc), AWS, GCP, Azure, 

Los volúmenes se definen a nivel de pod/template de pod... y se montan a nivel de contenedor.
Y ese documento de manifiesto (POD/TEMPLATE DE POD) es creado por el desarrollador. Y ese personaje no tiene los datos concretos de los volúmenes. Aquí salen:
- Las PVC (peticiones de volúmenes persistentes), las rellena el desarrollador, en un lenguaje próximo a él (sus necesidades... quiero un volumen de 10GiB, que sea rápido y redundante)
- Los PV (volúmenes persistentes), las rellena el administrador del cluster. Ahí es donde metemos los datos concretos del almacenamiento (los del backend... allí debe de existir ya, cuando se da de alta la pv)

            POD ---> PVC <> PV ---> BACKEND DE ALMACENAMIENTO
                      |      ^                      ^
                      |      +----------------------+
                      |                             |
                      +> tipo: StorageClass <---- Provisionador de almacenamiento dinámico  


Que el administrador cree volúmenes a cascoporro anticipadamente, no es práctico.
Tampoco que desarrollo tenga que esperar a que el administrador le cree el volumen en el backend y lo registre en el cluster (pv) cuando el lanza la pvc.



DESARROLLO (Metodología en cascada para gestionar su proyecto)
    Tomaban requisitos
    Hacían diseño
    Hacían desarrollo
    Hacían pruebas
    Listo! ----------> SISTEMAS (Operaciones)
                        Toma lo que había montado desarrollo
                        Lo pasa a producción
                        Lo mantiene 24x7... con legiones de Operadores con un busca vibrando cada vez que había un problema



DESARROLLO es quién pasa a producción... de forma automatizada.
Hay algo que hace desarrollo que desencadena un paso automático a producción.
Y SISTEMAS (Operaciones) DESAPARECE! No están en la empresa... estarán en otra!


Lo que tenemos hoy en día, desde el punto de vista de Administrador de Sistemas, Operaciones...
es a 3 personas administrando el cluster CORPORATIVO de la empresa... donde hay 300 apps.
La operación del entorno la hace kubernetes.

Esas personas crear un NAMESCAPE para cada proyecto (o entorno: dev, pre, prod)
LIMITAN EL USO DE RECURSOS (cpu, memoria, almacenamiento, número de pods, etc etc etc) a cada namespace.
Crean un usuario, que vinculan a ese namespace.
Y le dan permisos (role, rolebinding) para que pueda hacer lo que necesite en ese namespace.

Y la gestión de los despliegues, la definición de las cosas concretas que necesita cada app, la hace desarrollo.

---

DEVOPS! Es la cultura de la automatización de trabajos.

Pero esa palabra también se usa para definir a un perfil profesional.
DEVOPS es el administrador de sistemas del siglo XXI, reciclado.
Un tio que es capaz de manejar clouds (AWS, GCP, Azure)
De automatizar creación de entornos virtuales en en clouds (TERRAFORM, ANSIBLE)

Hay que montar estos procesos de automatización que usan los desarrolladores..
Y los desarrolladores no saben de eso. Me toca a mi (que sé de despliegues, de entornos de producción, montar estas cosas).

Y hoy en día estamos con arquitecturas de microservicios ->
 Una app de antes, era un monolito, siguiendo una metodología de desarrollo en cascada se instalaba 3 veces al año.
 Hoy en día, esa app son 50 microservicios, que se actualizan 3 veces al mes... 150 despliegues al mes / 20 = 7 despliegues al día. A mano no puedo... pero hay que automatizarlo... y quién lo hace? DEVOPS!

 Antes yo (Administrador de sistemas) desplegaba a mano lo que me pasaba desarrollo.
 Ahora yo (DEVOPS) monto los procesos de automatización de despliegue. Es decir, hago un programa que hace lo que yo antes hacía a mano.
 Sigo ocupado! Ese programa lleva horas.
 Lo que pasa es que antes hacíamos 10 despliegues al mes... y ahora 150. Esto nos permite ser más competitivos.

 El tester de hace 15 años hacía pruebas.
 El tester de hoy en día hace programas que prueben las apps automáticamente.
 Y es que antaño se hacían pruebas a los sistemas cuando iban a producción.... 3 veces al año.
 Hoy en día, las apps van a producción 150 veces al mes... y hay que probarlas 150 veces al mes.


Un Oracle gordo, sigue instalado en un servidor gordo de Oracle (Exadata, T8)

DEVOPS Es automatizar.

    Los desarrolladores hoy en día configuran programas que automatizan tareas que antes hacían a mano.. COMPILAR, EMPAQUETARLO.  Maven

    Los testers hoy en día configuran programas que automatizan tareas que antes hacían a mano.. PROBARLO. Selenium, Appium, SonarQube, Karate, JMeter, Karma, etc etc etc SOAPUI, Postman.
    
    Los administradores de sistemas de automatización hoy en día configuran programas que automatizan tareas que antes hacían a mano.. DESPLEGARLO. Jenkins, GitLab-CI, ArgoCD, Ansible, Terraform, Vagrant, Puppet, Clouds, Kubernetes, Openshift, Docker, Helm, etc etc etc ---> DEVOPS


                    Logstash > ELASTIC SEARCH < KIBANA
                    URL
                     ^
Tomcat               |
 Filebeat -----------+
 Métricas
  (Peticiones, tamaño de la cola, tiempos de respuesta, etc etc etc)
    |
    + Exportador de métricas a Prometheus <- Grafana
                                nombrecito 

DEVOPS


Desarrollo / Negocio
 Equipo de desarrollo que hoy en día tenemos en la empresas... y operan bajo lo que llamamos metodologías ágiles (SCRUM, KANBAN, SAFE, etc etc etc)


---

# Gestión de recursos en Kubernetes

Podemos Y DEBEMOS limitar el consumo de recursos (CPU, MEMORIA, ALMACENAMIENTO) que puede usar cada NAMESPACE.... y de cada contenedor!

El administrador del cluster limita el uso de recursos a nivel de NAMESPACE.
Y el desarrollador limita el uso de recursos a nivel de CONTENEDOR (NO SE HACE A NIVEL DE POD)

Hay 2 datos que damos: requests y limits

Los requests son los que usa el scheduler para decidir en qué nodo mete el pod.

            TOTAL           COMPROMETIDO      USO
            CPU    RAM      CPU    RAM      CPU    RAM
    NODO 1   10     20      10      20       2     20
     Pod1 / (C1: Programa1)  5      10       5     10
     Pod2 / (C1: Programa1)  5      10       0      0     sigterm... -> timeout -> sigkill
    NODO 2   10     20       5      10       1      1
     Pod3 / (C1: Programa1)  5      10       1      1

Pod / (C1: Programa1)   Request: 5 Cores y 10 GiB RAM (lo que se debe garantizar)
                        Limits: 10 Cores y 12 GiB RAM (lo que me permites si hay)
Otra réplica de lo mismo! Pod2 / (C1: Programa1)   Request: 5 Cores y 10 GiB RAM
Otra réplica de lo mismo! Pod3 / (C1: Programa1)   Request: 5 Cores y 10 GiB RAM

La recomendación con el limit de ram es ponerlo igual que el request.
-Xmx de la JVM = MAXIMA
-Xms de la JVM = INICIAL 