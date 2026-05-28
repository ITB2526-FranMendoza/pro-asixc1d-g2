# Infraestructura AWS — InnovateTech

> Documentación técnica de la infraestructura cloud desplegada en Amazon Web Services para el proyecto InnovateTech.  
> Región: **eu-north-1 (Estocolmo)**

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
│  │  │  │  t3.small              │  │    │                           │
│  │  │  │  13.63.204.200 (EIP)   │  │    │                           │
│  │  │  └────────────────────────┘  │    │                           │
│  │  │  ┌────────────────────────┐  │    │                           │
│  │  │  │  Samba_AD              │  │    │                           │
│  │  │  │  Active Directory      │  │    │                           │
│  │  │  │  t3.small              │  │    │                           │
│  │  │  │  13.62.146.201 (EIP)   │  │    │                           │
│  │  │  └────────────────────────┘  │    │                           │
│  │  │  ┌────────────────────────┐  │    │                           │
│  │  │  │  Server_Audio          │  │    │                           │
│  │  │  │  Streaming Audio       │  │    │                           │
│  │  │  │  t3.small              │  │    │                           │
│  │  │  │  16.171.107.58 (EIP)   │  │    │                           │
│  │  │  └────────────────────────┘  │    │                           │
│  │  │  ┌────────────────────────┐  │    │                           │
│  │  │  │  Gestionado            │  │    │                           │
│  │  │  │  Streaming Vídeo       │  │    │                           │
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
│  │  │  Streaming Vídeo adicional   │    │                           │
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
         │  VLAN 10 — Servidores          │
         │  VLAN 20 — Usuarios            │
         │  VLAN 30 — Servicios IP        │
         │  VLAN Gestión — Administración │
         └────────────────────────────────┘
```

### VPCs y Redes

| VPC | ID | Cuenta AWS | Subredes |
|---|---|---|---|
| `innovatetech-g2` | `vpc-0f50d9b1d874b8688` | `147826551864` | Pública + Privada |
| `Samba_AD` | `vpc-08a2ef35bf1f60aa9` | `107365729991` | Samba_Subnet (Pública) |
| _(sin nombre)_ | `vpc-0658e7daf52b29d35` | `1080795438887` | Pública |

### Segmentación VLAN (CPD)

| VLAN | ID | Propósito |
|---|---|---|
| Servidores | VLAN 10 | Tráfico entre servidores físicos y virtuales |
| Usuarios | VLAN 20 | Tráfico de equipos de usuario final |
| Servicios IP | VLAN 30 | Servicios de red (VoIP, streaming, etc.) |
| Gestión | VLAN Mgmt | Administración y monitorización de infraestructura |

---

## Servicios Desplegados

Todos los servicios corren sobre instancias EC2 en AWS (`eu-north-1`). Cada instancia está aislada y dedicada a un único rol.

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
| **DNS Privado** | `ip-172-16-34-17.eu-north-1.compute.internal` |
| **VPC** | `vpc-08a2ef35bf1f60aa9` (Samba_AD) |
| **Subred** | `subnet-01df6764bf6a2fabd` (Samba_Subnet) |
| **ARN** | `arn:aws:ec2:eu-north-1:107365729991:instance/i-0e19ac2dfbbd9ec0d` |
| **IMDSv2** | Required ✅ |

---

### EC2 Active Directory

| Campo | Valor |
|---|---|
| **ID** | `i-092074046b4f94411` |
| **Tipo** | `t3.small` |
| **IP Pública** | `13.62.146.201` (Elastic IP) |
| **IP Privada** | `172.16.34.18` |
| **DNS Privado** | `ip-172-16-34-18.eu-north-1.compute.internal` |
| **VPC** | `vpc-08a2ef35bf1f60aa9` (Samba_AD) |
| **Subred** | `subnet-01df6764bf6a2fabd` (Samba_Subnet) |
| **ARN** | `arn:aws:ec2:eu-north-1:107365729991:instance/i-092074046b4f94411` |
| **IMDSv2** | Required ✅ |

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

> Tipo `c7i-flex.large` para soportar la carga de ingesta y procesado de logs.

---

### EC2 Streaming Audio

| Campo | Valor |
|---|---|
| **ID** | `i-01d554ec3151e75a1` |
| **Tipo** | `t3.small` |
| **IP Pública** | `16.171.107.58` (Elastic IP) |
| **IP Privada** | `172.16.34.19` |
| **DNS Privado** | `ip-172-16-34-19.eu-north-1.compute.internal` |
| **VPC** | `vpc-08a2ef35bf1f60aa9` (Samba_AD) |
| **Subred** | `subnet-01df6764bf6a2fabd` (Samba_Subnet) |
| **ARN** | `arn:aws:ec2:eu-north-1:107365729991:instance/i-01d554ec3151e75a1` |
| **IMDSv2** | Required ✅ |

---

### EC2 Streaming Vídeo

Dos instancias cubren el servicio de streaming de vídeo:

**Gestionado**

| Campo | Valor |
|---|---|
| **ID** | `i-0b3447c1dd6810ac6` |
| **Tipo** | `t3.micro` |
| **IP Pública** | `16.171.115.99` ⚠️ Sin EIP |
| **IP Privada** | `172.16.34.26` |
| **DNS Privado** | `ip-172-16-34-26.eu-north-1.compute.internal` |
| **VPC** | `vpc-08a2ef35bf1f60aa9` (Samba_AD) |
| **Subred** | `subnet-01df6764bf6a2fabd` (Samba_Subnet) |
| **ARN** | `arn:aws:ec2:eu-north-1:107365729991:instance/i-0b3447c1dd6810ac6` |
| **IMDSv2** | Required ✅ |

**Fran-Video-Server**

| Campo | Valor |
|---|---|
| **ID** | `i-025eff8c23ec4f60a` |
| **Tipo** | `t3.micro` |
| **IP Pública** | `13.61.226.79` (Elastic IP) |
| **IP Privada** | `172.31.42.165` |
| **DNS Público** | `ec2-13-61-226-79.eu-north-1.compute.amazonaws.com` |
| **DNS Privado** | `ip-172-31-42-165.eu-north-1.compute.internal` |
| **VPC** | `vpc-0658e7daf52b29d35` |
| **Subred** | `subnet-0b7646a89d886ab00` |
| **ARN** | `arn:aws:ec2:eu-north-1:1080795438887:instance/i-025eff8c23ec4f60a` |
| **IMDSv2** | Required ✅ |

---

### EC2 Videoconferencia

| Campo | Valor |
|---|---|
| **ID** | `i-0d05703ab0f3272cb` |
| **Tipo** | `t3.small` |
| **IP Pública** | `13.62.220.208` (Elastic IP) |
| **IP Privada** | `172.31.8.42` |
| **DNS Público** | `ec2-13-62-220-208.eu-north-1.compute.amazonaws.com` |
| **DNS Privado** | `ip-172-31-8-42.eu-north-1.compute.internal` |
| **VPC** | `vpc-0f50d9b1d874b8688` (innovatetech-g2) |
| **Subred** | `subnet-0240adae937576373` (Privada) |
| **ARN** | `arn:aws:ec2:eu-north-1:147826551864:instance/i-0d05703ab0f3272cb` |
| **IMDSv2** | Optional ⚠️ |

---

### EC2 Base de Datos

| Campo | Valor |
|---|---|
| **ID** | `i-034ff7dd473f23011` |
| **Tipo** | `t3.small` |
| **IP Pública** | `13.53.152.159` (Elastic IP) |
| **IP Privada** | `172.31.24.225` |
| **DNS Público** | `ec2-13-53-152-159.eu-north-1.compute.amazonaws.com` |
| **DNS Privado** | `ip-172-31-24-225.eu-north-1.compute.internal` |
| **VPC** | `vpc-0f50d9b1d874b8688` (innovatetech-g2) |
| **Subred** | `subnet-09dc1cd56ab839493` (Pública) |
| **ARN** | `arn:aws:ec2:eu-north-1:147826551864:instance/i-034ff7dd473f23011` |
| **IMDSv2** | Optional ⚠️ |

---

## Seguridad

### Security Groups

Cada instancia dispone de un Security Group propio con reglas de mínimo privilegio:

| Servicio | Puerto(s) | Protocolo | Origen |
|---|---|---|---|
| **Web-SFTP** | 22 | TCP | IP administración |
| **Web-SFTP** | 80, 443 | TCP | `0.0.0.0/0` |
| **Active Directory** | 389, 636 (LDAP/S) | TCP | VPC CIDR |
| **Active Directory** | 445 (SMB) | TCP | VPC CIDR |
| **Active Directory** | 22 | TCP | IP administración |
| **Logs** | 5044, 9200 | TCP | VPC CIDR |
| **Streaming Audio** | 8000, 8080 | TCP | `0.0.0.0/0` |
| **Streaming Vídeo** | 1935 (RTMP) | TCP | `0.0.0.0/0` |
| **Videoconferencia** | 80, 443 | TCP | `0.0.0.0/0` |
| **Videoconferencia** | 10000 (WebRTC) | UDP | `0.0.0.0/0` |
| **Base de Datos** | 3306 / 5432 | TCP | VPC CIDR |

### Acceso SSH mediante clave pública

El acceso administrativo a todas las instancias se realiza exclusivamente mediante par de claves RSA, sin autenticación por contraseña:

```bash
# Sintaxis general de conexión
ssh -i <clave.pem> ec2-user@<IP_PUBLICA>

