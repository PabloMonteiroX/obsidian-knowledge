# LPIC-1 + LPIC-2 — Guía de preparación

## Objetivo

Conseguir:

```text
LPIC-1
101-500 + 102-500
        ↓
LPIC-2
201-450 + 202-450
```

LPIC-1 utiliza objetivos 5.0; LPIC-2 utiliza actualmente los objetivos publicados 4.5. Los objetivos oficiales deben ser la fuente principal del plan de estudio. 

---

# 1. Estrategia

No memorizar preguntas.

Usar:

```text
Theory
 ↓
Command
 ↓
Lab
 ↓
Verify
 ↓
Questions
 ↓
Mistake log
 ↓
Repeat
```

---

# 2. Laboratorio

Recomendación:

```text
Fedora Host
 ├── Debian/Ubuntu VM
 └── Rocky/Fedora VM
```

La razón es que LPIC es vendor-neutral.

Practicar con más de una familia Linux evita aprender únicamente la implementación de una distribución.

---

# 3. LPIC-1 — 101

## Bloque A — Architecture

```bash
lspci
lsusb
lsblk
lsmod
modprobe
dmesg
uname
```

Checklist:

- [ ] BIOS / UEFI
- [ ] Bootloader
- [ ] Kernel
- [ ] initramfs
- [ ] Modules
- [ ] `/proc`
- [ ] `/sys`
- [ ] `/dev`

---

## Bloque B — Packages

Debian:

```bash
apt
dpkg
```

RPM:

```bash
dnf
rpm
```

Practicar:

```bash
install
remove
query
search
dependencies
repositories
```

Checklist:

- [ ] apt
- [ ] dpkg
- [ ] dnf
- [ ] rpm
- [ ] repositories
- [ ] dependencies
- [ ] package files

---

## Bloque C — GNU/Unix commands

Prioridad máxima.

```bash
find
grep
sed
awk
cut
sort
uniq
wc
xargs
tar
gzip
bzip2
xz
```

Shell:

```bash
|
>
>>
<
2>
2>&1
```

Regular expressions:

```text
^
$
.
*
+
?
[]
{}
()
|
```

---

## Bloque D — Filesystems

```bash
lsblk
blkid
mount
umount
findmnt
df
du
```

Estudiar:

```text
FHS
ext4
XFS
mount
fstab
permissions
links
```

---

# 4. LPIC-1 — 102

## Shell

Dominar:

```bash
bash
PATH
export
env
printenv
alias
```

Scripting:

```bash
if
for
while
case
functions
exit status
```

---

## Administration

```bash
useradd
usermod
userdel
groupadd
passwd
chage
id
```

Scheduling:

```bash
cron
at
```

Time:

```bash
date
timedatectl
```

Logs:

```bash
journalctl
```

---

## Services

```bash
systemctl
journalctl
```

Entender:

```text
start
stop
restart
enable
disable
status
```

---

## Networking

Prioridad máxima.

```bash
ip
ss
ping
traceroute
dig
host
getent
```

Estudiar:

```text
IPv4
IPv6
CIDR
Routing
DNS
TCP
UDP
Ports
Sockets
```

---

## Security

```bash
chmod
chown
chgrp
umask
getfacl
setfacl
sudo
ssh
```

Conceptos:

```text
permissions
ACL
PAM
authentication
authorization
firewall
SSH
```

---

# 5. Primer checkpoint

Antes de pasar a LPIC-2:

- [ ] 101 completo
- [ ] 102 completo
- [ ] Todos los comandos principales practicados
- [ ] Labs sin copiar comandos
- [ ] Preguntas consistentes
- [ ] Capacidad de explicar respuestas
- [ ] 101-500 aprobado
- [ ] 102-500 aprobado
- [ ] LPIC-1 obtenido

---

# 6. LPIC-2 — 201

## Kernel

```bash
uname
lsmod
modprobe
modinfo
depmod
sysctl
dmesg
```

Estudiar:

```text
kernel architecture
modules
drivers
DKMS
udev
sysctl
/proc
/sys
```

---

## Boot

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
```

Practicar:

- modificar parámetros;
- analizar logs de boot;
- recuperar servicios;
- entender targets;
- regenerar initramfs.

---

## Storage

Profundizar:

```text
Partitioning
RAID
LVM
Filesystem
Mounting
Quotas
```

Practicar escenarios completos.

---

## Networking avanzado

```bash
ip
ss
tcpdump
ip route
```

Practicar:

```text
routing
interfaces
bridges
connectivity
packet analysis
```

---

## Troubleshooting

Crear fallos deliberadamente:

```text
Broken route
Wrong permissions
Failed service
Wrong mount
Bad DNS
Wrong firewall
Broken configuration
```

Después resolverlos.

---

# 7. LPIC-2 — 202

## DNS

Practicar:

```bash
dig
host
nslookup
```

Estudiar:

```text
zones
records
authoritative
recursive
forwarders
```

---

## Web

Practicar:

```text
Install
Configure
Virtual host
TLS
Logs
Troubleshooting
```

---

## NFS

Practicar:

```text
Server
 ↓
Export
 ↓
Client
 ↓
Mount
 ↓
Permissions
```

---

## Samba

Practicar:

```text
Share
 ↓
Authentication
 ↓
Permissions
 ↓
Windows client
```

---

## DHCP

Practicar:

```text
Discover
Offer
Request
ACK
```

Entender:

```text
leases
reservations
options
DNS
gateway
```

---

## Authentication

Estudiar:

```text
PAM
NSS
LDAP
```

Comprender la diferencia entre:

```text
Authentication
Authorization
Account management
```

---

## Mail

Estudiar:

```text
SMTP
IMAP
POP3
MTA
MDA
MUA
```

No limitarse a memorizar siglas: entender el flujo.

---

# 8. Labs LPIC-2

## Lab 01 — Kernel

```text
Inspect kernel
Load module
Unload module
Configure module
Inspect logs
Configure sysctl
Verify
```

## Lab 02 — Storage

```text
Disk
 ↓
