# Examen ASO

## Instalacion y configuracion de ldap

Lo primero que tenemos que hacer es instalar el paquete necesario. Se haria con el siguiente comando:

```
apt-get install slapd
````

Una vez se instale nos saldra una pequeña pantalla de configuracion donde debemos debemos de añadir la contraseña del administrador.

Tambien instalaremos el paquete tree para ayudarnos con la estructura de ficheros:

```
apt-get install tree
````

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

##### Con autenticacion

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

## Administracion de unidades organizativas, usuarios y grupos en LDAP con ldap-utils

### Administracion de Unidades Organizativas(OU)

#### ModeloOu.ldif

```
dn: ou=exemplo-unidade-organizativa,dc=exemplo,dc=local
objectClass: organizationalUnit
ou= exemplo-unidade-organizativa
````

#### Crear unidades organizativas

Primero tenemos que crear el fichero ldif con el que importaremos las OU. 

El fichero es el siguiente:

```
#OU usuarios
dn: ou=usuarios,dc=iescalquera,dc=local
objectClass: organizationalUnit
ou: usuarios
description: OU para almacenar usuarios

#OU profes
dn: ou=profes,ou=usuarios,dc=iescalquera,dc=local
objectClass: organizationalUnit
ou: profe
street: Rua Borrar n 3 (non usar tildes)
````

Ahora tenemos que ejecutar el comando:

```
ldapadd -D cn=admin,dc=iescalquera,dc=local -W -f ou.ldif
````

Como nos equivocamos al poner el nombre de la ou profes en la parte de ou aparece de la siguiente manera:

```
ldapsearch -x -b ou=usuarios,dc=iescalquera,dc=local
# extended LDIF
#
# LDAPv3
# base <ou=usuarios,dc=iescalquera,dc=local> with scope subtree
# filter: (objectclass=*)
# requesting: ALL
#

# usuarios, iescalquera.local
dn: ou=usuarios,dc=iescalquera,dc=local
objectClass: organizationalUnit
ou: usuarios
description: OU para almacenar usuarios.

# profes, usuarios, iescalquera.local
dn: ou=profes,ou=usuarios,dc=iescalquera,dc=local
objectClass: organizationalUnit
ou: profe
ou: profes
street: Rua Borrar n 3 (non usar tiles)

# search result
search: 2
result: 0 Success

# numResponses: 3
# numEntries: 2
````

### Modificar Unidades Organizativas

Primero tenemos que crear el fichero que en el que indicamos que es lo que vamos a modificar de la OU.

```
dn: ou=profes,ou=usuarios,dc=iescalquera,dc=local
changetype: modify
replace: ou
ou: profes
-
add: description
description: OU para almacenar usuarios profes.
-
delete: street
````

Ahora tenemos que ejecutar el siguiente comando:

```
ldapadd -D cn=admin,dc=iescalquera,dc=local -W -f ou_modif.lidf
````

### Eliminar Unidades Organizativas

Tenemos dos opciones o bien hacerlo con ldapadd o bien hacerlo con ldapdelete.

Primero lo haremos con ldapadd. Tenemos que crear un fichero ldif con el siguiente contendio:

```
dn: ou=usuarios,dc=iescalquera,dc=local
changetype: delete
````

Y despues ejecutar el comando:

```
ldapadd -D cn=admin,dc=iescalquera,dc=local -W -f ou_borrar.lidf
````

Ahora lo haremos con ldapdelete, para ello tambien debemos de crear  un fichero ldif, con el siguiente contenido:

```
dn: ou=usuarios,dc=iescalquera,dc=local
dn: ou=profes,ou=usuarios,dc=iescalquera,dc=local
````

Una vez creado el fichero ldif debemos de ejecutar el comando:

```
ldapdelete -D cn=admin,dc=iescalquera,dc=local -W -f ou_borrar2.lidf
````

### Administracion de Grupos

```
dn: cn=exemplo-nombre-grupo, ou=exemplo-unidade-organizativa,dc=iescalquera,dc=local
objectClass: posixGroup
cn: exemplo-nombre-grupo
gidNumber: N-gid-Mejor-mayor-de-9999
````

#### Crear Grupos

Tenemos que crear el siguiente fichero:

```
dn: cn=g-usuarios,ou=grupos,dc=iescalquera,dc=local
objectClass: posixGroup
cn: g-usuarios
gidNumber: 10000
````

