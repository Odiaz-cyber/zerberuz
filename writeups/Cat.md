**INDICE:**

1.git -> Githack

2.GET -> datos , viajan con GET osea en al url

3.Apache -> php

4.Sqlite

5.XSS -> cookie

6.High Jacking -> cookie

7.Injeccion SQL -> sqlite

8.python

9.logs -> apache , solicitudes por get , url logs

10.Gitea -> mail , index.php , xss , v1.22



Maquina -> Linux:

Reconicimiento :

```
nmap -p-  --open -sS -Pn -n --min-rate 5000 -vvv ip -oG allPorts
```

Recordar parametros :

**-p-**-> todos los puertos (65535)

 **--open** -> filtrar por puertos abiertos
 
**-sS** -> stelth scan lo que permite hacer un escaneo sigiliso evitadno el try way handshake 
SYN->SYN-ACK->RESET-PAQUET, asi eveita el TWH pq en ves de reset paquet hay que devolverle el ACK

**-Pn** -> Hay 2 acciones que nmap realiza que realentizan el escaneo y una  es host discovery (ARP) osea descubrimiento de host con este paramtro eviteamos que esto pase

**-n** -> Es es el segundo parametro que realintiza la operacion que es la resolucion DNS con este evitamos que ocurra

**--min-rate 5000** -> Este parametro lo que hace es filtar por paquetes no mas lentos que 5000 paqutes por segundo

**-vvv**  -> Este es el verbose , mostrar en tiempo real del output del comando


Puertos abiertos:

```
22 -> SSH

80 -> HTTP
```

Escaneo mas potente dirigido a los puertos abiertos

```
nmap -p22,80  -sCV ip -oN targeted
```

Recordando parametros:

**-sC** -> Este ejecuta un conjunto de script de nmap los mas populares

-sV -> Para ver las versiones de los servicios que corren en esos puertos 

Tambien se usan : -sCV

![[nmap.png]]
APACHE -> PHP
Podemos ver por la cabecera de OPENSSH el codename del Ubuntu  , esto es simplemente informativo.

Buscando en Google:

![[codename.png]]

Es un Ubuntu Focal

Se esta aplicando virtual hosting a :

```
http://cat.htb
```

Nuestra maquina no reconce este nmbre de dominio por lo que :

```
nano /etc/host
```

Dentro de este archivo pones :

```
ip    http://cat.htb
```


La web:

![[web.png]]

Podemos probar este script:

```
nmap -p80 --script http-enum ip  -oN webScan
```

--script -> ejecutamos script de nmap

http-enum -> es para fuzzear en la web para ver si ese directorio o recurso existe es pequeno la wordlist


Mientras investgando la web :

Podemos ver que es una web orientada a gatos donde se puede votar a ver que gatos gusta mas :

Hay varios gatos y panel de :

Registro:

![[register.png]]


Login:

![[login.png]]

Nos registramos y despues no logeamos y todo funciona 

Una vez logeados podemos acceder a la seccion de contest que donde te permniten participar en el concurso de gatos subiendo los datos de el gato junto con una foto

![[concurso.png]]

Una llenados los datos podemos esta solcitud pasarla por Burpsiute  a ver como funciona :

![[burpsuite (2).png]]


En este caso se podrian tratar  de vulnerar el sistema de subida de archivo pero la web esta bien montada por lo  que no funciona asi que descativamos el proxy de burpsuite para que fluya la peticion

![[inspection.png]]

Una vez subido , manda un mensaje de que alguien va a inpeccionar el contenido subido


El fuzzeo no nmap reporto un directorio :

```
.git
```

Sabiendo la existencia de directorio se pude usar herramientas como GITHACK para descargar el contenido de el y ver  los recursos de la pagina web

Clonamos el repositorio de GItchack

```
git clone enlace
```

Se ejcuta tan facil como:

```
python Githack.py
```

Te pide el url con la extencion /.git:

```
python Githack.py http://cat.htb/.git
```

Se te descarga todo el proyecto  :

si usas 

```
tree 
```

Se vizualiza el projecto  cat.htb


![[cat.png]]

1.Inspeccinando el archivo **join.php** lo mas interesante fue:

![[get.png]]
php

Que los recursos como nombre de usuario , email y passwd viajan por GET osea en la url osea una vez ganado acceso a la maquina son visibles en los logs

2.Inspeccionando el archivo **config.php** , estos archivos aveces tienen credeenciales  en texto claro

**![[config.php.png]]*
PDO -> seguridad a ijecciones SQL
Donde se almacena la base de datos

sqlite -> En `payloadallthethings ` -> en esta ruta al payload para usar en injecciones sql con sqlite o cualquier base da datos sql


3.Inspeccionando el archivo **accept_cat.php**

![[accept.png]]

username -> axel

Osea este recurso en la web es gestionado por axel por lo que en este hay una seccion del programa que si logramos modificar podemos  hacer cosas como:

![[accept (2).png]]

**XSS**

En esta seccion  vemos que uno de los campos que inserta es el del owner-username y al hora de hacer un  XSS no esta bien sanitzado por lo que parece que permite usuarios registrados con estructuras un tanto anomlas:

![[owner.php.png]]

Porque parace que permite xss pq en el archivo join.php pasa que toma el username y pasa sin ningun tipo de validacion o sanitizacion especifica para xss:

![[validacion.png]]


Como validar el XSS :

Te deslogearia y en le seccion de registro :

![[xss.png]]

En nuestra maquina nos montamos un server http 

```
python3 -m http.server 8080
```

Llenamos todos los campos del register

Te permite registrarte con ese usario y despues te logeas de manera rapida y subimos los datos del gato de nuevo

