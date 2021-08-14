# gentoo-personal
Guía de instalación personal desde una distro existente. Algunas características son las siguientes:

* Modo UEFI
* Gestor de servicios (Por defecto): OpenRC
* MTP (Tranferencia de datos Android) y Tethering (Internet Móvil Android)

**NOTA:** Para no tener problema con la conexión a internet recomiendo conexión por cable como primera opción. Como segunda opción puede conectar un telefono android que pueda compartir internet por USB, de esta manera tendrá internet inmediatamente sin configurar nada.

## **1. Preparación el disco para alojar la intalación Gentoo**

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

**NOTA:** El formato para introducir la fecha es: MMDDhhmmYYYY. En este ejemplo se setea: Agosto 13 de 2021, Hora: 10:00 p.m.

`cd /mnt/gentoo`

`wget https://bouncer.gentoo.org/fetch/root/all/releases/amd64/autobuilds/20210808T170546Z/stage3-amd64-openrc-20210808T170546Z.tar.xz`

`tar -xpf stage3-amd64-openrc-20210808T170546Z.tar.xz --numeric-owner --xattrs-include="*.*"`

**NOTA:** El stage utilizado es el disponible en la página en la fecha 13-08. Sustituya el stage por uno más actualizado. Para hacerlo vaya a la página: www.gentoo.org/downloads.

### **Modificación del archivo de opciones de compilación**

`nano -w /mnt/gentoo/etc/portage/make.conf`

```
-* archivo make.conf *-
COMMON_CFLAGS="-march=native -O2 -pipe"
MAKEOPTS="-j4"

GRUB_PLATFORMS="efi-64"
VIDEO_CARDS="intel i965 iris"

L10N="es-MX es"
USE="X alsa bindist dbus elogind networkmanager policykit -systemd udisks wayland mmx sse sse2"
```

## **3. Enjaulamiento**

Antes de montar los sistemas de archivos necesarios, copie la información de los DNS para asegurar que la red siga funcionando después de entrar al nuevo entorno.

`cp --dereference /etc/resolv.conf /mnt/gentoo/etc/`

### **Montaje los sistemas de archivos necesarios**

`mount --types proc /proc /mnt/gentoo/proc`

`mount --rbind /sys /mnt/gentoo/sys`

`mount --make-rslave /mnt/gentoo/sys`

`mount --rbind /dev /mnt/gentoo/dev`

`mount --make-rslave /mnt/gentoo/dev`

### **Entrar al nuevo entorno**

`chroot /mnt/gentoo /bin/bash`

`source /etc/profile`

`export PS1="(chroot) ${PS1}"`

## **4. Actualización del repositorio de Gento**

`emerge-webrsync -v`

`emerge --ask --sync --quiet`

## **5. Zona Horaria**

`echo "America/Mexico_City" > /etc/timezone`

`emerge --config sys-libs/timezone-data`

`echo "es_MX.UTF-8 UTF-8" > /etc/locale.gen`

`locale-gen`

`eselect locale list`

`eselect locale set 4`

**NOTA:** Cambie el valor "4" por el deseado.

## **6. Configurar el Nucleo Linux**

`emerge --ask sys-kernel/gentoo-sources`

Cree un enlace simbólico que punte a las fuentes del núcleo instaladas:

`eselect kernel list`

`eselect kernel set 1`

`echo "sys-kernel/linux-firmware @BINARY-REDISTRIBUTABLE" > /etc/portage/package.license`

`emerge --ask sys-kernel/linux-firmware`

### **Configuración manual del núcleo linux**

`cd /usr/src/linux`

`make menuconfig`

```
-* Sistema de inicio del kernel o init *-
Gentoo Linux --->
   Generic Driver Options --->
      [*] Gentoo Linux support
      [*] Linux dynamic and persistent device naming (userspace devfs) support
      [*] Select options required by Portage features
         Support for init systems, system and service managers --->
            [*] OpenRC, runit and other script based systems and managers
            [ ] systemd
```

```
-* Sistema de archivos para montar dispositivos (Todo es un archivo en Linux) - Precione Shift + /  y escriba CONFIG_DEVMPFS_MOUNT*-
Device drivers
   Generic Drivers Options -->
      [*] Maintain a devtmpfs filesystem to mount at /dev
```

```
-* Soporte de disco SCSI - Precione Shift + /  y escriba CONFIG_BLK_DEV_SD*-
Device drivers
   SCSI device support -->
      <*> SCSI disk support
      <*> SCSI CDROM support
      <*> SCSI generic support

      [ ] SCSI low-level drivers --->

   <*> Serial ATA and Parallel ATA drivers (libata) --->
```

```
-* Sistema de archivos - Precione Shift + /  y escriba CONFIG_EXT3_FS, CONFIG_EXT4_FS, CONFIG_MSDOS_FS, CONFIG_VFAT_FS, CONFIG_PROC_FS, CONFIG_FUSE_FS*-
File systems -->
   <*> Second extended fs support
   <*> The Extended 3 (ext3) filesystem
   <*> The Extended 4 (ext4) filesystem
   [*] Ext4 POSIX Access Control Lists
   [*] Ext4 Security Labels
   <*> FUSE (Filesystem in Userspace) support

   DOS/FAT/EXFAT/NT Filesystems -->
      <*> MSDOS fs support
      <*> VFAT (Windows-95) fs support
      <*> NTFS file system support
      [*] NTFS write support
      
Pseudo Filsystems --->
   [*] /proc file system support
   [*] Tmpfs virtual memory file system support (former shm fs)
```

```
-* Multinúcleo del procesador - Precione Shift + /  y escriba CONFIG_SMP*-
Processor type and features --->
   [*] Symmetric multi-processing support
```

```
-* Cargador EFI y variables EFI en el núcleo linux + /  y escriba CONFIG_PARTITION_ADVANCED, CONFIG_EFI_PARTITION*-
Enable the block layer -->
   Partition Types -->
      [*] Advanced partition selection
      [*] EFI GUID Partition support
```

```
-* Soporte GPT - Precione Shift + /  y escriba CONFIG_EFI, CONFIG_EFI_STUB, CONFIG_EFI_MIXED, CONFIG_EFI_VARS*-
Processor type and features
      [*] EFI runtime service support
      [*]   EFI stub support
      [*]      EFI mixed-mode support

Firmware Drivers
      EFI (Extensible Firmware Interface) Support -->
         <*> EFI Variable Support via sysfs
```

```
-* Emulación IA32 para ejecutar programas de 32 bits - Precione Shift + /  y escriba CONFIG_IA32_EMULATION*-
Binary Emulations
   [*] IA32 Emulation 
```

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