Ahora tenemos que ejecutar el comando:

```
ldapadd -D cn=admin,dc=iescalquera,dc=local -W -f grupos.lidf
````

#### Modificar/eliminar grupos

En cuanto a modificar y eliminar grupos la metodologia es igual que con las unidades organizativas.

### Administracion de usuarios

```
dn: uid=usuario,ou=ou_exemplo,dc=exemplo,dc=local
objectClass: inetOrgPerson
objectClass: posixAccount
objectClass: shadowAccount
uid: usuario
sn: apelido
givenName: Nome de pila
cn: Nome Completo
displayName: Nome para amosar
uidNumber: 10000
gidNumber: 10000
userPassword: contrasinal
gecos: Informacion sobre o usuario (Opcional. Sen tiles)
loginShell: /bin/bash
homeDirectory: /home/usuario
mail: usuario@exemplo.local
initials: UA

#As seguintes entradas son opcionais, e serven para controlar a caducidade do contrasinal.
shadowExpire: -1
shadowFlag: 0
shadowWarning: 7
shadowMin: 8
shadowMax: 999999
shadowLastChange: 10877
````

#### Crear usuarios

Tenemos que crear el siguiente fichero:

```
dn: uid=sol,ou=profes,ou=usuarios,dc=iescalquera,dc=local
objectClass: inetOrgPerson
objectClass: posixAccount
objectClass: shadowAccount
uid: sol
sn: Lúa
cn: Profe - Sol Lúa
givenName: Sol
displayName: Profe - Sol Lúa
uidNumber: 10000
gidNumber: 10000
userPassword: abc123.
gecos: Profe - Sol Lua
loginShell: /bin/bash
homeDirectory: /home/sol
mail: sol@iescalquera.local
initials: SL
shadowExpire: -1
````

Ahora tenemos que ejecutar el siguiente comando:

```
ldapadd -D cn=admin,dc=iescalquera,dc=local -W -f usuarios.ldif
````

#### Añadir a los usuarios a grupos secundarios

Primero creamos el fichero para declarar los grupos a los que los vamos a unir.

```
dn: cn=g-profes,ou=grupos,dc=iescalquera,dc=local
changetype: modify
add: memberUid
memberUid:sol

dn: cn=g-dam1-profes,ou=grupos,dc=iescalquera,dc=local
changetype: modify
add: memberUid
memberUid:sol

dn: cn=g-dam2-profes,ou=grupos,dc=iescalquera,dc=local
changetype: modify
add: memberUid
memberUid:sol
````

Ahora tenemos que ejecutar el siguiente comando:

```
ldapadd -D cn=admin,dc=iescalquera,dc=local -W -f grupos_secundarios.ldif
````

#### Eliminar/modificar usuarios

Se haria igual que con los grupos y las unidades organizativas. Pero hay un pequeñp detalle que es que aunque eliminemos un usuario no significa que este se elemine de la entrada correspondiente en el grupo secundario, por lo tanto debemos de borrar la entrada memberUid del grupo.

### Cambiar la contraseña de un usuario

Tenemos que ejecutar el siguiente comando:

```
ldappasswd -D cn=admin,dc=iescalquera,dc=local -W uid=sol,ou=profes,ou=usuarios,dc=iescalquera,dc=local
````
En este caso la contraseña se pone automaticamente; en caso de que quisiesemos ponerla nosotros deberiamos de utilizar el parametro -S junto con el comando anterior; por lo que quedaria de la siguiente manera:

```
ldappasswd -D cn=admin,dc=iescalquera,dc=local -W uid=sol,ou=profes,ou=usuarios,dc=iescalquera,dc=local -S
````

O pot ultimo otra forma de cambiar la contraseña con un archivo ldif con el siguiente contenido:

