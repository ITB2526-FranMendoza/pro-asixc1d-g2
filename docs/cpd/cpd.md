# CPD — InnovateTech

# 1. Diseño del CPD

## Objetivo

El objetivo del CPD de InnovateTech es proporcionar una infraestructura segura, escalable y tolerante a fallos capaz de soportar servicios empresariales, streaming multimedia, bases de datos y sistemas de videoconferencia.

---

# 2. Ubicación física del CPD

La sala del CPD ha sido diseñada en una planta baja sin ventanas, con acceso restringido y aislamiento térmico/acústico para mejorar tanto la seguridad física como el control ambiental.

## Características principales

- Acceso restringido mediante control RFID.
- Sala sin ventanas para dificultar identificación externa.
- Aislamiento acústico y térmico.
- Videovigilancia permanente.
- Sistemas antiincendios.

### 📸 CAPTURA
<img width="1256" height="846" alt="image" src="https://github.com/user-attachments/assets/9566be26-582f-495b-8ffb-83c458d79ded" />

---

# 3. Climatización y control ambiental

El CPD incorpora un sistema de refrigeración redundante N+1 para garantizar funcionamiento continuo incluso en caso de fallo de uno de los equipos de climatización.

## Configuración ambiental

| Elemento | Configuración |
|---|---|
| Temperatura | 21ºC |
| Humedad | 45% |
| Refrigeración | Redundante (N+1) |
| Filtrado de aire | HEPA |
| Monitorización | Sensores ambientales |

Además, se utilizan filtros HEPA para reducir polvo y partículas en suspensión.

### 📸 CAPTURA
<img width="1042" height="410" alt="image" src="https://github.com/user-attachments/assets/7288027a-d71a-42e2-8a1c-a1e345158031" />

---

# 4. Distribución y gestión del cableado

La infraestructura utiliza cableado estructurado Cat6 para soportar velocidades de hasta 10 Gbps.

## Características implementadas

- Uso de patch panels para facilitar mantenimiento.
- Etiquetado completo de conexiones.
- Separación física entre corriente eléctrica y datos.
- Organizadores horizontales y verticales de cableado.
- Canalizaciones independientes.

Cada conexión incluye:
- Identificador del rack.
- Número de patch panel.
- Puerto utilizado.
- Destino de conexión.

### 📸 CAPTURA
<img width="1203" height="805" alt="image" src="https://github.com/user-attachments/assets/822a773c-b849-4452-8070-786ffab3a619" />

---

# 5. Infraestructura eléctrica

El CPD dispone de una infraestructura eléctrica redundante diseñada para alta disponibilidad.

## Elementos principales

### Doble línea eléctrica

- Alimentación redundante independiente.
- Tolerancia ante fallo eléctrico.

### Sistemas SAI/UPS

- Protección frente a cortes eléctricos.
- Autonomía aproximada de 30 minutos.

### PDU por rack

- Distribución organizada de alimentación.

---

# 6. Estructura de racks

Se ha diseñado una separación física entre comunicaciones, servidores y almacenamiento para mejorar organización y escalabilidad.

---

## Rack 1 — Red y comunicaciones

Contiene:

- Switch Core Cisco Catalyst.
- Firewall pfSense.
- Router MikroTik.
- Patch panels.
- PDU y SAI.

### 📸 CAPTURA
<img width="561" height="860" alt="image" src="https://github.com/user-attachments/assets/55459c2f-ca5b-4f1f-beba-982825a53507" />

---

## Rack 2 — Servidores

Contiene:

- Servidor Web + SFTP.
- Samba Active Directory.
- Servidor de logs.
- Streaming de audio.
- Streaming de vídeo.
- Jitsi Meet.
- Base de datos.
- Backup y monitorización.

### 📸 CAPTURA
<img width="536" height="976" alt="image" src="https://github.com/user-attachments/assets/e6aa7e8d-95e2-44b7-b9ff-a4d09e7bf745" />

---

## Rack 3 — Almacenamiento y respaldo

Contiene:

- NAS Synology.
- Dell PowerEdge.
- pfSense.
- MikroTik.
- UPS APC.

### 📸 CAPTURA
<img width="734" height="932" alt="image" src="https://github.com/user-attachments/assets/982bab54-7788-4296-b037-b892ff8bc6c8" />

---

# 7. Seguridad física y lógica

La propuesta incorpora medidas de protección tanto físicas como lógicas.

## Seguridad física

- Control de acceso RFID.
- Videovigilancia CCTV.
- Sistemas antiincendios.
- Vías de evacuación.
- Señalización de emergencia.

## Seguridad lógica

- Firewall pfSense.
- Segmentación mediante VLANs.
- Backups automáticos.
- RAID para tolerancia a fallos.
- Centralización de logs.
- Monitorización continua.

### 📸 CAPTURA
<img width="567" height="418" alt="image" src="https://github.com/user-attachments/assets/38e1b372-818f-40ed-9956-952d73a0fac2" />

---

# 8. Infraestructura AWS

La infraestructura cloud se ha desplegado sobre AWS utilizando instancias EC2 independientes para cada servicio crítico.

## Servicios implementados

| Servicio | Tecnología |
|---|---|
| Web | NGINX |
| SFTP | OpenSSH |
| Directorio Activo | Samba AD |
| Logs | rsyslog + Graylog |
| Audio Streaming | Icecast2 |
| Video Streaming | NGINX + RTMP |
| Videoconferencia | Jitsi Meet |
| Base de Datos | MariaDB |

### 📸 CAPTURA
<img width="824" height="657" alt="image" src="https://github.com/user-attachments/assets/cb89293d-8fc5-42a2-b975-7206cf503a39" />

---
