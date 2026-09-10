# RHCSA — Red Hat Certified System Administrator

## EX200 — Guía de certificación

### 1. Qué es

La **RHCSA (Red Hat Certified System Administrator)** es la certificación de administración de sistemas Linux de Red Hat. El examen asociado es **EX200** y evalúa principalmente capacidad práctica sobre **Red Hat Enterprise Linux (RHEL)**.

La competencia central es:

```text
Configurar → Verificar → Hacer persistente → Reiniciar → Verificar de nuevo
```

No basta con conocer comandos: hay que saber administrar y diagnosticar un sistema.

---

## 2. Posición en la ruta Red Hat

```text
RHCSA
EX200
  │
  ▼
RHCE
EX294
  │
  ▼
RHCA
```

- **RHCSA:** Linux administration.
- **RHCE:** Linux + automatización con Ansible.
- **RHCA:** especialización avanzada.

---

## 3. Qué debes dominar

### Shell y command line

```bash
pwd ls cd cp mv rm mkdir touch file stat
find cat less head tail grep sort uniq cut awk sed wc
```

Dominar:

```bash
>
>>
<
2>
2>&1
|
```

Objetivo: trabajar eficazmente sin GUI.

### Files, permissions y ownership

```bash
chmod
chown
chgrp
umask
getfacl
setfacl
```

Dominar:

- permisos `rwx`
- SUID
- SGID
- sticky bit
- ACL
- ownership

### Users y groups

```bash
useradd usermod userdel
groupadd groupmod groupdel
passwd chage id groups
```

Archivos:

```text
/etc/passwd
/etc/shadow
/etc/group
/etc/gshadow
```

### sudo

```text
/etc/sudoers
/etc/sudoers.d/
```

Editar con:

```bash
visudo
```

---

## 4. Software

### DNF

```bash
dnf search PACKAGE
dnf info PACKAGE
dnf install PACKAGE
dnf remove PACKAGE
dnf update
dnf repolist
```

### RPM

```bash
rpm -q PACKAGE
rpm -qa
rpm -ql PACKAGE
rpm -qf /path/to/file
```

Entender:

```text
Repository → Package → Dependencies → DNF → Installed system
```

---

## 5. systemd

```bash
systemctl status SERVICE
systemctl start SERVICE
systemctl stop SERVICE
systemctl restart SERVICE
systemctl enable SERVICE
systemctl disable SERVICE
```

Muy importante:

```bash
systemctl enable --now SERVICE
```

Diferencia:

```text
start  = ahora
enable = durante el boot
```

Investigación:

```bash
systemctl status SERVICE
systemctl list-units
systemctl list-unit-files
journalctl -u SERVICE
```

---

## 6. Processes

```bash
ps
top
pgrep
pkill
kill
killall
jobs
bg
fg
```

Señales fundamentales:

```text
SIGTERM
SIGKILL
SIGHUP
```

Preferir terminación limpia antes de `SIGKILL`.

---

## 7. Storage

**Área crítica.**

Modelo:

```text
Disk
 ↓
Partition
 ↓
PV
 ↓
VG
 ↓
LV
 ↓
Filesystem
 ↓
Mount point
```

Identificación:

```bash
lsblk
blkid
fdisk -l
```

Particiones:

```bash
fdisk
parted
```

LVM:

```bash
pvcreate
pvs
vgcreate
vgs
lvcreate
lvs
```

Filesystems:

```bash
mkfs.xfs
mkfs.ext4
```

Mount:

```bash
mount
umount
df -h
du -sh
```

Persistencia:

```text
/etc/fstab
```

Verificación:

```bash
mount -a
reboot
df -h
```

Debes saber crear, montar, persistir, ampliar y diagnosticar storage.

---

## 8. Networking

Herramientas:

```bash
ip
nmcli
ss
ping
hostnamectl
```

NetworkManager:

```bash
nmcli device status
nmcli connection show
nmcli connection show NAME
nmcli connection modify ...
nmcli connection up ...
```

Debes entender:

```text
IP
Subnet
Gateway
DNS
Routing
Hostname
Sockets
```

### Troubleshooting

```text
¿Tengo IP?
    ↓
ip addr

¿Tengo ruta?
    ↓
ip route

¿Llego al gateway?
    ↓
ping GATEWAY

¿Resuelvo DNS?
    ↓
getent hosts example.com

¿Escucha el servicio?
    ↓
ss -lntup
```

---

## 9. SSH

Servicio:

```bash
systemctl status sshd
```

