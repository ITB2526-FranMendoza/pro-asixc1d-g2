#Active Directory (Samba AD)

> **Responsable:** jeremy [itb] ruiz hernandez  
> **Proyecto:** Proyecto Transversal 25/26 — InnovateTech  
> **Instancia AWS:** `i-092074046b4f94411` · t3.small · eu-north-1a  
> **IP pública:** `13.62.146.201`  
> **IP privada:** `172.16.34.18` (VLAN 20 — Servidores)

---

## Índice

1. [Descripción del servicio](#1-descripción-del-servicio)
2. [Tecnologías utilizadas](#2-tecnologías-utilizadas)
3. [Infraestructura AWS](#3-infraestructura-aws)
4. [Información del dominio](#4-información-del-dominio)
5. [Usuarios del dominio](#5-usuarios-del-dominio)
6. [Grupos del dominio](#6-grupos-del-dominio)
7. [Configuración DNS](#7-configuración-dns)
8. [Seguridad](#8-seguridad)
9. [Verificación del servicio](#9-verificación-del-servicio)
10. [Capturas de evidencia](#10-capturas-de-evidencia)

---

## 1. Descripción del servicio

Este servidor implementa un **controlador de dominio Active Directory** usando **Samba AD DC**, que proporciona autenticación centralizada, gestión de usuarios y grupos, y resolución de nombres DNS para toda la infraestructura de InnovateTech.

El servicio es **fundamental** para el resto de la infraestructura, ya que otros servicios (SFTP, videoconferencia, etc.) pueden autenticar usuarios contra este directorio.

| Servicio | Función | Puerto |
|----------|---------|--------|
| **Samba AD DC** | Controlador de dominio Active Directory | 389 (LDAP), 636 (LDAPS), 88 (Kerberos) |
| **DNS interno** | Resolución de nombres del dominio `empresa.local` | 53 (UDP/TCP) |
| **Kerberos** | Autenticación de tickets para usuarios del dominio | 88 (TCP/UDP) |

---

## 2. Tecnologías utilizadas

| Componente | Tecnología | Versión |
|------------|-----------|---------|
| Sistema operativo | Ubuntu Server 22.04 LTS | 22.04 |
| Directorio activo | Samba AD DC | 4.19.5-Ubuntu |
| Autenticación | Kerberos 5 | — |
| Directorio | LDAP (interno Samba) | — |
| Cloud | AWS EC2 | t3.small |

---

## 3. Infraestructura AWS

```
VPC: 10.0.0.0/16
└── Subnet Privada (Servicios): 10.0.2.0/24
    └── EC2-2 Samba AD
        ├── IP pública:  13.62.146.201
        └── IP privada:  172.16.34.18
```

**Security Group — puertos abiertos:**

| Puerto | Protocolo | Origen | Descripción |
|--------|-----------|--------|-------------|
| 22 | TCP | Admin | SSH administrativo |
| 53 | TCP/UDP | VPC | DNS interno |
| 88 | TCP/UDP | VPC | Kerberos |
| 389 | TCP | VPC | LDAP |
| 445 | TCP | VPC | SMB/CIFS |
| 636 | TCP | VPC | LDAPS |

---

## 4. Información del dominio

```
Forest          : empresa.local
Domain          : empresa.local
Netbios domain  : EMPRESA
DC name         : sambaad.empresa.local
DC netbios name : SAMBAAD
Server site     : Default-First-Site-Name
```

**Nivel funcional del dominio:**

```
Forest function level:         (Windows) 2008 R2
Domain function level:         (Windows) 2008 R2
Lowest function level of a DC: (Windows) 2008 R2
```

---

## 5. Usuarios del dominio

Los siguientes usuarios han sido creados en el dominio `empresa.local`:

| Usuario | Grupo asignado | Descripción |
|---------|---------------|-------------|
| `Administrator` | Domain Admins | Administrador del dominio (sistema) |
| `admin_db` | Gestores_BBDD | Administrador de base de datos |
| `ruben` | Admins_TI | Técnico de sistemas |
| `dev_web` | Desarrolladores_Web | Desarrollador web |
| `creador_media` | Gestores_Media | Gestor de contenido multimedia |
| `gerard` | Admins_TI | Técnico de sistemas |
| `jeremy` | Admins_TI | Técnico de sistemas |
| `auditor_sec` | Auditores_Seguridad | Auditor de seguridad |
| `empleado_1` | Usuarios_Empresa | Usuario estándar |
| `jefe_it` | Admins_TI | Jefe del departamento TI |
| `admin_comms` | Admins_Comms | Administrador de comunicaciones |
| `fran` | Admins_TI | Técnico de sistemas |

### Crear un nuevo usuario

```bash
sudo samba-tool user create nombre_usuario --given-name="Nombre" \
    --surname="Apellido" --mail-address="usuario@empresa.local"

# Añadir al grupo correspondiente
sudo samba-tool group addmembers "Nombre_Grupo" nombre_usuario
```

### Cambiar contraseña de usuario

```bash
sudo samba-tool user setpassword nombre_usuario
```

### Deshabilitar/habilitar usuario

```bash
sudo samba-tool user disable nombre_usuario
sudo samba-tool user enable nombre_usuario
```

---

## 6. Grupos del dominio

Los grupos personalizados creados para la organización InnovateTech son:

| Grupo | Descripción |
|-------|-------------|
| `Admins_TI` | Administradores del departamento de TI |
| `Desarrolladores_Web` | Desarrolladores web |
| `Gestores_Media` | Gestores de contenido multimedia |
| `Auditores_Seguridad` | Auditores de seguridad |
| `Admins_Comms` | Administradores de comunicaciones |
| `Gestores_BBDD` | Administradores de base de datos |
| `Usuarios_Empresa` | Usuarios estándar corporativos |
| `qa` | Equipo de quality assurance |

> Además de los grupos personalizados, existen los grupos estándar de Active Directory (`Domain Admins`, `Domain Users`, `Administrators`, etc.)

### Ver miembros de un grupo

```bash
sudo samba-tool group listmembers "Admins_TI"
```

### Crear un nuevo grupo

```bash
sudo samba-tool group add Nombre_Grupo
sudo samba-tool group addmembers "Nombre_Grupo" usuario1,usuario2
```

---

## 7. Configuración DNS

El servidor AD actúa también como servidor DNS interno para el dominio `empresa.local`.

### Verificación DNS

```bash
# Registro SRV LDAP (confirma que el DC es accesible)
host -t SRV _ldap._tcp.empresa.local
# _ldap._tcp.empresa.local has SRV record 0 100 389 sambaad.empresa.local.

# Resolución del dominio
host -t A empresa.local
# empresa.local has address 172.16.34.18
```

### Configuración en clientes

Para unir una máquina al dominio `empresa.local`, el DNS del cliente debe apuntar a `172.16.34.18`.

```bash
# En /etc/resolv.conf del cliente
nameserver 172.16.34.18
search empresa.local
```

---

## 8. Seguridad

### Medidas implementadas

- **Autenticación Kerberos:** Tickets con caducidad automática, sin contraseñas en tránsito.
- **LDAP sobre red privada:** El puerto 389 solo está accesible desde dentro de la VPC.
- **Acceso administrativo por clave pública:** Sin autenticación por contraseña en SSH.
- **Grupos con mínimos privilegios:** Cada usuario tiene solo los permisos del grupo que necesita.
- **Security Groups AWS:** LDAP y Kerberos no expuestos a Internet.

### Test de autenticación Kerberos

```bash
# Obtener ticket para un usuario
kinit dev_web@EMPRESA.LOCAL

# Verificar ticket obtenido
klist
# Ticket cache: FILE:/tmp/krb5cc_1000
# Default principal: dev_web@EMPRESA.LOCAL
# Valid starting: 05/27/26 17:19:07
# Expires:        05/28/26 03:19:07
```

---

## 9. Verificación del servicio

```bash
# Estado del servicio Samba AD
sudo systemctl status samba-ad-dc

# Información del dominio
sudo samba-tool domain info 127.0.0.1

# Listar usuarios
sudo samba-tool user list

# Listar grupos
sudo samba-tool group list

# Ver miembros de un grupo
sudo samba-tool group listmembers "Admins_TI"

# Información detallada de un usuario
sudo samba-tool user show dev_web

# Nivel funcional del dominio
sudo samba-tool domain level show

# Verificar DNS del dominio
host -t SRV _ldap._tcp.empresa.local
host -t A empresa.local
```

---

## 10. Capturas de evidencia

| # | Comando | Qué muestra |
|---|---------|-------------|
| 1 | `systemctl status samba-ad-dc` | Servicio activo (running) |
| 2 | `samba-tool domain info 127.0.0.1` | Dominio `empresa.local` configurado |
| 3 | `samba-tool user list` | Lista de usuarios del dominio |
| 4 | `samba-tool group list` | Grupos personalizados creados |
| 5 | `samba-tool group listmembers "Admins_TI"` | Miembros del grupo |
| 6 | `samba-tool domain level show` | Nivel funcional Windows 2008 R2 |
| 7 | `host -t SRV _ldap._tcp.empresa.local` | DNS del dominio funcionando |
| 8 | `kinit` + `klist` | Autenticación Kerberos operativa |

---

*Proyecto Transversal 25/26 — Institut Tecnològic Barcelona (ITB)*  
*ASIX1D — jeremy [itb] ruiz hernandez*
