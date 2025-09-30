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