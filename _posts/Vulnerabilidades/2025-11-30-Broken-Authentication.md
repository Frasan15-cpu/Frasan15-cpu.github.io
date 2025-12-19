---
layout: post
title: "Broken Authentication"
date: 2025-11-30 18:30:00 -0300
categories: [Vulnerabilidad,Broken Authentication]
tags: [Vulnerabilidad, Broken Authentication, Web]
description: "Vulnerabilidad Broken Authentication"
---
## Resumen de la Vulnerabilidad

**Broken Authentication** es una vulnerabilidad de seguridad que ocurre cuando los mecanismos de autenticación y gestión de sesiones de una aplicación están mal implementados, permitiendo a un atacante **suplantar identidades, acceder a cuentas ajenas o evadir el proceso de login**.

Esta vulnerabilidad afecta directamente a la **identidad de los usuarios** y suele ser una de las más críticas en aplicaciones web.

## ¿Qué puede hacer un atacante con Broken Authentication?

Un ataque exitoso puede permitir:

* Acceder a cuentas de otros usuarios
* Secuestrar sesiones activas
* Bypassear el sistema de login
* Escalar privilegios
* Obtener control administrativo de la aplicación

## ¿Por qué ocurre Broken Authentication?

Broken Authentication ocurre cuando la aplicación:

* Usa contraseñas débiles o sin políticas de complejidad
* No limita intentos de inicio de sesión
* Gestiona mal las sesiones (tokens predecibles, no expiración)
* No invalida sesiones tras cerrar sesión o cambiar contraseña
* Expone identificadores de sesión en URLs

Ejemplo inseguro:

```bash
GET /dashboard?sessionid=12345
```

## Ejemplos comunes de Broken Authentication

### 1️⃣ Fuerza bruta

* No existe limitación de intentos
* No hay bloqueo de cuentas

### 2️⃣ Credential Stuffing

* Uso de credenciales filtradas
* Falta de detección de patrones anómalos

### 3️⃣ Session Fixation

* El atacante fija un ID de sesión conocido

### 4️⃣ Tokens inseguros

* Tokens predecibles o reutilizables


## Ejemplos de pruebas

```bash
admin:admin
admin:123456
user:user
```

Manipulación de sesión:

```bash
Cookie: PHPSESSID=abcdef123456
```


## Impacto de Broken Authentication

| Impacto              | Descripción                          |
| -------------------- | ------------------------------------ |
| Account Takeover     | Control total de cuentas             |
| Privilege Escalation | Acceso a funciones administrativas   |
| Data Exposure        | Robo de información sensible         |
| System Compromise    | Compromiso completo de la aplicación |


## Cómo prevenir Broken Authentication

### Buenas prácticas

* Implementar políticas fuertes de contraseñas
* Limitar intentos de login
* Usar autenticación multifactor (MFA)
* Generar tokens de sesión seguros y aleatorios
* Expirar e invalidar sesiones correctamente
* No exponer tokens en URLs

Ejemplo de configuración segura:

```bash
Session timeout: 15 minutos
HTTPOnly: true
Secure: true
```


## Cómo detectar Broken Authentication en pentesting

* Probar fuerza bruta y credential stuffing
* Analizar gestión de sesiones
* Revisar cookies (`HttpOnly`, `Secure`)
* Verificar logout y expiración
* Usar herramientas como:

  * Burp Suite
  * Hydra
  * OWASP ZAP

## Severidad

- **Alta**
- **Crítica** si permite Account Takeover o acceso administrativo


