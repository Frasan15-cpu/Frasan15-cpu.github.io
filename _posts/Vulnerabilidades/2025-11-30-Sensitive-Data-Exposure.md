---
layout: post
title: "Sensitive Data Exposure"
date: 2025-11-30 18:30:00 -0300
categories: [Vulnerabilidad,Sensitive Data Exposure]
tags: [Vulnerabilidad, Sensitive Data Exposure, Web]
description: "Vulnerabilidad Sensitive Data Exposure"
---
## Resumen de la Vulnerabilidad

**Sensitive Data Exposure** es una vulnerabilidad de seguridad que ocurre cuando una aplicación web **no protege adecuadamente información sensible**, permitiendo que datos críticos sean expuestos, transmitidos o almacenados de forma insegura.

Este tipo de vulnerabilidad puede afectar directamente la **confidencialidad** de los usuarios y de la organización.


## ¿Qué tipo de datos pueden verse expuestos?

La exposición de datos sensibles puede incluir:

* Credenciales (usuarios y contraseñas)
* Información personal (PII)
* Datos financieros (tarjetas de crédito, cuentas bancarias)
* Tokens de sesión
* Claves API y secretos
* Información médica o legal


## ¿Por qué ocurre Sensitive Data Exposure?

Esta vulnerabilidad ocurre cuando la aplicación:

* Transmite datos sin cifrado (HTTP en lugar de HTTPS)
* Usa algoritmos criptográficos débiles o rotos
* Almacena contraseñas en texto plano
* Expone información sensible en logs o respuestas
* No protege backups o archivos de configuración

Ejemplo inseguro:

```bash
password=123456
```


## Ejemplos comunes de exposición

### 1️⃣ Contraseñas en texto plano

* Bases de datos sin hashing
* Archivos de configuración accesibles

### 2️⃣ Uso de HTTP sin TLS

* Credenciales interceptables
* Riesgo de ataques Man-in-the-Middle

### 3️⃣ Información sensible en respuestas

```bash
{
  "password": "admin123",
  "token": "abcd1234"
}
```


## Escenarios frecuentes

* APIs que devuelven datos innecesarios
* Backups accesibles públicamente
* Repositorios con secretos expuestos
* Logs con información sensible


## Impacto de Sensitive Data Exposure

| Impacto        | Descripción                      |
| -------------- | -------------------------------- |
| Data Breach    | Fuga de información confidencial |
| Identity Theft | Robo de identidad                |
| Financial Loss | Fraude económico                 |
| Legal Impact   | Multas y sanciones regulatorias  |


## Cómo prevenir Sensitive Data Exposure

### Buenas prácticas

* Usar HTTPS con TLS actualizado
* Aplicar hashing fuerte para contraseñas (bcrypt, Argon2)
* Cifrar datos sensibles en reposo
* No exponer información innecesaria
* Proteger backups y archivos de configuración
* Gestionar secretos de forma segura

Ejemplo seguro:

```bash
password_hash(password, PASSWORD_BCRYPT)
```

## Cómo detectar Sensitive Data Exposure en pentesting

* Revisar tráfico HTTP/HTTPS
* Analizar respuestas de la aplicación
* Buscar datos sensibles en código y repositorios
* Revisar archivos accesibles públicamente
* Usar herramientas como:

  * Burp Suite
  * OWASP ZAP
  * TruffleHog

## Severidad

* **Media a Alta**
* **Crítica** si se exponen credenciales o datos financieros
