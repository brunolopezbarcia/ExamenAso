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


## Herramientas para la administracion del LDAP: ldapscripts, LDAP Accoutn Manager y JXplorer

### Administracion mediante scripts

Para esto debemos de instalar el siguiente paquete en dserver00:

```
apt-get install ldapscripts
````

A continuacion debemos de modificar unos parametros en el archivo **/etc/ldapscripts/ldapscripts.conf**:

```
SERVER="ldap://localhost"
BINDPWDFILE="/etc/ldapscripts/ldapscripts.passwd"
SUFFIX="dc=iescalquera,dc=local"
GSUFFIX="ou=grupos"
USUFFIX="ou=usuarios"
MSUFFIX="ou=maquinas"
...
BINDDN="cn=admin,dc=iescalquera,dc=local"
BINDPWDFILE="/etc/ldapscripts/ldapscripts.passwd"
...
CREATEHOMES="yes"
````

Para terminar la configuracion del paquete debemos de actualizar la contraseña del administrador para este paquete con el siguiente comando:

```
sh -c "echo -n abc123." > /etc/ldapscripts/ldapscripts.passwd
````

Ahora tenemos muchas opciones como puede ser *ldapaddgroup, ldapadduser....*

Vamos a probar algunos comandos:

  - Añadir el grupo g-alum
    ```
    ldapaddgroup g-alum
    ````
  - Ahora crearemos un usuario que añadiremos a un grupo(debemos de tener creados los grupos grupo1-ficticio y grupo2-ficticio con anterioridad)
    ```
    ldapadduser pepe grupo1-ficticio
    ````
  - Podemos cambiarle la contraseña con:
    ```
    ldapsetpasswd pepe
    ````
  - Y podemo añadirlo a mas grupos con el comando:
    ```
    ldapaddusertogroup pepe grupo2-ficticio
    ````

Ahora veremos como eliminar el usuario pepe y los grupos.
  
  - Eliminar al usuario del grupo al que lo añadimos:
    ```
    ldapdeleteusertogroup pepe grupo2-ficticio
    ````
  - Eliminar al usuario:
    ```
    ldapdeleteuser pepe
    ````
  - Eliminar el grupo1-ficticio
    ```
    ldapdeletegroup grupo1-ficticio
    ````	

### Instalacion y uso del LDAP Account Manager(LAM)

Lo primero que debemos de hacer es instalar el paquete correspondiente:

```
apt-get install ldap-account-manager
````

Una vez instalado este paquete como se instala el servidor apache instalaremos una serie de paquete para su correcto funcionamiento.

```
apt-get install php-xml
apt-get install php-zip
````

Una vez instalados este paquete reiniciamos el servicio de apache:

```
service apache2 restart
````

### Configuracion de LAM

Como estamos en una maquina virtual lo que debemos de hacer es permitir un reenvio de puertos desde el puerto 80 de la maquina virtual a un puerto de la maquina real en nuestro caso el 8080. Para poder acceder a la interfaz del LAM debemos de poner **http://192.168.32.12:8080/lam**.

1. Una vez conectados presionamos en LAM configuration 
2. Una vez en esta ventana entraremos en la opcion *Edit Server profiles*
3. Aqui debemos de modificar la contraseña de la utilidad, que por defecto es lam; en este caso la cambiaremos a *abc123.*
4. Ahora debemos de iniciar sesion e ir a la pestaña general. Donde pone **Tree sufix** debemos de poner **dc=iescalquera,dc=local**. En la parte de **default language** pondremos el idioma español y tambien debemos de poner en la parte de **List of valid usuers** tendremos que poner **cn=admin,dc=iescalquera,dc=local**
5. Tendremos que guardar los cambios para que surjan efecto.

Ahora debemos de movernos a la opcion de **Tipos de cuentas** y dentro del apartado **Tipos de cuentas activos**, borraremos la cuenta de **Dominios de Samba**. Dentro de esta pestaña debemos de modificar los tipos de cuenta usuarios, grupos y equipos, poniendole los siguientes Sufijos de LDAP.

```
Usuarios -> ou=usuarios,dc=iescalquera,dc=local
Grupos -> ou=grupos,dc=iescalquera,dc=local
Equipos -> ou=maquinas,dc=iescalquera,dc=local
````

