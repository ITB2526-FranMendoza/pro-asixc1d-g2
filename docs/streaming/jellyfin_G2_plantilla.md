# Servei de Vídeo — Jellyfin (RA8)

---

## Índex

1. [Descripció del servei](#1-descripció-del-servei)
2. [Infraestructura](#2-infraestructura)
3. [Instal·lació de Jellyfin](#3-installació-de-jellyfin)
4. [Configuració de la biblioteca](#4-configuració-de-la-biblioteca)
5. [Publicació del vídeo](#5-publicació-del-vídeo)
6. [Verificació des del navegador](#6-verificació-des-del-navegador)
7. [Instal·lació i (intent de) connexió de MariaDB](#7-installació-i-intent-de-connexió-de-mariadb)
8. [Formats, còdecs i protocols](#8-formats-còdecs-i-protocols)
9. [Validació final](#9-validació-final)

---

## 1. Descripció del servei

Jellyfin és un servidor multimèdia de codi obert que permet emmagatzemar, organitzar i reproduir contingut de vídeo, àudio i imatges. En aquest projecte s'ha desplegat en una instància AWS per oferir un servei de streaming de vídeo intern per a un centre educatiu.

**Característiques principals:**
- Servidor: Jellyfin vX.X.X (vegeu captura de versió)
- Sistema operatiu: Ubuntu Server XX.04 LTS
- IP del servidor: `13.61.226.XX`
- Port d'accés: `8096` (HTTP)
- Protocol de streaming: HLS (HTTP Live Streaming)

---

## 2. Infraestructura

### Instància AWS

| Paràmetre        | Valor                  |
|------------------|------------------------|
| Tipus d'instància | `t2.micro` / `t3.small` |
| Sistema operatiu | Ubuntu Server 22.04 LTS |
| IP pública       | `13.61.226.XX`         |
| Port obert       | TCP `8096` (0.0.0.0/0) |

### Security Group AWS

<!-- CAPTURA: Security Group amb el port 8096 obert -->
> 📸 *Inserir captura del Security Group a AWS mostrant TCP 8096 obert des de 0.0.0.0/0*

---

## 3. Instal·lació de Jellyfin

### 3.1 Actualització del sistema i dependències

```bash
sudo apt update && sudo apt upgrade -y
```

**Sortida del terminal:**
```
# Enganxar aquí la sortida
```

### 3.2 Afegir repositori i instal·lar Jellyfin

```bash
curl -s https://repo.jellyfin.org/install-debuntu.sh | sudo bash
```

**Sortida del terminal:**
```
# Enganxar aquí la sortida
```

### 3.3 Verificar que el servei està actiu

```bash
sudo systemctl status jellyfin
```

**Sortida del terminal:**
```
# Enganxar aquí la sortida (ha de mostrar: active (running))
```

### 3.4 Primer accés i configuració inicial

Accés via navegador a `http://13.61.226.XX:8096`

<!-- CAPTURA: Pantalla de login de Jellyfin -->
> 📸 *Inserir captura de la pantalla d'inici de sessió amb l'usuari admin*

<!-- CAPTURA: Dashboard principal -->
> 📸 *Inserir captura del dashboard principal després del login*

### 3.5 Versió instal·lada

Ruta: **Dashboard → Sistema**

<!-- CAPTURA: Dashboard > Sistema amb la versió -->
> 📸 *Inserir captura de Dashboard → Sistema mostrant la versió instal·lada*

---

## 4. Configuració de la biblioteca

### 4.1 Creació del directori de medis

```bash
sudo mkdir -p /srv/jellyfin/media/videos
sudo chown -R jellyfin:jellyfin /srv/jellyfin/media
```

**Sortida del terminal:**
```
# Enganxar aquí la sortida
```

### 4.2 Creació de la biblioteca des del panell

Ruta: **Dashboard → Biblioteques → Afegir biblioteca**

- **Tipus:** Pel·lícules o Vídeos
- **Ruta:** `/srv/jellyfin/media/videos`

<!-- CAPTURA: Biblioteca creada al panell -->
> 📸 *Inserir captura de la biblioteca creada al panell de Jellyfin*

---

## 5. Publicació del vídeo

### 5.1 Descàrrega del vídeo H.264/MP4

```bash
sudo wget -O /srv/jellyfin/media/videos/bigbuckbunny.mp4 \
  "https://www.w3schools.com/html/mov_bbb.mp4"
sudo chown jellyfin:jellyfin /srv/jellyfin/media/videos/bigbuckbunny.mp4
```

**Sortida del terminal:**
```
# Enganxar aquí la sortida del wget
```

### 5.2 Indexació automàtica

Jellyfin indexa el vídeo automàticament un cop copiat al directori de la biblioteca.

<!-- CAPTURA: Vídeo apareixent a la biblioteca -->
> 📸 *Inserir captura del vídeo "bigbuckbunny" apareixent a la secció "Recent en Pel·lícules"*

---

## 6. Verificació des del navegador

### 6.1 Reproducció del vídeo

<!-- CAPTURA: Vídeo reproduint-se -->
> 📸 *Inserir captura del vídeo reproduint-se al navegador*

### 6.2 Estadístiques de reproducció (còdec H.264)

Durant la reproducció → botó **ℹ️ Info** → **Dades de reproducció**

<!-- CAPTURA: Estadístiques de reproducció -->
> 📸 *Inserir captura mostrant les estadístiques amb còdec H264 Main*

Informació verificada:
- **Reproductor:** Html Video Player
- **Mètode de reproducció:** Reproducció directa
- **Protocol:** HTTP
- **Contenidor:** mp4
- **Còdec de vídeo:** H264 Main
- **Resolució:** 320x176
- **Bitrate:** 629 kbps
- **Còdec d'àudio:** AAC LC — 160 kbps — 48000 Hz

---

## 7. Instal·lació i (intent de) connexió de MariaDB

### 7.1 Instal·lació del plugin des del panell

Ruta: **Dashboard → Plugins → Catàleg → MariaDB → Instal·lar**

```bash
# Reinici de Jellyfin després d'instal·lar el plugin
sudo systemctl restart jellyfin
```

**Sortida del terminal:**
```
# Enganxar aquí la sortida
```

### 7.2 Configuració de credencials

<!-- CAPTURA: Plugin MariaDB configurat i connectat -->
> 📸 *Inserir captura del plugin configurat amb les credencials*

> ⚠️ **Nota:** [Indicar si la connexió va funcionar correctament o si hi va haver algun problema]

---

## 8. Formats, còdecs i protocols

### 8.1 Verificació dels segments HLS

Mentre el vídeo es reproduïa al navegador, es van verificar els segments HLS generats per Jellyfin:

```bash
sudo ls /var/lib/jellyfin/transcodes/
```

**Sortida del terminal:**
```
stream-19628.ts  stream-19635.ts  stream-19642.ts  stream-19649.ts ...
stream-19629.ts  stream-19636.ts  stream-19643.ts  stream-19650.ts  stream.m3u8
...
```

<!-- CAPTURA: Directori amb segments .m3u8 i .ts -->
> 📸 *Inserir captura del directori amb els fitxers .m3u8 i .ts generats*

### 8.2 Resum de formats i protocols

| Element              | Valor                        |
|----------------------|------------------------------|
| Contenidor           | MP4                          |
| Còdec de vídeo       | H.264 (AVC) Main             |
| Còdec d'àudio        | AAC LC                       |
| Protocol de streaming| HTTP / HLS                   |
| Segments HLS         | `.ts` + manifest `.m3u8`     |
| Mètode de reproducció| Directe (sense transcodificació) |

---

## 9. Validació final

### Resum de l'estat dels serveis

| Servei       | Estat       | Port  | Observacions                  |
|--------------|-------------|-------|-------------------------------|
| Jellyfin     | ✅ Actiu    | 8096  | Accessible des del navegador  |
| MariaDB      | ⚠️ Parcial  | 3306  | Plugin instal·lat             |
| AWS SG       | ✅ Configurat | 8096 | Obert des de 0.0.0.0/0       |

### Accés final verificat

URL d'accés: `http://13.61.226.XX:8096`

<!-- CAPTURA FINAL: Dashboard amb la biblioteca i el vídeo indexat -->
> 📸 *Inserir captura final del dashboard mostrant el servei operatiu*

---

*Documentació elaborada pel Grup 2 — Projecte Transversal*
