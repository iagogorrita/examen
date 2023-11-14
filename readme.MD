# EXAMEN DNS - DOCKER

1. Explica métodos para 'abrir' una consola/shell a un contenedor que se está ejecutando

    Hay varias formas para abrir una consola o shell a un contenedor de docker:
    
    1.1. La primera puede ser a través del visual code con el plugin de docker y le podremos dar directamente a iniciar, dandole click derecho y start.
        ![Alt text](image-1.png)
    1.2. La segunda forma será con el comando **`docker exec -it *nombredelcontenedor* bash `**
2. En el contenedor anterior con que opciones tiene que haber sido arrancado para poder interactuar con las entradas y salidas del contenedor.

    Tiene que haber sido arrancado con stdin_open: true que mantiene la entrada estandar abierta lo que permite que el usuario interactue y tty: true que habilita un terminal simulado que nos permitira interactuar dentro del contenedor, estas son las opciones que nos permitiran interactuar con el contentedor.
3. ¿Cómo sería un fichero docker-compose para que dos contenedores se comuniquen entre si en una red solo de ellos?

    Para que dos contenedores interactuen entre si, unicamente tendremos que hacer que esten en la misma red levantando una red para que esten los dos solos y se puedan comunicar entre si. Para ello dejo la red de mi docker compose aquí:
```
networks: #nombre de nuestra red
  network: #nombre de la subred
    ipam: #el protocolo que utiliza podria ser bridge tambien
      config: 
        - subnet: 172.28.0.0/24 #la subred que utilizará
          gateway: 172.28.0.1 #el gateway de nuestra subred
```
Además obviamente tendremos que tener los dos servicios en la misma subred para que se puedan comunicar.

4. ¿Qué hay que añadir al fichero anterior para que un contenedor tenga la IP fija?

    Tendremos que ponerle a cada uno de los servicios, un network y en el tendremos que poner una ip en nuestro caso IPV4, dejo el codigo aquí:

    ```
    -Servicio de ubuntu
    ubuntu:
    container_name: asir_cliente
    #nombre del equipo de nuestro cliente
    image: ubuntu:latest
    #instalaremos la ultima imagen de ubuntu poniendole el latest
    networks:
      network: 
        ipv4_address: 172.28.0.70
        #la ip será 172.28.0.70
    -Servicio de httpd
    web1:
    container_name: asir_web1
    image: httpd:latest
    networks:
      network:
        ipv4_address: 172.28.0.80
    ```
5. ¿Que comando de consola puedo usar para saber las ips de los contenedores anteriores? Filtra todo lo que puedas la salida.
    
El comando es el siguiente:

    docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' nombre_del_contenedor

Si cambio el nombre_del_contenedor por asir_bind9 que es como se llama mi contenedor con el servicio dns me dirá la IP que tiene asignada este.

    docker inspect -f '{{.Name}} - {{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' $(docker ps -q)

Con este por ejemplo nos dira las ips de todos los contenedores en ejecución, con su nombre antes.

6. ¿Cual es la funcionalidad del apartado "ports" en docker compose?

El apartado ports de docker compose nos permite especificar los puertos que necesitemos exponer para nuestros servicios del docker compose. La exposición de puertos es fundamental para permitir que los servicios dentro de contenedores se comuniquen entre sí o con el mundo exterior.

Por ejemplo este apartado port de mi docker compose:
```
    ports:
     - '58080:80'
```
Esto hace que se exponga el puerto 58080 de nuestro contenedor al puerto 80 del host.

7. ¿Para que sirve el registro CNAME? Pon un ejemplo

El registro CNAME (Canonical Name) en DNS (Domain Name System) se utiliza para crear un alias o un apodo de un dominio. Esto permite que un nombre de un dominio, pueda asociarse con otro dominio.

Supongamos que tienes un sitio web llamado "mi-sitio-web.com" y también tienes un servidor de correo llamado "correo.miservidor.com". Puedes crear un registro CNAME para asociar el nombre "correo.miservidor.com" con "mi-sitio-web.com". Esto sería útil si, por ejemplo, decides cambiar la dirección IP de tu servidor de correo en el futuro; solo necesitarías actualizar el registro CNAME en lugar de cambiar múltiples registros A (direcciones IP) asociados al dominio.

8. ¿Como puedo hacer para que la configuración de un contenedor DNS no se borre si creo otro contenedor?

Podrias guardar la configuración del DNS en un volumen, y lo vincularas con el docker compose, cuando lo hagas aunque borres o crees otro DNS, este siempre se creara igual con el docker compose haciendo que sea muy facil de manejar los archivos desde fuera del contenedor, y además tendremos la certeza de que siempre será la misma configuración salvo que la cambies.