Esta ultima en caso de no estar añadida la añadimos.


Ahora el la pestaña modulos debemos de modificar los modulos seleccionados para cada tipo de cuenta:

Los modulos deben de ser los siguientes:
- En el caso de usuarios:
  - Personal(inetOrgPerson)(*)
  - Unix(posixAccount)
  - Sombra(shadowAccount)
*En caso de que haya mas modulos los eliminamos*

- En el caso de grupos:
  - Unix(posixAccount)
*En caso de que haya mas modulos los eliminamos*

- En el caso de usuarios:
  - Cuenta(account)(*)
  - Unix(posixAccount)
*En caso de que haya mas modulos los eliminamos*

Ahora le damos al boton de guardar para que los cambios surjan efecto.


#### Crear OUs en LAM

Ahora vamos a crear la OU dam2.

Debemos de abrir la vista de arbol y en la parte de ou=alumno abrimos el desplegable y le damos a crear nueva entrada aqui. Selecionamos OU generica y le indicamos el nombre y le damos a guardar.

##### Crear Grupos en LAM

En la pestaña de grupos le damos a nuevo grupo y donde pone nombre de grupo ponemos g-dam1-alum y le damos a guardar(no es necesario poner el nombre del grupo debido a que LAM pone automaticamente el primer GID libre.)

#### Crear Usuarios en LAM

###### Crear Usuarios con CSV

En usuarios le damos en File Upload y nos enseña los modulos que va a tener en cuenta le damos a aceptar y descargamos el modelo csv de ejemplo lo abrimos con una hoja de calculo y cambiamos los datos para los datos que nosotros queremos. Una vez cambiados lo guardamos y lo subimos a lam, en este momento LAM chequeara si todo el contenido esta bien lo sube. Y asi `podemos ver al usuarios creadom con el uid(no se indico en el archivo), tambien lo vemos dentro de su ou y dentro de su grupo secundario


### JXplorer

Jxplorer es una herramienta garfica de administracion de ficheros que nos permite conectarnos al servidor ldap.

Debemos de instalar esta herramienta en uno de los clientes en nuestro caso uclient01. Con el siguiente comando:

```
sudo apt-get install jxplorer
````

Una vez instalado le debemos de dar a conectar, donde debemos de introducir la direccion ip del servidor, en la parte de level debemos de añadir *dc=iescalquera,dc=local* y en la parte de user DN debemos de añadir *cn=admin,dc=iescalquera,dc=local* y en la password la contraseña que le establecimos al usuario admin.

Una vez conectados nos aparecera todo el servidor en forma de arbol donde haciendo doble click sobre un objeto podemos modificar sus valores.

## Autenticacion segura contra el LDAP. Uso de TLS/SSL: LDAPS

### Creacion de la Autirdad de Certificacion.

En dserver00, primero crearemos los directiorios para almacenar los cerficados y los archivos relacionados:

```
mkdir /etc/ssl/CA
mkdir /etc/ssl/newcerts
````

Ahora crearemos dos fichero que la CA precisara para mantener un numero de serie.
```
touch /etc/ssl/CA/index.txt
sh -c "echo '01'" > /etc/ssl/CA/serial
````

En el fichero **/etc/ssl/openssl.conf** modificaremos los siguientes parametros:
```
dir = /etc/ssl # Where everything is kept
...
database = $dir/CA/index.txt # database index file.
...
certificate = $dir/certs/cacert.pem # The CA certificate
serial = $dir/CA/serial # The current serial number
````

Creamos el certificado raiz.

```
openssl req -new -x509 -extensions v3_ca -keyout cakey.pem -out cacert.pem -days 3650
````

Y poner los siguientes datos. 

