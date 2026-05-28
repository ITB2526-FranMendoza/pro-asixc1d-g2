# Infraestructura AWS — InnovateTech

> Documentación técnica de la infraestructura cloud desplegada en Amazon Web Services para el proyecto InnovateTech.  
> **Responsable:** Gerard [itb] romo sanchex  
> **Proyecto:** Proyecto Transversal 25/26 — InnovateTech  


---

## Índice

- [Arquitectura Híbrida](#-arquitectura-híbrida)
- [Servicios Desplegados](#-servicios-desplegados)
- [Seguridad](#-seguridad)
- [Ventajas](#-ventajas)

---

## Arquitectura Híbrida

InnovateTech utiliza una infraestructura híbrida combinando CPD físico y AWS.

El CPD on-premise convive con instancias EC2 desplegadas en AWS (`eu-north-1`), distribuidas en **3 VPCs independientes** y segmentadas mediante **VLANs** para aislar el tráfico por tipo de servicio.

```
┌─────────────────────────────────────────────────────────────────────┐
│                        AWS  eu-north-1                              │
│                                                                     │
│  ┌──────────────────────────────────────┐                           │
│  │  VPC: innovatetech-g2                │                           │
│  │  vpc-0f50d9b1d874b8688               │                           │
│  │                                      │                           │
│  │  ┌──────────────────────────────┐    │                           │
│  │  │  Subred Pública              │    │                           │
│  │  │  subnet-09dc1cd56ab839493    │    │                           │
│  │  │  ┌────────────────────────┐  │    │                           │
│  │  │  │  BD-Ruben-G2           │  │    │                           │
│  │  │  │  Base de Datos         │  │    │                           │
│  │  │  │  t3.small              │  │    │                           │
│  │  │  │  13.53.152.159 (EIP)   │  │    │                           │
│  │  │  └────────────────────────┘  │    │                           │
│  │  └──────────────────────────────┘    │                           │
│  │                                      │                           │
│  │  ┌──────────────────────────────┐    │                           │
│  │  │  Subred Privada              │    │                           │
│  │  │  subnet-0240adae937576373    │    │                           │
│  │  │  ┌────────────────────────┐  │    │                           │
│  │  │  │  Jitsi-Ruben-G2        │  │    │                           │
│  │  │  │  Videoconferencia      │  │    │                           │
│  │  │  │  t3.small              │  │    │                           │
│  │  │  │  13.62.220.208 (EIP)   │  │    │                           │
│  │  │  └────────────────────────┘  │    │                           │
│  │  │  ┌────────────────────────┐  │    │                           │
│  │  │  │  Server-logs           │  │    │                           │
│  │  │  │  Gestión de Logs       │  │    │                           │
│  │  │  │  c7i-flex.large        │  │    │                           │
│  │  │  │  172.31.6.43 (priv.)   │  │    │                           │
│  │  │  └────────────────────────┘  │    │                           │
│  │  └──────────────────────────────┘    │                           │
│  └──────────────────────────────────────┘                           │
│                                                                     │
│  ┌──────────────────────────────────────┐                           │
│  │  VPC: Samba_AD                       │                           │
│  │  vpc-08a2ef35bf1f60aa9               │                           │
│  │                                      │                           │
│  │  ┌──────────────────────────────┐    │                           │
│  │  │  Samba_Subnet (Pública)      │    │                           │
│  │  │  subnet-01df6764bf6a2fabd    │    │                           │
│  │  │  ┌────────────────────────┐  │    │                           │
│  │  │  │  Web-SFTP              │  │    │                           │
│  │  │  │  VLAN 20 — Servidores  │  │    │                           │
│  │  │  │  t3.small              │  │    │                           │
│  │  │  │  13.63.204.200 (EIP)   │  │    │                           │
│  │  │  └────────────────────────┘  │    │                           │
│  │  │  ┌────────────────────────┐  │    │                           │
│  │  │  │  Samba_AD              │  │    │                           │
│  │  │  │  Active Directory      │  │    │                           │
│  │  │  │  VLAN 20 — Servidores  │  │    │                           │
│  │  │  │  t3.small              │  │    │                           │
│  │  │  │  13.62.146.201 (EIP)   │  │    │                           │
│  │  │  └────────────────────────┘  │    │                           │
│  │  │  ┌────────────────────────┐  │    │                           │
│  │  │  │  Server_Audio          │  │    │                           │
│  │  │  │  Streaming Audio       │  │    │                           │
│  │  │  │  VLAN 30 — Streaming   │  │    │                           │
│  │  │  │  t3.small              │  │    │                           │
│  │  │  │  16.171.107.58 (EIP)   │  │    │                           │
│  │  │  └────────────────────────┘  │    │                           │
│  │  │  ┌────────────────────────┐  │    │                           │
│  │  │  │  Gestionado            │  │    │                           │
│  │  │  │  Streaming Vídeo       │  │    │                           │
│  │  │  │  VLAN 30 — Streaming   │  │    │                           │
│  │  │  │  t3.micro              │  │    │                           │
│  │  │  │  16.171.115.99         │  │    │                           │
│  │  │  └────────────────────────┘  │    │                           │
│  │  └──────────────────────────────┘    │                           │
│  └──────────────────────────────────────┘                           │
│                                                                     │
│  ┌──────────────────────────────────────┐                           │
│  │  VPC: vpc-0658e7daf52b29d35          │                           │
│  │                                      │                           │
│  │  subnet-0b7646a89d886ab00            │                           │
│  │  ┌──────────────────────────────┐    │                           │
│  │  │  Fran-Video-Server           │    │                           │
│  │  │  Streaming Vídeo             │    │                           │
│  │  │  VLAN 30 — Streaming         │    │                           │
│  │  │  t3.micro                    │    │                           │
│  │  │  13.61.226.79 (EIP)          │    │                           │
│  │  └──────────────────────────────┘    │                           │
│  └──────────────────────────────────────┘                           │
└─────────────────────────────────────────────────────────────────────┘
                          │
               [Internet / VPN Site-to-Site]
                          │
         ┌────────────────────────────────┐
         │        CPD Físico              │
         │        On-Premise              │
         │                                │
         │  VLAN 10 — Gestión             │
         │  VLAN 20 — Servidores          │
         │  VLAN 30 — Streaming           │
         │  VLAN 40 — Invitados           │
         │  VLAN 50 — Usuarios            │
         └────────────────────────────────┘
```

### VPCs y Redes

| VPC | ID | Cuenta AWS | Subredes |
|---|---|---|---|
| `innovatetech-g2` | `vpc-0f50d9b1d874b8688` | `147826551864` | Pública + Privada |
| `Samba_AD` | `vpc-08a2ef35bf1f60aa9` | `107365729991` | Samba_Subnet (Pública) |
| _(sin nombre)_ | `vpc-0658e7daf52b29d35` | `1080795438887` | Pública |

### Segmentación VLAN (CPD)

| VLAN | ID | Red | Propósito |
|---|---|---|---|
| Gestión | VLAN 10 | `192.168.10.0/24` | Administración y monitorización de infraestructura |
| Servidores | VLAN 20 | `192.168.20.0/24` | Tráfico entre servidores (AD, Web, SFTP, BD) |
| Streaming | VLAN 30 | `192.168.30.0/24` | Servicios multimedia (audio, vídeo, Jitsi) |
| Invitados | VLAN 40 | `192.168.40.0/24` | Red aislada para dispositivos invitados |
| Usuarios | VLAN 50 | `192.168.50.0/24` | Equipos de usuario final corporativos |

### Equipos de red (CPD)

| Equipo | Función |
|---|---|
| MikroTik CCR2004 | Router principal |
| pfSense | Firewall perimetral |
| Cisco Catalyst | Switch core con soporte VLAN |

---

## Servicios Desplegados

Todos los servicios corren sobre instancias EC2 en AWS (`eu-north-1`). Cada instancia está aislada y dedicada a un único rol. La automatización del aprovisionamiento se gestiona mediante **Ansible**.

### Inventario completo

| Nombre | Servicio | ID de Instancia | Tipo | IP Pública | IP Privada | VPC |
|---|---|---|---|---|---|---|
| **Web-SFTP** | Web + SFTP | `i-0e19ac2dfbbd9ec0d` | `t3.small` | `13.63.204.200` ✅ | `172.16.34.17` | Samba_AD |
| **Samba_AD** | Active Directory | `i-092074046b4f94411` | `t3.small` | `13.62.146.201` ✅ | `172.16.34.18` | Samba_AD |
| **Server-logs** | Logs | `i-0169f47bf1fcc7a3a` | `c7i-flex.large` | `16.192.46.108` | `172.31.6.43` | innovatetech-g2 |
| **Server_Audio** | Streaming Audio | `i-01d554ec3151e75a1` | `t3.small` | `16.171.107.58` ✅ | `172.16.34.19` | Samba_AD |
| **Gestionado** | Streaming Vídeo | `i-0b3447c1dd6810ac6` | `t3.micro` | `16.171.115.99` ⚠️ | `172.16.34.26` | Samba_AD |
| **Fran-Video-Server** | Streaming Vídeo | `i-025eff8c23ec4f60a` | `t3.micro` | `13.61.226.79` ✅ | `172.31.42.165` | vpc-0658e7 |
| **Jitsi-Ruben-G2** | Videoconferencia | `i-0d05703ab0f3272cb` | `t3.small` | `13.62.220.208` ✅ | `172.31.8.42` | innovatetech-g2 |
| **BD-Ruben-G2** | Base de Datos | `i-034ff7dd473f23011` | `t3.small` | `13.53.152.159` ✅ | `172.31.24.225` | innovatetech-g2 |

> ✅ Elastic IP asignada — IP fija que no cambia entre reinicios.  
> ⚠️ Sin Elastic IP — la IP pública puede cambiar al reiniciar la instancia.

---

### EC2 Web + SFTP

| Campo | Valor |
|---|---|
| **ID** | `i-0e19ac2dfbbd9ec0d` |
| **Tipo** | `t3.small` |
| **IP Pública** | `13.63.204.200` (Elastic IP) |
| **IP Privada** | `172.16.34.17` |
| **VLAN** | VLAN 20 — Servidores |
| **DNS Privado** | `ip-172-16-34-17.eu-north-1.compute.internal` |
| **VPC** | `vpc-08a2ef35bf1f60aa9` (Samba_AD) |
| **Subred** | `subnet-01df6764bf6a2fabd` (Samba_Subnet) |
| **ARN** | `arn:aws:ec2:eu-north-1:107365729991:instance/i-0e19ac2dfbbd9ec0d` |
| **IMDSv2** | Required ✅ |
| **Servicios** | NGINX (HTTP :80), OpenSSH SFTP (:22 chroot) |

---

### EC2 Active Directory

| Campo | Valor |
|---|---|
| **ID** | `i-092074046b4f94411` |
| **Tipo** | `t3.small` |
| **IP Pública** | `13.62.146.201` (Elastic IP) |
| **IP Privada** | `172.16.34.18` |
| **VLAN** | VLAN 20 — Servidores |
| **DNS Privado** | `ip-172-16-34-18.eu-north-1.compute.internal` |
| **VPC** | `vpc-08a2ef35bf1f60aa9` (Samba_AD) |
| **Subred** | `subnet-01df6764bf6a2fabd` (Samba_Subnet) |
| **ARN** | `arn:aws:ec2:eu-north-1:107365729991:instance/i-092074046b4f94411` |
| **IMDSv2** | Required ✅ |
| **Servicios** | Samba AD DC · Kerberos · DNS interno (`empresa.local`) |

---

### EC2 Logs

| Campo | Valor |
|---|---|
| **ID** | `i-0169f47bf1fcc7a3a` |
| **Tipo** | `c7i-flex.large` |
| **IP Pública** | `16.192.46.108` |
| **IP Privada** | `172.31.6.43` |
| **DNS Público** | `ec2-16-192-46-108.eu-north-1.compute.amazonaws.com` |
| **DNS Privado** | `ip-172-31-6-43.eu-north-1.compute.internal` |
| **VPC** | `vpc-0f50d9b1d874b8688` (innovatetech-g2) |
| **Subred** | `subnet-0240adae937576373` (Privada) |
| **AMI** | `ami-067bcf851477ebb78` |
| **ARN** | `arn:aws:ec2:eu-north-1:147826551864:instance/i-0169f47bf1fcc7a3a` |
| **IMDSv2** | Required ✅ |
| **SO** | Linux/UNIX |
| **Servicios** | Elasticsearch 8.17.4 · Kibana 8.17.4 · Filebeat |
| **Acceso Kibana** | `http://16.192.46.108:5601` |

> Tipo `c7i-flex.large` para soportar la carga de ingesta y procesado de logs. Filebeat también está desplegado en los servidores BD y Jitsi para centralizar sus logs aquí.

---

### EC2 Streaming Audio

| Campo | Valor |
|---|---|
| **ID** | `i-01d554ec3151e75a1` |
| **Tipo** | `t3.small` |
| **IP Pública** | `16.171.107.58` (Elastic IP) |
| **IP Privada** | `172.16.34.19` |
| **VLAN** | VLAN 30 — Streaming |
| **DNS Privado** | `ip-172-16-34-19.eu-north-1.compute.internal` |
| **VPC** | `vpc-08a2ef35bf1f60aa9` (Samba_AD) |
| **Subred** | `subnet-01df6764bf6a2fabd` (Samba_Subnet) |
| **ARN** | `arn:aws:ec2:eu-north-1:107365729991:instance/i-01d554ec3151e75a1` |
| **IMDSv2** | Required ✅ |
| **Servicios** | Icecast2 (:8000) · Liquidsoap |
| **Stream URL** | `http://16.171.107.58:8000/stream` |

---

### EC2 Streaming Vídeo

Dos instancias cubren el servicio de streaming de vídeo:

**Gestionado — Jellyfin**

| Campo | Valor |
|---|---|
| **ID** | `i-0b3447c1dd6810ac6` |
| **Tipo** | `t3.micro` |
| **IP Pública** | `16.171.115.99` ⚠️ Sin EIP |
| **IP Privada** | `172.16.34.26` |
| **VLAN** | VLAN 30 — Streaming |
| **DNS Privado** | `ip-172-16-34-26.eu-north-1.compute.internal` |
| **VPC** | `vpc-08a2ef35bf1f60aa9` (Samba_AD) |
| **Subred** | `subnet-01df6764bf6a2fabd` (Samba_Subnet) |
| **ARN** | `arn:aws:ec2:eu-north-1:107365729991:instance/i-0b3447c1dd6810ac6` |
| **IMDSv2** | Required ✅ |
| **Servicios** | Jellyfin (:8096) · HLS |

**Fran-Video-Server — NGINX RTMP**

| Campo | Valor |
|---|---|
| **ID** | `i-025eff8c23ec4f60a` |
| **Tipo** | `t3.micro` |
| **IP Pública** | `13.61.226.79` (Elastic IP) |
| **IP Privada** | `172.31.42.165` |
| **VLAN** | VLAN 30 — Streaming |
| **DNS Público** | `ec2-13-61-226-79.eu-north-1.compute.amazonaws.com` |
| **DNS Privado** | `ip-172-31-42-165.eu-north-1.compute.internal` |
| **VPC** | `vpc-0658e7daf52b29d35` |
| **Subred** | `subnet-0b7646a89d886ab00` |
| **ARN** | `arn:aws:ec2:eu-north-1:1080795438887:instance/i-025eff8c23ec4f60a` |
| **IMDSv2** | Required ✅ |
| **Servicios** | NGINX + módulo RTMP (:1935) · HLS (:80) · FFmpeg |
| **Stream URL** | `http://13.61.226.79/hls/stream.m3u8` |

---

### EC2 Videoconferencia

| Campo | Valor |
|---|---|
| **ID** | `i-0d05703ab0f3272cb` |
| **Tipo** | `t3.small` |
| **IP Pública** | `13.62.220.208` (Elastic IP) |
| **IP Privada** | `172.31.8.42` |
| **VLAN** | VLAN 30 — Streaming |
| **DNS Público** | `ec2-13-62-220-208.eu-north-1.compute.amazonaws.com` |
| **DNS Privado** | `ip-172-31-8-42.eu-north-1.compute.internal` |
| **VPC** | `vpc-0f50d9b1d874b8688` (innovatetech-g2) |
| **Subred** | `subnet-0240adae937576373` (Privada) |
| **ARN** | `arn:aws:ec2:eu-north-1:147826551864:instance/i-0d05703ab0f3272cb` |
| **IMDSv2** | Optional ⚠️ |
| **Servicios** | Jitsi Meet (Docker) · WebRTC · Let's Encrypt SSL |
| **Acceso** | `https://innovatetech-g2.ddns.net` |

---

### EC2 Base de Datos

| Campo | Valor |
|---|---|
| **ID** | `i-034ff7dd473f23011` |
| **Tipo** | `t3.small` |
| **IP Pública** | `13.53.152.159` (Elastic IP) |
| **IP Privada** | `172.31.24.225` |
| **VLAN** | VLAN 20 — Servidores |
| **DNS Público** | `ec2-13-53-152-159.eu-north-1.compute.amazonaws.com` |
| **DNS Privado** | `ip-172-31-24-225.eu-north-1.compute.internal` |
| **VPC** | `vpc-0f50d9b1d874b8688` (innovatetech-g2) |
| **Subred** | `subnet-09dc1cd56ab839493` (Pública) |
| **ARN** | `arn:aws:ec2:eu-north-1:147826551864:instance/i-034ff7dd473f23011` |
| **IMDSv2** | Optional ⚠️ |
| **Servicios** | MariaDB · Backups automáticos · Triggers de auditoría |
| **Base de datos** | `innovatetech_db` |

---

## Seguridad

### Security Groups

Cada instancia dispone de un Security Group propio con reglas de mínimo privilegio:

| Servicio | Puerto(s) | Protocolo | Origen |
|---|---|---|---|
| **Web-SFTP** | 22 | TCP | IP administración |
| **Web-SFTP** | 80, 443 | TCP | `0.0.0.0/0` |
| **Active Directory** | 22 | TCP | IP administración |
| **Active Directory** | 53 | TCP/UDP | VPC CIDR |
| **Active Directory** | 88 | TCP/UDP | VPC CIDR (Kerberos) |
| **Active Directory** | 389, 636 (LDAP/S) | TCP | VPC CIDR |
| **Active Directory** | 445 (SMB) | TCP | VPC CIDR |
| **Logs** | 22 | TCP | IP administración |
| **Logs** | 5601 (Kibana) | TCP | VPC CIDR |
| **Logs** | 9200 (Elasticsearch) | TCP | VPC CIDR |
| **Streaming Audio** | 22 | TCP | IP administración |
| **Streaming Audio** | 8000 (Icecast2) | TCP | `0.0.0.0/0` |
| **Streaming Vídeo** | 80 (HLS) | TCP | `0.0.0.0/0` |
| **Streaming Vídeo** | 1935 (RTMP) | TCP | `0.0.0.0/0` |
| **Streaming Vídeo** | 8096 (Jellyfin) | TCP | `0.0.0.0/0` |
| **Videoconferencia** | 80, 443 | TCP | `0.0.0.0/0` |
| **Videoconferencia** | 10000 (WebRTC) | UDP | `0.0.0.0/0` |
| **Base de Datos** | 22 | TCP | IP administración |
| **Base de Datos** | 3306 (MariaDB) | TCP | VPC CIDR |


### Subred pública y privada

| Subred | ID | VPC | Tipo | Servicios alojados |
|---|---|---|---|---|
| — | `subnet-09dc1cd56ab839493` | innovatetech-g2 | Pública | Base de Datos |
| — | `subnet-0240adae937576373` | innovatetech-g2 | Privada | Logs, Videoconferencia |
| Samba_Subnet | `subnet-01df6764bf6a2fabd` | Samba_AD | Pública | Web-SFTP, AD, Audio, Vídeo (Gestionado) |
| — | `subnet-0b7646a89d886ab00` | vpc-0658e7 | Pública | Fran-Video-Server |

Los servicios internos como Logs están en subred privada y con acceso restringido al CIDR de la VPC. El Active Directory expone LDAP y Kerberos únicamente dentro de la VPC, sin exposición a Internet.

### Dominio corporativo

El servidor Active Directory gestiona el dominio interno `empresa.local` con autenticación Kerberos. Los tickets tienen caducidad automática y las contraseñas nunca viajan en texto claro por la red.

---

## Ventajas

| Ventaja | Descripción |
|---|---|
| **Escalabilidad** | Las instancias EC2 permiten ajustar recursos vertical u horizontalmente según la demanda sin interrupciones de servicio. |
| **Alta disponibilidad** | Cada servicio opera en una instancia independiente; el fallo de una no afecta al resto. |
| **Modularidad** | Arquitectura de servicios aislados que facilita el mantenimiento, actualización y sustitución independiente de cada componente. |
| **Tolerancia a fallos** | La separación por VPCs, subredes y VLANs limita el radio de impacto ante cualquier incidencia. |

---
