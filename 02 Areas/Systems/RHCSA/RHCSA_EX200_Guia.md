# RHCSA (EX200) — Guía de preparación profesional

> **Objetivo:** preparar y aprobar el examen Red Hat Certified System Administrator (RHCSA), **EX200**, desarrollando capacidad práctica real de administración de RHEL.

---

## 0. Cómo usar esta guía

Esta guía está pensada como un **runbook de estudio**, no como un resumen teórico.

Para cada tema sigue siempre este ciclo:

```text
Concepto
   ↓
Comando
   ↓
Configuración
   ↓
Verificación
   ↓
Reboot / prueba de persistencia
   ↓
Troubleshooting
```

### Regla de dominio

No marques un objetivo como completado porque hayas leído sobre él.

Márcalo cuando puedas:

- hacerlo desde terminal;
- explicar qué estás haciendo;
- verificar el resultado;
- repetirlo sin consultar apuntes;
- solucionar un fallo deliberadamente introducido.

---

# 1. Mapa de competencias

```text
RHCSA / EX200
│
├── Shell & command line
├── Files & permissions
├── Users & groups
├── Software management
├── systemd & services
├── Processes
├── Storage
│   ├── partitions
│   ├── LVM
│   ├── filesystems
│   └── mounts / fstab
├── Networking
├── SSH
├── SELinux
├── firewalld
├── Scheduling
├── Logs
├── Boot & recovery
└── Containers
```

---

# 2. Entorno de laboratorio

## 2.1 Plataforma recomendada

Para aprender:

- RHEL si tienes acceso a una suscripción/developer subscription.
- Fedora como entorno de práctica cuando una funcionalidad sea equivalente.
- Idealmente una VM adicional para practicar networking, SSH y servicios.

### Recomendación

No conviertas tu sistema Fedora principal en el laboratorio de prácticas destructivas.

Usa una VM.

```text
Fedora host
   │
   └── VM RHEL / laboratorio
          ├── disco principal
          ├── disco adicional
          └── red
```

---

# 3. Fase 1 — Command line

## Objetivos

Debes dominar:

```bash
pwd
ls
cd
cp
mv
rm
mkdir
touch
file
stat
find
cat
less
head
tail
grep
sort
uniq
cut
awk
sed
wc
```

## Redirecciones

Practicar:

```bash
>
>>
<
2>
2>&1
|
```

Ejemplos:

```bash
command > output.txt
command >> output.txt
command 2> errors.txt
command > output.txt 2>&1
command | grep pattern
```

## Pipes

Debes poder construir comandos compuestos:

```bash
grep ERROR /var/log/messages | sort | uniq -c
```

## Búsqueda

Practicar:

```bash
find /var/log -type f
find /etc -name "*.conf"
find /var -type f -size +100M
```

### Checklist

- [ ] Navegar por filesystem sin GUI
- [ ] Buscar archivos
- [ ] Buscar contenido
- [ ] Redirigir stdout
- [ ] Redirigir stderr
- [ ] Encadenar comandos con `|`
- [ ] Usar `grep`, `sed`, `awk`
- [ ] Trabajar con comodines y quoting

---

# 4. Fase 2 — Files, permissions & ownership

## Comandos

```bash
ls -l
chmod
chown
chgrp
umask
```

## Permisos

Debes entender:

```text
r = 4
w = 2
x = 1
```

Ejemplo:

```bash
chmod 750 script.sh
```

Resultado:

```text
owner   rwx
group   r-x
others  ---
```

## Permisos especiales

Dominar:

```text
SUID
SGID
sticky bit
```

Ejemplo:

```bash
chmod 1777 /shared
```

## Ownership

```bash
chown user file
chown user:group file
chgrp group file
```

## ACL

Practicar también:

```bash
getfacl
setfacl
```

### Checklist

- [ ] Interpretar `ls -l`
- [ ] Cambiar permisos
- [ ] Cambiar owner
- [ ] Cambiar group
- [ ] Entender `umask`
- [ ] SUID
- [ ] SGID
- [ ] Sticky bit
- [ ] ACL

---

# 5. Fase 3 — Users & groups

## Comandos

```bash
useradd
usermod
userdel

groupadd
groupmod
groupdel

passwd
chage
id
groups
```

## Archivos

Conocer:

```text
/etc/passwd
/etc/shadow
/etc/group
/etc/gshadow
```

