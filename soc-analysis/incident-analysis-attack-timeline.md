# Caso práctico: Detección de ataque completo (Web RCE → Reverse Shell → Kerberoasting → DCSync)

---

## 1. Resumen del escenario

Se simula un compromiso completo en un entorno corporativo:

1. Explotación de aplicación web vulnerable (RCE)
2. Obtención de reverse shell
3. Enumeración interna
4. Ataque Kerberoasting
5. Escalada a dominio mediante DCSync

---

## 2. Objetivo

Demostrar cómo un SOC puede:

* Detectar cada fase del ataque
* Correlacionar eventos
* Identificar compromiso completo

---

## 3. Timeline del ataque

```text id="timeline1"
10:01 → Exploit web (RCE)
10:02 → Ejecución de bash desde Apache
10:03 → Reverse shell (conexión saliente)
10:10 → Enumeración interna
10:15 → Kerberos requests anómalos (Kerberoasting)
10:25 → DCSync (extracción de hashes)
```

---

## 4. Fase 1: Explotación web (RCE)

### Evidencia

* Requests HTTP anómalos
* Parámetros sospechosos (#post_render, command injection)

---

### Detección

```spl id="t1"
index=web_logs ("#post_render" OR "cmd=" OR "bash")
```

---

### Análisis

* Peticiones fuera de comportamiento normal
* Indicador de explotación activa

---

## 5. Fase 2: Ejecución de comandos (Web → sistema)

### Evidencia

```text id="t2"
apache → bash
```

---

### Detección

```spl id="t3"
index=sysmon parent_process_name=apache process_name=bash
```

---

### Análisis

* Un servidor web no debería lanzar shells
* Indicador directo de RCE

---

## 6. Fase 3: Reverse Shell

### Evidencia

```text id="t4"
bash conecta a IP externa (puerto 4444)
```

---

### Detección

```spl id="t5"
index=sysmon EventCode=3
| search dest_port=4444 OR dest_port>1024
| stats count by dest_ip, process_name
```

---

### Análisis

* Conexión saliente sospechosa
* Relacionada con ejecución previa

---

## 7. Fase 4: Kerberoasting

### Evidencia

* Solicitudes masivas de tickets Kerberos

---

### Detección

```spl id="t6"
index=wineventlog EventCode=4769
| stats count by Account_Name, src_ip
| where count > 10
```

---

### Análisis

* Usuario solicita múltiples tickets
* Indicador de extracción de hashes

---

## 8. Fase 5: DCSync

### Evidencia

* Acceso a replicación de Active Directory

---

### Detección

```spl id="t7"
index=wineventlog EventCode=4662
| search "Replicating Directory Changes"
```

---

### Análisis

* Actividad crítica
* Solo debería ocurrir en DC

---

## 9. Correlación del ataque

```text id="timeline2"
RCE web
→ ejecución bash
→ reverse shell
→ actividad Kerberos
→ DCSync
```

👉 Mismo host / misma IP / mismo usuario

---

## 10. Conclusión del incidente

El sistema ha sido completamente comprometido:

* Acceso inicial mediante RCE
* Persistencia mediante shell
* Escalada mediante credenciales
* Compromiso total del dominio

---

## 11. Respuesta recomendada (SOC)

1. Aislar el host comprometido
2. Revocar credenciales afectadas
3. Analizar movimiento lateral
4. Revisar logs históricos
5. Aplicar parches y hardening

---

## 12. MITRE ATT&CK

* T1190 – Exploit Public-Facing Application
* T1059 – Command Execution
* T1071 – Application Layer Protocol
* T1558.003 – Kerberoasting
* T1003.006 – DCSync

---

## 13. Valor para SOC

Este caso demuestra:

* Detección por fases
* Correlación de eventos
* Comprensión del ciclo completo de ataque
* Capacidad de análisis realista (nivel junior)

---

## 14. Conclusión final

Un ataque completo no se detecta con una sola alerta.

* correlacionar eventos
* entender el contexto
* detectar comportamiento anómalo
