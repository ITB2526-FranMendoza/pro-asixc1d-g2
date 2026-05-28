# CPD — InnovateTech

> **Responsable:** fran [itb] mendoza jimenez  
> **Proyecto:** Proyecto Transversal 25/26 — InnovateTech  

## Índice

1. [Diseño del CPD](#1-diseño-del-cpd)
2. [Ubicación física del CPD](#2-ubicación-física-del-cpd)
3. [Climatización y control ambiental](#3-climatización-y-control-ambiental)
4. [Distribución y gestión del cableado](#4-distribución-y-gestión-del-cableado)
5. [Infraestructura eléctrica](#5-infraestructura-eléctrica)
6. [Estructura de racks](#6-estructura-de-racks)
7. [Seguridad física y lógica](#7-seguridad-físicaa-y-lógica)
8. [Infraestructura AWS](#8-infraestructura-aws)
9. [Prevención de Riesgos Laborales](#9-prevención-de-riesgos-laborales)

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

<img width="708" height="840" alt="image" src="https://github.com/user-attachments/assets/aabff421-ef32-443a-a775-59d9ba67cdad" />

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

<img width="449" height="867" alt="image" src="https://github.com/user-attachments/assets/5616816b-f957-42f8-97ca-88cea8bd841a" />

---

## Rack 3 — Almacenamiento y respaldo

Contiene:

- NAS Synology.
- Dell PowerEdge.
- UPS APC.
- PDU.

<img width="698" height="881" alt="image" src="https://github.com/user-attachments/assets/48804dda-6800-4857-900e-43ff926e30e2" />

---

# 7. Seguridad física y lógica

La propuesta incorpora medidas de protección tanto físicas como lógicas.

## Seguridad física

- Control de acceso RFID.
- Videovigilancia CCTV.
- Sistemas antiincendios.
- Vías de evacuación.
- Señalización de emergencia.
  
<img width="1536" height="1024" alt="image" src="https://github.com/user-attachments/assets/1b387f30-0933-4b51-ac57-d2b072b6f574" />

PROPUESTA DE SAI:

<img width="434" height="407" alt="image" src="https://github.com/user-attachments/assets/70cb3ca4-71e8-43cb-b538-cc9686f92c31" />


## Seguridad lógica

- Firewall pfSense.
- Segmentación mediante VLANs.
- Backups automáticos.
- RAID para tolerancia a fallos.
- Centralización de logs.
- Monitorización continua.

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

<img width="824" height="657" alt="image" src="https://github.com/user-attachments/assets/cb89293d-8fc5-42a2-b975-7206cf503a39" />

---

# 9. Prevención de Riesgos Laborales

## Normativa aplicable

- Ley 31/1995 de Prevención de Riesgos Laborales
- Real Decreto 486/1997 — condiciones mínimas de seguridad en lugares de trabajo
- Reglamento Electrotécnico de Baja Tensión (REBT)

---

## Riesgos identificados

| Riesgo | Causa | Personal afectado |
|---|---|---|
| Eléctrico | Cableado, PDUs, SAI/UPS | Técnicos de mantenimiento |
| Incendio | Calor de servidores, cortocircuitos | Todo el personal |
| Sobrecarga postural | Instalación de equipos en rack | Técnicos de sistemas |
| Exposición al ruido | Ventiladores y climatización | Personal en sala |
| Caídas | Cables mal gestionados en suelo técnico | Todo el personal |
| Riesgo térmico | Temperatura elevada en pasillos calientes | Técnicos en sala |
| Agentes químicos | Gas FM-200 del sistema de extinción | Todo el personal |

---

## Medidas preventivas

**Señalización**
La sala dispone de señalización normalizada según la norma ISO 7010, visible y permanente en todos los puntos de acceso y zonas de riesgo.

**Evacuación**
- La puerta del CPD abre hacia el exterior para facilitar la salida rápida.
- Señalización luminiscente visible sin electricidad.
- Iluminación de emergencia automática ante corte de suministro.
- Tiempo de evacuación estimado: menos de 3 minutos.
- El sistema de extinción FM-200 tiene un retardo de 30 segundos con alarma acústica y luminosa previa a la descarga, dando tiempo al personal para evacuar.

**EPI obligatorios**
- Guantes aislantes
- Calzado de seguridad con suela aislante

---

## Mantenimiento periódico

| Tarea | Frecuencia | Responsable |
|---|---|---|
| Revisión sistema eléctrico (PDUs, SAI) | Trimestral | Técnico eléctrico |
| Limpieza filtros HEPA climatización | Mensual | Técnico de sistemas |
| Prueba sistema extinción FM-200 | Anual | Empresa de mantenimiento |
| Revisión salidas emergencia e iluminación | Semestral | Responsable PRL |
| Inspección cableado y organización racks | Trimestral | Técnico de sistemas |
| Test descarga SAI/UPS | Semestral | Técnico eléctrico |
| Revisión sensores ambientales | Mensual | Técnico de sistemas |

---

## Formación del personal

Todo el personal con acceso al CPD recibe formación obligatoria antes de trabajar en la sala:

- Riesgos específicos del CPD: eléctrico, incendio, gas FM-200 y ergonómico
- Procedimiento de evacuación y punto de reunión
- Actuación ante activación del sistema de extinción automática
- Normas de acceso y orden en sala
- Uso correcto de EPIs

Se realiza además un **simulacro de evacuación anual** y se mantiene un registro documental de la formación recibida por cada trabajador.

---

## Normas de orden en sala

- Prohibido dejar herramientas o material fuera de su lugar de almacenamiento.
- Tras cada intervención, el técnico debe dejar la sala en el mismo estado en que la encontró.
- Los pasillos frío y caliente deben mantenerse siempre despejados para garantizar el flujo de aire de climatización.

---