```
enerating a 1024 bit RSA private key
......++++++
........++++++
unable to write 'random state'
writing new private key to 'cakey.pem'
Enter PEM pass phrase:
Verifying - Enter PEM pass phrase:
-----
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [AU]:ES
State or Province Name (full name) [Some-State]:Galicia
Locality Name (eg, city) []:
Organization Name (eg, company) [Internet Widgits Pty Ltd]:IES Calquera
Organizational Unit Name (eg, section) []:
Common Name (eg, YOUR name) []:dserver00.iescalquera.local
Email Address []:
````

Ahora movemos en los directorios de la CA tanto la llave privada como el certificado creado:

```
mv cakey.pem /etc/ssl/private/
mv cacert.pem /etc/ssl/certs/
````

### Generar la solicitud de firmado del certificado

Lo primero que tenemos que hacer es ejecutar este comando. 

```
openssl genrsa -des3 -out server.key 1024
````

Ahora ejecutaremos este comando:

```
openssl rsa -in server.key -out server.key.insecure
````

Y guardamos en server.key la clave sin contraseña.

```
mv server.key server.key.secure
mv server.key.insecure server.key
````

Y por ultimo creamos el CSR:

```
openssl req -new -key server.key -out server.csr
````

Donde pondremos los siguientes datos:

```
Country Name (2 letter code) [AU]:ES
State or Province Name (full name) [Some-State]:Galicia
Locality Name (eg, city) []:
Organization Name (eg, company) [Internet Widgits Pty Ltd]:IES Calquera
Organizational Unit Name (eg, section) []:
Common Name (eg, YOUR name) []:dserver00.iescalquera.local
Email Address []:
Please enter the following 'extra' attributes
Xenerar a solicitude de firma do certificado (CSR)
to be sent with your certificate request
A challenge password []:
An optional company name []:
````

### Generar certificado apartir del CSR

Lo primero que tenemos que hacer es ejecutar este comando:

```
openssl ca -in server.csr -config /etc/ssl/openssl.cnf
````

Y debemos de añadir nuestra clave que usamos al crear la CA y lsito ya tenemos nuestro certificado:

```
-----BEGIN CERTIFICATE-----
MIICpzCCAhCgAwIBAgIBATANBgkqhkiG9w0BAQUFADBbMQswCQYDVQQGEwJFUzEQ
MA4GA1UECBMHR2FsaWNpYTEVMBMGA1UEChMMSUVTIGNhbHF1ZXJhMSMwIQYDVQQD
ExpzZXJ2ZXIwMC5pZXNjYWxxdWVyYS5sb2NhbDAeFw0xMDAzMDQyMzI1MjNaFw0x
MTAzMDQyMzI1MjNaMFsxCzAJBgNVBAYTAkVTMRAwDgYDVQQIEwdHYWxpY2lhMRUw
EwYDVQQKEwxJRVMgY2FscXVlcmExIzAhBgNVBAMTGnNlcnZlcjAwLmllc2NhbHF1
ZXJhLmxvY2FsMIGfMA0GCSqGSIb3DQEBAQUAA4GNADCBiQKBgQC/QWqoi12reBRA
/3p6+KyWTAoN3XqLU8VaNhpAAP4LTRuuzeeCKxkPyj2QZk+rWehmqkqbwX6Zdrqi
BSfeKuoRokTV7e2bbMJmaomEbvez5bwr7sDSXl2UyFhVyJWtQBkI8m2pkqjWt9Fn
2OotV+c43HNncXN3/mGoVwpE7OMivwIDAQABo3sweTAJBgNVHRMEAjAAMCwGCWCG
SAGG+EIBDQQfFh1PcGVuU1NMIEdlbmVyYXRlZCBDZXJ0aWZpY2F0ZTAdBgNVHQ4E
FgQU3/xzDTawr9pH9+NX+UH9/4ivF64wHwYDVR0jBBgwFoAUAciyrRu3hkU+yjfM
wZWOqCLD0ZswDQYJKoZIhvcNAQEFBQADgYEAVHDWexRWbz6nPWVA+x/4KaXA9KaE
atZ1cu2Mep+29duZyAFcQEf4pivXCallmkmbAhurpUH61SLFHOb7YHl71EPLvru0
U3kDx48wSDGqBzdCKWhoh1SBrFryxlovEredZ44q/1AxldJ8py9r77e2kqJ7u+TC
6v0/CnJRUYvWZh0=
-----END CERTIFICATE-----
````

Todo esto lo copiamos en un archivo que se llame server.crt

Y luego ejecutaremos este comando:

```
cp server.crt /etc/ssl/cert
cp server.key /etc/ssl/private
````