# Ejemplos
ssh -i innovatetech.pem ec2-user@13.63.204.200   # Web-SFTP
ssh -i innovatetech.pem ec2-user@13.62.146.201   # Active Directory
ssh -i innovatetech.pem ec2-user@13.53.152.159   # Base de Datos
ssh -i innovatetech.pem ec2-user@13.62.220.208   # Videoconferencia
ssh -i innovatetech.pem ec2-user@16.171.107.58   # Streaming Audio
```

### Subred pública y privada

| Subred | ID | VPC | Servicios alojados |
|---|---|---|---|
| Pública | `subnet-09dc1cd56ab839493` | innovatetech-g2 | Base de Datos |
| Privada | `subnet-0240adae937576373` | innovatetech-g2 | Logs, Videoconferencia |
| Pública | `subnet-01df6764bf6a2fabd` | Samba_AD | Web-SFTP, AD, Audio, Vídeo |
| Pública | `subnet-0b7646a89d886ab00` | vpc-0658e7 | Fran-Video-Server |

Los servicios internos como la Base de Datos y los Logs se mantienen en subredes privadas o con acceso restringido al CIDR de la VPC, reduciendo la superficie de exposición.

---

## Ventajas

| Ventaja | Descripción |
|---|---|
| **Escalabilidad** | Las instancias EC2 permiten ajustar recursos vertical u horizontalmente según la demanda sin interrupciones de servicio. |
| **Alta disponibilidad** | Cada servicio opera en una instancia independiente; el fallo de una no afecta al resto. |
| **Modularidad** | Arquitectura de servicios aislados que facilita el mantenimiento, actualización y sustitución independiente de cada componente. |
| **Tolerancia a fallos** | La separación por VPCs, subredes y VLANs limita el radio de impacto ante cualquier incidencia. |

---