Cliente:

```bash
ssh user@server
```

Configuración:

```text
/etc/ssh/sshd_config
```

Claves:

```text
Cliente
 ├── private key
 └── public key
          ↓
Servidor
 └── ~/.ssh/authorized_keys
```

Permisos habituales:

```bash
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys
```

---

## 10. SELinux

**Área crítica.**

Estado:

```bash
getenforce
sestatus
setenforce
```

Estados:

```text
Enforcing
Permissive
Disabled
```

Contextos:

```bash
ls -Z
ps -Z
```

Restauración:

```bash
restorecon -Rv /data
```

Configuración persistente:

```bash
semanage fcontext
restorecon
```

Patrón:

```bash
semanage fcontext -a -t TYPE "/data(/.*)?"
restorecon -Rv /data
```

No tratar `chcon` como sustituto automático de una configuración persistente.

Modelo mental:

```text
Process
 ↓
Resource
 ↓
SELinux context
 ↓
Policy
 ↓
Allow / Deny
```

---

## 11. firewalld

```bash
firewall-cmd --state
firewall-cmd --get-active-zones
firewall-cmd --list-all
```

Ejemplo:

```bash
firewall-cmd --permanent --add-service=http
firewall-cmd --reload
```

Conceptos:

```text
zone
service
port
runtime
permanent
```

Siempre verificar después del cambio.

---

## 12. Logs

Principalmente:

```bash
journalctl
journalctl -b
journalctl -u SERVICE
journalctl -p err
journalctl --since today
```

Workflow:

```text
Problema
 ↓
¿Cuándo?
 ↓
¿Qué servicio?
 ↓
systemctl status
 ↓
journalctl
 ↓
¿Network?
 ↓
¿firewalld?
 ↓
¿SELinux?
```

---

## 13. Scheduling

### cron

```bash
crontab -e
crontab -l
```

Dominar:

```text
minute
hour
day
month
weekday
```

### systemd timers

```bash
systemctl list-timers
```

---

## 14. Boot y recovery

Modelo:

```text
Firmware
 ↓
Bootloader
 ↓
Kernel
 ↓
initramfs
 ↓
systemd
 ↓
target
 ↓
services
```

Comandos:

```bash
systemctl get-default
systemctl set-default
journalctl -b
```

Debes poder investigar problemas de arranque y servicios.

---

## 15. Containers

Herramienta:

```bash
podman
```

Conceptos:

```text
image
container
registry
volume
port
```

Comandos:

```bash
podman images
podman ps
podman pull
podman run
podman stop
podman rm
```

El objetivo es administración básica de containers, no Kubernetes.

---

# 16. Troubleshooting profesional

## Servicio

```bash
systemctl status SERVICE
journalctl -u SERVICE
ss -lntup
```

## Red

```bash
ip addr
ip route
nmcli device status
ping GATEWAY
getent hosts example.com
```

## Acceso a archivos

```bash
ls -l FILE
getfacl FILE
ls -Z FILE
id USER
```

Pensar siempre en:

```text
Linux permissions
+
ACL
+
SELinux
```

## Servicio inaccesible

```text
1. ¿Existe el proceso?
2. ¿Está funcionando systemd?
3. ¿Está escuchando?
4. ¿La red funciona?
5. ¿firewalld permite el tráfico?
6. ¿SELinux lo permite?
7. ¿La configuración es correcta?
```

---

# 17. Método de preparación

## Fase 1 — Fundamentals

```text
Shell
 ↓
Files
 ↓
Permissions
 ↓
Users
 ↓
Packages
```

## Fase 2 — Administration

```text
systemd
 ↓
Processes
 ↓
Storage
 ↓
Networking
```

## Fase 3 — Security

```text
SSH
 ↓
SELinux
 ↓
firewalld
 ↓
Logs
```

## Fase 4 — Operations

```text
Scheduling
 ↓
Boot
 ↓
Recovery
 ↓
Containers
```

## Fase 5 — Exam mode

```text
Labs
 ↓
Timed labs
 ↓
No notes
 ↓
Verification
 ↓
Reboot
 ↓
Troubleshooting
```

---

# 18. Prioridad

| Prioridad | Área |
|---|---|
| 🔴 Máxima | Storage / LVM / Filesystems |
| 🔴 Máxima | Users / Groups / Permissions |
| 🔴 Máxima | systemd / Services |
| 🔴 Máxima | Networking |
| 🔴 Máxima | SELinux |
| 🔴 Máxima | Shell |
| 🟠 Alta | firewalld |
| 🟠 Alta | SSH |
| 🟠 Alta | DNF / RPM |
| 🟠 Alta | Logs / Troubleshooting |
| 🟡 Media | Scheduling |
| 🟡 Media | Boot / Recovery |
| 🟡 Media | Containers |

