> Guía rápida para LPIC-1, RHCSA y LPIC-2
---
## 1. ¿Qué es el kernel?

El **kernel** es el núcleo del sistema operativo. Gestiona los recursos del hardware y proporciona servicios a los programas.

```text
Aplicaciones
     ↓
Shell / Libraries
     ↓
System Calls
     ↓
┌──────────────────────────┐
│      LINUX KERNEL        │
│                          │
│ Procesos                 │
│ Memoria                  │
│ Dispositivos / Drivers   │
│ Filesystems              │
│ Networking               │
│ Seguridad                │
└────────────┬─────────────┘
             ↓
       Hardware
```

**Linux**, estrictamente, es el kernel. Una distribución como Fedora, Debian o RHEL añade herramientas, librerías, systemd, gestores de paquetes y aplicaciones.

---

# 2. Funciones principales

## Procesos

El kernel administra:

- procesos;
- threads;
- CPU scheduling;
- señales;
- prioridades.

Comandos útiles:

```zsh
ps
top
pgrep
kill
nice
renice
```
---

## Memoria

Gestiona:

- RAM;
- memoria virtual;
- paging;
- swap;
- protección de memoria.

```text
Proceso
   ↓
Memoria virtual
   ↓
Kernel
   ↓
RAM / Swap
```

---

## Dispositivos

El kernel utiliza **drivers** para comunicarse con el hardware.

```text
Aplicación
    ↓
Kernel
    ↓
Driver
    ↓
Hardware
```

Ejemplo:

```text
USB
 ↓
Kernel USB subsystem
 ↓
Driver
 ↓
Dispositivo
```

---

## Filesystems

El kernel proporciona soporte para diferentes sistemas de archivos:

```text
XFS
ext4
VFAT
NFS
...
```

---

## Networking

El kernel implementa gran parte de la pila de red:

```text
Application
     ↓
Socket
     ↓
TCP / UDP
     ↓
IP
     ↓
Network driver
     ↓
NIC
```

---

# 3. System Calls

Los programas utilizan **system calls** para solicitar servicios al kernel.

Ejemplos conceptuales:

```text
open()
read()
write()
fork()
execve()
socket()
```

Modelo:

```text
User Space
     ↓
System Call
     ↓
Kernel Space
     ↓
Hardware / Resources
```

Esto separa:

- **User space** → aplicaciones.
- **Kernel space** → código privilegiado del kernel.

---

# 4. Kernel Space vs User Space

```text
┌─────────────────────────┐
│       USER SPACE        │
│                         │
│ bash                    │
│ ssh                     │
│ nginx                   │
│ python                  │
│ aplicaciones           │
└────────────┬────────────┘
             │
        System Calls
             │
┌────────────▼────────────┐
│       KERNEL SPACE      │
│                         │
│ Scheduler               │
│ Memory manager          │
│ Drivers                 │
│ Filesystems             │
│ Networking              │
└─────────────────────────┘
```

Una aplicación normal no tiene acceso directo y arbitrario al hardware.

---

# 5. Kernel Modules

Un **kernel module** es código que puede cargarse dinámicamente en el kernel.

Se utiliza habitualmente para:

- drivers;
- filesystems;
- funcionalidades adicionales.

Comandos fundamentales:

```bash
lsmod
modinfo
modprobe
insmod
rmmod
depmod
```

### Ver módulos cargados

```bash
lsmod
```

### Información de un módulo

```bash
modinfo <module>
```

### Cargar

```bash
sudo modprobe <module>
```

### Descargar

```bash
sudo modprobe -r <module>
```

### Diferencia importante

```text
modprobe
  → carga módulos y resuelve dependencias

insmod
  → inserta directamente un módulo
```

Los módulos del kernel se encuentran normalmente bajo:

```text
/lib/modules/<kernel-version>/
```

---

# 6. Kernel version

Para saber qué kernel estás ejecutando:

```bash
uname -r
```

Más información:

```bash
uname -a
```

Ejemplo conceptual:

```text
6.x.x-xxx.x86_64
```

También:

```bash
cat /proc/version
```

---

# 7. /proc

`/proc` es un filesystem virtual que proporciona información sobre procesos y el kernel.

Ejemplos:

```bash
cat /proc/cpuinfo
cat /proc/meminfo
cat /proc/version
cat /proc/uptime
```

Conceptualmente:

```text
/proc
 ├── Información de procesos
 ├── CPU
 ├── Memoria
 ├── Kernel
 └── Runtime information
```

---

# 8. /sys

`/sys` es un filesystem virtual asociado principalmente con dispositivos, drivers y subsistemas del kernel.

