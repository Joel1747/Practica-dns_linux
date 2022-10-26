# Creación docker compose

Crearemos el docker-compose compose con los ficheros como vimos en la practica anterior en este caso nuestro fichero quedará de la sigunete manera

~~~ 
version: "3.9" 
services:
  asir_cliente: 
    container_name: asir_clientep1
    image: alpine
    networks:
      - bind9_subnetp1
    stdin_open: true
    tty: true
    dns:
      - 10.1.1.254
  bind9:
    container_name: asir2_bind9_p1
    image: internetsystemsconsortium/bind9:9.16
    ports:
      - 5400:53/udp
      - 5400:53/tcp
    networks:
      bind9_subnetp1:
        ipv4_address: 10.1.1.254 #ip fija del servidor
    volumes:
      - /home/asir2a/Documentos/SRI/p2/confg:/etc/bind
      - /home/asir2a/Documentos/SRI/p2/zonas:/var/lib/bind
networks:
  bind9_subnetp1:
    external: true   
~~~

Como podemos ver en el código anterior tenemos como dos apartados uno que es _asir_cliente_  y otro que es _bind9_, nuestro _asir_cliente_ será nuestro cliente _alpine_ desde el que nos conectaremos despues, arriba podemos ver la conf que le hemos dado al cliente; y el apartado de _Bind9_ es el de nuestro servidor DNS

# Creación de volumenes

tenemos que crear los volumenes en el mismo directorio que el docker-compose en el cual crearemos el directorio _confg_ y otro que se llame _zonas_

## directorio confg 

Dentro deberemos tener 3 archivos uno llamado _named.conf_ donde en el interior se hará referencia a los dos archivos a continuación

~~~
include "/etc/bind/named.conf.options";
include "/etc/bind/named.conf.local";
~~~

### fichero named.conf.options(configuración forwardes) 

Tendremos que crear el fichero _named.conf.options_  en el cual tendremos un código parecido a este segun las opciones que neceitemos

~~~ 
options {
        directory "/var/cache/bind";

        forwarders {
            8.8.8.8;
            8.8.4.4;
        };
        forward only;

        listen-on { any; };
        listen-on-v6 { any; };
        allow-query {
            any;
        };

        allow-recursion {
                none;
        };
        allow-transfer {
                none;
        };
        allow-update {
                none;
        };
};
~~~
### fichero named.conf.local

Tendremos que crear el fichero _named.conf.local_  en el cual tendremos un código parecido a este donde haremos referencia a la zona

~~~
zone "asircastelaop1.com." {
        type master;
        file "/var/lib/bind/db.asircastelaop1.com";
        notify explicit;
        allow-query {
            any;
        };
};

~~~

Estos serian los archivos del directorio confg

## Directorio zonas(Creación zona propia)

En el directorio zonas solo crearemos un archivo que será el nombre de la zona en este caso _db.asircastelaop1.com_ en el cual habrá el siguiente código

~~~
$TTL    36000
@       IN      SOA     ns.asircastelaop1.com. joel.danielcastelao.org. (
                     06102022           ; Serial
                         3600           ; Refresh [1h]
                          600           ; Retry   [10m]
                        86400           ; Expire  [1d]
                          600 )         ; Negative Cache TTL [1h]
;
@       IN      NS     ns.asircastelaop1.com.
@       IN      A      10.1.1.2   
ns      IN      A      10.1.1.254
test    IN      A      10.1.1.25
email   IN      MX     10.1.1.2  
alias   IN      CNAME  test

text    IN      TXT    "Texto default"  

~~~


### Network 
acordarse de crear la network antes de crear y levantar el servicio para ello usaremos el siguiente comando:
~~~
  docker network create --subnet 10.1.1.0/24 --gateway 10.1.1.1 bind9_subnet
~~~


## Gestion del servicio
Con el siguiente comando crearemos el servicio en segundo plano al añadirle el parametro _-d_
~~~
    docker-compose up -d
~~~

Con el comando start y stop arrancaremos y pararemos el servicio una vez este creado
~~~
    docker-compose start
    docker-compose stop
~~~
En caso de querer eliminar el servicio usaremos
~~~
    docker-compose down -v
~~~
cuando modifiquemos el servicio tendremos que usar el comando down y up para borrar el viejo y subir el nuevo.

Con esto tendremos nuestro Docker-compose listo para funcionar.

![foto ping](https://github.com/Joel1747/Practica-dns_linux/blob/main/ping.png)
