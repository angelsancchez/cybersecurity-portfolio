# Informe de Seguridad - Evaluación de Host "<NOMBRE_MAQUINA>"

## 1. Resumen Ejecutivo

Se ha realizado una evaluación de seguridad sobre el host "<NOMBRE_MAQUINA>", identificado como sistema <OS> con servicios expuestos.

Durante el análisis se identificó <VECTOR_INICIAL> que permitió obtener acceso inicial al sistema. Posteriormente, mediante <VECTOR_ESCALADA>, se logró escalar privilegios hasta root.

### Impacto

- Acceso inicial no autorizado
- Ejecución remota de comandos
- Escalada de privilegios
- Compromiso total del sistema

### Nivel de riesgo: **ALTO / CRÍTICO**

---

## 2. Alcance

- Objetivo: <NOMBRE_MAQUINA>
- Sistema: <Linux/Windows>
- Tipo de prueba: Black Box
- Metodología: Enumeración → Explotación → Escalada

---

## 3. Enumeración

### 3.1 Escaneo de puertos

```bash
nmap -p- -sS --min-rate 5000 -Pn <IP>