## Práctica

Crear:

```text
grupo: developers
usuario: dev01
```

Añadir usuario al grupo:

```bash
usermod -aG developers dev01
```

Verificar:

```bash
id dev01
```

## Password aging

Practicar:

```bash
chage -M 90 dev01
chage -l dev01
```

## sudo

Conocer:

```text
/etc/sudoers
/etc/sudoers.d/
```

Editar mediante:

```bash
visudo
```

### Checklist

- [ ] Crear usuario
- [ ] Eliminar usuario
- [ ] Crear grupo
- [ ] Añadir usuario a grupo
- [ ] Configurar password
- [ ] Configurar password aging
- [ ] Configurar sudo
- [ ] Verificar identidad con `id`

---

# 6. Fase 4 — Software management

## DNF

```bash
dnf search package
dnf info package
dnf install package
dnf remove package
dnf update
dnf repolist
```

## RPM

```bash
rpm -q package
rpm -qa
rpm -ql package
rpm -qf /path/to/file
```

Debes entender:

```text
RPM
 ↓
package
 ↓
repository
 ↓
DNF
 ↓
dependencies
```

### Checklist

- [ ] Buscar paquete
- [ ] Instalar
- [ ] Eliminar
- [ ] Consultar información
- [ ] Listar repositorios
- [ ] Consultar archivos de un paquete
- [ ] Identificar qué paquete proporciona un archivo

---

# 7. Fase 5 — systemd

## Servicios

```bash
systemctl status sshd
systemctl start sshd
systemctl stop sshd
systemctl restart sshd
systemctl enable sshd
systemctl disable sshd
```

Combinación habitual:

```bash
systemctl enable --now sshd
```

### Diferencia fundamental

```text
start  → inicia ahora
enable → inicia automáticamente durante boot
```

## Investigación

```bash
systemctl list-units
systemctl list-unit-files
systemctl status SERVICE
journalctl -u SERVICE
```

### Checklist

- [ ] Start
- [ ] Stop
- [ ] Restart
- [ ] Enable
- [ ] Disable
- [ ] Status
- [ ] Investigar servicio fallido
- [ ] Relacionar systemd con journalctl

---

# 8. Fase 6 — Processes

## Comandos

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

## Señales

Conocer al menos:

```text
SIGTERM
SIGKILL
SIGHUP
```

Regla:

```text
SIGTERM → terminación solicitada
SIGKILL → terminación forzada
```

Preferir terminación limpia cuando sea posible.

### Checklist

- [ ] Encontrar proceso
- [ ] Identificar PID
- [ ] Terminar proceso
- [ ] Diferenciar SIGTERM/SIGKILL
- [ ] Trabajar con jobs
- [ ] Background / foreground

---

# 9. Fase 7 — Storage

> **Prioridad máxima.**

Debes pensar en capas:

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

## Identificación

```bash
lsblk
blkid
fdisk -l
```

## Particiones

Practicar:

```bash
fdisk
parted
```

## LVM

```bash
pvcreate
pvs

vgcreate
vgs

lvcreate
lvs
```

## Filesystems

```bash
mkfs.xfs
mkfs.ext4
```

## Mount

```bash
mount
umount
df -h
du -sh
```

## Persistencia

Archivo:

```text
/etc/fstab
```

Preferir UUID:

```bash
blkid
```

Después comprobar:

```bash
mount -a
```

Y finalmente:

```bash
reboot
df -h
```

### Checklist

- [ ] Identificar discos
- [ ] Crear partición
- [ ] Crear PV
- [ ] Crear VG
- [ ] Crear LV
- [ ] Crear filesystem
- [ ] Montar filesystem
- [ ] Configurar `/etc/fstab`
- [ ] Verificar después de reboot
- [ ] Ampliar LV
- [ ] Ampliar filesystem
- [ ] Diagnosticar errores de montaje

---

# 10. Fase 8 — Networking

## Comandos

```bash
ip addr
ip route
ss -lntup
nmcli
hostnamectl
ping
```

## NetworkManager

```bash
nmcli device status
nmcli connection show
nmcli connection show <connection>
```

Practicar configuración mediante:

```bash
nmcli connection modify ...
nmcli connection up ...
```

## Modelo de troubleshooting

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

¿Resuelve DNS?
    ↓
getent hosts example.com

¿Está escuchando el servicio?
    ↓
