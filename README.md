# gentoo-personal
Guía de instalación personal desde una distro existente. Algunas características son las siguientes:

Modo UEFI

Gestor de servicios: OpenRC

Perfil estándar


## **1. Preparación el disco**

Para mi intalación personal usaré el disco: /dev/sda. Para particionarlo usaré siguiente comando:

`cfdisk /dev/sda`

**NOTA:** Si el disco no cuenta con tabla de particiones seleccione GPT (GUID Partition Table).

```
dev/sda1; Tipo: Efi system; Tamaño: 150M
dev/sda2; Tipo: Linux file system; Tamaño: Resto del disco
```


### **Creación y monteje sistemas de archivos**

`mkfs.vfat -F 32 /dev/sda1`

`mkfs.ext4 /dev/sda2`

### **Montaje de sistemas de archivos**

`mkdir /mnt/gentoo`

`mount /dev/sda2 /mnt/gentoo`

`mkdir /mnt/gentoo/boot`

`mount /dev/sda1 /mnt/gentoo/boot`

## **2. Descargar y extraer el stage3**

`date 081309002021`

**NOTA:** El formato para introducir la fecha es: MMDDhhmmYYYY. En este ejemplo se setea: Junio 06 2021, Hora: 10:00 p.m.

`cd /mnt/gentoo`

`wget https://bouncer.gentoo.org/fetch/root/all/releases/amd64/autobuilds/20210808T170546Z/stage3-amd64-openrc-20210808T170546Z.tar.xz`

`tar -xpf stage3-amd64-openrc-20210808T170546Z.tar.xz --numeric-owner --xattrs-include="*.*"`

**NOTA:** El stage utilizado es el disponible en la página en la fecha 13-08. Sustituya el stage por uno más actualizado. Para hacerlo vaya a la página: www.gentoo.org/downloads.

`nano -w /mnt/gentoo/etc/portage/make.conf`

```
-* archivo make.conf *-
COMMON_CFLAGS="-march=native -O2 -pipe"
MAKEOPTS="-j5"

GRUB_PLATFORMS="efi-64"
VIDEO_CARDS="intel i965 iris"

L10N="es-MX es"
USE="X dbus bindist elogind networkmanager -systemd mmx sse sse2"
```

## **3. Enjaulamiento**

`cp --dereference /etc/resolv.conf /mnt/gentoo/etc/`

`mount --types proc /proc /mnt/gentoo/proc`

`mount --rbind /sys /mnt/gentoo/sys`

`mount --make-rslave /mnt/gentoo/sys`

`mount --rbind /dev /mnt/gentoo/dev`

`mount --make-rslave /mnt/gentoo/dev`

### **Entrar al nuevo entorno**

`chroot /mnt/gentoo /bin/bash`

`source /etc/profile`

`export PS1="(chroot) ${PS1}"`
