# TwoMillion – Informe de Pentesting

## 1. Resumen Ejecutivo

La máquina objetivo expone una aplicación web que utiliza un sistema de registro basado en códigos de invitación junto con una API interna.

Mediante el análisis de la lógica de la aplicación, fue posible eludir el sistema de invitaciones y obtener acceso no autorizado. Posteriormente, abusando de la API, se logró escalar privilegios dentro de la aplicación hasta alcanzar acceso al sistema.

Finalmente, se explotó una vulnerabilidad local en GLIBC (CVE-2023-4911 – Looney Tunables), lo que permitió escalar privilegios hasta root.

El sistema fue completamente comprometido.

---

## 2. Alcance

- Objetivo: TwoMillion (Hack The Box)  
- Tipo de prueba: Caja individual (entorno controlado)  
- Objetivo: Identificar vulnerabilidades y comprometer el sistema  

---

## 3. Enumeración

Se realizó un escaneo completo de puertos TCP:

```bash
nmap -p- -sS --min-rate 5000 -T4 -n -Pn <IP>
```

Puertos abiertos detectados:

- 80/tcp (HTTP)
- 443/tcp (HTTPS)

Enumeración de servicios:

```bash
nmap -p80 -sCV <IP>
```

No se identificaron vulnerabilidades explotables directamente a partir de versiones o banners.

---

## 4. Análisis de la Aplicación Web

La aplicación web requiere un código de invitación para el registro de usuarios.

Se añadió el dominio al archivo local:

```bash
<IP> 2million.htb
```

Durante el análisis del código cliente (JavaScript), se identificaron llamadas a endpoints internos de la API:

```
/api/v1/invite/...
```

---

## 5. Identificación de Vulnerabilidades

### 5.1 Fallo de Control de Acceso

El sistema de invitaciones presentaba un fallo de lógica que permitía interactuar directamente con la API sin autenticación adecuada.

Fue posible generar códigos de invitación válidos sin autorización.

Tipo de vulnerabilidad:

- Broken Access Control  
- Lógica débil de aplicación  

---

## 6. Explotación

Mediante la manipulación de los endpoints de la API:

- Se generó un código de invitación válido  
- Se registró un usuario  
- Se escalaron privilegios dentro de la aplicación  

Se obtuvo acceso a funcionalidades administrativas.

---

## 7. Acceso Inicial

Se consiguió acceso al sistema vía SSH como usuario `admin`.

Verificación:

```bash
whoami
id
```

---

## 8. Escalada de Privilegios

### 8.1 Enumeración del Sistema

```bash
sudo -l
find / -perm -4000 2>/dev/null
getcap -r / 2>/dev/null
uname -a
```

No se identificaron vectores clásicos de escalada.

---

### 8.2 Vulnerabilidad GLIBC (CVE-2023-4911)

Comprobación de versión:

```bash
ldd --version
```

La versión era vulnerable a “Looney Tunables”.

Se utilizó un exploit público:

```bash
gcc exploit.c -o exploit
./exploit
```

Se obtuvo acceso root:

```bash
id
```

---

## 9. Impacto

- Control total del sistema  
- Acceso a datos sensibles  
- Posibilidad de persistencia  
- Movimiento lateral  

Impacto: Crítico  

---

## 10. Recomendaciones

- Implementar controles de acceso adecuados en la API  
- Validar correctamente el sistema de invitaciones  
- Restringir endpoints internos  
- Mantener el sistema actualizado  
- Monitorizar actividad sospechosa  

---

## 11. Conclusión

La máquina demuestra cómo una vulnerabilidad lógica en la aplicación puede derivar en un compromiso total del sistema.

El ataque se basa en:

- Análisis de lógica de negocio  
- Enumeración estructurada  
- Identificación de vulnerabilidades reales  

---

## 12. Conceptos Clave

- Broken Access Control  
- APIs inseguras  
- Enumeración web  
- Escalada de privilegios  
- CVE-2023-4911  
