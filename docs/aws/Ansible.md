# Automatización con Ansible — InnovateTech
**Proyecto Transversal 25/26**
**Responsable:** Gerard Romo Sánchez
**Módulo:** 0371 — Fundamentos de Hardware (Implementación AWS)

---

## Índice
 
1. [Introducción](#1-introducción)
2. [Instalación de Ansible](#2-instalación-de-ansible)
3. [Gestión de claves SSH](#3-gestión-de-claves-ssh)
4. [Fichero de inventario](#4-fichero-de-inventario-inventoryini)
5. [Verificación de conectividad](#5-verificación-de-conectividad)
6. [Playbooks de configuración](#6-playbooks-de-configuración)
7. [Verificación general de los servicios](#7-verificación-general-de-los-servicios)
8. [Resumen de puertos y servicios](#8-resumen-de-puertos-y-servicios)


---

## 1. Introducción

Para automatizar la gestión de los servidores de InnovateTech se ha utilizado **Ansible**, una herramienta de automatización de infraestructura que permite gestionar múltiples servidores de forma centralizada sin necesidad de instalar ningún agente en las máquinas remotas. Ansible se conecta directamente por SSH utilizando claves públicas/privadas, sin contraseña, tal como establece el enunciado del proyecto.

Las máquinas gestionadas son las siguientes, todas desplegadas en la misma región AWS **eu-north-1 (Estocolmo)**:

| Nombre | Función | IP pública | Región AWS |
|---|---|---|---|
| web-sftp | Servidor Web + SFTP | 13.60.253.209 | eu-north-1 (Estocolmo) |
| server-audio | Servidor de Audio (Icecast2) | 13.62.46.183 | eu-north-1 (Estocolmo) |

La máquina de control desde la que se ejecuta Ansible es el servidor propio del grupo, desplegado también en AWS con IP **16.171.115.99**.

---

## 2. Instalación de Ansible

Ansible se ha instalado en el servidor de control mediante los repositorios oficiales de Ubuntu, sin usar `pip`, para garantizar la integración correcta con el sistema:

```bash
sudo apt update
sudo apt install -y ansible python3-boto3 awscli
```

La versión instalada es **Ansible core 2.16.3**.

<img width="1035" height="198" alt="image" src="https://github.com/user-attachments/assets/8ce92770-ca5e-409d-b75e-660c8e48142f" />

---

## 3. Gestión de claves SSH

El proyecto utiliza **dos claves SSH diferenciadas** según su función:

### 3.1 Clave AWS — `Samba_AD.pem`

Clave de tipo RSA proporcionada por uno de mis compañeros en el momento de creación de las instancias EC2. Se utiliza para acceder inicialmente a las máquinas como usuario `ubuntu` (usuario por defecto de las AMIs de Ubuntu en AWS). Se encuentra en el directorio `~/.ssh/Samba_AD.pem` del servidor de control.

```
~/.ssh/Samba_AD.pem   → Clave privada AWS (acceso inicial como ubuntu)
```

### 3.2 Clave de administrador — `sysadmin_innovatetech`

Clave de tipo **ed25519** generada específicamente para el usuario administrador `sysadmin` que crean los playbooks. Ansible la añade automáticamente al fichero `~/.ssh/authorized_keys` del nuevo usuario durante la ejecución del playbook para permitir que Ansible se conecte sin contraseña.

```bash
# Generación de la clave (ejecutar en el servidor de control antes del playbook)
ssh-keygen -t ed25519 -C "sysadmin@innovatetech" -f ~/.ssh/sysadmin_innovatetech
cat ~/.ssh/sysadmin_innovatetech.pub
# → Copiar la salida y pegarla en la variable admin_pubkey de los playbooks
```

```
~/.ssh/sysadmin_innovatetech      → Clave privada (solo en el servidor de control)
~/.ssh/sysadmin_innovatetech.pub  → Clave pública (desplegada en los servidores vía Ansible)
```
<img width="955" height="56" alt="image" src="https://github.com/user-attachments/assets/16e018e2-a073-4bce-bfe5-b56b7baec371" />

---

## 4. Fichero de inventario (`inventory-conf.ini`)

El inventario define los servidores gestionados y los parámetros de conexión comunes:

```ini
[web]
web-sftp-p  ansible_host=[ip de la maquina creada]

[audio]
server-audio-p  ansible_host=[ip de la maquina creada]

[all:vars]
ansible_user=ubuntu
ansible_ssh_private_key_file=~/.ssh/Samba_AD.pem
# Evita el prompt de fingerprint la primera vez
ansible_ssh_common_args='-o StrictHostKeyChecking=no'
# Python en el servidor remoto
ansible_python_interpreter=/usr/bin/python3
# Usuario administrador que creará Ansible (no ubuntu, no root)
admin_user=sysadmin
```

<img width="573" height="363" alt="image" src="https://github.com/user-attachments/assets/e68ea01f-6ff6-467d-ac3c-1cc261a5df58" />

---

## 5. Verificación de conectividad

Antes de ejecutar ningún playbook, se verifica que Ansible puede alcanzar correctamente todos los servidores mediante el módulo `ping`:

```bash
ansible -i inventory.ini all -m ping
```

El resultado esperado es `"ping": "pong"` para cada host, confirmando que la conexión SSH con clave funciona correctamente.

<img width="560" height="140" alt="image" src="https://github.com/user-attachments/assets/d00c3578-08b8-4484-8775-5c1e1e0be2d4" />

---

## 6. Playbooks de configuración

Se han creado dos playbooks específicos: uno para el servidor web + SFTP y otro para el servidor de audio. Cada playbook es completamente autónomo y se ejecuta de forma independiente sobre su servidor correspondiente.

### 6.1 Servidor Web + SFTP — `02_web_sftp.yml`

```bash
ansible-playbook -i inventory.ini 02_web_sftp.yml
```

Este playbook configura desde cero el servidor web de InnovateTech. Primero actualiza el sistema e instala todos los paquetes necesarios (Nginx, OpenSSH, UFW, fail2ban, entre otros). A continuación, crea el usuario administrador `sysadmin` con shell bash, le añade la clave pública SSH y le otorga privilegios `sudo` sin contraseña, de modo que Ansible pueda gestionar el servidor en el futuro sin depender del usuario `ubuntu` de AWS.

Seguidamente, endurece la configuración SSH del servidor: desactiva la autenticación por contraseña, prohíbe el login directo como `root`, limita los intentos de autenticación a 3 y añade un banner de advertencia (MOTD) que se muestra en cada conexión. Activa el firewall UFW permitiendo únicamente los puertos 22 (SSH), 80 (HTTP) y 443 (HTTPS), y configura fail2ban para bloquear automáticamente las IPs que hagan más de 3 intentos fallidos de conexión SSH.

En cuanto a los servicios web, despliega Nginx con la página corporativa de InnovateTech y configura el bloque de servidor para servir el contenido desde `/var/www/html`. Finalmente, crea los usuarios SFTP (`usuario1` y `usuario2`) en un grupo dedicado y los confina dentro de un entorno chroot jail en `/srv/sftp/<usuario>/upload`, de modo que no pueden salir de su directorio ni acceder a la shell del sistema.

<details>
<summary>📄 Ver el código completo del playbook</summary>

```yaml
---
- name: "InnovateTech · Servidor Web + SFTP"
  hosts: web-sftp-p
  become: yes

  vars:
    admin_user: "sysadmin"
    admin_pubkey: "ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIHsP+dIyOcZoxZosYtURuHjAfCj0FT7UShIb3X1amZi2 sysadmin@innovatetech"
    sftp_group: "sftpusers"
    sftp_users:
      - usuario1
      - usuario2

  tasks:

    # BLOQUE 1 – Sistema base
    - name: "Actualizar lista de paquetes"
      apt:
        update_cache: yes
        cache_valid_time: 3600

    - name: "Actualizar todos los paquetes del sistema"
      apt:
        upgrade: dist
        autoremove: yes

    - name: "Instalar paquetes básicos"
      apt:
        name:
          - curl
          - wget
          - git
          - vim
          - htop
          - ufw
          - fail2ban
          - rsyslog
          - logrotate
          - unattended-upgrades
          - python3
          - nginx
          - openssh-server
        state: present

    # BLOQUE 2 – Usuario administrador
    - name: "Crear grupo '{{ admin_user }}'"
      group:
        name: "{{ admin_user }}"
        state: present

    - name: "Crear usuario '{{ admin_user }}'"
      user:
        name: "{{ admin_user }}"
        group: "{{ admin_user }}"
        groups: sudo
        shell: /bin/bash
        create_home: yes
        state: present

    - name: "Crear directorio .ssh para '{{ admin_user }}'"
      file:
        path: "/home/{{ admin_user }}/.ssh"
        state: directory
        owner: "{{ admin_user }}"
        group: "{{ admin_user }}"
        mode: "0700"

    - name: "Añadir clave pública SSH a '{{ admin_user }}'"
      authorized_key:
        user: "{{ admin_user }}"
        key: "{{ admin_pubkey }}"
        state: present

    - name: "Permitir sudo sin contraseña para '{{ admin_user }}'"
      copy:
        dest: "/etc/sudoers.d/{{ admin_user }}"
        content: "{{ admin_user }} ALL=(ALL) NOPASSWD:ALL\n"
        mode: "0440"
        validate: "visudo -cf %s"

    # BLOQUE 3 – SSH seguro
    - name: "SSH – desactivar autenticación por contraseña"
      lineinfile:
        path: /etc/ssh/sshd_config
        regexp: "^#?PasswordAuthentication"
        line: "PasswordAuthentication no"

    - name: "SSH – desactivar login de root"
      lineinfile:
        path: /etc/ssh/sshd_config
        regexp: "^#?PermitRootLogin"
        line: "PermitRootLogin no"

    - name: "SSH – limitar intentos de autenticación"
      lineinfile:
        path: /etc/ssh/sshd_config
        regexp: "^#?MaxAuthTries"
        line: "MaxAuthTries 3"

    - name: "SSH – reducir tiempo de login"
      lineinfile:
        path: /etc/ssh/sshd_config
        regexp: "^#?LoginGraceTime"
        line: "LoginGraceTime 30"

    - name: "Crear banner MOTD"
      copy:
        dest: /etc/motd
        content: |
          ╔══════════════════════════════════════════════════════╗
          ║        InnovateTech – Servidor Web + SFTP            ║
          ║      Acceso restringido a personal autorizado        ║
          ║  Toda actividad es monitorizada y registrada         ║
          ╚══════════════════════════════════════════════════════╝

    - name: "Reiniciar SSH"
      service:
        name: ssh
        state: restarted
        enabled: yes

    # BLOQUE 4 – Firewall UFW
    - name: "UFW – denegar todo el tráfico entrante"
      ufw:
        default: deny
        direction: incoming

    - name: "UFW – permitir SSH (22)"
      ufw:
        rule: allow
        port: "22"
        proto: tcp

    - name: "UFW – permitir HTTP (80)"
      ufw:
        rule: allow
        port: "80"
        proto: tcp

    - name: "UFW – permitir HTTPS (443)"
      ufw:
        rule: allow
        port: "443"
        proto: tcp

    - name: "UFW – activar firewall"
      ufw:
        state: enabled

    # BLOQUE 5 – Fail2ban
    - name: "Configurar fail2ban"
      copy:
        dest: /etc/fail2ban/jail.local
        content: |
          [DEFAULT]
          bantime  = 3600
          findtime = 600
          maxretry = 5
          backend  = systemd

          [sshd]
          enabled  = true
          port     = ssh
          logpath  = %(sshd_log)s
          maxretry = 3

    - name: "Activar fail2ban"
      service:
        name: fail2ban
        state: started
        enabled: yes

    # BLOQUE 6 – Nginx
    - name: "Crear directorio web"
      file:
        path: /var/www/html
        state: directory
        owner: www-data
        group: www-data
        mode: "0755"

    - name: "Página de inicio InnovateTech"
      copy:
        dest: /var/www/html/index.html
        content: |
          <!DOCTYPE html>
          <html lang="ca">
          <head>
            <meta charset="UTF-8">
            <title>InnovateTech Solutions</title>
          </head>
          <body>
            <h1>InnovateTech</h1>
            <p>Infraestructura Cloud Empresarial · Projecte Transversal ASIXc1</p>
          </body>
          </html>

    - name: "Configurar bloque de servidor Nginx"
      copy:
        dest: /etc/nginx/sites-available/default
        content: |
          server {
              listen 80 default_server;
              listen [::]:80 default_server;
              root /var/www/html;
              index index.html;
              server_name _;
              location / {
                  try_files $uri $uri/ =404;
              }
          }

    - name: "Activar e iniciar Nginx"
      service:
        name: nginx
        state: restarted
        enabled: yes

    # BLOQUE 7 – SFTP con chroot jail
    - name: "Crear grupo {{ sftp_group }}"
      group:
        name: "{{ sftp_group }}"
        state: present

    - name: "Crear usuarios SFTP"
      user:
        name: "{{ item }}"
        group: "{{ sftp_group }}"
        shell: /usr/sbin/nologin
        create_home: no
        state: present
      loop: "{{ sftp_users }}"

    - name: "Crear directorios chroot"
      file:
        path: "/srv/sftp/{{ item }}"
        state: directory
        owner: root
        group: root
        mode: "0755"
      loop: "{{ sftp_users }}"

    - name: "Crear subdirectorio upload para cada usuario"
      file:
        path: "/srv/sftp/{{ item }}/upload"
        state: directory
        owner: "{{ item }}"
        group: "{{ sftp_group }}"
        mode: "0750"
      loop: "{{ sftp_users }}"

    - name: "Configurar SFTP chroot en sshd_config"
      blockinfile:
        path: /etc/ssh/sshd_config
        marker: "# {mark} ANSIBLE – SFTP CHROOT"
        block: |
          Match Group {{ sftp_group }}
              ChrootDirectory /srv/sftp/%u
              ForceCommand internal-sftp
              PasswordAuthentication no
              AllowTcpForwarding no
              X11Forwarding no

    - name: "Reiniciar SSH para aplicar SFTP"
      service:
        name: ssh
        state: restarted
```

</details>

**Resultado de la ejecución:**

<img width="890" height="433" alt="image" src="https://github.com/user-attachments/assets/3e74932c-714c-4b26-8c81-9830e06c36a5" />

---

### 6.2 Servidor de Audio — `03_audio.yml`

```bash
ansible-playbook -i inventory.ini 03_audio.yml
```

Este playbook configura el servidor de audio corporativo de InnovateTech. Al igual que el playbook anterior, comienza actualizando el sistema e instalando los paquetes base, crea el usuario `sysadmin` con su clave pública y aplica la misma configuración de seguridad SSH (sin acceso por contraseña, sin login de root, banner MOTD). El firewall UFW se activa permitiendo únicamente el puerto 22 (SSH) y el puerto **8000** (Icecast2), y fail2ban protege el servicio SSH de fuerza bruta.

La parte principal de este playbook es la instalación y configuración de **Icecast2**, el servidor de streaming de audio. Se despliega el fichero `icecast.xml` con los parámetros del proyecto: hostname dinámico tomado del inventario, límite de 100 clientes simultáneos, 2 fuentes de entrada máximo, y las credenciales de autenticación para la fuente (`ITB2026`), el relay (`ITB2026relay`) y el administrador (`ITB2026admin`). El servicio se habilita e inicia automáticamente.

A continuación se instala **Liquidsoap**, que actúa como cliente fuente de Icecast2: lee los ficheros MP3 del directorio `/srv/audio/musica`, los mezcla de forma aleatoria y los emite en formato MP3 a 128 Kbps hacia el mountpoint `/stream` de Icecast2. Todo ello se encapsula en un servicio **systemd** (`liquidsoap-radio`) que arranca automáticamente con el sistema y se reinicia solo si falla. Finalmente, el playbook verifica que Icecast2 escucha correctamente en el puerto 8000 y muestra un resumen con todas las URLs del servicio.

Para añadir música al servidor y que Liquidsoap la pueda emitir:

```bash
scp -i ~/.ssh/sysadmin_innovatetech.pem arch.mp3 sysadmin@13.62.46.183:/srv/audio/musica/
siendo arch.mp3 cualquier cancion que se desee.
```

<details>
<summary>📄 Ver el código completo del playbook</summary>

```yaml
---
- name: "InnovateTech · Servidor de audio (Icecast2 + Liquidsoap)"
  hosts: server-audio-p
  become: yes

  vars:
    admin_user: "sysadmin"
    admin_pubkey: "ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIHsP+dIyOcZoxZosYtURuHjAfCj0FT7UShIb3X1amZi2 sysadmin@innovatetech"
    icecast_source_password: "ITB2026"
    icecast_relay_password:  "ITB2026relay"
    icecast_admin_password:  "ITB2026admin"
    audio_dir: "/srv/audio/musica"

  tasks:

    # BLOQUE 1 – Sistema base
    - name: "Actualizar lista de paquetes"
      apt:
        update_cache: yes
        cache_valid_time: 3600

    - name: "Instalar paquetes básicos"
      apt:
        name:
          - curl
          - wget
          - ufw
          - fail2ban
          - python3
          - openssh-server
        state: present

    # BLOQUE 2 – Usuario administrador (idéntico al playbook web)
    - name: "Crear usuario '{{ admin_user }}'"
      user:
        name: "{{ admin_user }}"
        groups: sudo
        shell: /bin/bash
        create_home: yes
        state: present

    - name: "Añadir clave pública SSH a '{{ admin_user }}'"
      authorized_key:
        user: "{{ admin_user }}"
        key: "{{ admin_pubkey }}"
        state: present

    - name: "Permitir sudo sin contraseña"
      copy:
        dest: "/etc/sudoers.d/{{ admin_user }}"
        content: "{{ admin_user }} ALL=(ALL) NOPASSWD:ALL\n"
        mode: "0440"
        validate: "visudo -cf %s"

    # BLOQUE 3 – SSH seguro
    - name: "SSH – desactivar autenticación por contraseña"
      lineinfile:
        path: /etc/ssh/sshd_config
        regexp: "^#?PasswordAuthentication"
        line: "PasswordAuthentication no"

    - name: "SSH – desactivar login de root"
      lineinfile:
        path: /etc/ssh/sshd_config
        regexp: "^#?PermitRootLogin"
        line: "PermitRootLogin no"

    - name: "Crear banner MOTD"
      copy:
        dest: /etc/motd
        content: |
          ╔══════════════════════════════════════════════════════╗
          ║       InnovateTech – Servidor de Audio (Icecast2)    ║
          ║      Acceso restringido a personal autorizado        ║
          ╚══════════════════════════════════════════════════════╝

    - name: "Reiniciar SSH"
      service:
        name: ssh
        state: restarted
        enabled: yes

    # BLOQUE 4 – Firewall UFW
    - name: "UFW – denegar todo el tráfico entrante"
      ufw:
        default: deny
        direction: incoming

    - name: "UFW – permitir SSH (22)"
      ufw:
        rule: allow
        port: "22"
        proto: tcp

    - name: "UFW – permitir Icecast2 (8000)"
      ufw:
        rule: allow
        port: "8000"
        proto: tcp

    - name: "UFW – activar firewall"
      ufw:
        state: enabled

    # BLOQUE 5 – Fail2ban
    - name: "Configurar y activar fail2ban"
      copy:
        dest: /etc/fail2ban/jail.local
        content: |
          [DEFAULT]
          bantime  = 3600
          findtime = 600
          maxretry = 5

          [sshd]
          enabled  = true
          maxretry = 3

    - name: "Activar fail2ban"
      service:
        name: fail2ban
        state: started
        enabled: yes

    # BLOQUE 6 – Icecast2
    - name: "Instalar Icecast2"
      apt:
        name: icecast2
        state: present
      environment:
        DEBIAN_FRONTEND: noninteractive

    - name: "Crear directorio de música"
      file:
        path: "{{ audio_dir }}"
        state: directory
        owner: icecast2
        group: icecast
        mode: "0775"

    - name: "Configurar icecast.xml"
      copy:
        dest: /etc/icecast2/icecast.xml
        owner: icecast2
        group: icecast
        mode: "0640"
        content: |
          <icecast>
            <location>Catalunya, Spain</location>
            <admin>admin@innovatetech.local</admin>
            <limits>
              <clients>100</clients>
              <sources>2</sources>
            </limits>
            <authentication>
              <source-password>{{ icecast_source_password }}</source-password>
              <relay-password>{{ icecast_relay_password }}</relay-password>
              <admin-user>admin</admin-user>
              <admin-password>{{ icecast_admin_password }}</admin-password>
            </authentication>
            <hostname>{{ ansible_host }}</hostname>
            <listen-socket>
              <port>8000</port>
            </listen-socket>
            <paths>
              <basedir>/usr/share/icecast2</basedir>
              <logdir>/var/log/icecast2</logdir>
              <webroot>/usr/share/icecast2/web</webroot>
              <adminroot>/usr/share/icecast2/admin</adminroot>
            </paths>
            <logging>
              <accesslog>access.log</accesslog>
              <errorlog>error.log</errorlog>
              <loglevel>3</loglevel>
            </logging>
          </icecast>

    - name: "Habilitar Icecast2"
      lineinfile:
        path: /etc/default/icecast2
        regexp: "^ENABLE="
        line: 'ENABLE=true'

    - name: "Iniciar Icecast2"
      service:
        name: icecast2
        state: started
        enabled: yes

    # BLOQUE 7 – Liquidsoap
    - name: "Instalar Liquidsoap"
      apt:
        name: liquidsoap
        state: present

    - name: "Crear script radio.liq"
      copy:
        dest: /home/icecast2/radio.liq
        owner: icecast2
        group: icecast
        mode: "0750"
        content: |
          set("log.stdout", true)
          set("log.level", 3)

          radio = mksafe(
            playlist(
              mode   = "randomize",
              reload = 3600,
              "{{ audio_dir }}"
            )
          )

          output.icecast(
            %mp3(bitrate = 128),
            host     = "localhost",
            port     = 8000,
            password = "{{ icecast_source_password }}",
            mount    = "/stream",
            name     = "InnovateTech Radio",
            radio
          )

    - name: "Crear servicio systemd para Liquidsoap"
      copy:
        dest: /etc/systemd/system/liquidsoap-radio.service
        mode: "0644"
        content: |
          [Unit]
          Description=InnovateTech Radio – Liquidsoap
          After=network.target icecast2.service
          Requires=icecast2.service

          [Service]
          Type=simple
          User=icecast2
          Group=icecast
          ExecStart=/usr/bin/liquidsoap /home/icecast2/radio.liq
          Restart=on-failure
          RestartSec=15

          [Install]
          WantedBy=multi-user.target

    - name: "Recargar systemd e iniciar Liquidsoap"
      systemd:
        daemon_reload: yes

    - name: "Activar Liquidsoap"
      service:
        name: liquidsoap-radio
        state: started
        enabled: yes

    # VERIFICACIÓN
    - name: "Comprobar puerto 8000"
      wait_for:
        port: 8000
        host: localhost
        delay: 5
        timeout: 30
      ignore_errors: yes
```

</details>

**URLs del servicio una vez desplegado:**

| Recurso | URL |
|---|---|
| Panel de estado Icecast2 | `http://13.62.46.183:8000` |
| Panel de administración | `http://13.62.46.183:8000/admin` |
| Stream de audio en directo | `http://13.62.46.183:8000/stream` |

**Resultado de la ejecución:**

<img width="1897" height="349" alt="image" src="https://github.com/user-attachments/assets/c3799527-083d-4a24-8830-4d3fa1dff8f9" />

<img width="1322" height="803" alt="image" src="https://github.com/user-attachments/assets/92ef1a2a-1d97-4182-b287-449fd20f4483" />

<img width="1265" height="715" alt="image" src="https://github.com/user-attachments/assets/d1cda1df-6268-4f8b-8541-6f835135419e" />

---

## 7. Verificación general de los servicios

Una vez ejecutados los dos playbooks, se puede verificar el estado de todos los servicios con los siguientes comandos desde cada servidor:

**Servidor Web + SFTP:**
```bash
sudo systemctl status nginx
sudo systemctl status ssh
sudo ufw status
sudo fail2ban-client status sshd
```

**Servidor de Audio:**
```bash
sudo systemctl status icecast2
sudo systemctl status liquidsoap-radio
sudo ufw status
curl -s http://localhost:8000/status.xsl | head -20
```

<img width="960" height="253" alt="image" src="https://github.com/user-attachments/assets/3d2f08de-c479-4934-b531-9506652aee33" />

<img width="722" height="222" alt="image" src="https://github.com/user-attachments/assets/eff2731f-4943-4e09-8190-de5da78ef986" />

---

## 8. Resumen de puertos y servicios

| Servidor | Servicio | Puerto | Estado |
|---|---|---|---|
| web-sftp (13.60.253.209) | Nginx (HTTP) | 80 | ✅ Activo |
| web-sftp (13.60.253.209) | SSH / SFTP | 22 | ✅ Activo |
| server-audio (13.62.46.183) | Icecast2 | 8000 | ✅ Activo |
| server-audio (13.62.46.183) | SSH | 22 | ✅ Activo |