Partition
 ↓
RAID/LVM
 ↓
Filesystem
 ↓
Mount
 ↓
Persist
```

## Lab 03 — DNS

```text
DNS server
 ↓
Zone
 ↓
Records
 ↓
Client
 ↓
dig
```

## Lab 04 — Web

```text
Web server
 ↓
Virtual host
 ↓
TLS
 ↓
Logs
 ↓
Troubleshoot
```

## Lab 05 — NFS

```text
NFS server
 ↓
Export
 ↓
Client
 ↓
Mount
 ↓
Permissions
```

## Lab 06 — Samba

```text
Samba server
 ↓
Share
 ↓
User
 ↓
Authentication
 ↓
Client
```

## Lab 07 — DHCP

```text
DHCP server
 ↓
Client
 ↓
Lease
 ↓
Gateway
 ↓
DNS
```

## Lab 08 — Authentication

```text
PAM
 ↓
NSS
 ↓
LDAP
 ↓
Authentication
```

---

# 9. Orden recomendado

```text
LPIC-1
│
├── 101
│   ├── Architecture
│   ├── Packages
│   ├── Commands
│   └── Filesystems
│
└── 102
    ├── Shell
    ├── Administration
    ├── Services
    ├── Networking
    └── Security

        ↓

LPIC-2
│
├── 201
│   ├── Capacity
│   ├── Kernel
│   ├── Boot
│   ├── Storage
│   └── Networking
│
└── 202
    ├── DNS
    ├── Web
    ├── NFS
    ├── Samba
    ├── DHCP
    ├── Authentication
    ├── Mail
    └── Security
```

---

# 10. Prioridades

## LPIC-1

```text
🔴 Commands
🔴 Shell
🔴 Filesystems
🔴 Users / Permissions
🔴 Networking
🟠 Packages
🟠 Services
🟠 Security
🟡 Desktop
```

## LPIC-2

```text
🔴 Networking
🔴 Storage
🔴 Kernel
🔴 Troubleshooting
🔴 DNS
🔴 Web
🟠 NFS
🟠 Samba
🟠 Authentication
🟠 DHCP
🟡 Mail
```

---

# 11. Sistema de estudio

Por cada objetivo:

```text
OBJECTIVE
    ↓
What?
    ↓
Why?
    ↓
Command?
    ↓
Config file?
    ↓
Lab?
    ↓
Verify?
    ↓
Failure mode?
```

Para cada comando:

```text
NAME:
PURPOSE:
SYNTAX:
OPTIONS:
FILES:
EXAMPLE:
VERIFY:
COMMON MISTAKE:
```

---

# 12. Error log

Archivo:

```text
lpic-mistakes.md
```

Formato:

```text
## Error

Question:
...

My answer:
...

Correct answer:
...

Why:
...

Related command/concept:
...

How to avoid:
...
```

---

# 13. Simulacros

No usar simulacros como primera fase.

Orden:

```text
Study
 ↓
Lab
 ↓
Questions
 ↓
Mock
 ↓
Analyze
 ↓
Weak objectives
 ↓
Repeat
```

Antes del examen:

```text
Timed
No notes
No internet
No guessing by pattern
```

---

# 14. Criterio de preparación

```text
0 — No conozco
1 — Reconozco
2 — Entiendo
3 — Respondo
4 — Practico
5 — Explico y diagnostico
```

Objetivo:

```text
LPIC-1
Áreas críticas → 4–5

LPIC-2
Áreas críticas → 4–5
```

---

# 15. Relación con RHCSA

Hay bastante overlap.

No estudiar dos veces:

```text
Linux commands
Users
Permissions
Storage
Networking
Services
Security
```

Usar LPIC para amplitud y RHCSA para práctica RHEL:

```text
LPIC-1
   ↓
Linux breadth
   ↓
RHCSA
   ↓
Hands-on RHEL
   ↓
LPIC-2
   ↓
Advanced infrastructure
```

Una alternativa:

```text
LPIC-1
   ↓
RHCSA
   ↓
LPIC-2
   ↓
RHCE
```

Esto produce una progresión muy fuerte de **Linux fundamentals → administration → advanced infrastructure → automation**.

---

# 16. Checklist final

## LPIC-1

- [ ] 101-500
- [ ] 102-500
- [ ] Commands
- [ ] Shell
- [ ] Filesystems
- [ ] Packages
- [ ] Users
- [ ] Permissions
- [ ] Services
- [ ] Networking
- [ ] Security

## LPIC-2

- [ ] 201-450
- [ ] 202-450
- [ ] Kernel
- [ ] Boot
- [ ] Storage
- [ ] Networking
- [ ] DNS
- [ ] Web
- [ ] NFS
- [ ] Samba
- [ ] DHCP
- [ ] Authentication
- [ ] Mail
- [ ] Security
- [ ] Troubleshooting

---

# 17. Meta profesional

```text
LPIC-1
    ↓
Linux Administrator
    ↓
LPIC-2
    ↓
Advanced Linux Administrator
    ↓
RHCSA / RHCE
    ↓
Automation / Infrastructure
    ↓
Cybersecurity
```

La certificación debe ser la consecuencia de la habilidad.

El objetivo real es poder mirar un servidor y razonar:

```text
What is happening?
        ↓
Where is the problem?
        ↓
What evidence do I have?
        ↓
What should I change?
        ↓
How do I verify it?
```