#### Configuracion del servidor LDAP

Añadimos si no existe el grupo local **ssl-cert**

```
#Se non existe o grupo local ssl-cert, creámolo.
#addgroup ssl-cert,
adduser openldap ssl-cert
chmod 750 /etc/ssl/private
chgrp ssl-cert /etc/ssl/private
chmod 640 /etc/ssl/private/server.key
chgrp ssl-cert /etc/ssl/private/server.key
````

Y tenemos que reiniciar el servicio ldap:

```
/etc/init.d/slapd restart
````

Ahora utilizaremos este comando:

```
ldapmodify -Y EXTERNAL -H ldapi://
````
Pegaremos los siguientes datos:

```
dn: cn=config
add: olcTLSCACertificateFile
olcTLSCACertificateFile: /etc/ssl/certs/cacert.pem
-
add: olcTLSCertificateFile
olcTLSCertificateFile: /etc/ssl/certs/server.crt
-
add: olcTLSCertificateKeyFile
olcTLSCertificateKeyFile: /etc/ssl/private/server.key
````
Utilizaremos Ctrl+D para procesar los datos introducidos.

Ahora tenemos que modificar el ficher **/etc/default/sldap** y establecer en el paramentro *SLAPD_SERVICES* el siguiente valor:

```
SLAPD_SERVICES="ldap:/// ldaps:/// ldapi:///"
````

### Configuracion del cliente LDAP

Utilizaremos el comando:

```
sudo dpkg-reconfigure nslcd
````

Y modificaremos la direccion ip del servidor poniendole lo siguiente:

```
ldaps://172.16.5.10/
````

El resto lo dejamos como esta. Excepto en la pantalla de que te pregunte sobre conseguir el certificado que seleccionaremos permitir.

Se reiniciara el servicio automaticamente pero podemos forzarlo con:

```
sudo service nscd restart
sudo service nslcd restart
````

Comprobamos que nos funciona o bien con el comando getent

```
getent passwd
````
Si aparecen los usuarios del dominio es que funciona

O bien iniciando sesion con un usuario del dominio:

```
su - sol
````

Si inicia sesion es que nos funciona.


## Creacion e configuracion del esqueleto de la estructura de carpetas.

Lo primero que debemos de hacer es añadir un nuevo disco a la maquina virtual desde el VB.

Una vez hayamos añadido el disco debemos de iniciar la maquina virtual y poner estos comandos:

```
ls /dev/sd*
````

Esto sirve para comprobar que el disco este añadido.

Ahora tenemos que particionar el disco, primero miraremos si esta particionado con el comando *fdisk -l* que permite ver los discos y sus particiones.

Para particionarlo haremos lo siguiente:

```
cfdisk /dev/sdb
````
En donde pone elegir la tabla de particiones elegimos la opcion de *gpt* que es soportada por casi todos los sistemas operativos.

Ahora debemos de darle a la opcion de nueva y pondremos en el tamaño de particion 5G, una vez tengamos esta particion creada le nos pondremos sobre donde pone espacio libre y le daremos a nueva para crear otra particion que tambien tendra 5G de tamaño y le daremos al boton de escribir para que cree las particiones, y le indicaremos que si queremos escribir la tabla de particiones.


Una vez hecho esto comprobamos el particionado del disco con el siguiente comando:

```
fdisk -l
o con
ls -l /dev/sd*
````

Ahora le daremos el formato y le pondremos la etiqueta con el siguiente comando:

```
mkfs.ext4 -L Usuarios /dev/sdb1
````

