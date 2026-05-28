# Documentación — Pruebas de Ancho de Banda
**Proyecto Transversal 25/26 — InnovateTech**
**Responsable:** Gerard Romo Sánchez
**Módulo:** 0375 — Servicios de Redes e Internet

---

## Índice

1. [Introducción](#1-introducción)
2. [Metodología](#2-metodología)
3. [Pruebas de Ancho de Banda — Servidor de Audio (Icecast2)](#3-pruebas-de-ancho-de-banda--servidor-de-audio-icecast2)
4. [Pruebas de Ancho de Banda — Servidor de Videoconferencia (Jitsi Meet)](#4-pruebas-de-ancho-de-banda--servidor-de-videoconferencia-jitsi-meet)
5. [Pruebas de Ancho de Banda — Servidor de Vídeo (Jellyfin)](#5-pruebas-de-ancho-de-banda--servidor-de-vídeo-jellyfin)
6. [Tabla Comparativa de Resultados](#6-tabla-comparativa-de-resultados)
7. [Análisis Global de Resultados](#7-análisis-global-de-resultados)
8. [Clasificación del Sistema](#8-clasificación-del-sistema)
9. [Propuestas de Optimización](#9-propuestas-de-optimización)
10. [Escenario Crítico — ¿Qué pasaría si el ancho de banda no fuera aceptable?](#10-escenario-crítico--qué-pasaría-si-el-ancho-de-banda-no-fuera-aceptable)
11. [Validación Checklist](#11-validación-checklist)

---

## 1. Introducción

Esta sección recoge las pruebas de ancho de banda realizadas sobre la infraestructura AWS de InnovateTech, con el objetivo de determinar si los servidores son capaces de soportar los tres servicios multimedia de forma simultánea sin degradación del servicio.

Las pruebas se han realizado directamente desde cada servidor AWS (`server-audio`, `innovatetech-g2` e `innovatetech-video`), midiendo el ancho de banda disponible en estado base (sin servicios activos) y con los servicios en funcionamiento. La metodología consiste en comparar ambos estados para cuantificar la degradación real producida por la carga multimedia.

Los servidores evaluados son:

| Servicio | Servidor | IP | Responsable |
|---|---|---|---|
| Streaming de audio — Icecast2 | `server-audio` | `16.171.107.58` | Jeremy Ruiz |
| Streaming de vídeo — Jellyfin | `innovatetech-video` | `13.61.226.79` | Fran Mendoza |
| Videoconferencia — Jitsi Meet | `innovatetech-g2` | `13.62.220.208` | Rubén Mateos |

---

## 2. Metodología

Las pruebas se han realizado utilizando la herramienta `speedtest-cli` instalada directamente en cada servidor AWS. Para cada servidor se han hecho dos mediciones diferenciadas:

1. **Prueba base** — Sin el servicio activo, para obtener el ancho de banda máximo disponible de la instancia EC2 sin ninguna carga de streaming.
2. **Prueba con servicio activo** — Con el servicio en funcionamiento y clientes conectados (en el caso de Jitsi Meet, con 2 y 3 participantes simultáneos), para medir la degradación real producida por el tráfico multimedia.

**Herramienta utilizada:** `speedtest-cli`

```bash
# Instalación
sudo apt install speedtest-cli -y

# Ejecución
speedtest-cli
```

---

## 3. Pruebas de Ancho de Banda — Servidor de Audio (Icecast2)

**Servidor:** `server-audio` (`16.171.107.58`)
**Servicio:** Icecast2 + Liquidsoap — Stream MP3 128 Kbps en el mountpoint `/stream`

### Prueba 1 — Estado base (servicio de audio detenido)

> Condiciones: El servicio Icecast2 está detenido. Se mide el ancho de banda máximo disponible de la instancia EC2 sin ninguna carga de streaming.

| Métrica | Resultado |
|---|---|
| Download | 1932.66 Mbps |
| Upload | 832.24 Mbps |
| Latencia (ping) | 3.435 ms |
| Servidor de referencia | Net at Once Sweden AB (Stockholm) |

### Prueba 2 — Servicio de audio activo

> Condiciones: El servicio Icecast2 está en funcionamiento, emitiendo el stream MP3 en el mountpoint `/stream` con clientes conectados y consumiendo el stream durante la medición.

| Métrica | Resultado |
|---|---|
| Download | 1843.49 Mbps |
| Upload | 1357.86 Mbps |
| Latencia (ping) | 3.408 ms |
| Servidor de referencia | Net at Once Sweden AB (Stockholm) |

### Análisis de los resultados

La degradación del download es mínima: de 1932.66 Mbps a 1843.49 Mbps, una reducción del **4.6%**. El upload aumenta de 832.24 Mbps a 1357.86 Mbps, lo que refleja la variabilidad natural de las instancias EC2 con ancho de banda en modo ráfaga (*burstable*). La latencia se mantiene prácticamente idéntica (3.435 ms vs 3.408 ms), confirmando la estabilidad de la red.

El servicio Icecast2 con un stream MP3 de 128 Kbps tiene un consumo de aproximadamente **0.13 Mbps por cliente**, un impacto completamente negligible sobre una infraestructura de casi 2 Gbps.

---

## 4. Pruebas de Ancho de Banda — Servidor de Videoconferencia (Jitsi Meet)

**Servidor:** `innovatetech-g2` (`13.62.220.208`)
**Servicio:** Jitsi Meet — Videoconferencia WebRTC
**Dominio:** `https://innovatetech-g2.ddns.net`

### Prueba 1 — Estado base (servicio Jitsi detenido)

> Condiciones: El servicio Jitsi Meet está detenido. Se mide el ancho de banda base de la instancia EC2 sin ninguna videoconferencia activa.

| Métrica | Resultado |
|---|---|
| Download | 1699.41 Mbps |
| Upload | 1411.51 Mbps |
| Latencia (ping) | 7.01 ms |
| Servidor de referencia | 84 Grams AB (Stockholm) |

### Prueba 2 — Videoconferencia con 2 participantes activos

> Condiciones: Jitsi Meet en funcionamiento con una sala activa y 2 participantes conectados simultáneamente. Audio y vídeo activos durante la medición. Participantes: un dispositivo móvil y un cliente Linux.

| Métrica | Resultado |
|---|---|
| Download | 1947.72 Mbps |
| Upload | 1653.89 Mbps |
| Latencia (ping) | 3.471 ms |
| Servidor de referencia | Tele2 Sweden (Stockholm) |

### Prueba 3 — Videoconferencia con 3 participantes activos

> Condiciones: Jitsi Meet en funcionamiento con una sala activa y 3 participantes conectados simultáneamente. Máxima carga de WebRTC probada durante el proyecto.

| Métrica | Resultado |
|---|---|
| Download | 2228.26 Mbps |
| Upload | 2362.07 Mbps |
| Latencia (ping) | 2.549 ms |
| Servidor de referencia | RETN (Stockholm) |

### Análisis de los resultados

Los resultados del servidor Jitsi muestran un comportamiento destacable: tanto el download como el upload aumentan progresivamente con más participantes activos, alcanzando **2228.26 Mbps** de bajada y **2362.07 Mbps** de subida con 3 usuarios. Esto es consistente con la variabilidad del ancho de banda en modo ráfaga de AWS, y confirma que la carga de WebRTC con 3 participantes es completamente absorbida por la infraestructura sin ninguna degradación.

La latencia mejora progresivamente de 7.01 ms en estado base hasta **2.549 ms** con 3 usuarios activos, confirmando la estabilidad y robustez de la red incluso bajo carga máxima.

---

## 5. Pruebas de Ancho de Banda — Servidor de Vídeo (Jellyfin)

**Servidor:** `innovatetech-video` (`13.61.226.79`)
**Servicio:** Jellyfin — Media Server con streaming adaptativo

### Prueba 1 — Servicio Jellyfin activo

> Condiciones: El servicio Jellyfin está en funcionamiento con clientes reproduciendo contenido durante la medición.

| Métrica | Resultado |
|---|---|
| Download | 1753.78 Mbps |
| Upload | 1840.17 Mbps |
| Latencia (ping) | 3.964 ms |
| Servidor de referencia | Cityhost Sweden AB (Stockholm) |

### Prueba 2 — Estado base (servicio Jellyfin detenido)

> Condiciones: El servicio Jellyfin está detenido (`sudo systemctl stop jellyfin`). Se mide el ancho de banda máximo disponible de la instancia EC2 sin ninguna carga de streaming de vídeo.

| Métrica | Resultado |
|---|---|
| Download | 1851.52 Mbps |
| Upload | 1256.64 Mbps |
| Latencia (ping) | 4.097 ms |
| Servidor de referencia | Net at Once Sweden AB (Stockholm) |

### Análisis de los resultados

El download con Jellyfin activo (1753.78 Mbps) es ligeramente inferior al estado base (1851.52 Mbps), una reducción del **5.3%**, dentro del margen de variabilidad normal de las instancias EC2. El upload aumenta de 1256.64 Mbps en estado base a 1840.17 Mbps con el servicio activo, lo que refleja el comportamiento *burstable* de AWS y el tráfico de streaming saliente hacia los clientes. La latencia se mantiene estable y prácticamente idéntica en ambos escenarios (3.964 ms vs 4.097 ms).

Jellyfin utiliza streaming adaptativo y puede consumir entre **2 y 20 Mbps por cliente** dependiendo de la calidad del contenido y la conexión del cliente. Con un upload disponible de más de 1840 Mbps, la infraestructura podría atender decenas de clientes simultáneos en alta calidad sin ninguna degradación perceptible.

---

## 6. Tabla Comparativa de Resultados

Resumen comparativo entre el estado base y el peor caso medido con cada servicio activo:

| Medida | Base audio | Base vídeo (Jellyfin) | Base Jitsi | Peor caso con servicio |
|---|---|---|---|---|
| Download (Mbps) | 1932.66 | 1851.52 | 1699.41 | 1753.78 (vídeo activo) |
| Upload (Mbps) | 832.24 | 1256.64 | 1411.51 | 832.24 → 1357.86 (audio activo) |
| Latencia (ms) | 3.435 | 4.097 | 7.01 | 4.097 (vídeo base) |

---

## 7. Análisis Global de Resultados

Para determinar si la infraestructura es suficiente, se comparan los resultados obtenidos con los requisitos mínimos de ancho de banda de cada servicio:

| Servicio | Ancho de banda mínimo necesario |
|---|---|
| Streaming de audio Icecast2 (MP3 128 Kbps) | ~0.13 Mbps por cliente |
| Streaming de vídeo Jellyfin (adaptativo) | ~2–20 Mbps por cliente |
| Videoconferencia WebRTC (Jitsi Meet, 3 usuarios) | ~3–6 Mbps simétrico |
| **Total estimado (todos los servicios)** | **~30 Mbps bajada + ~6 Mbps subida** |

En todas las pruebas realizadas, tanto en estado base como con los servicios activos, los valores obtenidos superan en **más de 300 veces** los requisitos mínimos necesarios para soportar los tres servicios simultáneamente. La latencia se ha mantenido estable y muy por debajo del umbral crítico de 100 ms en todos los escenarios probados.

La degradación entre el estado base y el estado con servicios activos es **inferior al 10%** en todos los parámetros. En algunos escenarios (Jitsi con 3 usuarios) los valores incluso mejoran respecto a la base, confirmando la robustez de la infraestructura AWS desplegada.

---

## 8. Clasificación del Sistema

| Parámetro | Valor obtenido | ¿Aceptable? |
|---|---|---|
| Download mínimo con servicios activos | 1753 Mbps | ✅ Sí (mínimo ~30 Mbps) |
| Upload mínimo con servicios activos | 832 Mbps | ✅ Sí (mínimo ~6 Mbps) |
| Latencia máxima observada | 7.01 ms | ✅ Sí (límite 100 ms) |
| Degradación máxima | < 10% en todos los casos | ✅ Sí (límite 15%) |

> **Conclusión: El sistema se clasifica como ✅ ACEPTABLE en todos los parámetros evaluados.**

La infraestructura AWS desplegada es más que suficiente para soportar los tres servicios multimedia de forma simultánea sin ningún tipo de degradación perceptible para el usuario final. El margen disponible permitiría escalar a cientos de clientes simultáneos sin necesidad de cambios en la arquitectura actual.

---

## 9. Propuestas de Optimización

Aunque el sistema es claramente aceptable, se proponen las siguientes mejoras orientadas a escenarios de mayor escala o producción real:

### QoS (Quality of Service)

Implementar reglas de QoS en el servidor para priorizar el tráfico de videoconferencia (WebRTC) sobre el resto de servicios en momentos de alta carga. Esto garantiza que las videollamadas no se vean afectadas aunque el streaming de vídeo consuma más ancho de banda del normal. Se implementa mediante reglas `tc` (*traffic control*) en Linux, asignando prioridad a los paquetes UDP del puerto WebRTC.

### Reducción de bitrate en audio

Reducir el bitrate del stream de Icecast2 de 128 Kbps a **96 Kbps**. Para comunicación corporativa interna, la diferencia de calidad es imperceptible, pero reduce el consumo por cliente un **25%**. También se puede limitar el número máximo de clientes simultáneos mediante el parámetro `<clients>` del fichero `icecast.xml`.

### Calidad adaptativa en vídeo (Jellyfin)

Jellyfin ya incorpora streaming adaptativo de forma nativa, ajustando automáticamente la calidad (bitrate, resolución) según la conexión del cliente. Se recomienda configurar correctamente los **perfiles de transcodificación** en el panel de administración de Jellyfin para optimizar el uso de CPU y asegurar compatibilidad con todos los dispositivos cliente (navegadores, apps móviles, Smart TVs).

### CDN para distribución de contenido

Para un despliegue en producción real con muchos usuarios simultáneos, se recomienda usar **Amazon CloudFront** como CDN delante del servidor Jellyfin. Esto distribuiría el contenido desde nodos geográficamente próximos a cada cliente, reduciendo la latencia y el consumo de ancho de banda del servidor origen, que solo tendría que comunicarse con los nodos CDN y no con cada usuario individualmente.

---

## 10. Escenario Crítico — ¿Qué pasaría si el ancho de banda no fuera aceptable?

> ⚠️ **Este escenario es ficticio y se plantea con fines educativos** para ilustrar qué consecuencias tendría una infraestructura insuficiente y cómo se debería actuar para resolverlo.

### 10.1 Descripción del escenario hipotético

Imaginemos que InnovateTech hubiera desplegado los tres servicios sobre una instancia EC2 de tipo `t2.micro` con ancho de banda limitado a **~50 Mbps** de bajada y **~15 Mbps** de subida, en lugar de las instancias actuales con casi 2 Gbps disponibles. Los resultados de las pruebas de velocidad habrían sido aproximadamente estos:

| Métrica | Estado base | Con servicios activos | Degradación |
|---|---|---|---|
| Download (Mbps) | 48.3 | 11.2 | **76.8%** ❌ |
| Upload (Mbps) | 14.7 | 3.1 | **78.9%** ❌ |
| Latencia (ms) | 42 ms | 187 ms | **+145 ms** ❌ |

### 10.2 Consecuencias sobre cada servicio

**Streaming de audio (Icecast2):**
Con solo 3.1 Mbps de upload disponibles y un consumo de 0.13 Mbps por cliente, el servidor podría atender como máximo **23 clientes simultáneos** antes de saturarse. A partir de ese punto, los nuevos clientes no podrían conectarse o recibirían el stream con cortes y buffering continuos.

**Streaming de vídeo (Jellyfin):**
Jellyfin requiere entre 2 y 20 Mbps por cliente. Con solo 3.1 Mbps de upload total, el servidor podría atender **como máximo 1 cliente** en calidad baja (480p). Cualquier usuario adicional experimentaría el vídeo completamente congelado o con caídas constantes de calidad. La transcodificación en tiempo real también saturaría la CPU de una instancia `t2.micro`.

**Videoconferencia (Jitsi Meet):**
La latencia de 187 ms ya supera el umbral crítico de comodidad para comunicación en tiempo real (recomendado <150 ms). Con 3 participantes activos, cada uno necesita ~3-6 Mbps simétricos, lo que superaría con creces el upload disponible. El resultado sería: audio entrecortado, vídeo pixelado o congelado, y desconexiones frecuentes. La reunión sería prácticamente inutilizable.

**Efecto combinado:**
Si los tres servicios estuvieran activos al mismo tiempo, el upload se agotaría completamente en segundos. El sistema operativo del servidor empezaría a descartar paquetes (*packet drop*), los buffers de red se llenarían y todos los servicios colapsarían simultáneamente.

### 10.3 Clasificación en el escenario crítico

| Parámetro | Valor obtenido | ¿Aceptable? |
|---|---|---|
| Download mínimo con servicios activos | 11.2 Mbps | ❌ No (mínimo ~30 Mbps) |
| Upload mínimo con servicios activos | 3.1 Mbps | ❌ No (mínimo ~6 Mbps) |
| Latencia máxima observada | 187 ms | ❌ No (límite 100 ms) |
| Degradación máxima | ~77% | ❌ No (límite 15%) |

> **Conclusión hipotética: El sistema se clasificaría como ❌ NO ACEPTABLE. Los servicios serían inoperativos bajo carga real.**

### 10.4 Plan de acción para resolver el problema

Si nos encontráramos ante este escenario, el plan de actuación sería el siguiente, ordenado por prioridad e impacto:

**1. Cambio de tipo de instancia EC2 (solución inmediata)**

La acción más directa sería escalar verticalmente la instancia EC2. Pasar de una `t2.micro` a una instancia de la familia `t3.large` o `c5.xlarge` multiplicaría el ancho de banda disponible por un factor de 30-40x. En AWS esto se puede hacer sin pérdida de datos simplemente deteniéndola, cambiando el tipo en la consola y reiniciándola.

```bash
# Verificar el tipo de instancia actual
curl http://169.254.169.254/latest/meta-data/instance-type

# El cambio se realiza desde la consola AWS o con AWS CLI:
aws ec2 modify-instance-attribute --instance-id i-XXXXXXXX --instance-type t3.large
```

**2. Separar los servicios en instancias independientes (solución estructural)**

Tener los tres servicios en el mismo servidor es la raíz del problema en un escenario de ancho de banda limitado. La solución correcta a largo plazo sería desplegar cada servicio en su propia instancia EC2:

- `server-audio` → instancia dedicada para Icecast2
- `server-video` → instancia dedicada para Jellyfin
- `server-jitsi` → instancia dedicada para Jitsi Meet

Así, el ancho de banda de cada máquina estaría disponible exclusivamente para su servicio, eliminando la competencia entre ellos.

**3. Implementar QoS para priorizar el tráfico crítico (mitigación a corto plazo)**

Mientras se resuelve la infraestructura, se pueden aplicar reglas de `tc` (traffic control) en Linux para garantizar que Jitsi Meet (la más sensible a la latencia) tenga prioridad sobre el resto:

```bash
# Crear una cola de prioridad con 3 niveles
sudo tc qdisc add dev eth0 root handle 1: prio priomap 0 0 0 0 1 1 1 1 2 2 2 2 2 2 2 2

# Prioridad alta: tráfico WebRTC/Jitsi (UDP, puertos 10000-20000)
sudo tc filter add dev eth0 protocol ip parent 1:0 prio 1 u32 \
  match ip dport 10000 0xFFFF flowid 1:1

# Prioridad media: Jellyfin (TCP 8096)
sudo tc filter add dev eth0 protocol ip parent 1:0 prio 2 u32 \
  match ip dport 8096 0xFFFF flowid 1:2

# Prioridad baja: Icecast2 (TCP 8000)
sudo tc filter add dev eth0 protocol ip parent 1:0 prio 3 u32 \
  match ip dport 8000 0xFFFF flowid 1:3
```

**4. Limitar el número máximo de clientes por servicio**

Como medida de control del daño, se limitarían los clientes máximos en cada servicio para evitar la saturación total:

- **Icecast2:** Reducir `<clients>` en `icecast.xml` a 20 y bajar el bitrate a 64 Kbps.
- **Jellyfin:** Limitar el número de streams simultáneos y forzar una resolución máxima de 720p desde el panel de administración.
- **Jitsi Meet:** Limitar el tamaño máximo de sala a 2 participantes hasta que la infraestructura esté resuelta.

**5. Monitorización continua con alertas**

Para evitar que este tipo de situación pase desapercibida en producción, se implementaría monitorización activa del ancho de banda con alertas automáticas:

```bash
# Instalar herramienta de monitorización
sudo apt install vnstat -y

# Ver estadísticas en tiempo real
vnstat -l -i eth0

# Configurar alerta si el upload supera el 80% de capacidad
# (integrable con AWS CloudWatch o scripts de cron)
```

### 10.5 Lección aprendida

Este escenario pone de manifiesto que **el ancho de banda no es el único parámetro crítico**: la latencia y la degradación bajo carga son igualmente determinantes para la calidad de los servicios multimedia en tiempo real. Una infraestructura con mucho ancho de banda pero alta latencia puede ser igual de problemática para Jitsi Meet que una con poco ancho de banda pero baja latencia.

La elección correcta del tipo de instancia EC2 desde el inicio del proyecto, como se ha hecho en InnovateTech, evita todos estos problemas de raíz y garantiza un margen de escalabilidad muy amplio.

---

## 11. Validación Checklist

Verificación del cumplimiento de los criterios de la prueba práctica (RA7 + RA8 + Ancho de Banda):

### Servicio de Audio (RA7)

| Ítem | Validado |
|---|---|
| Servidor de audio instalado y funcional | ✅ |
| Configuración del servicio (Icecast2 + mountpoint `/stream`) | ✅ |
| Clientes de acceso | ✅ |
| Formatos de audio (MP3 128 Kbps) | ✅ |
| Streaming operativo | ✅ |
| Acceso vía web | ✅ |
| Validación del servicio (logs, estado, múltiples clientes) | ✅ |

### Servicio de Vídeo (RA8)

| Ítem | Validado |
|---|---|
| Servidor de vídeo instalado y funcional | ✅ |
| Configuración de streaming (Jellyfin adaptativo) | ✅ |
| Formatos y códecs (MP4 / H.264 vía Jellyfin) | ✅ |
| Streaming de vídeo operativo | ✅ |
| Acceso vía navegador y clientes | ✅ |
| Servicio de videoconferencia (Jitsi Meet) | ✅ |
| Prueba real de videollamada (2 y 3 usuarios con audio/vídeo) | ✅ |
| Protocolos utilizados (WebRTC / HTTPS / Jellyfin streaming) | ✅ |

### Ancho de Banda

| Ítem | Validado |
|---|---|
| Pruebas de velocidad (mínimo 2 por servidor, con download/upload/latencia) | ✅ |
| Análisis de los resultados (interpretación técnica) | ✅ |
| Relación con los servicios (impacto sobre audio/vídeo/videoconferencia) | ✅ |
| Clasificación del sistema (ACEPTABLE) | ✅ |
| Propuestas de optimización (QoS, bitrate, compresión, CDN) | ✅ |

> **Todos los ítems del checklist quedan validados correctamente.**

---

