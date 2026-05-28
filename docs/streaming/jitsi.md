# Servicio de Videoconferencia — Jitsi Meet

> Servicio de videoconferencia Jitsi Meet desplegado en AWS EC2 mediante Docker, con acceso seguro HTTPS y comunicación en tiempo real entre usuarios.

## Índice

1. [Descripción del servicio de videoconferencia](#1-descripción-del-servicio-de-videoconferencia)  
2. [Infraestructura](#2-infraestructura)  
3. [Instalación](#3-instalación)  
4. [IP Elástica en AWS](#4-ip-elástica-en-aws)  
5. [Dominio gratuito en No-IP](#5-dominio-gratuito-en-no-ip)  
6. [Acceso al servicio desde el navegador](#6-acceso-al-servicio-desde-el-navegador)  
7. [Prueba real de videoconferencia](#7-prueba-real-de-videoconferencia)  
8. [Prueba de sonido](#8-prueba-de-sonido)  
9. [Protocolos utilizados](#9-protocolos-utilizados)  
10. [Incidencias detectadas y soluciones aplicadas](#10-incidencias-detectadas-y-soluciones-aplicadas)  
11. [Conclusiones técnicas](#11-conclusiones-técnicas)

---

## 1. Descripción del servicio de videoconferencia

Jitsi Meet es una plataforma de videoconferencia de código abierto y completamente gratuita. Funciona desde el navegador sin necesidad de instalar nada en los clientes. Utiliza WebRTC como protocolo principal para enviar audio y vídeo en tiempo real entre usuarios. Se ha desplegado en una instancia EC2 de AWS con Ubuntu 22.04 LTS mediante Docker.

---

## 2. Infraestructura

| Elemento | Valor |
|----------|-------|
| Instancia AWS EC2 | t3.small (2 vCPU, 2GB RAM) |
| Sistema Operativo | Ubuntu Server 22.04 LTS |
| IP pública | 13.62.220.208 |
| Dominio | innovatetech-g2.ddns.net |
| Puertos abiertos | TCP 80, TCP 443, UDP 10000 |

---

## 3. Instalación

### 3.1 Actualización del sistema e instalación de dependencias

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install curl wget gnupg -y
```

> <img width="568" height="201" alt="image" src="https://github.com/user-attachments/assets/c13d109f-b98f-45f6-8f93-4360bfb7aba6" />
> <img width="447" height="169" alt="image" src="https://github.com/user-attachments/assets/11377cc3-a907-4697-ad51-b8e862d580a5" />



### 3.2 Configuración del hostname

```bash
sudo hostnamectl set-hostname innovatetech-g2.ddns.net
```

> <img width="599" height="157" alt="image" src="https://github.com/user-attachments/assets/6fb18975-a36c-4392-b4d3-e54af3918c06" />


### 3.3 Configuración del fichero /etc/hosts

```bash
sudo nano /etc/hosts
```

Contenido añadido:
```
 innovatetech-g2.ddns.net
```

> <img width="586" height="266" alt="image" src="https://github.com/user-attachments/assets/9b172ac4-0d93-4b98-b22b-b687ef7b7afc" />


### 3.4 Instalación de Docker

```bash
sudo apt install docker.io docker-compose -y
sudo systemctl enable docker
sudo systemctl start docker
sudo docker --version
```

> <img width="598" height="241" alt="image" src="https://github.com/user-attachments/assets/e69511a1-8c61-42ac-a4f2-aa7a73783393" />


### 3.5 Descarga de la configuración oficial de Jitsi para Docker

```bash
cd ~
wget https://github.com/jitsi/docker-jitsi-meet/archive/refs/heads/master.zip
sudo apt install unzip -y
unzip master.zip
cd docker-jitsi-meet-master
cp env.example .env
```

> <img width="524" height="338" alt="image" src="https://github.com/user-attachments/assets/b3f118dd-8351-4732-bd33-02f0065201c8" />


### 3.6 Configuración del fichero .env con el dominio y Let's Encrypt

```bash
sed -i 's/HTTP_PORT=8000/HTTP_PORT=80/' .env
sed -i 's/HTTPS_PORT=8443/HTTPS_PORT=443/' .env
sed -i 's|TZ=UTC|TZ=Europe/Madrid|' .env
sed -i 's|#PUBLIC_URL=https://meet.example.com:${HTTPS_PORT}|PUBLIC_URL=https://innovatetech-g2.ddns.net|' .env
sed -i 's|#ENABLE_LETSENCRYPT=1|ENABLE_LETSENCRYPT=1|' .env
sed -i 's|#LETSENCRYPT_DOMAIN=meet.example.com|LETSENCRYPT_DOMAIN=innovatetech-g2.ddns.net|' .env
sed -i 's|#LETSENCRYPT_EMAIL=alice@atlanta.net|LETSENCRYPT_EMAIL=ruben.mateos.7e7@itb.cat|' .env
sed -i 's|#LETSENCRYPT_ACME_SERVER="letsencrypt"|LETSENCRYPT_ACME_SERVER="letsencrypt"|' .env
echo "RESTART_POLICY=always" >> .env
```

> <img width="808" height="144" alt="image" src="https://github.com/user-attachments/assets/e2e78b90-3b48-45ff-b290-2111014f7b1e" />


### 3.7 Generación de contraseñas con gen-passwords.sh

```bash
./gen-passwords.sh
```

> **Nota:** El script `gen-passwords.sh` genera automáticamente contraseñas seguras para los componentes internos de Jitsi (Jicofo, Videobridge, etc.) que necesitan para comunicarse entre ellos.

> <img width="707" height="200" alt="image" src="https://github.com/user-attachments/assets/be6046e9-496c-4241-97a3-109c6cbb91e6" />


### 3.8 Creación de los directorios necesarios y arranque de los contenedores

```bash
mkdir -p ~/.jitsi-meet-cfg/{web,transcripts,prosody/config,prosody/prosody-plugins-custom,jicofo,jvb,jigasi,jibri}
sudo docker-compose up -d
```

> <img width="962" height="494" alt="image" src="https://github.com/user-attachments/assets/af0cc7ea-9778-47a2-9da8-075f103c554d" />


### 3.9 Verificación de que todos los contenedores están activos

```bash
sudo docker-compose ps
```

> <img width="962" height="135" alt="image" src="https://github.com/user-attachments/assets/38f2fe50-d3af-4828-888b-610d4e0b95a9" />


---

## 4. IP Elástica en AWS

Para evitar que la IP pública cambie al reiniciar la instancia, se asignó una IP Elástica fija desde AWS.

### 4.1 Creación y asignación de la IP Elástica

> <img width="963" height="159" alt="image" src="https://github.com/user-attachments/assets/b11e5b4a-d2b7-49e6-929b-918d91ac7362" />
> <img width="967" height="346" alt="image" src="https://github.com/user-attachments/assets/04b5078d-121c-4593-bdc8-64dfe2cbf5e1" />



---

## 5. Dominio gratuito en No-IP

Durante la instalación manual de Jitsi se detectaron problemas con la configuración de nginx. Además, sin dominio no era posible obtener un certificado SSL válido con Let's Encrypt. Para solucionarlo, se creó un dominio gratuito en No-IP apuntando a la IP pública de la instancia EC2.

**Pasos realizados:**
1. Registro en [noip.com](https://www.noip.com)
2. Creación del hostname `innovatetech-g2.ddns.net` con tipo A apuntando a la IP pública

> <img width="969" height="360" alt="image" src="https://github.com/user-attachments/assets/91dce3e0-ff69-4638-9434-824be4500201" />


---

## 6. Acceso al servicio desde el navegador

Acceso via HTTPS: [https://innovatetech-g2.ddns.net](https://innovatetech-g2.ddns.net)

> <img width="922" height="520" alt="image" src="https://github.com/user-attachments/assets/f420bcdb-1394-4388-b327-cf3b6985555f" />


---

## 7. Prueba real de videoconferencia

### 7.1 Creación de una sala de videoconferencia

> <img width="596" height="562" alt="image" src="https://github.com/user-attachments/assets/55e0e991-813e-461b-8a00-f4dac2f518dd" />
> <img width="864" height="485" alt="image" src="https://github.com/user-attachments/assets/104944c4-4948-4bb4-aba2-051f8c6cb06f" />


### 7.2 Videollamada funcional entre dos participantes

**Vista desde PC:**

> <img width="923" height="521" alt="image" src="https://github.com/user-attachments/assets/2f5cd8ac-b21e-4256-b5fc-6c2fc1bbc3ac" />


**Vista desde móvil:**

> <img width="590" height="617" alt="image" src="https://github.com/user-attachments/assets/d76cb89a-7e20-4f51-83c8-71464692dfd1" />


---

## 8. Prueba de sonido

> [prueba de sonido de la videoconferencia.webm](https://github.com/user-attachments/assets/8a6b6db5-3c1e-4650-be4e-e5fa703a1dba)


---

## 9. Protocolos utilizados

| Protocolo | Función |
|-----------|---------|
| **WebRTC** | Protocolo principal para enviar audio y vídeo en tiempo real entre navegadores |
| **XMPP / Prosody** | Sistema de mensajería interno que gestiona los usuarios de cada sala |
| **STUN** | Ayuda a los clientes a descubrir su IP pública para conectarse |
| **TURN** | Cuando dos usuarios no pueden conectarse directamente, el servidor retransmite el tráfico |

---

## 10. Incidencias detectadas y soluciones aplicadas

### 10.1 Problema con el certificado autofirmado

La cámara no funcionaba porque los navegadores detectaban el certificado SSL como no seguro, bloqueando el acceso a la cámara y al micrófono. Se solucionó creando un dominio en No-IP e instalando un certificado SSL válido mediante Let's Encrypt.

### 10.2 Problema con la configuración manual de nginx

La instalación manual de Jitsi generó problemas con nginx y el procesamiento SSI de los ficheros HTML. Se optó por utilizar Docker, que incluye toda la configuración preconfigurada correctamente.

---

## 11. Conclusiones técnicas

El servicio de videoconferencia Jitsi Meet se ha desplegado correctamente en una instancia EC2 de AWS mediante Docker. El uso de Docker ha simplificado la instalación y configuración respecto a la instalación manual. El servicio es accesible desde cualquier navegador moderno via HTTPS con certificado SSL válido.
