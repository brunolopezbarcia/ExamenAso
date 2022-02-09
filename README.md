# Examen ASO

### Instalacion y configuracion de ldap

Lo primero que tenemos que hacer es instalar el paquete necesario. Se haria con el siguiente comando:

```
apt-get install slapd
````

Una vez se instale nos saldra una pequeña pantalla de configuracion donde debemos debemos de añadir la contraseña del administrador.

Tambien instalaremos el paquete tree para ayudarnos con la estructura de ficheros:

```
apt-get install tree


Ahora debemos de reconfigurar el slapd; lo haremos con el siguiente comando:

```
dpkg-reconfigure slapd
````

Nos aparacera una pantalla donde debemos de de decir que *no* queremos saltar la configuracion de ldap; en la siguiente pantalla debemos de introducir el nombre de dominio (en nuestro ejemplo es **iescalquera.local**). 
Lugo nos saltara la pantalla del nombre de la organizacion (en nuestro ejemplo es **iescalquera**)
Añadimos la contraseña del administrador; esta pantalla es para cambiarla por lo tanto pondremos la misma que en el caso anterior.

En el motor de base de datos seleccionaremos la tercera opcion que es *MDB*; no eliminaremos la base de datos para que en un futuro podremos restaurar 
cuando se desinstale el servidor. Pero si que aceptaremos trasladaremos la base de datos antigua, y por ultimo indicaremos que no queremos utilizar el protocolo LDAPv2.


### Uilidades ldap: paquete ldap-utils

Instalaremos el paquete de utilidades para ldap que nos proporcionara poder consultar el directorio LDAP, añadir/modificar/eliminar objectos y poder cambiar la contraseña de un usuario.

Para instalarlo debemos de aplicar el siguiente comando:

```
apt-get install ldap-utils
````

##### Utilizar ldapsearch

###### Consultar el dominio creado

```
ldapsearch -x -b '' -s base objectClass=* namingcontext
`````

##### Consultar objecto de la rama dc=iescalquera,dc=local

##### Sin autenticacion

```
ldapsearch -x -b 'dc=iescalquera,dc=local'
```

#### Con autenticacion

```
ldapsearch -Y EXTERNAL -H ldapi:// -b 'dc=iescalquera,dc=local'
```

##### Indicando el usuario y la contraseña

```
ldapsearch -D cn=admin,dc=iescalquera,dc=local -w abc123. -H ldap://localhost -b 'dc=iescalquera,dc=local'
`````
##### Sin indicar la contraseña

```
ldapsearch -D cn=admin,dc=iescalquera,dc=local -W -H ldap://dserver00 -b 'dc=iescalquera,dc=local'
````

##### Indicando que solo queremos consultar los valores base pero no los hijos

```
ldapsearch -x -b 'dc=iescalquera,dc=local' -s base
````

##### Indicando que solo queremos consultar los valores de los objectos hijos, no el objeto base

```
ldapsearch -x -b 'dc=iescalquera,dc=local' -s one
````

##### Ponemos un filtro en el que indicamos que deseamos consultar los objectos que tengan el atributo objectClass independientemente del valor que tenga este atributo.

```
ldapsearch -x -b 'dc=iescalquera,dc=local' objectClass=*
````

##### Ponemos un filtro en el que indicamos que deseamos consultar los objectos que tengan el atributo objectClass con valor dcObject

```
ldapsearch -x -b 'dc=iescalquera,dc=local' objectClass=dcObject
````

##### Ponemos un filtro en el que indicamos que deseamos consultar los objectos que tengan el atributo cn con valor admin

```
ldapsearch -x -b 'dc=iescalquera,dc=local' cn=admin
````

