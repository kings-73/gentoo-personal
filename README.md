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

## **4. Configurar Portage y Elegir perfil**

`emerge-webrsync -v`

`emerge --ask --sync --quiet`

`eselect profile list`

`eselect profile set 5`

**NOTA:** Cambia el valor "5" por el deseado.

## **5. Zona Horaria**

`echo "America/Mexico_City" > /etc/timezone`

`emerge --config sys-libs/timezone-data`

`echo "es_MX.UTF-8 UTF-8" > /etc/locale.gen`

`locale-gen`

`eselect locale list`

`eselect locale set 4`

**NOTA:** De igual manera, cambie el valor "4" por el deseado.

## **6. Configurar el Nucleo Linux**

`emerge --ask sys-kernel/gentoo-sources`

`ls -l /usr/src/linux`

**NOTA:** el comando "ls -l" muestra el enlace simbólico a linux hecho por portage. En caso de no tenerlo realizarlo manualmente con los siguientes comandos:

### **(Opcional) sólo en caso que no aparezca enlace simbólico**

`eselect kernel list`

`eselect kernel set 1`

**NOTA:** Sustituya el valor "1" por el deseado en caso de ser necesario.

### **Compilación del núcleo linux**

`emerge --ask sys-kernel/genkernel`

`emerge --ask sys-kernel/linux-firmware`

`genkernel all` ó `genkernel --menuconfig all`

**NOTA:** genkernel all para compilación completa (nuevos usuarios) ó genkernel --menuconfig all para usuarios experimentados que conozcan su harwware.

### **7. Archivos de configuración**

`nano -w /etc/fstab`

```
-* archivo fstab *-
/dev/sda1  boot  vfat  noatime  0 0
/dev/sda2  none  swap  sw       0 0
/dev/sda3  /     ext4  noatime  0 1
```

`nano -w /etc/conf.d/hostname`

```
-* archivo hostname *-
localhost="gentoo"
```

`nano -w /etc/hosts`

```
-* archivo hosts *-
127.0.0.1 gentoo.redhogar gentoo localhost
::1 gentoo
```

`nano -w /etc/conf.d/hwclock`

```
-* archivo hwclock *-
clock="local"
```

`nano -w /etc/conf.d/keymaps`

```
-* archivo keymaps *-
keymap="es"
```

### **8. Instalacion de GRUB EFI**

`emerge -av sys-boot/grub:2`

`grub-install --target=x86_64-efi --efi-directory=/boot`

`grub-mkconfig -o /boot/grub/grub.cfg`

### **9. Instalar Herramientas**

`emerge -av net-misc/dhcpcd`

`emerge -av gentoolkit`

### **10. Password ROOT y salir del sistema**

`passwd`

`exit`

`cd`

`umount -l /mnt/gentoo/dev{/shm,/pts,} `

`umount -R /mnt/gentoo`

`reboot`

**¡Felicidades, ya has instalado gentoo!**