---

# 19. Laboratorio

Recomendado:

```text
Fedora Host
    │
    └── RHEL VM
          ├── OS disk
          ├── Data disk
          └── Network
```

No usar el sistema Fedora principal para prácticas destructivas.

### Labs mínimos

#### Lab 01 — Users

```text
Crear usuarios
Crear grupos
Configurar passwords
Configurar sudo
Verificar
```

#### Lab 02 — Permissions

```text
chmod
chown
ACL
SUID
SGID
sticky bit
```

#### Lab 03 — Storage

```text
Disk
Partition
LVM
Filesystem
Mount
fstab
Reboot
```

#### Lab 04 — Networking

```text
IP
Gateway
DNS
Hostname
Routes
```

#### Lab 05 — Service

```text
Install
Configure
systemctl
firewalld
SELinux
Verify
```

#### Lab 06 — Troubleshooting

Introducir deliberadamente:

```text
Servicio parado
Firewall incorrecto
Contexto SELinux incorrecto
fstab incorrecto
Configuración de red incorrecta
```

Después resolver sin reinstalar.

---

# 20. Mock Exam

Antes de presentarte:

- [ ] Resolver laboratorios sin documentación
- [ ] Trabajar con tiempo limitado
- [ ] Verificar cada tarea
- [ ] Reiniciar cuando sea necesario
- [ ] Resolver problemas sin reinstalar
- [ ] No depender de GUI
- [ ] Explicar cada configuración

---

# 21. Checklist global

## Shell
- [ ] Navigation
- [ ] Files
- [ ] Pipes
- [ ] Redirection
- [ ] grep
- [ ] sed
- [ ] awk
- [ ] find

## Users
- [ ] Users
- [ ] Groups
- [ ] Passwords
- [ ] Aging
- [ ] sudo

## Permissions
- [ ] chmod
- [ ] chown
- [ ] chgrp
- [ ] umask
- [ ] ACL
- [ ] SUID
- [ ] SGID
- [ ] Sticky bit

## Services
- [ ] systemctl
- [ ] systemd
- [ ] journalctl

## Storage
- [ ] Partitions
- [ ] LVM
- [ ] XFS
- [ ] ext4
- [ ] mount
- [ ] fstab
- [ ] Resize

## Network
- [ ] nmcli
- [ ] IP
- [ ] Routes
- [ ] DNS
- [ ] Hostname
- [ ] Sockets

## Security
- [ ] SELinux
- [ ] firewalld
- [ ] SSH
- [ ] Permissions

## Operations
- [ ] Processes
- [ ] Logs
- [ ] cron
- [ ] timers
- [ ] boot
- [ ] recovery
- [ ] containers

---

# 22. Criterio de preparación

No uses como criterio:

> "He terminado el temario."

Usa:

> "Puedo resolver los escenarios administrativos sin ayuda y verificar que sobreviven a un reboot."

### Niveles

```text
0 — No conozco el concepto
1 — Conozco el comando
2 — Puedo hacerlo con documentación
3 — Puedo hacerlo sin documentación
4 — Puedo diagnosticar un fallo
5 — Puedo explicarlo y automatizarlo
```

Para EX200:

**Objetivo: nivel 3–4 en todas las áreas críticas.**

---

# 23. RHCSA → RHCE → Cybersecurity

```text
Unix / C / Shell
       ↓
Linux fundamentals
       ↓
RHCSA / EX200
       │
       ├── Administration
       ├── Networking
       ├── Storage
       ├── Security
       └── Troubleshooting
       ↓
RHCE / EX294
       ↓
Ansible / Automation
       ↓
Cybersecurity
```

RHCSA no es una certificación de hacking. Su valor para cybersecurity está en proporcionar una base fuerte de administración Linux, seguridad, networking, procesos, servicios y troubleshooting.

---

# 24. Regla de oro

```text
UNDERSTAND
    ↓
CONFIGURE
    ↓
VERIFY
    ↓
PERSIST
    ↓
REBOOT
    ↓
VERIFY AGAIN
```

La meta no es memorizar comandos.

La meta es poder recibir un sistema Linux con requisitos concretos y dejarlo **correctamente configurado, seguro, persistente y verificable**.