```bash
ls /sys
```

Modelo:

```text
/sys
 ↓
Kernel device model
 ↓
Devices / Drivers / Subsystems
```

Diferencia rápida:

```text
/proc → procesos + información del kernel

/sys  → dispositivos + drivers + subsistemas
```

---

# 9. dmesg

`dmesg` muestra mensajes generados por el kernel.

```bash
dmesg
```

Buscar información:

```bash
dmesg | grep -i usb
dmesg | grep -i error
dmesg | grep -i network
```

Especialmente útil para:

- hardware;
- drivers;
- dispositivos;
- boot;
- errores del kernel.

En sistemas con systemd:

```bash
journalctl -k
```

es especialmente útil para consultar mensajes del kernel desde el journal.

---

# 10. Boot y kernel

Secuencia simplificada:

```text
Power On
   ↓
BIOS / UEFI
   ↓
GRUB
   ↓
Linux Kernel
   ↓
initramfs
   ↓
systemd
   ↓
Services
   ↓
User Space
```

---

# 11. GRUB

**GRUB** es el bootloader.

Su función principal es cargar el kernel y los componentes necesarios para iniciar el sistema.

Conceptualmente:

```text
GRUB
 ├── Kernel
 └── initramfs
        ↓
      RAM
        ↓
      Kernel
```

---

# 12. initramfs

**initramfs** significa *initial RAM filesystem*.

Es un filesystem temporal cargado en RAM durante el arranque.

Puede contener:

- drivers;
- herramientas;
- scripts;
- soporte para storage;
- soporte para LVM;
- soporte para RAID;
- módulos necesarios para encontrar `/`.

Modelo:

```text
Kernel
  ↓
initramfs
  ↓
Drivers / Storage
  ↓
Root filesystem
```

Esto es especialmente importante cuando `/` está sobre LVM, RAID o storage que necesita drivers adicionales.

---

# 13. Kernel parameters

El kernel puede recibir parámetros durante el arranque.

Ejemplo conceptual:

```text
GRUB
  ↓
Kernel parameters
  ↓
Kernel
```

Pueden utilizarse para modificar temporalmente determinados comportamientos o realizar troubleshooting durante el boot.

En runtime, muchos parámetros se exponen mediante:

```text
/proc/sys/
```

Y se pueden consultar/modificar mediante:

```bash
sysctl
```

Ejemplo:

```bash
sysctl -a
```

---

# 14. sysctl

`sysctl` permite consultar y modificar parámetros del kernel en runtime.

Consultar:

```bash
sysctl net.ipv4.ip_forward
```

Modificar temporalmente:

```bash
sudo sysctl -w net.ipv4.ip_forward=1
```

Para configuración persistente se utiliza normalmente:

```text
/etc/sysctl.conf
/etc/sysctl.d/
```

Concepto:

```text
sysctl
   ↓
/proc/sys
   ↓
Kernel parameters
```

---

# 15. udev

`udev` gestiona dinámicamente los dispositivos en user space, trabajando con los eventos del kernel.

Modelo:

```text
Hardware
   ↓
Kernel detects device
   ↓
udev
   ↓
/dev/*
```

Ejemplo:

```text
USB serial device
      ↓
Kernel
      ↓
udev
      ↓
/dev/ttyACM0
```

También permite utilizar **udev rules** para establecer configuraciones específicas de dispositivos.

---

# 16. Kernel compilation — LPIC-2

LPIC-2 exige más profundidad.

Modelo:

```text
Kernel source
     ↓
Configuration
     ↓
Compilation
     ↓
Modules
     ↓
Installation
     ↓
initramfs
     ↓
GRUB
     ↓
Boot
```

Archivo importante:

```text
.config
```

La configuración determina si una funcionalidad está:

```text
y → built-in
m → module
n → disabled
```

Herramientas habituales:

```bash
make
make menuconfig
make oldconfig
```

Y conceptos:

```text
bzImage
modules
modules_install
```

No es un objetivo central de RHCSA, pero sí de LPIC-2.

---

# 17. DKMS — LPIC-2

**DKMS = Dynamic Kernel Module Support**

Permite gestionar módulos externos al kernel y reconstruirlos cuando cambia la versión del kernel.

Modelo:

```text
Nuevo kernel
    ↓
Módulo externo
    ↓
Rebuild
    ↓
Instalación
    ↓
Carga
```

Es especialmente relevante para drivers o módulos que no forman parte del kernel principal.

---

# 18. Kernel troubleshooting

Cuando algo relacionado con hardware o drivers falla:

```text
1. ¿Qué kernel estoy ejecutando?
        ↓
   uname -r

2. ¿Está cargado el módulo?
        ↓
   lsmod

3. ¿Qué información tiene?
        ↓
   modinfo

4. ¿Puedo cargarlo?
        ↓
   modprobe

5. ¿Qué dice el kernel?
        ↓
   dmesg
   journalctl -k

6. ¿Existe el dispositivo?
        ↓
   /dev
   /sys

7. ¿Qué gestiona udev?
        ↓
   udev rules / events
```

---

# 19. Herramientas esenciales

| Comando | Función |
|---|---|
| `uname -r` | Kernel activo |
| `uname -a` | Información completa |
| `lsmod` | Módulos cargados |
| `modinfo` | Información de módulo |
| `modprobe` | Cargar/descargar módulos |
| `insmod` | Insertar módulo |
| `rmmod` | Eliminar módulo |
| `depmod` | Generar dependencias |
| `dmesg` | Mensajes del kernel |
| `journalctl -k` | Logs del kernel |
| `sysctl` | Parámetros del kernel |
| `ls /proc` | Información runtime |
| `ls /sys` | Devices/drivers |
| `ls /lib/modules` | Módulos instalados |

---

# 20. Qué estudiar para cada certificación

## LPIC-1

Debes dominar:

```text
Kernel
 ├── Concepto
 ├── Boot
 ├── GRUB
 ├── initramfs
 ├── Modules
 ├── /proc
 ├── /sys
 ├── dmesg
 └── Kernel parameters
```

**Objetivo:** administrar y diagnosticar.

---

## RHCSA

Enfócate en:

```text
Kernel
 ├── Boot
 ├── Modules
 ├── Drivers
 ├── dmesg
 ├── journalctl -k
 ├── systemd
 └── Troubleshooting
```

**Objetivo:** administrar RHEL y solucionar problemas del sistema.

No necesitas aprender a compilar un kernel desde cero para RHCSA.

---

## LPIC-2

Profundiza en:

```text
Kernel
 ├── Architecture
 ├── Components
 ├── Modules
 ├── Module dependencies
 ├── udev
 ├── Runtime management
 ├── sysctl
 ├── Kernel parameters
 ├── DKMS
 ├── Compilation
 ├── .config
 ├── initramfs
 └── Bootloader
```

**Objetivo:** administrar el kernel a nivel avanzado.

---

# 21. Mapa mental

```text
                         LINUX KERNEL
                              │
       ┌──────────────────────┼──────────────────────┐
       │                      │                      │
    PROCESSES              MEMORY                HARDWARE
       │                      │                      │
   scheduler              virtual               drivers
   signals                memory                modules
       │                      │                      │
       └──────────────────────┼──────────────────────┘
                              │
                       FILESYSTEMS
                              │
                         NETWORKING
                              │
                          SECURITY
                              │
                       SYSTEM CALLS
```

---

# 22. Kernel + boot

```text
BIOS / UEFI
     ↓
   GRUB
     ↓
  Kernel
     ↓
initramfs
     ↓
 systemd
     ↓
 services
     ↓
 user space
```

---

# 23. Kernel + hardware

```text
Hardware
   ↓
Kernel
   ↓
Driver
   ↓
Module
   ↓
/dev
   ↓
Application
```

---

# 24. Kernel + troubleshooting

```text
PROBLEM
   ↓
uname -r
   ↓
lsmod
   ↓
modinfo
   ↓
dmesg / journalctl -k
   ↓
/proc + /sys
   ↓
udev
   ↓
modprobe / configuration
   ↓
VERIFY
```

---

# 25. Diferencia de profundidad

```text
LPIC-1
  ↓
"¿Qué es el kernel y cómo se administra?"

RHCSA
  ↓
"¿Cómo administro y diagnostico un RHEL?"

LPIC-2
  ↓
"¿Cómo funciona, se configura, se compila
y se administra el kernel a nivel avanzado?"
```

---

# 26. Regla de oro

```text
Don't memorize commands.
Understand the flow.
```

Debes visualizar siempre:

```text
Application
    ↓
System Call
    ↓
Kernel
    ↓
Driver / Subsystem
    ↓
Hardware
```

Y durante el boot:

```text
UEFI
 ↓
GRUB
 ↓
Kernel
 ↓
initramfs
 ↓
systemd
 ↓
Services
```

Si entiendes esos dos flujos, gran parte del tema **kernel** de LPIC-1, RHCSA y LPIC-2 deja de ser memorización y pasa a ser administración lógica.
