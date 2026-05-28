# Servicio de Vídeo — Jellyfin (RA8)

---

## Índice

1. [Descripción del servicio](#1-descripción-del-servicio)
2. [Infraestructura](#2-infraestructura)
3. [Instalación de Jellyfin](#3-instalación-de-jellyfin)
4. [Configuración de la biblioteca](#4-configuración-de-la-biblioteca)
5. [Publicación del vídeo](#5-publicación-del-vídeo)
6. [Verificación desde el navegador](#6-verificación-desde-el-navegador)
7. [Instalación y (intento de) conexión de MariaDB](#7-instalación-y-intento-de-conexión-de-mariadb)
8. [Formatos, códecs y protocolos](#8-formatos-códecs-y-protocolos)
9. [Validación final](#9-validación-final)

---

## 1. Descripción del servicio

Jellyfin es un servidor multimedia de código abierto que permite almacenar, organizar y reproducir contenido de vídeo, audio e imágenes. En este proyecto se ha desplegado en una instancia AWS para ofrecer un servicio de streaming de vídeo interno para un centro educativo.

**Características principales:**
- Servidor: Jellyfin vX.X.X (ver captura de versión)
- Sistema operativo: Ubuntu Server XX.04 LTS
- IP del servidor: `13.61.226.XX`
- Puerto de acceso: `8096` (HTTP)
- Protocolo de streaming: HLS (HTTP Live Streaming)

---

## 2. Infraestructura

### Instancia AWS

| Parámetro         | Valor                   |
|-------------------|-------------------------|
| Tipo de instancia | `t2.micro` / `t3.small` |
| Sistema operativo | Ubuntu Server 22.04 LTS |
| IP pública        | `13.61.226.XX`          |
| Puerto abierto    | TCP `8096` (0.0.0.0/0)  |

### Security Group AWS

<img width="793" height="241" alt="image" src="https://github.com/user-attachments/assets/b43cb1e4-1538-4f5d-8224-34914b3e04c3" />

---

## 3. Instalación de Jellyfin

### 3.1 Actualización del sistema y dependencias

```bash
sudo apt update && sudo apt upgrade -y
```

### 3.2 Añadir repositorio e instalar Jellyfin

```bash
curl -s https://repo.jellyfin.org/install-debuntu.sh | sudo bash
```

### 3.3 Verificar que el servicio está activo

```bash
sudo systemctl status jellyfin
```

<img width="955" height="445" alt="image" src="https://github.com/user-attachments/assets/ac1f7122-fe97-4f73-9e85-28edf905c565" />

### 3.4 Primer acceso y configuración inicial

Acceso vía navegador a `http://13.61.226.XX:8096`

<img width="653" height="351" alt="image" src="https://github.com/user-attachments/assets/fb907eca-6014-4db8-974a-4939e00ac86e" />

### 3.5 Versión instalada

Ruta: **Dashboard → Sistema**

<img width="948" height="436" alt="image" src="https://github.com/user-attachments/assets/0f1f55bb-d67b-41e1-986e-f6fe260544b2" />

---

## 4. Configuración de la biblioteca

### 4.1 Creación del directorio de medios

```bash
sudo mkdir -p /srv/jellyfin/media/videos
sudo chown -R jellyfin:jellyfin /srv/jellyfin/media
```

**Salida del terminal:**
```
# Pegar aquí la salida
```

### 4.2 Creación de la biblioteca desde el panel

Ruta: **Dashboard → Bibliotecas → Añadir biblioteca**

- **Tipo:** Películas o Vídeos
- **Ruta:** `/srv/jellyfin/media/videos`

<img width="648" height="344" alt="image" src="https://github.com/user-attachments/assets/64eb4fed-80a3-4cc7-8838-9180069fa35b" />

---

## 5. Publicación del vídeo

### 5.1 Descarga del vídeo H.264/MP4

```bash
sudo wget -O /srv/jellyfin/media/videos/bigbuckbunny.mp4 \
  "https://www.w3schools.com/html/mov_bbb.mp4"
sudo chown jellyfin:jellyfin /srv/jellyfin/media/videos/bigbuckbunny.mp4
```

### 5.2 Indexación automática

Jellyfin indexa el vídeo automáticamente una vez copiado al directorio de la biblioteca.

<img width="181" height="287" alt="image" src="https://github.com/user-attachments/assets/c9fcc6ec-0b47-486b-b32b-9748f3b95b73" />

---

## 6. Verificación desde el navegador

### 6.1 Reproducción del vídeo

<img width="954" height="473" alt="image" src="https://github.com/user-attachments/assets/3668828c-25ae-4138-8ed6-a820713dcd82" />

### 6.2 Estadísticas de reproducción (códec H.264)

Durante la reproducción → botón **ℹ️ Info** → **Datos de reproducción**

<img width="283" height="315" alt="image" src="https://github.com/user-attachments/assets/ae9fb6e0-87c4-4347-93bd-dd5b39b7ff13" />

Información verificada:
- **Reproductor:** Html Video Player
- **Método de reproducción:** Reproducción directa
- **Protocolo:** HTTP
- **Contenedor:** mp4
- **Códec de vídeo:** H264 Main
- **Resolución:** 320x176
- **Bitrate:** 629 kbps
- **Códec de audio:** AAC LC — 160 kbps — 48000 Hz

---

## 8. Formatos, códecs y protocolos

### 8.1 Verificación de los segmentos HLS

Mientras el vídeo se reproducía en el navegador, se verificaron los segmentos HLS generados por Jellyfin:

```bash
sudo ls /var/lib/jellyfin/transcodes/
```

**Salida del terminal:**
```
stream-19628.ts  stream-19635.ts  stream-19642.ts  stream-19649.ts ...
stream-19629.ts  stream-19636.ts  stream-19643.ts  stream-19650.ts  stream.m3u8
...
```

<img width="647" height="108" alt="image" src="https://github.com/user-attachments/assets/6295aa69-049d-4777-8204-e97333424139" />


### 8.2 Resumen de formatos y protocolos

| Elemento               | Valor                          |
|------------------------|--------------------------------|
| Contenedor             | MP4                            |
| Códec de vídeo         | H.264 (AVC) Main               |
| Códec de audio         | AAC LC                         |
| Protocolo de streaming | HTTP / HLS                     |
| Segmentos HLS          | `.ts` + manifiesto `.m3u8`     |
| Método de reproducción | Directo (sin transcodificación)|

---

## 9. Validación final

### Resumen del estado de los servicios

| Servicio  | Estado          | Puerto | Observaciones                     |
|-----------|-----------------|--------|-----------------------------------|
| Jellyfin  | ✅ Activo       | 8096   | Accesible desde el navegador      |
| AWS SG    | ✅ Configurado  | 8096   | Abierto desde 0.0.0.0/0           |

---

*Documentación elaborada por el Grupo 2 — Proyecto Transversal*
