# Base de Datos — InnovateTech

## 1. Descripción general

En este apartado se documenta la implementación de la base de datos de InnovateTech utilizando MariaDB sobre AWS EC2.

La base de datos permite gestionar empleados, clientes, videollamadas, auditoría y backups automáticos mediante triggers y eventos programados.

---

## 2. Instalación de MariaDB

Primero se actualizó el sistema operativo Ubuntu:

```bash
sudo apt update && sudo apt upgrade -y
```

Posteriormente se instaló MariaDB Server:

```bash
sudo apt install mariadb-server -y
```

Se comprobó que el servicio estuviera funcionando correctamente:

```bash
sudo systemctl status mariadb
```

<img width="455" height="229" alt="Captura de pantalla 2026-05-27 234547" src="https://github.com/user-attachments/assets/e81d8365-7f2d-4aa4-a64e-636d93000dbb" />


---

## 3. Creación de la base de datos

Se creó la base de datos principal del proyecto:

```sql
CREATE DATABASE innovatetech_db;
```

Después se accedió a MariaDB:

```bash
sudo mariadb
```

Y se seleccionó la base de datos:

```sql
USE innovatetech_db;
```

<img width="478" height="280" alt="Captura de pantalla 2026-05-27 234658" src="https://github.com/user-attachments/assets/75865993-c49e-47bc-aab8-08716ccdb039" />


---

## 4. Creación de tablas y carga de datos

Se ejecutaron los scripts de creación de tablas y carga de datos de prueba:

```sql
SOURCE /home/ubuntu/01_crear_tablas_innovatetech.sql;
```

```sql
SOURCE /home/ubuntu/02_datos_prueba_innovatetech.sql;
```

Se verificó que las tablas y los datos se hubieran creado correctamente:

```sql
SHOW TABLES;
```
<img width="359" height="497" alt="Captura de pantalla 2026-05-27 235408" src="https://github.com/user-attachments/assets/d85477ed-753b-4a2c-82bc-7984ca044cba" />

```sql
SELECT * FROM CLIENTES;
```
<img width="800" height="222" alt="Captura de pantalla 2026-05-27 235514" src="https://github.com/user-attachments/assets/2a27ddf2-7096-4373-bef8-3d767a6e7b65" />


---

## 5. Roles y permisos

Se crearon los roles del sistema:

- `admin`
- `vendes`
- `administracio`
- `treballador`

Ejemplo de creación de rol:

```sql
CREATE ROLE admin;
```

También se asignaron permisos específicos a cada rol utilizando `GRANT`.

Finalmente, se concedió el permiso especial `FILE` al usuario administrador para poder realizar exportaciones de archivos y backups:

```sql
GRANT FILE ON *.* TO 'ruben'@'localhost';
```

Se verificaron los permisos con:

```sql
SHOW GRANTS FOR 'ruben'@'localhost';
```

<img width="1009" height="216" alt="Captura de pantalla 2026-05-27 235927" src="https://github.com/user-attachments/assets/0ac07614-3905-4545-85a7-ade3e922a60a" />


---

## 6. Script automático de usuarios

Se creó un script Bash para automatizar la creación de usuarios y permisos.

Script utilizado:

```text
crear_usuarios.sh
```

Este script genera automáticamente un fichero SQL con las sentencias `CREATE USER` y `GRANT`.

También genera el fichero:

```text
crear_usuario.sql
```

<img width="385" height="169" alt="image" src="https://github.com/user-attachments/assets/7610df5a-ceb7-408a-82bd-242a4ba05bf2" />


---

## 7. Trigger de auditoría

Se implementó un trigger de auditoría para registrar cambios sobre la tabla `NOMINAS`.

Se verificó su funcionamiento realizando modificaciones de prueba.

```sql
SHOW TRIGGERS;
```

<img width="840" height="190" alt="image" src="https://github.com/user-attachments/assets/a668c5d2-bb24-4216-af3c-0a160bc281b9" />