Una vez enviado en nuestro server de http sale :

![[server.png]]

Porque la respuesta al server llega solo una vez envias los datos de los gatos ?

Pq las accept_cat.php en la web al no sanitizar el campo de username join.php , las etiquetas script en la web se ejecutan  y es vulnerable a xss

Ahora la PHPSESSID -> esta cookie se session no es mia es la del usuario peritmitido en la web donde se revisan los datos pq recordar que los datos se envian en una parte para su revision que es la parte vulnerable esa es la cookie en es ssesion que obtuvimos

**High Jacking**

![[cookie.png]]

HTTPOnly -> permite el XSS

Si sustituyes la cookie tuya por la que obtuviste usas `CTRL + R`

Vemos que tenemos otra seccion

Una vez obtendio al usario Axel , haciendo si no me equivoco Lateral Movment para un usuario con mayor privilegio para poder acceder al recuro de acept_cat.php para poder realizar la injeccion sql con el parametro que vimos anteriormente

Ahora hay que interceptar el **accept_cat.php** lo ingresamos el navegador pq recordar que aqui es donde se realiza la injeccion SQL

![[accept.png]]
Y cuando lo haces , solo te redirige osea no te da error , lo que podemos es interecpetar esta peticion con BURPSUITE , podemos alterar los valores para ingrear al base de datos


Ponemos  la ruta , despues esto aparace en burpsuite , cuando  usamos Click + Derecho , change request method esto cambia que originalmente es por get a post que te lo pide el   accept_cat que vimos anteriormente
![[burp.png]]
Te pide que proporciones el cat id y el cat name 

ingresas

```
catname=test&catid=1
```

ese nombre va  a :
![[accept (2).png]]

cat_name -> aqui va el nombre del gato , en este caso test

```
INSERT INTO accepted_cats (name) VALLUES ('$cat_name')";
```
Entonces si aqui pones:

```
catname=test')&catid=1
```

![[internal 1.png]]

Y envias en bupsuite da un internal server error

Esto quiere decir que hay una injeccion SQL y sabemos q se usa sqlite

Para sqlite para explotar manualmente se puede concatenar cadenas :

```
INSERT INTO accepted_cats (name) VALLUES ('test'||1/1||'')";
```

```
catname=test'||1/1||''&catid=1
```

Claro 1/1 = 0 , por lo que esta bien , pero si pones 1/0 da error pq no puedes dividir ningun numero por cero 

Si usas esta condicion para concatenar cadenas , puedes  aprovechar ese espacio para retornar un 1 o un 0 en dependencia de una query que especifique , con un  1 o 0 se refieres a error o no basicamente:

```
catname=test'||1/(substr((select usernames from users limit 0,1),1,1))=
'a'||''&catid=1
```

Que hace esto -> sencillo :

```
(substr((select usernames from users limit 0,1),1,1))='a'
```

En este caso estamos probando suponiendo que halla una tabla llamada users un campo llamado usaernames estamos con substr el primer valor de ese campo y si es valido devuelve 1 sino 0 y 1/1 = correcto  200  ok , 1/0 = incorrecto 500 bad

```
0,1
```

Esta seccion determina a q parte usuario estamos fila estamos accediendo osea:

En este caso a la primero fila pero si ponemos:

1,1 -> segunda fila etc

1,1 -> este refleja a que letra estamso accediendo de ese valor

Con burpsuite se puede probar automatizar esta tarea de buscar

Usuarios:

axel
rosa
robert

Ahora va a crear un script en python para determinar las contrasenas de cada usuario:

Si quiero ver es script ver el video de savitar maquina CAT min 50 o alrededor

md5 -> a-f , 0-9

hashes.com -> para crakear hashes

Hay que buscar la contrasena de cada usario encontrado pero el adelanto la busqueda y dijo que rosa es la objetivo asi , con ros usamos el script de pyhton da una passwd hasheada con md5  y despues usamos hashes.com la crakeamos y la passwd de rosa es :

```
soyunaprincesarosa
```

nos conectamos con ssh 

```
ssh rosa@ip
```

entramos

```
export TERM=xterm
```

Para usar ctrl + l

**Escalada de privilegios**:

```
sudo -l 
```
nada

primero vemos a que grupo pertenece rosa

```
id 
```
Estamos en los grupos adm -> este gurpo lo que permite es ver ciertos logs

```
find  / -group adm 2>/dev/null
```

Buscamos desde la raiz los archivos que tengan como grupo adm

si recordamos que habia datos que viajaban con get osea en la url en los archivos logs pueden quedar registro de eso

```
cat /var/logs/apache2/access.logs
```

vemos las credenciales de axel

![[axel.png]]

nos conectamos por ssh

![[correo.png]]

Entramos y dice que tenemos un correo

La ruta para ver correos es en :

```
/var/mail
```


El correo dice que envie un correo a jobert en el localhsot con info de mi repo de gitea
El puerto por default de gitea es -> 3000

![[gitea.png]]

Para ver puertos abiertos en la maquina
```
ss -ntlp
```

Vemos el puerto 3000

Ya que tenemos el usario axel para ver el contendio de gitea podemos aplicar un local port forwarding para verlos

```
ssh -L axel@ip_victima 3000:127.0.0.1:3000 
```

Para que el puerto 3000  de mi maquina sea el mismo que el de la maquina victima

Si en mi maquna en el navegador ponemos localhost:3000 vemos:

![[Pasted image 20250917131825.png]]


La version usada en este caso de gitea es la 1.22 si buscamos en google gitea 1.22 exploit , encotramos un xss como exploit

A partir de aqui ver el video para ver como uso ese exploit , el correo y otras cosas para escalar privilegios -> min 1.05 -> final