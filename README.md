# Examen ASO

### DNS

Primero debemos de instalar el dns:

```
apt-get install bind9
````

Una vez instalado debemos de de poner en el archivo /etc/bind/named.conf.options la linea de dnssec-validation en *yes*

En el archivo /etc/bind/named.conf.local debemos de declarar las zonas:

```
zone "iescalquera.local" {
    type master;
    file "db.iescalquera.local";
};

zone "5.12.172.in-addr.arpa" {
    type master;
    file "db.172.16.5"
};
````
Ahora debemos de copiar las zonas de ejemplo para poder modificarlas.

```
cp /etc/bind/db.empty /var/cache/bind/db.iescalquera.local
cp /etc/bind/db.empty /var/cache/bind/db.172.16.5
````

Ahora en el fichero /var/cache/bind/db.iescalquera.local
```
$TTL 86400	
@	IN SOA	iescalquera.local. root.iescalquera.local. (
				1          ; serial
				604800     ; refresh
				86400      ; retry 
				2419200    ; expire
				86400      ; minimum 
				)
;
@	IN		NS	ns.iescalquera.local.
ns			A	172.16.5.10; IP do servidor DNS
dserver00		A	172.16.5.10
uclient01		A	172.16.5.20
````

En el fichero /var/cache/bind/db.172.16.5
```
$TTL 86400
@	IN SOA	iescalquera.local. root.iescalquera.local. (
				1          ; serial
				604800     ; refresh
				86400      ; retry
				2419200    ; expire
				86400      ; minimum
				)
;
@	IN		NS	ns.iescalquera.local.
10			PTR	ns.iescalquera.local.
			PTR	dserver00.iescalquera.local.
20			PTR	uclient01.iescalquera.local.
````

Ahora podemos checkear las zonas con el *named-checkzone*

```
named-checkzone iescalquera.local /var/cache/bind/db.iescalquera.local

named-checkzone 5.16.172.in-addr.arpa /var/cache/bind/db.172.16.5
````

### Configuracion parte cliente dserver00

Editamos el fichero /etc/resolv.conf

Añadimos lo siguiente:

```
nameserver 172.16.5.10
search iescalquera.local
````

### Configuracion cliente uclient01

Ahora tenemos que configurar el cliete01 que se hace de forma grafica; la ip de este cliente es la **172.16.5.20** y la direccion del servidor DNS es **172.16.5.10** y en dominios de busqueda **iescalquera.local** 

Tenemos que configurar el archivo /etc/nsswitch.conf para poder hacer ping con el nombre dns debido a que nuestro nombre dns termina en .local.
Tenemos que modificar la linea host a:

```
hosts   files dns
````

# 

### Instalacion del servidor DHCP

Lo primero que tenemos que hacer es instalar el paquete con el siguiente comando:

```
apt-get install isc-dhcp-server
````

Ahora tenemos que editar el fichero */etc/dhcp/dhcp.conf*

Debemos de añadir lo siguiente:

```
option domain-name "iescalquera.local";
option domain-name-servers 172.16.5.10;
option routers 172.16.5.1;

default-lease-time 3600;
max-lease-time 7200;
````

Tamen tenemos que desmarcar la linea de *authoritative*

Ahora debemos de agregar el rango de las ips del servidor DHCP:

```
subnet 172.16.5.0 netmask 255.255.255.0 {
    range 172.16.5.100 172.16.5.119;
}
````
En el fichero */etc/default/isc-dhcp-server.conf* y en el parametro *INTERFACESv4=* y poner el nombre de la interfaz de nuestra tarjeta de red en este caso *enp0s3*

Y ahora debemos de reiniciar el servicio:

```
systemctl restart isc-dhcp-server
````

#### Configuracion del cliente02 

La configuracion de este cliente al igual que con el otro cliente se hace de manera grafica debemos de ir a editar la conexion a internet y en *Configuracion IPv4* poner en metodo *Automatico(DHCP)* una vez hecho esto debemos de desconectar  y volver a conectarnos a internet; esto se hace haciendo click izquierdo sobre el icono de red y en la parte de **Activar rede** debemos de desactivarla y despues volver a activarla.

Al igual que con el otro cliente debemos de modificar el fichero */etc/nsswitch.conf*.

Tenemos que modificar la linea host a:

```
hosts   files dns
````

### Reservas DHCP

Ahora debemos de crear una reserva del cliente02. Lo que debemos de añadir al fichero */etc/dhcp/dhcp.conf* es lo siguiente:

```
host uclient02{
    hardware ethernet 08:00:27:34:21:be;
    fixed-address 172.16.5.121;
};
````
Tenemos que reiniciar el servicio para que se apliquen los cambios con el comando:

```
service isc-dhcp-service restart
````

#### Configuracion cliente02
Ahora debemos de reiniciar el servicio de red en el cliente con el comando:

```
sudo dhclient -v enp0s3
````

### Asignar nombre al equipo cliente

Ahora podemos hacer que se establezca automaticamente el hostname del cliente02. Para eso tenemos que añadir lo siguiente en el */etc/dhcp/dhcp.conf* en sustitucion de la reserva del cliente02

```
host uclient02{
    hardware ethernet 08:00:27:34:21:be;
    fixed-address 172.16.5.121;
    option host-name "uclient02";
};
````

Tenemos que reiniciar el servicio para que se apliquen los cambios con el comando:

```
service isc-dhcp-service restart
````

### Configuracion del cliente para configurar su nombre de equipo

Tenemos que crear el fichero */etc/NetworkManager/dispatcher.d/99hostname* con el siguiente contenido:

```bash
#!/bin/sh

### Script para configurar o nome do equipo
### a partir dos datos enviados polo servidor DHCP


case """"""""$2"""""""" in

        up|dhcp4-change)
                # Non facemos nada se o servidor
                # non nos deu a variable adecuada
                if [ -z """"""""$DHCP4_HOST_NAME"""""""" ]
                then
                        exit 0
                else
                        hostname $DHCP4_HOST_NAME
                        echo $DHCP4_HOST_NAME > /etc/hostname
                fi
        ;;
esac
````

Ahora debemos de cambiarle los permisos con el siguiente comando:

```
sudo chmod 755 /etc/NetworkManager/dispatcher.d/99hostname
````

Y volver a desconectar y conectar la red para que los cambios sean efectivos(se pueden hacer desde la interfaz grafica). 

Para ver el nombre del cliente tenemos dos opciones o bien poner:

```
hostname
````
o bien poner:
```
cat /etc/hostname
````