---

## 8. Trigger de bloqueo de usuarios

Se implementó un trigger encargado de impedir llamadas cuando el usuario está bloqueado.

Se comprobó su funcionamiento con una prueba real de inserción en `VIDEOLLAMADAS`.

<img width="853" height="78" alt="image" src="https://github.com/user-attachments/assets/a559868c-5634-478a-bc1c-0394bbf90a54" />


---

## 9. Trigger de límite de llamadas diarias

Se implementó un trigger para controlar el número máximo de videollamadas permitidas por usuario y día.

Cuando se supera el límite, el usuario queda bloqueado automáticamente.

<img width="940" height="202" alt="image" src="https://github.com/user-attachments/assets/618be96a-39a3-44e1-b41a-e6bb3aa53601" />

<img width="653" height="150" alt="image" src="https://github.com/user-attachments/assets/7615cbe1-a867-40dc-ba6e-a64667de308c" />


---

## 10. Trigger de minutos mensuales

Se implementó un trigger para controlar el límite mensual de minutos consumidos en videollamadas.

<img width="1349" height="358" alt="image" src="https://github.com/user-attachments/assets/ae57cadc-f738-44e3-b4b8-c87b8a80a4d4" />


---

## 11. Sistema de auditoría

Se creó la tabla `AVISOS` para almacenar eventos importantes y acciones relevantes del sistema.

Comprobación de registros:

```sql
SELECT * FROM AVISOS;
```

<img width="1328" height="211" alt="image" src="https://github.com/user-attachments/assets/dd7dc44d-69ef-4235-a7f5-2f2b089728fa" />



---

## 12. Backups automáticos

Se configuró un sistema automático de backups utilizando eventos de MariaDB y un script Bash adicional para la compresión final.

El evento se ejecuta diariamente a las 02:00 y genera los ficheros CSV en:

```text
/var/lib/mysql/backups
```

Después, el script Bash comprime los archivos en ZIP y los guarda en:

```text
/home/ubuntu/backups
```

Script utilizado:

```text
04_event_backup.sql
backup_completo_innovatetech_db.sh
```

Se verifica el scheduler con:

```sql
SHOW EVENTS;
```

Y se comprueba la compresión automática con:

```bash
crontab -l
```

También se consulta la tabla:

```sql
SELECT * FROM CONTROL_BACKUPS;
```

<img width="1251" height="87" alt="image" src="https://github.com/user-attachments/assets/957ced57-e390-4d4c-877d-1c0fbcb64a2a" />


<img width="730" height="114" alt="image" src="https://github.com/user-attachments/assets/484ee31c-4f7a-40ce-81b0-4b282474dfd2" />


---

## 13. Scripts utilizados

| Script | Función |
|---|---|
| `01_crear_tablas_innovatetech.sql` | Creación de tablas |
| `02_datos_prueba_innovatetech.sql` | Inserción de datos de prueba |
| `crear_usuarios.sh` | Automatización de usuarios |
| `crear_usuario.sql` | SQL generado por el script de usuarios |
| `03_01_trigger_auditoria_nominas.sql` | Auditoría automática |
| `03_02_trigger_usuario_bloqueado.sql` | Bloqueo de usuarios |
| `03_03_trigger_limite_llamadas.sql` | Límite de llamadas diarias |
| `03_04_trigger_minutos_mensuales.sql` | Control de minutos mensuales |
| `04_event_backup.sql` | Evento automático de backups |
| `backup_completo_innovatetech_db.sh` | Compresión automática de backups |

---

## 14. Conclusión

La base de datos de InnovateTech se ha implementado correctamente utilizando MariaDB sobre AWS EC2.

Además de la gestión de datos, se han configurado sistemas automáticos de auditoría, control de permisos y backups automáticos para mejorar la seguridad y la administración de la infraestructura.
