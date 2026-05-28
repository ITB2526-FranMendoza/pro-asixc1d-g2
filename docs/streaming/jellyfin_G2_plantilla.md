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

<!-- CAPTURA: Security Group con el puerto 8096 abierto -->
> 📸 *Insertar captura del Security Group en AWS mostrando TCP 8096 abierto desde 0.0.0.0/0*

---

## 3. Instalación de Jellyfin

### 3.1 Actualización del sistema y dependencias

```bash
sudo apt update && sudo apt upgrade -y
```

**Salida del terminal:**
```
# Pegar aquí la salida
```

### 3.2 Añadir repositorio e instalar Jellyfin

```bash
curl -s https://repo.jellyfin.org/install-debuntu.sh | sudo bash
```

**Salida del terminal:**
```
# Pegar aquí la salida
```

### 3.3 Verificar que el servicio está activo

```bash
sudo systemctl status jellyfin
```

**Salida del terminal:**
```
# Pegar aquí la salida (debe mostrar: active (running))
```

### 3.4 Primer acceso y configuración inicial

Acceso vía navegador a `http://13.61.226.XX:8096`

<!-- CAPTURA: Pantalla de login de Jellyfin -->
> 📸 *Insertar captura de la pantalla de inicio de sesión con el usuario admin*

<!-- CAPTURA: Dashboard principal -->
> 📸 *Insertar captura del dashboard principal después del login*

### 3.5 Versión instalada

Ruta: **Dashboard → Sistema**

<!-- CAPTURA: Dashboard > Sistema con la versión -->
> 📸 *Insertar captura de Dashboard → Sistema mostrando la versión instalada*

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

<!-- CAPTURA: Biblioteca creada en el panel -->
> 📸 *Insertar captura de la biblioteca creada en el panel de Jellyfin*

---

## 5. Publicación del vídeo

### 5.1 Descarga del vídeo H.264/MP4

```bash
sudo wget -O /srv/jellyfin/media/videos/bigbuckbunny.mp4 \
  "https://www.w3schools.com/html/mov_bbb.mp4"
sudo chown jellyfin:jellyfin /srv/jellyfin/media/videos/bigbuckbunny.mp4
```

**Salida del terminal:**
```
# Pegar aquí la salida del wget
```

### 5.2 Indexación automática

Jellyfin indexa el vídeo automáticamente una vez copiado al directorio de la biblioteca.

<!-- CAPTURA: Vídeo apareciendo en la biblioteca -->
> 📸 *Insertar captura del vídeo "bigbuckbunny" apareciendo en la sección "Reciente en Películas"*

---

## 6. Verificación desde el navegador

### 6.1 Reproducción del vídeo

<!-- CAPTURA: Vídeo reproduciéndose -->
> 📸 *Insertar captura del vídeo reproduciéndose en el navegador*

### 6.2 Estadísticas de reproducción (códec H.264)

Durante la reproducción → botón **ℹ️ Info** → **Datos de reproducción**

<!-- CAPTURA: Estadísticas de reproducción -->
> 📸 *Insertar captura mostrando las estadísticas con códec H264 Main*

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

## 7. Instalación y (intento de) conexión de MariaDB

### 7.1 Instalación del plugin desde el panel

Ruta: **Dashboard → Plugins → Catálogo → MariaDB → Instalar**

```bash
# Reinicio de Jellyfin después de instalar el plugin
sudo systemctl restart jellyfin
```

**Salida del terminal:**
```
# Pegar aquí la salida
```

### 7.2 Configuración de credenciales

<!-- CAPTURA: Plugin MariaDB configurado y conectado -->
> 📸 *Insertar captura del plugin configurado con las credenciales*

> ⚠️ **Nota:** [Indicar si la conexión funcionó correctamente o si hubo algún problema]

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

<!-- CAPTURA: Directorio con segmentos .m3u8 y .ts -->
> 📸 *Insertar captura del directorio con los ficheros .m3u8 y .ts generados*

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
| MariaDB   | ⚠️ Parcial      | 3306   | Plugin instalado                  |
| AWS SG    | ✅ Configurado  | 8096   | Abierto desde 0.0.0.0/0           |

### Acceso final verificado

URL de acceso: `http://13.61.226.XX:8096`

<!-- CAPTURA FINAL: Dashboard con la biblioteca y el vídeo indexado -->
> 📸 *Insertar captura final del dashboard mostrando el servicio operativo*

---

*Documentación elaborada por el Grupo 2 — Proyecto Transversal*