ss -lntup
```

### Checklist

- [ ] IP
- [ ] Gateway
- [ ] DNS
- [ ] Hostname
- [ ] Routes
- [ ] NetworkManager
- [ ] Servicios escuchando
- [ ] Diagnóstico de conectividad

---

# 11. Fase 9 — SSH

## Servicio

```bash
systemctl status sshd
```

## Cliente

```bash
ssh user@server
```

## Configuración

```text
/etc/ssh/sshd_config
```

## Claves

Conceptualmente:

```text
private key → cliente
public key  → servidor
```

Archivo habitual:

```text
~/.ssh/authorized_keys
```

Permisos típicos:

```bash
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys
```

### Checklist

- [ ] Conectar por SSH
- [ ] Configurar sshd
- [ ] Reiniciar correctamente sshd
- [ ] Autenticación por clave
- [ ] Verificar permisos SSH
- [ ] Diagnosticar SSH

---

# 12. Fase 10 — SELinux

> **Prioridad máxima.**

## Estado

```bash
getenforce
sestatus
```

Estados:

```text
Enforcing
Permissive
Disabled
```

## Contextos

```bash
ls -Z
ps -Z
```

## Restauración

```bash
restorecon -Rv /path
```

## Configuración persistente

Conocer:

```bash
semanage fcontext
restorecon
```

Patrón:

```bash
semanage fcontext -a -t TYPE "/data(/.*)?"
restorecon -Rv /data
```

## Principio

No memorices SELinux como una colección de comandos.

Piensa:

```text
¿Quién?
   ↓
¿Qué proceso?
   ↓
¿Qué recurso?
   ↓
¿Qué contexto?
   ↓
¿Qué política?
```

### Checklist

- [ ] Enforcing / permissive
- [ ] Contextos
- [ ] `ls -Z`
- [ ] `restorecon`
- [ ] `semanage fcontext`
- [ ] Diagnosticar denegaciones
- [ ] Configuración persistente

---

# 13. Fase 11 — firewalld

## Estado

```bash
firewall-cmd --state
```

## Zonas

```bash
firewall-cmd --get-active-zones
firewall-cmd --list-all
```

## Servicios

```bash
firewall-cmd --permanent --add-service=http
firewall-cmd --reload
```

## Conceptos

```text
zone
service
port
runtime
permanent
```

### Checklist

- [ ] Consultar zona activa
- [ ] Añadir servicio
- [ ] Añadir puerto
- [ ] Distinguir runtime/permanent
- [ ] Reload
- [ ] Verificar configuración

---

# 14. Fase 12 — Logs

## journalctl

```bash
journalctl
journalctl -b
journalctl -u sshd
journalctl -p err
journalctl --since today
```

## Workflow

```text
Problema
   ↓
¿Cuándo ocurrió?
   ↓
¿Qué servicio?
   ↓
systemctl status
   ↓
journalctl -u
   ↓
¿SELinux?
   ↓
¿firewalld?
   ↓