9. Añade una zona tiendadeelectronica.int en tu docker DNS que tenga:

    - www a la IP 172.16.0.1
    ![Alt text](image-3.png)
    - owncloud sea un CNAME de www
    ![Alt text](image-4.png)
    - un registro de texto con el contenido "1234ASDF"
    ![Alt text](image-5.png)
    - Comprueba que todo funciona con el comando "dig"
    - Muestra en los logs que el servicio arranca correctamente
    ![Alt text](image-6.png)
    
10. Realiza el apartado 9 en la máquina virtual con DNS

Para hacer un DNS en la máquina virtual, tendremos que cambiar los mismos archivos que en el de Docker, y haremos un **`systemctl restart named.service`**. Una vez hecho todo esto haremos un dig para comprobar todo esto. 

  - www a la IP 172.16.0.1
  ```
  asir2@A09equipo05:~$ dig @10.0.9.110 www.tiendadeelectronica.int 

; <<>> DiG 9.18.16-1~deb12u1-Debian <<>> @10.0.9.110 www.tiendadeelectronica.int
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 49112
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
; COOKIE: ce2a80a384c297be01000000655395dbbd43b2f0e3306ad3 (good)
;; QUESTION SECTION:
;www.tiendadeelectronica.int.	IN	A

;; ANSWER SECTION:
www.tiendadeelectronica.int. 38400 IN	A	172.16.0.1

;; Query time: 4 msec
;; SERVER: 10.0.9.110#53(10.0.9.110) (UDP)
;; WHEN: Tue Nov 14 16:44:27 CET 2023
;; MSG SIZE  rcvd: 100

  ```
  - owncloud sea un CNAME de www
  ```
  asir2@A09equipo05:~$ dig @10.0.9.110 owncloud.tiendadeelectronica.int 

; <<>> DiG 9.18.16-1~deb12u1-Debian <<>> @10.0.9.110 owncloud.tiendadeelectronica.int
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 27292
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 2, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
; COOKIE: 086f7801e5d63a1a0100000065539633d3cad7bc59f22368 (good)
;; QUESTION SECTION:
;owncloud.tiendadeelectronica.int. IN	A

;; ANSWER SECTION:
owncloud.tiendadeelectronica.int. 38400	IN CNAME www.tiendadeelectronica.int.
www.tiendadeelectronica.int. 38400 IN	A	172.16.0.1

;; Query time: 0 msec
;; SERVER: 10.0.9.110#53(10.0.9.110) (UDP)
;; WHEN: Tue Nov 14 16:45:55 CET 2023
;; MSG SIZE  rcvd: 123

  ```
  - un registro de texto con el contenido "1234ASDF"
  ```
  asir2@A09equipo05:~$ dig @10.0.9.110 texto.tiendadeelectronica.int TXT

; <<>> DiG 9.18.16-1~deb12u1-Debian <<>> @10.0.9.110 texto.tiendadeelectronica.int TXT
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 62813
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
; COOKIE: 78b68e392086c26901000000655396718d193d1073ce771d (good)
;; QUESTION SECTION:
;texto.tiendadeelectronica.int.	IN	TXT

;; ANSWER SECTION:
texto.tiendadeelectronica.int. 38400 IN	TXT	"1234ASDF"

;; Query time: 0 msec
;; SERVER: 10.0.9.110#53(10.0.9.110) (UDP)
;; WHEN: Tue Nov 14 16:46:57 CET 2023
;; MSG SIZE  rcvd: 107

  ```
  - Comprueba que todo funciona con el comando "dig"
  Ya lo hemos hecho todo arriba
  - Muestra en los logs que el servicio arranca correctamente
  
    En la maquina virtual estos seran los logs:
    ```
    pruebas@Ubuntupruebas:~$ sudo tail /var/log/syslog | grep named
    [sudo] contraseña para pruebas: 
    Nov 14 16:38:44 Ubuntupruebas named[11416]: zone localhost/IN: loaded serial 2
    Nov 14 16:38:44 Ubuntupruebas named[11416]: zone tiendadeelectronica.int/IN: loaded serial 20231009
    Nov 14 16:38:44 Ubuntupruebas named[11416]: zone web1asir.int/IN: loaded serial 20231009
    Nov 14 16:38:44 Ubuntupruebas named[11416]: all zones loaded
    Nov 14 16:38:44 Ubuntupruebas named[11416]: running
    Nov 14 16:38:44 Ubuntupruebas named[11416]: managed-keys-zone: No DNSKEY RRSIGs found for '.': success
    ```