Y haremos lo mismo para la segunda particion

```
mkfs.ext4 -L Comun /dev/sdb2
````

### Solo nos queda montar las particiones.

Primero Tenemos que crear las carpetas donde se van a montar

```
mkdir /home/iescalquera
mkdir /comun
````

Esto lo haremos en el */etc/fstab* donde añadiremos los puntos de montaje que tendran la siguiente forma:


```
/dev/sdb1 /home/iescalquera ext4 default 0 0
/dev/sdb2 /comun ext4 default 0 0
````

Una vez añadido esos parametros al archivo debemos de ejecutar el mismo comando para montar las carpetas:

```
mount -a
````

Y utilizaremos el comando ```mount o con df -h ```` para ver si se han montado correctamente.

## Crear Esqueleto

### Scripts para crear el esqueleto.

Primero debemos de crear la carpeta donde estaran localizados los scripts.

```
mkdir Scripts
cd Scripts
````

Tenemos que crear un archivo con los cursos que queremos crear.

Nosotros le llamaremos f00_cursos.txt

El contenido del archivo debe de ser el siguiente:
```
dam1
dam2
````

Debemos de crear el script con las variables globales, le llamaremos 00_variables.sh

El contenido del archivo es el siguiente:

```bash
#!/bin/bash

#Variables
DIR_HOME_LDAP=/home/iescalquera
DIR_COMUN=/comun

#Ahora debemos de exportar las variables.

export DIR_HOME_LDAP
export DIR_COMUN
```

Ahora crearemos el script para crear el esqueleto, le llamaremos 01_crear_esqueleto.sh

El contenido sera el siguiente:

```bash

#!/bin/bash

#Primero llamaremos al script de las variables

. ./00_variables.sh

#Creamos el esqueleto de los profes:

test -d $DIR_HOME_LDAP/profes || mkdir -p $DIR_HOME_LDAP/profes

#Crear el esqueleto de los alumnos y los cursos

#Cursos

#Debemos primero de leer el archivo cursos

for curso in $(cat f00_cursos.txt); do
  test -d $DIR_HOME_LDAP/alumnos/$curso || mkdir -p $DIR_HOME_LDAP/alumnos/$curso
  test -d $DIR_COMUN/$curso || mkdir -p $DIR_COMUN/$curso
done

test -d $DIR_COMUN/departamentos || mkdir -p $DIR_COMUN/departamentos
````

Ahora debemos de ejecutar el script:

```
bash ./01_crear_esqueleto.sh
````

### Crear permisos del esqueleto.

Para poder realizar esto haremos un script que le llamaremos 02_permisos_esqueleto.sh

El contenido sera el siguiente:

```bash

#!/bin/bash

#Primero llamaremos al script de las variables

. ./00_variables.sh

#Cartafol /home/iescalquera

chown root:g-usuarios $DIR_HOME_LDAP
chmod 750 $DIR_HOME_LDAP

#Cartafol /home/iescalquera/profes

chown root:g-profes $DIR_HOME_LDAP/profes
chmod 750 $DIR_HOME_LDAP/profes

#Cartafol /home/iescalquera/alumnos

chown root:g-usuarios $DIR_HOME_LDAP/alumnos
chmod 750 $DIR_HOME_LDAP/alumnos

#Cartafoles de los cursos

for curso in $(cat f00_cursos.txt); do
  chown root:g-usuarios $DIR_HOME_LDAP/alumnos/$curso
  chmod 750 $DIR_HOME_LDAP/alumnos/$curso
done

#Cartafol comun
chown root:g-usuarios $DIR_COMUN
chmod 750 $DIR_COMUN

#Cartafoles comun/dam1
chown root:g-dam1-profes $DIR_COMUN/dam1
chmod 775 $DIR_COMUN/dam1

#Cartafoles comun/dam2
chown root:g-dam2-profes $DIR_COMUN/dam2
chmod 775 $DIR_COMUN/dam2

#Cartafoles comun/departamentos
chown root:g-profes $DIR_COMUN/departamentos
chmod 775 $DIR_COMUN/departamentos
````

