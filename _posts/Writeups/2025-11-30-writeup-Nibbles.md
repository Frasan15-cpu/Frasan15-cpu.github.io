---
layout: post
title: "Nibbles"
date: 2025-11-30 18:30:00 -0300
categories: [Writeups,Nibbles]
tags: [Htb, Nibbles, Linux]
description: "Writeup de la máquina nibbles de Hackthebox"
image: /assets/images/hackthebox.png
---
## Resumen de la resolución

**Nibbles** es una máquina **Linux** de dificultad **Easy** de la plataforma de **HackTheBox**, fue una máquina de Hack The Box la cual se puede enumerar un nombre de usuario y su contraseña de forma sencilla.

---
## Enumeración

En primer lugar, debemos desplegar la máquina para poder obtener la **Dirección IP** todo ello desde la web de **HackTheBox** y luego desde la **terminal** debemos conectarnos a la VPN usando el fichero correspondiente de la siguiente forma

```bash
openvpn lab_Frasan15.opvn
```

Luego le lanzamos un **ping** para comprobar si la maquina esta activa. Verificamos que responde correctamente devolviendo el paquete enviado. A continuacion, utilizamos un script propio que, a partir del valor del  **ttl**  permite identificar el tipo de sistema operativo: **Linux (TTL 64 )** o **Windows (TTL 128)**, en este caso, observamos un **TTL** próximo a 64 (**64**) por lo que se trata de una máquina **Linux**

> El motivo por el cual el **TTL** es de **64** es porque el paquete pasa por unos intermediarios (routers) antes de llegar a su destino (máquina atacante). Esto podemos comprobarlo con el comando `ping -c 1 -R 10.129.42.197`.

### Nmap

En segundo lugar, realizaremos un escaneo por **TCP** usando **Nmap** para ver que puertos de la máquina víctima se encuentra abiertos.

```bash
nmap 10.129.42.197 -p- --open -T5 -v -n -oG allPorts
```

Observamos como nos reporta los puertos abiertos.

Puerto | Protocolo | Servicio | Versión        | 
------ | --------- | -------- | -------------- | 
22     | TCP       | SSH      | OpenSSH 7.2    | 
80     | TCP       | HTTP     | Apache 2.4.18  | 

Ahora, gracias a la utilidad **extractPorts** definida en nuestra **.zshrc** podremos copiarnos cómodamente todos los puerto abiertos de la máquina víctima a nuestra **clipboard**.

A continuación, volveremos a realizar un escaneo con **Nmap**, pero esta vez se trata de un escaneo más exhaustivo pues lanzaremos unos script básicos de reconocimiento, además de que nos intente reportar la versión y servicio que corre para cada puerto:

```bash
nmap -sC -sV -p22,80 10.129.42.197 -oN targeted
```

En el segundo escaneo de **Nmap**, descubrimos las versiones tanto del puerto 22 ssh como del puerto 80 http.

___
### Puerto 22 - SSH

Como **SSH** permite inicios de sesion, intentamos conectarnos sin auteniticacion pero no tenemos exito.


### Puerto 80 - HTTP (Web)

Lo primero que realizaremos es identificar tecnologias utilizadas en dicho servicio para eso utilizamos el siguiente comando:

```bash
whatweb http://10.129.42.197
```

Dicho comando nos proporciono informacion la cual no nos es de mucha ayuda actualmente.

### Investigar Puerto 80 - HTTP (Web)

Una vez hecho eso abrimos el sitio web y realizamos una inspeccion de codigo, dentro de la inspeccion encontramos una ruta /nibblesblog/.
Entramos al sitio con dicha ruta y buscamos informacion dentro del mismo

### Gobuster

Realizamos un escaneo con gobuster para lograr identificar rutas dentro del sitio.

```bash
gobuster dir -u http://10.129.42.197/nibbleblog/ -w /usr/share/wordlists/dirb/common.txt
```
El escaneo nos reporto rutas como **/admin.php**, la cual es una ruta de inicio de sesion.
Otra ruta que nos dio fue  **/nibbleblog/content/** la cual al ingresar y  entrar a el archivo private dentro de ella identificamos un nombre de usuario **admin**

### Loging

Como comentamos anteriormente ya tenemos una ruta con un login y logramos identificar un nombre de usuario por lo que comenzamos probando con contraseñas por defecto o basicas de forma manual como lo son las siguientes:
 
- admin
- 12345678
- Top
- password
- root

Al probar con dichas contraseñas y no poder, ya casi rendidos probe con una contraseña basica como la es el nombre de la maquina **nibbles** y tuvimos exito, logramos iniciar sesion.
En el caso de no haber tenido suerte hubieramos probado con la herramienta hydra.

## Explotacion

Al investigar el sitio logramos identificar un apartado **plugins**, ingresamos en el mismo y vemos que en el archivo **myimage** podemos subir archivos, por lo que subimos un archivo de una reverse shell.

En otra terminal utilizamos el siguiente comando para escuchar la peticion y ver si logramos conseguir acceso al sistema.


```bash
nc -lvnp 9001
```
Una vez teniendo el puerto en escucha ingresamos al sitio web con el archivo creado, es decir **http://10.129.42.197/nibbleblog/content/private/plugins/my_image ** y volvemos a la terminal con el **nc** abierto y comprobamos que se conseguimos acceso a dicha consola.

Realizamos un **whoami** y confirmamos que somos el usuario nibbler

## Escalada de privilegios

Para escalar privilegios y pasar a root debemos realizar lo siguiente:

```bash
cd /home/nibbler/personal/stuff
echo "/bin/bash" > monitor.sh
chmod +x monitor.sh
sudo ./monitor.sh
whoami
```
Una vez realizado eso ya deberiamos de tener privilegios **ROOT**


La maquina **nibbles** me parecio una maquina muy buena y divertida en cuanto a las maquinas http, muy recoemndable de hacer.
