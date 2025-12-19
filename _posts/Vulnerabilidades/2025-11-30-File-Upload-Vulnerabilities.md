---
layout: post
title: "File Upload Vulnerabilities"
date: 2025-11-30 18:30:00 -0300
categories: [Vulnerabilidad,File Upload Vulnerabilities]
tags: [Vulnerabilidad, File Upload Vulnerabilities, Web]
description: "Vulnerabilidad File Upload Vulnerabilities"
---

## Resumen de la Vulnerabilidad

Las **File Upload Vulnerabilities** ocurren cuando una aplicación web permite a los usuarios **subir archivos sin aplicar controles de seguridad adecuados**, lo que puede permitir a un atacante subir archivos maliciosos que comprometan el sistema.

Este tipo de vulnerabilidad suele conducir a **ejecución remota de código (RCE)** y es considerada de alto impacto.


## ¿Qué puede hacer un atacante con una vulnerabilidad de subida de archivos?

Un ataque exitoso puede permitir:

* Ejecutar código arbitrario en el servidor
* Subir web shells
* Acceder o modificar archivos del sistema
* Escalar privilegios
* Comprometer completamente el servidor


## ¿Por qué ocurren estas vulnerabilidades?

Las File Upload Vulnerabilities ocurren cuando la aplicación:

* No valida el tipo de archivo
* Confía únicamente en la extensión del archivo
* No verifica el contenido real (MIME type)
* Permite ejecutar archivos subidos
* Almacena archivos en directorios accesibles públicamente

Ejemplo inseguro:

```bash
move_uploaded_file($_FILES['file']['tmp_name'], 'uploads/' . $_FILES['file']['name']);
```

## Ejemplos comunes de ataques

### 1️⃣ Subida de Web Shell

Archivo malicioso:

```bash
<?php system($_GET['cmd']); ?>
```

Nombre del archivo:

```bash
shell.php
shell.php.jpg
shell.phtml
```

---

### 2️⃣ Bypass de validaciones

* Doble extensión (`file.php.jpg`)
* Manipulación del Content-Type
* Uso de extensiones alternativas (`.php5`, `.phtml`)


## Payloads comunes

```bash
shell.php
shell.php.jpg
backdoor.phtml
cmd.php
```

## Impacto de File Upload Vulnerabilities

| Impacto         | Descripción                      |
| --------------- | -------------------------------- |
| RCE             | Ejecución remota de código       |
| Web Shell       | Control persistente del servidor |
| Data Breach     | Robo de información              |
| Full Compromise | Compromiso total del sistema     |


## Cómo prevenir File Upload Vulnerabilities

### Buenas prácticas

* Validar extensiones permitidas (lista blanca)
* Verificar el tipo MIME real del archivo
* Renombrar archivos al subirlos
* Almacenar archivos fuera del web root
* Deshabilitar ejecución en directorios de subida
* Limitar tamaño de archivos

Ejemplo seguro:

```bash
$allowed = ['jpg', 'png', 'pdf'];
```

## Cómo detectar File Upload Vulnerabilities en pentesting

* Probar extensiones maliciosas
* Manipular headers MIME
* Subir archivos con doble extensión
* Verificar ejecución del archivo
* Usar herramientas como:

  * Burp Suite
  * OWASP ZAP

## Severidad

* **Alta**
* **Crítica** si permite ejecución de código en el servidor

