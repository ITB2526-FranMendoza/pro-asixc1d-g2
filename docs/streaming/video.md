# Servicio de Vídeo — NGINX RTMP

> **Responsable:** fran [itb] mendoza jimenez  
> **Proyecto:** Proyecto Transversal 25/26 — InnovateTech  
> **Instancia AWS:** `i-025eff8c23ec4f60a` · t3.small · eu-north-1a  
> **IP pública:** `13.61.226.79`  
> **IP privada:** `172.16.34.20` (VLAN 30 — Streaming)

## Índice

1. [Descripción del servicio](#1-descripción-del-servicio)
2. [Infraestructura del servicio de vídeo](#2-infraestructura-del-servicio-de-vídeo)
3. [Instalación de NGINX + RTMP](#3-instalación-de-nginx-+-rtmp)
4. [Configuración del servicio RTMP/HLS](#4-configuración-del-servicio-rtmp/hls)
5. [Publicación del vídeo en streaming](#5-publicación-del-vídeo-en-streaming)
6. [Verificación desde navegador](#6-verificación-desde-navegador)
7. [Verificación desde VLC](#7-verificación-desde-vlc)
8. [Validación final del servicio](#8-validación-final-del-servicio)

## 1. Descripción del servicio

Se ha implementado un servidor de streaming de vídeo utilizando NGINX con el módulo RTMP (Real-Time Messaging Protocol). El sistema permite recibir vídeo mediante RTMP y redistribuirlo mediante HLS (HTTP Live Streaming) para reproducción desde navegador o VLC.

### Arquitectura del servicio

```text
FFmpeg → RTMP → NGINX → HLS → Navegador / VLC
```

---

# 2. Infraestructura del servicio de vídeo

## Puertos abiertos en AWS

| Puerto | Protocolo | Uso |
|---|---|---|
| 80 | TCP | HTTP/HLS |
| 1935 | TCP | RTMP |

El puerto 1935 se utiliza para streaming RTMP y debe estar habilitado en el Security Group de AWS.

<img width="1570" height="312" alt="image" src="https://github.com/user-attachments/assets/3f213221-fbb0-48b7-a886-90345b8922e9" />

---

# 3. Instalación de NGINX + RTMP

## Instalación de paquetes

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install nginx libnginx-mod-rtmp ffmpeg -y
```

## Verificación

```bash
nginx -v
ffmpeg -version
```

<img width="341" height="44" alt="image" src="https://github.com/user-attachments/assets/ee2640be-156e-4620-95ca-9a5f3f29d13b" />
<img width="1850" height="328" alt="image" src="https://github.com/user-attachments/assets/f2d46348-827b-439b-9f6f-15235c74f040" />

---

# 4. Configuración del servicio RTMP/HLS

## Configuración RTMP

Archivo:

```bash
/etc/nginx/nginx.conf
```

Configuración añadida:

```nginx
rtmp {
    server {
        listen 1935;
        chunk_size 4096;

        application live {
            live on;
            record off;

            #Convierte el stream RTMP a HLS
            hls on;
            hls_path /var/www/html/hls;
            hls_fragment 3;
            hls_playlist_length 60;
        }
    }
}
```

## Configuración HLS

```nginx
location /hls {
    types {
        application/vnd.apple.mpegurl m3u8;
        video/mp2t ts;
    }

    root /var/www/html;

    add_header Cache-Control no-cache;
    add_header Access-Control-Allow-Origin *;
}
```

## Creación del directorio HLS

```bash
sudo mkdir -p /var/www/html/hls
sudo chmod 777 /var/www/html/hls
```

## Reinicio del servicio

```bash
sudo systemctl restart nginx
sudo systemctl enable nginx
sudo systemctl status nginx
```

<img width="679" height="333" alt="image" src="https://github.com/user-attachments/assets/a926ec20-6e20-4024-99f4-1bcc5e04e211" />

---

# 5. Publicación del vídeo en streaming

## Descarga del vídeo de prueba

```bash
wget -O video.mp4 "https://www.w3schools.com/html/mov_bbb.mp4"
```

## Envío del stream mediante FFmpeg

```bash
ffmpeg -re -stream_loop -1 -i /home/ubuntu/video.mp4 \
-c:v libx264 -c:a aac \
-f flv rtmp://localhost/live/stream
```

## Formatos utilizados

| Elemento | Tecnología |
|---|---|
| Contenedor | MP4 |
| Códec vídeo | H.264 |
| Streaming | RTMP |
| Distribución web | HLS |

<img width="1849" height="1008" alt="image" src="https://github.com/user-attachments/assets/2ebbf030-210d-4590-98f7-d29a1c8e06d1" />
<img width="1841" height="541" alt="image" src="https://github.com/user-attachments/assets/c3ee4e95-9ede-4246-aea2-d7e3edcd7984" />

---

# 6. Verificación desde navegador

Crear una página HTML sencilla con un player HLS (HTTPS live stream) para ver el vídeo en el navegador:

<img width="812" height="547" alt="image" src="https://github.com/user-attachments/assets/b6040c40-e155-4aa6-9623-13e0acfb442b" />

El streaming HLS puede visualizarse directamente desde navegador utilizando un reproductor compatible con HLS.js.

URL utilizada:

```text
http://IP_PUBLICA_AWS/hls/stream.m3u8
```
--> EN NUESTRO CASO LA IP PÚBLICA SERIA 13.60.54.35

<img width="742" height="531" alt="image" src="https://github.com/user-attachments/assets/69996240-b86f-458c-b0f7-10f05bbd94d6" />

---

# 7. Verificación desde VLC

Se ha validado la reproducción del stream desde VLC utilizando la URL:

```text
http://IP_PUBLICA_AWS/hls/stream.m3u8
```
--> EN NUESTRO CASO LA IP PÚBLICA SERIA 13.60.54.35

<img width="741" height="615" alt="image" src="https://github.com/user-attachments/assets/bb51b228-8e69-49e1-9085-f789804c3a37" />

---

# 8. Validación final del servicio

## Comprobaciones realizadas

- Servidor NGINX operativo.
- Streaming RTMP funcional.
- Conversión HLS operativa.
- Reproducción desde navegador.
- Reproducción desde VLC.
- Acceso remoto funcional desde clientes externos.

- Ficheros HLS generados:
<img width="1615" height="394" alt="image" src="https://github.com/user-attachments/assets/79277c95-8551-4f8f-b755-bd17e6723e1f" />

## Resultado

El sistema de streaming cumple correctamente los requisitos mínimos establecidos para el RA8, proporcionando distribución multimedia estable y accesible desde distintos clientes.