¿network?
```

### Checklist

- [ ] Consultar boot actual
- [ ] Filtrar por servicio
- [ ] Filtrar errores
- [ ] Filtrar por tiempo
- [ ] Usar logs para troubleshooting

---

# 15. Fase 13 — Scheduling

## cron

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

## systemd timers

```bash
systemctl list-timers
```

### Checklist

- [ ] Crear cron job
- [ ] Verificar cron job
- [ ] Entender sintaxis
- [ ] Identificar timers

---

# 16. Fase 14 — Boot & recovery

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

Debes ser capaz de investigar problemas de boot y recuperar configuraciones incorrectas.

### Checklist

- [ ] Entender boot
- [ ] Identificar target
- [ ] Cambiar default target
- [ ] Consultar boot logs
- [ ] Troubleshooting de servicios durante boot

---

# 17. Fase 15 — Containers

Herramienta principal:

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

### Checklist

- [ ] Descargar imagen
- [ ] Ejecutar container
- [ ] Ver containers
- [ ] Parar container
- [ ] Eliminar container
- [ ] Entender image vs container
- [ ] Persistencia básica

---

# 18. Troubleshooting profesional

Esta sección debe convertirse en una habilidad transversal.

## Servicio caído

```bash
systemctl status SERVICE
journalctl -u SERVICE
ss -lntup
```

## Problema de red

```bash
ip addr
ip route
nmcli device status
ping GATEWAY
getent hosts example.com
```

## Problema de acceso a archivo

```bash
ls -l FILE
ls -Z FILE
id USER
getfacl FILE
```

Pensar en:

```text
Linux permissions
+
ACL
+
SELinux
```

## Problema de servicio accesible desde red

Comprobar en este orden:

```text
1. Process
2. systemd
3. Listening socket
4. Network
5. firewalld
6. SELinux
```

---

# 19. Simulaciones de examen

No hagas solo ejercicios aislados.

Crea escenarios completos.

## Simulación A — User administration

Objetivo:

```text
Crear grupo developers.
Crear usuarios dev01 y dev02.
Configurar pertenencia a grupos.
Configurar expiración de passwords.
Configurar sudo.
Verificar permisos.
```

Tiempo objetivo:

```text
15–20 min
```

---

## Simulación B — Storage

Objetivo:

```text
Añadir disco.
Crear partición.
Crear PV.
Crear VG.
Crear LV.
Formatear XFS.
Montar /data.
Configurar fstab.
Reboot.
Verificar.
```

Tiempo objetivo:

```text
20–30 min
```

---

## Simulación C — Service + firewall + SELinux

Objetivo:

```text
Instalar servicio.
Configurar servicio.
Activarlo con systemd.
Abrir firewall.
Configurar contexto SELinux.
Verificar acceso desde otra máquina.
```

Tiempo objetivo:

```text
20–30 min
```

---

# 20. Checklist final

## Linux

- [ ] Shell
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
- [ ] sudo
- [ ] Aging

## Permissions

- [ ] chmod
- [ ] chown
- [ ] chgrp
- [ ] umask
- [ ] ACL
- [ ] SUID
- [ ] SGID
- [ ] sticky bit

## Services

- [ ] systemctl
- [ ] systemd
- [ ] journalctl

## Storage

- [ ] partitions
- [ ] LVM
- [ ] XFS
- [ ] ext4
- [ ] mount
- [ ] fstab
- [ ] resize

## Network

- [ ] nmcli
- [ ] IP
- [ ] routes
- [ ] DNS
- [ ] hostname
- [ ] sockets

## Security

- [ ] SELinux
- [ ] firewalld
- [ ] SSH
- [ ] permissions

## Operations

- [ ] processes
- [ ] logs
- [ ] cron
- [ ] timers
- [ ] boot
- [ ] recovery
- [ ] containers

---

# 21. Método de estudio recomendado

## Etapa 1 — Fundamentos

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

## Etapa 2 — Administración

```text
systemd
↓
Processes
↓
Storage
↓
Networking
```

## Etapa 3 — Enterprise Linux

```text
SSH
↓
SELinux
↓
firewalld
↓
Logs
```

## Etapa 4 — Operación

```text
Scheduling
↓
Boot
↓
Recovery
↓
Containers
```

## Etapa 5 — Exam mode

```text
Timed labs
↓
No documentation
↓
Verification
↓
Reboot
↓
Troubleshooting
↓
Repeat
```

---

# 22. Regla de oro del EX200

Ante cualquier tarea:

```text
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

Una configuración que funciona ahora pero desaparece después del reboot **no está terminada**.

El objetivo final no es saber muchos comandos.

Es poder recibir un sistema Linux con requisitos concretos y dejarlo **correctamente configurado, seguro, persistente y verificable**.

---

# 23. Progreso

| Área | Estado | Confianza |
|---|---|---:|
| Shell | ⬜ | 0% |
| Files | ⬜ | 0% |
| Permissions | ⬜ | 0% |
| Users/Groups | ⬜ | 0% |
| Packages | ⬜ | 0% |
| systemd | ⬜ | 0% |
| Processes | ⬜ | 0% |
| Storage/LVM | ⬜ | 0% |
| Networking | ⬜ | 0% |
| SSH | ⬜ | 0% |
| SELinux | ⬜ | 0% |
| firewalld | ⬜ | 0% |
| Logs | ⬜ | 0% |
| Scheduling | ⬜ | 0% |
| Boot/Recovery | ⬜ | 0% |
| Containers | ⬜ | 0% |
| Mock exams | ⬜ | 0% |

---

## Objetivo de salida

Antes de presentarte al EX200 deberías poder abrir una terminal y resolver un laboratorio completo sin depender de copiar comandos de una guía.

**Target:** convertir conocimientos de Linux en administración práctica reproducible.