Ahora debemos de ejecutar el script:

```
bash ./02_ajustar_permisos_esqueleto.sh
````

### Creacion de las carpertas personales de los usuarios(home)

Todos los usuarios deben de tener la carpeta personal en /home/iescalquera

Tenemos que traer el skel de un cliente debido a que es un ubuntu; lo haremos con el siguiente comando:

```
scp -r uadmin@uclient01:/etc/skel ./
````

Una vez la tengamos en dserver00 la renombraremos con el comando:

```
mv skel skel_ubuntu
````



Para eso crearemos un script que se llamara 03_crear_carpertas_personales_ajustar_permisos.sh

```bash

#!/bin/bash

#Primero llamaremos al script de las variables globales

. ./00_variables.sh

#Volcamos todos los usuarios

getent passwd> usuarios.txt

#Extraemos los campos que nos interesan:

for USUARIO in $(awk -F':' '$3>=10000 $$ $3<60000 {print $1:$6}' usuarios.txt)
do

NOMBRE_USUARIO=$(echo $USUARIO | awk -F':' '{print $1}')
HOME_USUARIO=$(echo $USUARIO | awk -F':' '{print $2}')
GRUPO_GLOBAL_USUARIO=$(echo $HOME_USUARIO | awk -F'/' '{print $4}')

#Creamos la carpeta personal de cada usuario

test -d $HOME_USUARIO || mkdir -p $HOME_USUARIO

#Copiamos dentro de cada carpeta el contenido del skel

cp skel_ubuntu/\. $HOME_USUARIO

if [$GRUPO_GLOBAL_USUARIO = "profes"]
then 
  chown -R $NOME_USUARIO:g-usuario $HOME_USUARIO
  chmod -R 700 $HOME_USUARIO
else
#Si el grupo es alumno
  chown -R $NOME_USUARIO:g-usuario $HOME_USUARIO
  chmod -R 700 $HOME_USUARIO
fi
done

rm usuarios.txt
````

Ahora debemos de ejecutar el script:

```
bash ./03_crear_carpertas_personales_ajustar_permisos.sh
````

Y comprobamos que se ejecuto bien con un tree de */home/iescalquera*.

Si aparte queremos ver los permisos haremos:

```
tree -ugm /home/iescalquera
````

Ahora podemos iniciar sesion en el servidor y comprobar que entra en la carpeta del usuario.


## Instalacion del servicio NFS. Exportaciones

Lo primero que tenemos que hacer es instalar el paquete:

```
apt-get install nfs-kernel-server
````

Una vez hayamos creado el debemos de añadir en el fichero **/etc/exports** lo siguiente:

```
/home/iescalquera 172.16.5.0/255.255.255.0(rw)
/comun            172.16.5.0/255.255.255.0(rw)
````

Y tenemos que reiniciar el servicio

```
service nfs-kernel-server restart
````

Y comprobamos que se hayan exportado.

```
exportfs -v
````

##  Configuracion del cliente NFS

Primero debemos de instalar el paquete:

```
sudo apt-get install nfs-common
````

Ahora debemos de acceder al archivo de */etc/fstab* y añadir las carpetas compartidas en red. Pero primero debemos de crear las carpetas en la maquina real.

```
sudo mkdir /home/iescalquera
sudo mkdir /media/comun
````

En el archivo */etc/fstab* debemos añadir estas lineas:

```
dserver00:/home/iescalquera /home/iescalquera nfs default,_netdev 0 0
dserver00:/comun /media/comun nfs default,_netdev 0 0
````

Y realizaremos un ```sudo mount -a```` para que los cambios surjan efecto. 

### Iniciar sesion en el modo grafico.

Para poder iniciar sesion en el modo grafico debemos de modificar el siguiente archivo:

```
sudo nano /usr/share/lightdm/lightdm.conf.d/50-ubuntu.conf
````

Y añadir la siguiente linea:

```
greeter-show-manual-login=true
````

Y tendremos que reiniciar el servicio con el comando:

