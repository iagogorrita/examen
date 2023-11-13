# EXAMEN DNS - DOCKER

1. Explica métodos para 'abrir' una consola/shell a un contenedor que se está ejecutando

    Hay varias formas para abrir una consola o shell a un contenedor de docker:
    
    1.1. La primera puede ser a través del visual code con el plugin de docker y le podremos dar directamente a iniciar, dandole click derecho y start.
        
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
    