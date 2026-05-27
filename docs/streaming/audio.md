[Servidor_Audio.md](https://github.com/user-attachments/files/28325792/EC2-4_Servidor_Audio.md)
# Servidor de Streaming de Audio

> **Responsable:** jeremy [itb] ruiz hernandez  
> **Proyecto:** Proyecto Transversal 25/26 — InnovateTech  
> **Instancia AWS:** `i-01d554ec3151e75a1` · t3.small · eu-north-1a  
> **IP pública:** `16.171.107.58`  
> **IP privada:** `172.16.34.19` (VLAN 30 — Streaming)

---

## Índice

1. [Descripción del servicio](#1-descripción-del-servicio)
2. [Tecnologías utilizadas](#2-tecnologías-utilizadas)
3. [Infraestructura AWS](#3-infraestructura-aws)
4. [Arquitectura del sistema de audio](#4-arquitectura-del-sistema-de-audio)
5. [Configuración Icecast2](#5-configuración-icecast2)
6. [Configuración Liquidsoap](#6-configuración-liquidsoap)
7. [Seguridad](#7-seguridad)
8. [Verificación del servicio](#8-verificación-del-servicio)
9. [Capturas de evidencia](#9-capturas-de-evidencia)

---

## 1. Descripción del servicio

Este servidor proporciona el servicio de **streaming de audio corporativo** de InnovateTech, denominado **InnovateTech Radio**. Combina dos herramientas especializadas:

- **Liquidsoap**: Motor de emisión que gestiona la playlist y envía el stream al servidor.
- **Icecast2**: Servidor de distribución que recibe el stream y lo sirve a los oyentes.

| Servicio | Función | Puerto |
|----------|---------|--------|
| **Icecast2** | Servidor de distribución de audio streaming | 8000 |
| **Liquidsoap** | Motor de playlist y emisión de audio | — (interno) |

El mountpoint de emisión es `/stream`, accesible en `http://16.171.107.58:8000/stream`.

---

## 2. Tecnologías utilizadas

| Componente | Tecnología | Descripción |
|------------|-----------|-------------|
| Sistema operativo | Ubuntu Server 22.04 LTS | — |
| Servidor de streaming | Icecast2 | Distribución de audio a oyentes |
| Motor de emisión | Liquidsoap | Gestión de playlist y transcodificación |
| Formato de audio | MP3 | Bitrate: 128 kbps |
| Cloud | AWS EC2 | t3.small |

---

## 3. Infraestructura AWS

```
VPC: 10.0.0.0/16
└── Subnet Privada (Servicios): 10.0.2.0/24
    └── EC2-4 Streaming Audio
        ├── IP pública:  16.171.107.58
        └── IP privada:  172.16.34.19
```

**Security Group — puertos abiertos:**

| Puerto | Protocolo | Origen | Descripción |
|--------|-----------|--------|-------------|
| 22 | TCP | Admin | SSH administrativo |
| 8000 | TCP | 0.0.0.0/0 | Icecast2 (streaming público) |

---

## 4. Arquitectura del sistema de audio

```
┌─────────────────────────────────────────────┐
│              EC2-4 Audio Server              │
│                                             │
│  /home/ubuntu/radio/   ←── Archivos MP3     │
│           ↓                                 │
│      Liquidsoap                             │
│   (radio.liq — playlist random)             │
│           ↓                                 │
│      output.icecast()                       │
│   host=localhost · port=8000                │
│   mount=/stream · bitrate=128kbps           │
│           ↓                                 │
│        Icecast2                             │
│   puerto 8000 · mount /stream               │
│           ↓                                 │
└─────────────────────────────────────────────┘
           ↓
   Oyentes externos
   http://16.171.107.58:8000/stream
```

El flujo de datos es: **archivos de audio → Liquidsoap (codificación) → Icecast2 (distribución) → oyentes**.

---

## 5. Configuración Icecast2

### Parámetros principales (`/etc/icecast2/icecast.xml`)

| Parámetro | Valor | Descripción |
|-----------|-------|-------------|
| `hostname` | `13.60.10.65` | Hostname del servidor |
| `port` | `8000` | Puerto de escucha |
| `clients` | `100` | Máximo de oyentes simultáneos |
| `sources` | `2` | Máximo de fuentes de audio |
| `burst-size` | `65535` (64KB) | Buffer inicial para reducir latencia de conexión |
| `client-timeout` | `30` | Tiempo de espera del cliente (segundos) |
| `source-timeout` | `10` | Tiempo de espera de la fuente (segundos) |

### Autenticación

| Rol | Usuario | Contraseña |
|-----|---------|-----------|
| Fuente (Liquidsoap) | `source` | `ITB2026` |
| Relay | `relay` | `ITB2026` |
| Administración web | `admin` | `ITB2026` |

> **Nota:** Las contraseñas están configuradas para el entorno de laboratorio del proyecto.

### Logging

```xml
<logging>
    <accesslog>access.log</accesslog>
    <errorlog>error.log</errorlog>
    <loglevel>3</loglevel>   <!-- 3 = Info -->
    <logsize>10000</logsize>
</logging>
```

Los logs se almacenan en `/var/log/icecast2/`.

### Panel de administración web

Accesible en: `http://16.171.107.58:8000/admin/`  
Usuario: `admin` · Contraseña: `ITB2026`

---

## 6. Configuración Liquidsoap

### Script de emisión (`/home/ubuntu/radio.liq`)

```liquidsoap
set("log.stdout", true)

radio = mksafe(
  playlist(mode="random", "/home/ubuntu/radio")
)

output.icecast(
  %mp3(bitrate=128),
  host="localhost",
  port=8000,
  password="ITB2026",
  mount="/stream",
  name="InnovateTech Radio",
  description="Radio corporativa",
  genre="Tech",
  radio
)
```

### Explicación del script

| Directiva | Valor | Descripción |
|-----------|-------|-------------|
| `playlist(mode="random")` | `/home/ubuntu/radio` | Reproduce los archivos de audio en orden aleatorio |
| `mksafe()` | — | Evita que el stream se corte si no hay archivos disponibles |
| `%mp3(bitrate=128)` | 128 kbps | Codifica el audio en MP3 a 128 kbps |
| `host="localhost"` | localhost | Liquidsoap envía el stream a Icecast en la misma máquina |
| `mount="/stream"` | `/stream` | Mountpoint de emisión |
| `name` | InnovateTech Radio | Nombre de la emisora visible en el panel de Icecast |
| `genre` | Tech | Género de la emisora |

### Directorio de audio

Los archivos de audio (MP3, OGG, etc.) se almacenan en `/home/ubuntu/radio/`. Liquidsoap los lee y los emite en orden aleatorio de forma continua.

### Arranque de Liquidsoap

```bash
# Iniciar manualmente en background
nohup liquidsoap /home/ubuntu/radio.liq &

# Para que arranque automáticamente con el sistema,
# se puede crear un servicio systemd:
sudo nano /etc/systemd/system/liquidsoap.service
```

Ejemplo de unidad systemd para Liquidsoap:

```ini
[Unit]
Description=Liquidsoap Radio InnovateTech
After=network.target icecast2.service

[Service]
ExecStart=/usr/bin/liquidsoap /home/ubuntu/radio.liq
Restart=always
User=ubuntu

[Install]
WantedBy=multi-user.target
```

```bash
sudo systemctl daemon-reload
sudo systemctl enable liquidsoap
sudo systemctl start liquidsoap
```

---

## 7. Seguridad

### Medidas implementadas

- **Puerto 8000 público:** Solo el puerto de streaming está expuesto a Internet; el panel de administración está protegido por contraseña.
- **Acceso administrativo SSH:** Solo por clave pública, sin contraseña.
- **Liquidsoap en local:** La fuente de audio se conecta a Icecast en `localhost`, no expuesta externamente.
- **CORS habilitado:** La cabecera `Access-Control-Allow-Origin: *` permite embeber el stream en páginas web externas.
- **Security Groups AWS:** Solo puertos 22 y 8000 accesibles desde exterior.

---

## 8. Verificación del servicio

### Comprobaciones Icecast2

```bash
# Estado del servicio
systemctl status icecast2

# Verificar que escucha en el puerto 8000
ss -tlnp | grep 8000

# Ver logs en tiempo real
tail -f /var/log/icecast2/access.log
tail -f /var/log/icecast2/error.log

# Comprobar estado del stream desde terminal
curl -I http://localhost:8000/stream
```

### Comprobaciones Liquidsoap

```bash
# Verificar que el proceso está corriendo
ps aux | grep liquidsoap

# Ver archivos de audio disponibles
ls -lh /home/ubuntu/radio/

# Ver el script activo
cat /home/ubuntu/radio.liq
```

### Acceso al stream desde un cliente

El stream se puede escuchar directamente desde un navegador o reproductor de audio:

```
URL del stream: http://16.171.107.58:8000/stream
```

Compatible con: VLC, navegadores web, MPV, cualquier reproductor que soporte HTTP streaming.

---

## 9. Capturas de evidencia

| # | Comando / Acción | Qué muestra |
|---|-----------------|-------------|
| 1 | `systemctl status icecast2` | Servicio activo (running) |
| 2 | `ps aux \| grep liquidsoap` | Proceso Liquidsoap en ejecución |
| 3 | `cat /home/ubuntu/radio.liq` | Script de emisión configurado |
| 4 | `ls /home/ubuntu/radio/` | Archivos de audio disponibles |
| 5 | Panel admin Icecast `http://IP:8000/admin/` | Mountpoint `/stream` activo con oyentes |
| 6 | Stream reproduciendo en VLC o navegador | Audio funcionando en `http://IP:8000/stream` |
| 7 | `tail /var/log/icecast2/access.log` | Conexiones de oyentes registradas |

---

*Proyecto Transversal 25/26 — Institut Tecnològic Barcelona (ITB)*  
*ASIX1D — jeremy [itb] ruiz hernandez*
