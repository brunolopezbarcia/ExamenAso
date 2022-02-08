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

Tenemos que configurar el archivo /etc/nsswitch.conf pàra poder hacer ping con el nombre dns debido a que nuestro nombre dns termina en .local.
Tenemos que modificar la linea host a:

```
hosts   files dns
````








