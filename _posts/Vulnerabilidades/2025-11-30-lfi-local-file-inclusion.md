---
layout: post
title: "LFI (Local File Inclusion)"
date: 2025-11-30 18:30:00 -0300
categories: [Vulnerabilidad,Web]
tags: [Vulnerabilidad, LFI, Web]
description: "Vulnerabilidad LFI (Local File Inclusion)"
---
## Resumen de la Vulnerabilidad

La Local File Inclusion (LFI) es una vulnerabilidad web que ocurre cuando una aplicación permite incluir archivos locales del servidor utilizando datos proporcionados por el usuario sin una validación adecuada.
Es decir podemos acceder a archivos locales de la maquina victima aunque no tengamos permisos.

## ¿Cómo se ve un código vulnerable?


```bash
$file = $_GET['page'];
include($file . ".php");
```
En este caso el servidor toma y abre lo que sea que venga en 'page'


## Buscar parametro vulnerable

La mejor opcion es ingresar al codigo fuente del sitio (ctrl+u) y buscar algo del siguiente estilo:


```bash
<a href=index.php?page=home>
<a href=load.php?file=about>
```
## Comprobar que sea vulnerable

Para comprobar que lo que encontramos sea vulnerable debemos pegarlo en el buscador y cambiar el valor como dijimos anteriormente el 'page' de la siguiente forma:

**Parametro vulnerable:**

```bash
index.php?page=home
```
**Comprobacion**

```bash 
index.php?page=/../../../etc/passwd 
```
Al buscar eso en el navegador si es vulnerable deberiamos poder acceder a dichos archivos

## Archivos comúnmente atacados mediante LFI

**Sistema Linux**

/etc/passwd

/etc/hostname

/etc/issue

/proc/self/environ

Logs del servidor web (/var/log/apache2/access.log)

**Sistema Windows**

C:\Windows\win.ini

C:\Windows\System32\drivers\etc\hosts

## Impacto de la vulnerabilidad LFI

El impacto de una vulnerabilidad LFI puede ser alto o crítico, ya que permite:

- Lectura de archivos sensibles del sistema

- Exposición de credenciales y configuraciones

- Acceso al código fuente de la aplicación

- Recolección de información para ataques posteriores

- Escalada a Remote Code Execution (RCE) en escenarios avanzados

## Cómo identificar una LFI durante un análisis de seguridad

Durante una auditoría o pentesting web, se debe prestar atención a:

- Parámetros dinámicos como page, file, path, lang, template

- Cambios en el contenido al modificar valores de la URL

- Mensajes de error del servidor relacionados con include() o require()

- Posibilidad de realizar directory traversal (../)

## Medidas de mitigación

Para prevenir vulnerabilidades LFI se recomienda:

- Validar y sanitizar estrictamente todas las entradas del usuario

- Utilizar listas blancas de archivos permitidos

- Evitar el uso directo de datos del usuario en funciones de inclusión

- Implementar rutas absolutas controladas

- Aplicar el principio de mínimo privilegio en el servidor

- Deshabilitar configuraciones inseguras del lenguaje o servidor

## Clasificación de la vulnerabilidad

**Tipo:** Inclusión de archivos

**OWASP Top 10:** Security Misconfiguration / Injection

**Severidad:** Alta (Crítica si permite RCE)

## Conclusión

La vulnerabilidad Local File Inclusion (LFI) representa un riesgo significativo para la seguridad de aplicaciones web cuando no se gestionan correctamente los datos proporcionados por el usuario. Su correcta identificación y mitigación es fundamental dentro de cualquier estrategia de seguridad ofensiva y defensiva, ya que puede ser el punto de entrada a compromisos más graves del sistema.
