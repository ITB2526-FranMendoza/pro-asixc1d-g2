# Servidor Web + SFTP

> **Responsable:** jeremy [itb] ruiz hernandez  
> **Proyecto:** Proyecto Transversal 25/26 — InnovateTech  
> **Instancia AWS:** `i-0e19ac2dfbbd9ec0d` · t3.small · eu-north-1a  
> **IP pública:** `13.63.204.200`  
> **IP privada:** `10.0.1.10` (Subnet Pública AWS)

---

## Índice

1. [Descripción del servicio](#1-descripción-del-servicio)
2. [Tecnologías utilizadas](#2-tecnologías-utilizadas)
3. [Infraestructura AWS](#3-infraestructura-aws)
4. [Configuración NGINX](#4-configuración-nginx)
5. [Configuración SFTP](#5-configuración-sftp)
6. [Seguridad](#6-seguridad)
7. [Verificación del servicio](#7-verificación-del-servicio)
8. [Capturas de evidencia](#8-capturas-de-evidencia)

---

## 1. Descripción del servicio

Este servidor aloja dos servicios independientes sobre la misma instancia EC2:

| Servicio | Función | Puerto |
|----------|---------|--------|
| **NGINX** | Servidor web corporativo de InnovateTech | 80 (HTTP) |
| **SFTP** | Transferencia segura de ficheros con jaula chroot por usuario | 22 (SSH/SFTP) |

El servidor forma parte de la **subred pública** de la infraestructura AWS (`10.0.1.0/24`), siendo el punto de entrada principal para el acceso web externo y la transferencia de archivos autenticada.

---

## 2. Tecnologías utilizadas

| Componente | Tecnología | Versión |
|------------|-----------|---------|
| Sistema operativo | Ubuntu Server 22.04 LTS | 22.04 |
| Servidor web | NGINX | 1.24.0 |
| Transferencia de ficheros | OpenSSH (subsistema SFTP interno) | — |
| Cloud | AWS EC2 | t3.small |

---

## 3. Infraestructura AWS

```
VPC: 10.0.0.0/16
└── Subnet Pública: 10.0.1.0/24
    └── EC2-1 Web+SFTP
        ├── IP pública:  13.63.204.200
        └── IP privada:  10.0.1.10
```

**Security Group — puertos abiertos:**

| Puerto | Protocolo | Origen | Descripción |
|--------|-----------|--------|-------------|
| 22 | TCP | 0.0.0.0/0 | SSH / SFTP |
| 80 | TCP | 0.0.0.0/0 | HTTP (NGINX) |
| 443 | TCP | 0.0.0.0/0 | HTTPS (reservado) |

---

## 4. Configuración NGINX

NGINX actúa como servidor web corporativo sirviendo contenido estático desde `/var/www/html`.

### Configuración activa (`/etc/nginx/sites-enabled/default`)

```nginx
server {
    listen 80 default_server;
    listen [::]:80 default_server;

    root /var/www/html;
    index index.html;

    server_name _;

    location / {
        try_files $uri $uri/ =404;
    }
}
```

### Estado del servicio

```
● nginx.service - A high performance web server and a reverse proxy server
     Loaded: loaded (/usr/lib/systemd/system/nginx.service; enabled; preset: enabled)
     Active: active (running) since Tue 2026-05-26 10:21:58 UTC
    Main PID: 25755 (nginx)
      Tasks: 3 (limit: 2209)
     Memory: 3.1M (peak: 3.6M)
```

NGINX está configurado para **arrancar automáticamente** con el sistema (`enabled`).

### Verificación de respuesta HTTP

```
HTTP/1.1 200 OK
Server: nginx/1.24.0 (Ubuntu)
Date: Wed, 27 May 2026 13:58:12 GMT
Content-Type: text/html
```

---

## 5. Configuración SFTP

El servicio SFTP se implementa usando el **subsistema interno de OpenSSH** (`internal-sftp`), sin necesidad de software adicional. Cada usuario queda enjaulado en su propio directorio mediante **chroot**.

### Configuración relevante (`/etc/ssh/sshd_config`)

```sshd
# Subsistema SFTP
Subsystem    sftp    /usr/lib/openssh/sftp-server

# Jaula chroot por grupo
Match Group sftpusers
    ChrootDirectory /sftp/%u
    ForceCommand internal-sftp
    PasswordAuthentication yes
    X11Forwarding no
    AllowTcpForwarding no
```

### Explicación de la configuración

| Directiva | Valor | Función |
|-----------|-------|---------|
| `ChrootDirectory /sftp/%u` | Directorio del usuario | Cada usuario solo ve su propio directorio |
| `ForceCommand internal-sftp` | SFTP obligatorio | Impide acceso SSH a la shell |
| `X11Forwarding no` | Desactivado | Seguridad: sin reenvío gráfico |
| `AllowTcpForwarding no` | Desactivado | Seguridad: sin túneles TCP |
| `Match Group sftpusers` | Grupo específico | Solo los usuarios del grupo tienen SFTP |

### Estructura de directorios chroot

```
/sftp/
└── <usuario>/          ← ChrootDirectory del usuario
    └── files/          ← Subdirectorio con permisos de escritura
```

> **Nota técnica:** El directorio raíz del chroot debe ser propiedad de `root:root` con permisos `755` para que OpenSSH lo acepte como válido. El usuario solo puede escribir en subdirectorios dentro de su jaula.

### Gestión de usuarios SFTP

```bash
# Crear usuario sin acceso shell
sudo useradd -s /usr/sbin/nologin -G sftpusers nuevo_usuario
sudo passwd nuevo_usuario

# Crear y configurar directorio chroot
sudo mkdir -p /sftp/nuevo_usuario
sudo chown root:root /sftp/nuevo_usuario
sudo chmod 755 /sftp/nuevo_usuario

# Crear subdirectorio donde el usuario puede escribir
sudo mkdir /sftp/nuevo_usuario/files
sudo chown nuevo_usuario:nuevo_usuario /sftp/nuevo_usuario/files

# Reiniciar SSH para aplicar cambios
sudo systemctl restart ssh
```

---

## 6. Seguridad

### Medidas implementadas

- **Chroot obligatorio:** Los usuarios SFTP no pueden navegar fuera de su directorio asignado.
- **Shell deshabilitada:** `ForceCommand internal-sftp` impide el acceso a la shell del sistema.
- **Sin reenvío de puertos:** `AllowTcpForwarding no` evita el uso del servidor como proxy.
- **Acceso administrativo solo por clave pública** (usuario `ubuntu`).
- **Security Groups AWS:** Solo puertos estrictamente necesarios habilitados.

### Acceso administrativo al servidor

```bash
# Solo con clave pública — sin contraseña
ssh -i ~/.ssh/clave_privada.pem ubuntu@13.63.204.200
```

---

## 7. Verificación del servicio

### Comprobaciones NGINX

```bash
# Estado del servicio
systemctl status nginx

# Verificar sintaxis de configuración
sudo nginx -t
# nginx: configuration file /etc/nginx/nginx.conf syntax is ok
# nginx: configuration file /etc/nginx/nginx.conf test is successful

# Comprobar respuesta HTTP
curl -I http://localhost
curl -I http://13.63.204.200

# Logs en tiempo real
tail -f /var/log/nginx/access.log
tail -f /var/log/nginx/error.log
```

### Comprobaciones SFTP

```bash
# Estado del servicio SSH/SFTP
systemctl status ssh

# Verificar configuración
grep -i sftp /etc/ssh/sshd_config
grep -i chroot /etc/ssh/sshd_config

# Probar conexión SFTP
sftp usuario@13.63.204.200

# Dentro de la sesión SFTP
sftp> pwd
sftp> ls
sftp> put archivo_prueba.txt
sftp> ls

# Logs de autenticación
tail -f /var/log/auth.log
```

---

## 8. Capturas de evidencia

| # | Comando | Qué muestra |
|---|---------|-------------|
| 1 | `systemctl status nginx` | Servicio activo (running) |
| 2 | `curl -I http://13.63.204.200` | Respuesta `200 OK` de NGINX |
| 3 | `systemctl status ssh` | Servicio SSH activo (running) |
| 4 | `grep -i sftp /etc/ssh/sshd_config` | Subsistema SFTP configurado |
| 5 | `grep -i chroot /etc/ssh/sshd_config` | ChrootDirectory activo |
| 6 | Sesión SFTP activa | `ls` y `put` funcionando dentro de la jaula |
| 7 | `tail /var/log/auth.log` | Login SFTP registrado correctamente |

---

*Proyecto Transversal 25/26 — Institut Tecnològic Barcelona (ITB)*  
*ASIX1D — jeremy [itb] ruiz hernandez*