```
sudo service lightdm restart
````

### Agregar accesos directos a las carpetas montadas por NFS.

Crearemos un script cuyo nombre sera **sudo nano /opt/engadir_marcadores.sh**

```bash
#!/bin/bash
#Comprobamos si el usuario que inicia sesion es profe y no tiene creado un marcador.

if (groups ${u} | grep profes) && !(cat ~/.config/gtk-3.0/bookmarks | grep Alumnos)
then
#Añadimos el favorito al fichero pero no lo machacamos por si tiene otros marcadores.
echo file:///home/iescalquera/alumnos Alumnos>>~/.config/gtk-3.0/bookmarks
fi
```

Este script tiene que ejecutarse al iniciar la maquina por lo que añadiremos una linea en el **/etc/profile** que haga una llamada a este script.

```
sh /opt/engadir_marcadores.sh
````


## Cuotas de disco.

Primero tenemos que añadir el paquete:

```
apt-get install quota
````

Luego debemos de añadir los usuarios y grupo quota a donde queremos aplicar las quotas. Esto se hace en el **/etc/fstab**

```
UUID=f7d9a85b-5847-449a-9f98-29dfecf4239e  /home/iescalquera   ext4   defaults,usrquota,grpquota   0 0

#/dev/sdb2: LABEL="Comun" UUID="726f54f6-960b-4ad0-9ec2-35eace42290a" 
/dev/sdb2		                   /comun 	       ext4   defaults,usrquota,grpquota   0 0
````

Ahora tenemos que utilizar el siguiente comando para hacer efectivos los cambios:

```
mount -o remount /home/iescalquera
mount -o remount /comun
````


Ahora tenemos que utilizar el siguiente comando para comprobar el estado de las quotas(si da error la primera vez se vuelve a ejecutar para que no de error, si en la segunda ejecuccion hay algun error hay algun problema)

```
quotacheck -avug
````

El comando anterior se debe de ejecutar con las quotas desactivadas para lo que vamos a utilizar el comando:

```
quotaoff -avug
````

Y ahora podremos encender las quotas con el comando:

```
quotaon -av
````

### Establecer quotas a los usuarios.

Añadiremos una entrada de quotas para los profesores y para los alumnos. Para ello es necesario que exista el usuario.

```
useradd profe -M -N -u 1001
useradd alumno  -M -N -u 1002
````

Y bloquearemos la cuenta:

```
passwd -l profe
passwd -l alumno
````

Ahora estableceremos las quotas:

```
setquota -u profe 250000 300000 0 0 /home/iescalquera
setquota -u alumno 150000 200000 0 0 /home/iescalquera
````

Para comprobar los consumos de quota se puede utilizar *quota* y *repquota*

```
quota profe
````

```
repquota -a
````

Ahora crearemos un script para asignar las quotas a los usuarios.

```bash
#!/bin/bash

#Script quota usuarios

getent passwd > usuarios.txt

for usuario in $(awk -F':' '$3>=10000 && $3<60000 {print $1}' usuarios.txt)
do

if (groups $usuario | grep profes)
then
  edquota -p  profe $usuario
else
  edquota -p alumno $usuario
fi
done

rm usuarios.txt
````

Ahora ejecutamos el script:

```
bash 10_cuotas_usuario.sh
````

Y comprobamos que funcionase:

```
repquota -a
````

### Las quotas en los equipos clientes

Lo primero que tenemos que hacer es instalar el paquete:

```
sudo apt-get install quota
````

Ahora para que funcione debemos de ejecutar el siguiente comando:

```
systemctl start quotarpc
````

Ahora comprobamos si funciona conectandonos con un usuario y ejecutando el comando

```
quota
````

Para crear archivos y ver si aumenta la quota el comando es:

```
dd if=/dev/zero of=fichero50MB bs=50M count=1
````

Para editar el periodo de gracia es:

```
edquota -Tu noe
````

Para quitar las quotas a un usuario usaremos el comando:

```
setquota -u USUARIO 0 0 0 0 /home/iescalquera
````