```
dn: uid=sol,ou=profes,ou=usuarios,dc=iescalquera,dc=local
changetype: modify
replace: userPassword
userPassword:novo contrasinal
````

Y ejecutarlo con ldapadd.


### Ferramentas incluidas con el servidor ldap.

- *slapindex* : Este comando pode ser moi útil en caso de apagados accidentais do servidor LDAP, xa que é posible que os índices usados para acceder á información do directorio se corrompan, o que pode producir erros na busca ou inserción de información no directorio ou incluso que o servidor LDAP non poida arrancar. A función do comando é rexenerar os índices a partir da información almacenada no directorio, creando así de novo as estruturas necesarias para que o servizo LDAP funcione correctamente.

-  O comando debe executarse co mesmo usuario que executa o servidor LDAP (no caso de Debian/Ubuntu o usuario openldap), xa que os ficheiros de índices que este comando crea só poderán ser modificados polo usuario que executa o comando e executalo con outro usuario podería dar problemas de permisos ao arrancar o servidor LDAP.

```
root@dserver00:~# service slapd stop
  [ ok ] Stopping OpenLDAP: slapd.
root@dserver00:~# su - openldap -s /bin/bash
openldap@dserver00:~$ /usr/sbin/slapindex -v
  indexing id=00000001
  indexing id=00000002
  indexing id=00000014
  indexing id=00000015
  indexing id=00000016
  indexing id=00000017
  indexing id=00000018
  indexing id=00000019
  indexing id=0000001a
  indexing id=00000024
  indexing id=00000025
openldap@dserver00:~$ exit
  logout
root@dserver00:~# service slapd start
  [ ok ] Starting OpenLDAP: slapd.
````


## Configuracion Cliente LDAP

### Instalar los paquetes necesarios

Lo primero que debemos de hacer es instalar los paquetes necesarios para que nos funcione, en este caso instalaremos el siguiente paquete:

```
sudo apt-get install libpam-ldapd
````

Esto nos abrira un sistema de configuracion donde debemos de hacer los siguientes pasos:

1. Tendremos que declarar la direccion ip del servidor de ldap. Por el momento haremos una conexion no segura al mismo. Lo que debemos de escribir es lo siguiente:
  ```
  ldap://172.16.5.10/
  ````
Y le daremos al boton de aceptar. 
2. Una vez añadida la direccion ip del servidor debemos de poner lo siguiente, que es el nombre de dominio del servidor:
  ```
  dc=iescalquera,dc=local
  ````
Una vez hecho esto pasaremos a la siguiente pantalla dandole a aceptar.
3. En esta pantalla debemo de seleccionar las opciones group, passwd y shadow. Esto se hace poniendose sobre la opcion y dandole al espacio para que aparezca un asterisco. Una vez hecho esto le daremos a aceptar y se terminara la configuracion.

4. Para comprobar que funcione probaremos los siguientes comandos:

  ```
  getent passwd
  getent group
  getent shadow
  ```

5. Tambien ahoremos el siguiente comando y miraremos que este todo con asterisco (menos los de crear el directorio personal si no queremos que se cree en local)
  ```
  sudo pam-auth-update
  ````

Si al final de todo aparecen los grupos y usuarios de nuestro dominio todo funciona correctamente; si no debemos de volver a configurar. Para poder volver a configurarlo debemos de utilizar este comando:
  ```
  sudo dpkg-reconfigure nslcd
  ````

Y reiniciar el servicio con el comando:

```
sudo service nslcd restart
sudo service nscd restart
````

#### Comprobar el funcionamiento del cliente

Pata comprobar si un cliente funciona lo podemos hacer de dos maneras; o bien con el comando getent o bien conectadonos con un usuarios del servidor ldap.

#### Con el comando getent

Lo que debemos de hacer es lo siguiente:

```
getent passwd
getent group
getent shadow
````

Debemos de poner uno de esos tres comandos y si al final de todo aparecen los grupos y usuarios de nuestro dominio todo funciona correctamente


#### Conectandonos con un usuario

Tambien podemos comprobar si funciona conectandonos con un usuario del servidor ldap; en este caso lo haremos con sol.

```
su - sol
````

Si nos deja iniciar sesion funciona perfectamente. Una vez conectado tiene los mismo comandos que un usuarios local.