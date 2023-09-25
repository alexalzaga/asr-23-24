# PRÁCTICA 2 ASR : VM

## Parte 1: creación de máquina de salto

1. Se crea una máquina de salto y un webserver con las especificaciones de bajo coste revisadas en clase. En este punto ambos poseen IP externa.
![maquinasp1](/p2/maquinasp1.png)

2. Creamos reglas de firewall para manejar acceso y restringir el acceso a las maquinas. Sólo se permite el acceso desde la red IP de ICAI al jump server, y desde el jump server al web server.
![firewalllp1](/p2/firewallp1.png)

3. Se comprueban las conexiones.
![sshjumpp1](/p2/sshjumpp1.png)
![sshwebp1](/p2/sshwebp1.png)

## Parte 2: introducción a los WAF

1. Retiramos la IP pública del web server. Comprobamos que no se puede establecer conexión.
![webnoIPp2](/p2/webnoIPp2.png)
![timeoutwebp2](/p2/timeoutwebp2.png)

2. Creamos el load balancer usando el certificado asr.es.crt y KEY.key generados.
![instancegroupp2](/p2/instancegroupp2.png)
![loadbalancerp2](/p2/loadbalancerp2.png)

3. Creamos un health check para el backend, y una regla de firewall para permitir a las direcciones IP indicadas por Google el acceso.
![healthfirewallp2](/p2/healthfirewallp2.png)

4. Establecemos una Cloud NAT para poder acceder a internet desde el web server y descargar nginx.
![cloudnatp2](/p2/cloudnatp2.png)

5. Comprobamos conexión.
![conexionp2](/p2/conexionp2.png)

6. Añadimos la policy de Cloud Armor con las tres reglas indicadas en el enunciado, usando configuración de Google predeterminida para XSS y SQL Injection, y estableciendo una norma que permita acceso a llamadas cuyo campo de país de origen contenga uno de los códigos indicados.
![cloudarmorp2](/p2/cloudarmorp2.png)

## Parte 3: zero trust

Para cumplir este apartado es necesario:
a) Cambiar el backend del load balancer a HTTPS.
b) Cambiar los health checks de Google para que funcionen en el puerto 443.
c) Modificar la configuración de nginx en el webserver para que sólo escuche en el puerto 443 y que use los certificados generados por nosotros.

1. Modificamos el backend del load balancer, cambiando el puerto del instance group, y el endpoint protocol del health check.
![loadbalancerp3](/p2/loadbalancerp3.png)

2. Cambiamos la regla de firewall del health check para que funcione por el puerto 443.
![healthfirewallp3](/p2/healthfirewallp3.png)

3. Mediante scp subimos asr.es.crt y KEY.key al webserver, pasando por el jump server. Modificamos /etc/nginx/sites-enabled/default para que sólo escuche por el puerto 443 y use selfconfig.conf, nuestro archivo de configuración que especifica el uso de nuestro certificado y clave.
![configp3](/p2/configp3.png)
![nginxconfp3](/p2/nginxconfp3.png)

4. Comprobamos la conexión, que ahora se debe hacer especificando el puerto en la barra de búsqueda.
![conexionp3](/p2/conexionp3.png)

## Parte 4: posibles mejoras
La seguridad se puede seguir mejorando retocando y añadiendo reglas a Cloud Armor, protegiendo distintos posibles ataques. Google ofrece reglas predeterminadas al igual que con XSS y SQL injection. Para mejorar la disponibilidad se podría usar Google Cloud CDN (Content Delivery Network) para reducir la latencia, almacenando en caché assets estáticos, o crear un plan para la recuperación contra desastres mediante la generación regular de backups o mecanismos automatizados en caso de fallos.

