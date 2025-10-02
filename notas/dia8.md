
# Kubernetes

Kubernetes es una base... sobre la que ir montando cosas.
1. Plugin de red virtual para el cluster
2. Provider de almacenamiento para el cluster (montaremos varios)
3. Ingress Controller (Proxy inverso) (al menos 1)
4. Monitorización MetricServer -> (Prometheus + Grafana)
5. MetalLB (Balanceador de carga externo)
6. Gestión de DNS Externo (external-dns)
7. Alguien que genere certificados TLS (cert-manager)
8. Dashboard web (opcional)

Esto es lo que resuelve OPENSHIFT.
Es un Kubernetes empaquetado por la gente de Red Hat, con todo esto ya instalado y configurado. Y otras 300 cosas!


---

1 app que antes era un .ear que desplegábamos en un Weblogic, o e 3 weblogic, hoy en día son 50 microservicios .war, que van cada uno en su tomcat, con su propio autoescalado... En un momento del tiempo podemos tener 400 tomcats de una app.

Y ahora queremos tener esos 400 tomcats por https... y cada uno con su propio certificado SSL.... generado por una CA propia de la empresa.... y reconociéndose entre ellos.

Y regenerando los certificados cada 3 meses.

Hay algunas herramientas que nos ayudan con estas cosas: ISTIO, LINKERD, KUMA. Son especiales para kubernetes.

ISTIO lo que hace es un man-in-the-middle. Nos autohackeamos.

Se inyecta un proxy en cada pod del cluster. 

    POD App1
        C1: Tomcat
             ^ localhost:8080
        C2: Proxy ENVOY (sidecar)
             Con reglas de netfilter, cualquier petición que llegue al pod, pasa primero por el proxy ENVOY.

    POD 1                   POD 2
        C1: Tomcat          C1: Weblogic
            ^                  v             <- Localhost (no pisan red. http)
        C2: ENVOY  <------   C2: ENVOY (ssl)
                     ^^^^
                     https


Si tengo en mi app 400 tomcats, al ponerle istio, tengo además 400 proxys ENVOY.

Por un lado, al tener todas las comunicaciones interceptadas por el proxy, puedo hacer cosas como:
- Autenticación mutua entre servicios (mTLS)
- Monitoreo centralizado de todas las comunicaciones entre servicios.
- Securización (FIREWALL A NIVEL DE RED): Network Policies
- Pasos a producción:
  - Web principal del banco ---> Servicio de simulación de hipotecas v1 -> v2
  Monto los 2 en paralelo: v1 y v2
  - Cuando la web redirige a v1, el proxy ENVOY lo detecta y el 1% del tráfico lo manda a v2. Lo miro... va bien... 5%... 10%... 50%... 100%
  - Va mal.. he jodido al 1% de los usuarios... Desactivo interceptor y resuelto


SELINUX? 

- Desactivar
- Modo permisivo    <<<. Permite todo, pero me va generando las reglas de seguridad.
- Modo forzado <<<<< Aplican las reglas de seguridad.