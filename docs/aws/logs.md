# SERVIDOR DE LOGS (Elastic y Kibana)
> Servicio de centralización de logs desplegado en AWS EC2 con Elasticsearch, Kibana y Filebeat.
## Índice

- [1. Creación del usuario, asignación del password y permisos](#1-creación-del-usuario-asignación-del-password-y-permisos)
- [2. Instalación de Elasticsearch i Kibana](#2-instalación-de-elasticsearch-i-kibana)
- [3. Descomprimir los ficheros comprimidos y asignar los permisos](#3-descomprimir-los-ficheros-comprimidos-y-asignar-los-permisos)
- [4. Iniciar elastic](#4-iniciar-elastic)
- [5. Configuración de Kibana](#5-configuración-de-kibana)
- [6. Arrancamos kibana-setup](#6-arrancamos-kibana-setup)
- [7. Arrancamos Kibana](#7-arrancamos-kibana)
- [8. Probamos que se puede entrar a kibana y que registra logs](#8-probamos-que-se-puede-entrar-a-kibana-y-que-registra-logs)

### Centralización con Filebeat

- [1. Descargamos el repositorio de Filebeat](#1-descargamos-el-repositorio-de-filebeat)
- [2. Instalamos el Filebeat](#2-instalamos-el-filebeat)
- [3. Configuramos Filebeat y verificamos que la config está bien](#3-configuramos-filebeat-y-verificamos-que-la-config-está-bien)
- [4. Ejecutamos Filebeat Setup para que cargue los dashboards](#4-ejecutamos-filebeat-setup-para-que-cargue-los-dashboards)
- [5. Reiniciamos el servicio y comprobamos que registre los logs en kibana](#5-reiniciamos-el-servicio-y-comprobamos-que-registre-los-logs-en-kibana)
- [6. Prueba de que kibana recibe registra acciones de mi máquina](#6-prueba-de-que-kibana-recibe-registra-acciones-de-mi-máquina)
- [7. Implantación de Filebeat en las máquinas de nuestro equipo](#7-implantación-de-filebeat-en-las-máquinas-de-nuestro-equipo)
- [7.1 Antes de implantarlo en las máquinas de mis compañeros verifiqué que funcionaba correctamente en mis máquinas](#71-antes-de-implantarlo-en-las-máquinas-de-mis-compañeros-verifiqué-que-funcionaba-correctamente-en-mis-máquinas)
- [Conclusión](#conclusión)

## 1. Creación del usuario, asignación del password y permisos

Primero se crea el usuario `elastic`, se le asigna contraseña y se añaden permisos de sudo.

```bash
sudo useradd -m -s /bin/bash elastic
sudo passwd elastic
sudo usermod -aG sudo elastic
```
> <img width="509" height="135" alt="image" src="https://github.com/user-attachments/assets/b8d8e1ff-c89b-455b-84bc-dbd6c4e6a0ec" />


### Log de creación del usuario elastic en auth.log

> <img width="620" height="235" alt="image" src="https://github.com/user-attachments/assets/7cb2ae5b-23b8-49e5-b481-748584b9124d" />


```bash
sudo grep "elastic" /var/log/auth.log
```

---

## 2. Instalación de Elasticsearch i Kibana

Se descargan las versiones `.tar.gz` oficiales desde Elastic.

```bash
wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-8.17.4-linux-x86_64.tar.gz
wget https://artifacts.elastic.co/downloads/kibana/kibana-8.17.4-linux-x86_64.tar.gz
```

> <img width="637" height="365" alt="image" src="https://github.com/user-attachments/assets/3c22d305-fe92-4699-aa66-d86a9b6a39aa" />


---

## 3. Descomprimir los ficheros comprimidos y asignar los permisos

Se descomprimen los paquetes y se cambian los permisos al usuario `elastic`.

```bash
tar -xzf elasticsearch-8.17.4-linux-x86_64.tar.gz
tar -xzf kibana-8.17.4-linux-x86_64.tar.gz
sudo chown -R elastic:elastic elasticsearch-8.17.4 kibana-8.17.4
```

> <img width="749" height="246" alt="image" src="https://github.com/user-attachments/assets/aa85705a-73dc-4949-950c-66371ed16daf" />


---

## 4. Iniciar elastic

Se arranca Elasticsearch para que genere automáticamente la contraseña, el fingerprint del certificado y el enrollment token.

```bash
./bin/elasticsearch
```

> <img width="750" height="136" alt="image" src="https://github.com/user-attachments/assets/f9b514f4-98c5-4b40-a95d-e25bb1f690c9" />


> <img width="745" height="403" alt="image" src="https://github.com/user-attachments/assets/ca8c758e-482c-42e9-9c0e-c3676e302c23" />


### Per obtenir CA certificate SHA-256 fingerprint utilitzarem aquesta comanda:

```bash
openssl x509 -fingerprint -sha256 -in config/certs/http_ca.crt
```

> <img width="627" height="462" alt="image" src="https://github.com/user-attachments/assets/8c2581b7-ae3d-4c69-b436-75b6587e0267" />


---

## 5. Configuración de Kibana

En el fichero `config/kibana.yml` se pone la IP de escucha para que se pueda conectar desde fuera.

```yaml
server.host: "0.0.0.0"
```

> <img width="623" height="294" alt="image" src="https://github.com/user-attachments/assets/14fd4df1-39a0-4d18-a611-7674a0e17d48" />


---

## 6. Arrancamos kibana-setup

Antes de arrancarlo se regenera el token para poder entrar a Kibana.

```bash
./bin/elasticsearch-create-enrollment-token -s kibana
```

> <img width="744" height="94" alt="image" src="https://github.com/user-attachments/assets/f1b63904-ad32-43a6-8650-f927026c5991" />

Después se lanza el setup de Kibana.

```bash
./bin/kibana-setup
```

> <img width="746" height="196" alt="image" src="https://github.com/user-attachments/assets/47699984-5990-432f-8585-2d320b63efab" />


---

## 7. Arrancamos Kibana

```bash
./bin/kibana
```

> <img width="748" height="361" alt="image" src="https://github.com/user-attachments/assets/de6e77cd-c65c-4b14-8a96-2190d0177edf" />


---

## 8. Probamos que se puede entrar a kibana y que registra logs

Se accede desde el navegador a la interfaz web de Kibana.

```text
http://16.192.46.108:5601
```

Usuario:

```text
elastic
```

Password:

```text
_*6ZXwiE3Q2_QsG3e=P2
```

> <img width="573" height="618" alt="image" src="https://github.com/user-attachments/assets/e3b82e53-41b6-4ced-902b-dbe11c930c3b" />


> <img width="604" height="553" alt="image" src="https://github.com/user-attachments/assets/62efc1b8-6cad-4149-b290-128894dca93c" />


---

# SERVIDOR DE LOGS (Centralización con Filebeat)

## 1. Descargamos el repositorio de Filebeat

```bash
wget https://artifacts.elastic.co/downloads/beats/filebeat/filebeat-8.17.4-amd64.deb
```

> <img width="926" height="283" alt="image" src="https://github.com/user-attachments/assets/42d4fdab-c19e-4610-9768-7a7aa024598e" />


---

## 2. Instalamos el Filebeat

```bash
sudo dpkg -i filebeat-8.17.4-amd64.deb
```

> <img width="898" height="193" alt="image" src="https://github.com/user-attachments/assets/429a53d1-22aa-4c4e-878b-667b4a13baf6" />


---

## 3. Configuramos Filebeat y verificamos que la config está bien

### Archivo de configuración de filebeat

Se mantiene la configuración original del fichero y se modifica la salida hacia Elasticsearch.

```yaml
filebeat.inputs:
- type: filestream
  enabled: true
  paths:
    - /var/log/*.log

output.elasticsearch:
  hosts: ["https://16.192.46.108:9200"]
  username: "elastic"
  password: "_*6ZXwiE3Q2_QsG3e=P2"
  ssl:
    verification_mode: none
```

> <img width="776" height="580" alt="image" src="https://github.com/user-attachments/assets/893cc517-533d-4255-971f-8a753e5a51bf" />


### Y también habilitó la lectura de logs locales del sistema Linux mediante el filestream:

> <img width="761" height="580" alt="image" src="https://github.com/user-attachments/assets/fea37877-f2c5-4951-bf60-9fc436218a47" />


### Y finalmente verificamos con `filebeat test output` que la configuración es correcta:

```bash
sudo filebeat test output
```

> <img width="764" height="353" alt="image" src="https://github.com/user-attachments/assets/ce157a42-bfd9-4f0b-83dc-c3fcd7916f7a" />


---

## 4. Ejecutamos Filebeat Setup para que cargue los dashboards

```bash
sudo filebeat setup
```

> <img width="1043" height="174" alt="image" src="https://github.com/user-attachments/assets/a39af6de-76e8-4180-9ac1-9d90fb9c9045" />


---

## 5. Reiniciamos el servicio y comprobamos que registre los logs en kibana

```bash
sudo systemctl enable filebeat
sudo systemctl restart filebeat
sudo systemctl status filebeat
```

> <img width="932" height="561" alt="image" src="https://github.com/user-attachments/assets/4bf02854-8f4b-49fd-8284-6412c876fcf2" />


---

## 6. Prueba de que kibana recibe registra acciones de mi máquina

Se creó un usuario de prueba y se cambió la contraseña para comprobar que los eventos aparecían en Kibana.

```bash
sudo useradd -m -s /bin/bash hola
sudo passwd hola
```

> <img width="815" height="159" alt="image" src="https://github.com/user-attachments/assets/eaa4727b-5fc7-424e-ad7e-f3f770a9aaa0" />


> <img width="1150" height="478" alt="image" src="https://github.com/user-attachments/assets/4c9d2b56-d97f-4a53-a944-35a74bb18f75" />


---

## 7. Implantación de Filebeat en las máquinas de nuestro equipo

Se implantó Filebeat en las máquinas del equipo:
- Servidor Logs
- Servidor BDD
- Servidor Jitsi

Como hacerlo manualmente era demasiado tardado, se utilizó un script para automatizar la instalación y configuración.

> <img width="787" height="674" alt="image" src="https://github.com/user-attachments/assets/3edcc864-da3d-47d6-a3af-b4679cdaf1a0" />


---

## 7.1 Antes de implantarlo en las máquinas de mis compañeros verifiqué que funcionaba correctamente en mis máquinas

La primera prueba se realizó en el servidor BDD.  
Se creó un usuario nuevo, se cambió la contraseña y se eliminó, comprobando que los eventos aparecían en Kibana.

> <img width="691" height="235" alt="image" src="https://github.com/user-attachments/assets/47e9cdc9-c543-495d-ba21-83c5959f1a60" />


> <img width="791" height="363" alt="image" src="https://github.com/user-attachments/assets/3f2bbe94-671c-4aec-ad4f-1d2412cfa71a" />


Después también se probó en la máquina de Jitsi creando carpetas y ficheros.

> <img width="605" height="48" alt="image" src="https://github.com/user-attachments/assets/a15baf66-017f-43a2-aeec-67fbfda987c9" />


> <img width="791" height="234" alt="image" src="https://github.com/user-attachments/assets/8aef3e4a-5d6e-4469-a475-707c2d1d1023" />


---

## Conclusión

Al final se consiguió montar un servidor de logs centralizado utilizando Elasticsearch, Kibana y Filebeat. Gracias a esto, los logs de las diferentes máquinas del proyecto se pueden ver desde un mismo sitio en Kibana, haciendo más fácil controlar y revisar lo que ocurre en los servidores.
