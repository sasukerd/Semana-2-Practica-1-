# Laboratorio de Seguridad de Redes

## Demostración en Video
> **Enlace directo al video:** ([https://youtu.be/bq-lt1bbQ_0](https://youtu.be/bq-lt1bbQ_0))  
> *En el video se muestra la cara, voz, fecha/hora del sistema y la validación de cumplimiento de todos los objetivos de seguridad.*

## Propósito del Laboratorio
Este laboratorio tiene como objetivo implementar y validar una infraestructura de red segura utilizando un Next-Generation Firewall (FortiGate), reglas de filtrado en Switch Core, y microsegmentación en servidores Linux. Se aplican políticas de inspección profunda de paquetes (DPI), prevención de intrusiones (IPS), control de aplicaciones y límites de tasa (Rate Limiting) para mitigar ataques como SQL Injection, denegación de servicio (DoS) y comunicaciones no autorizadas.

## Diagrama de Topología y Direccionamiento IP

### Topología de Red
![Diagrama de Topología] 

<img width="697" height="637" alt="image" src="https://github.com/user-attachments/assets/24f138d8-1a62-4d04-bbc9-5f5694f3e072" />


### Esquema de Direccionamiento (Matrícula: 2025-0873)

| Dispositivo | Interfaz | Dirección IP / Máscara | Gateway Defecto | VLAN |
| :--- | :--- | :--- | :--- | :--- |
| **PC1 (Usuario)** | eth0 | `192.168.73.51/25` (DHCP) | `192.168.73.1` | VLAN 10 |
| **WEB-Server** | ens4 | `192.168.87.10/28` | `192.168.87.1` | VLAN 20 |
| **DB-Server** | ens4 | `192.168.87.20/28` | `192.168.87.1` | VLAN 20 |
| **FortiGate (LAN Usuarios)** | port1 | `192.168.73.1/25` | — | VLAN 10 |
| **FortiGate (DMZ Servidores)**| port2 | `192.168.87.1/28` | — | VLAN 20 |
| **FortiGate (WAN)** | port3 | IP asignada por ISP | Gateway ISP | N/A |

## Resumen de Configuraciones e Implementación

### 1. FortiGate (Configuración vía GUI)
- **Ruta por Defecto & NAT:** Configurada ruta estática `0.0.0.0/0` vía `port3` y NAT habilitado para la salida a Internet.
- **Políticas de Firewall:**
  - `Permitir_Usuarios_WEB`: Permite tráfico desde `192.168.73.0/25` hacia `192.168.87.10/32` en el puerto **HTTPS (443)**.
  - `Bloquear_Usuarios_DB`: Deniega tráfico explícito desde `192.168.73.0/25` hacia `192.168.87.20/32` en el puerto **MySQL (3306)**.
- **Deep Packet Inspection (DPI):** Perfil SSL/SSH Inspection configurado en modo *Deep Inspection* para decodificar tráfico HTTPS.
- **IPS (SQL Injection) & Cuarentena:** Perfil IPS activo detectando la firma `SQL.Injection`, con acción **Block** y aislamiento automático del atacante en **Quarantine Monitor**.
- **Application Control:** Regla de filtrado de archivos configurada para bloquear descargas de extensiones `.exe`.
- **DoS Policy (Rate Limiting):** Umbrales de protección para ataques TCP SYN Flood e ICMP/UDP Flood aplicados a las interfaces de entrada.

### 2. Microsegmentación en WEB-Server
- Regla de `iptables` configurada para permitir comunicación saliente exclusivamente hacia `192.168.87.20:3306` (DB-Server) y bloquear todo otro tráfico entre servidores.
