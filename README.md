# gentoo-personal
Esta es mi guía personal e informal de instalación de Gentoo en mi PC de escritorio. Por lo tanto, no la recomiendo para uso personal sino para fines educativos. Es decir, esta guía provee un aporte de como alguien realiza la instalación Gentoo en su equipo.

Hardware:

* Tarjeta Madre B460M Aorus Elite
* Procesador: Intel core i3 10100
* Memoria RAM: 8GB
* SSD: Western Digital M.2 240 GB
* Tarjeta de red inalámbrica PCI-Express (RTL8192EE)

Configuración Gentoo:
* Perfil: Por defecto
* Init: OpenRC (Por defecto)
* Formateo del Disco: GPT
* Arranque GRUB: EFI

## **☑ 0. Conexión a Wifi**
Esta guía supone que la tarjeta de red se encuentra dentro de los drivers disponibles en el CD de intalación de Gentoo.

Tenga en cuenta que para conectarse a una red Wi-Fi necesita conocer al menos 3 datos:
1. Nombre del dispositivo de red inalámbrica (INTERFAZ)
2. El nombre de nuestra red Wi-Fi (SSID)
3. La contraseña de red (PASSWOWRD)

El  nombre de la red y la contraseña son proporcionados por su proveedor de internet. Sin embargo, es posible obtener el nombre de la interfaz de red con el siguiente comando:

`iwconfig` o `ifconfig -a`

Una vez que se dispone del nombre de la interfaz puede conectarse a la red inalámbrica de la siguiente manera:

`wpa_supplicant -B -i INTERFAZ -c<(wpa_passphrase SSID PASSWORD)`

**NOTA:** Recuerde cambiar la palabra INTERFAZ por el nombre del dispositivo de red que obtuvo con el comando ifconfig. No olvide también cambiar el nombre de la red (SSID) y su respectiva contraseña.

Por último, verifique la conexión a internet de la siguiente manera:

`ping c3 www.gentoo.org`

## **☑ 1. Preparación el disco para alojar la intalación Gentoo**

Para mi intalación personal usaré el disco: /dev/sda. Para particionarlo usaré siguiente comando:

`cfdisk /dev/sda`

**NOTA:** Si el disco no cuenta con tabla de particiones seleccione GPT (GUID Partition Table).

```
dev/sda1; Tipo: Efi system; Tamaño: 150M
dev/sda2; Tipo: Linux file system; Tamaño: 20G
dev/sda3; Tipo: Linux file system; Resto del disco
```


### **Creación y monteje sistemas de archivos**

`mkfs.vfat -F 32 /dev/sda1`

`mkfs.ext4 /dev/sda2`

`mkfs.ext4 /dev/sda3`

### **Montaje de sistemas de archivos**

`mount /dev/sda2 /mnt/gentoo`

`mkdir /mnt/gentoo/home`

`mount /dev/sda3 /mnt/gentoo/home`

## **2. Descargar y extraer el stage3**

Verifique que la fecha y hora sean correctas:

`date`

En caso de ser necesario, cambie la fecha y hora usando el mismo comando con el siguiente formato:

`date 081310002021`

**NOTA:** El formato para introducir la fecha es: MMDDhhmmYYYY. En este ejemplo se setea: Agosto 13 de 2021, Hora: 10:00 p.m.

`cd /mnt/gentoo`

`links www.gentoo.org/downloads`

`ls`

`tar -xpf stage3-amd64-openrc-*.tar.xz --numeric-owner --xattrs-include="*.*"`

**NOTA:** Sustituya el stage por el más actual llendo a la página: www.gentoo.org/downloads.

### **Modificación del archivo de opciones de compilación**

`nano -w /mnt/gentoo/etc/portage/make.conf`

```
-* archivo make.conf *-
COMMON_CFLAGS="-march=native -O2 -pipe"
MAKEOPTS="-j4"

GRUB_PLATFORMS="efi-64"
VIDEO_CARDS="intel i965 iris"

L10N="es-MX es"
USE="X alsa bindist dbus egl elogind introspection networkmanager policykit -systemd udev udisks wayland mmx mmxext sse sse2"
```

## **☑ 3. Enjaulamiento**

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

`mount /dev/sda1 /boot`

## **☑ 4. Actualización del repositorio de Gento**

`emerge-webrsync -v`

`emerge --ask --sync --quiet`

`df`

`emerge -av dev-lang/rust`

`emerge -av dev-lang/spidermonkey`

`emerge --ask --verbose --update --deep --newuse @world`

## **☑ 5. Zona Horaria**

`echo "America/Mexico_City" > /etc/timezone`

`emerge --config sys-libs/timezone-data`

`echo "es_MX.UTF-8 UTF-8" > /etc/locale.gen`

`locale-gen`

`eselect locale list`

`eselect locale set 4`

**NOTA:** Cambie el valor "4" por el deseado.

## **6. Configuración manual del Nucleo Linux**

`emerge --ask sys-kernel/gentoo-sources`

Cree un enlace simbólico que apunte a las fuentes del núcleo instaladas:

`eselect kernel list`

`eselect kernel set 1`

### **Configuraciones necesarias**

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
-* Sistema de archivos para montar dispositivos (Todo es un archivo en Linux) - Precione Shift + /  y escriba CONFIG_DEVTMPFS_MOUNT *-
Device drivers  --->
     Generic Drivers Options -->
          [*] Maintain a devtmpfs filesystem to mount at /dev
```

```
-* Soporte de disco SCSI - Precione Shift + /  y escriba CONFIG_BLK_DEV_SD*-
Device drivers  --->
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

```
-* Dispositivos de entrada USB - Precione Shift + /  y escriba CONFIG_HID_GENERIC, CONFIG_USB_HID, CONFIG_USB_SUPPORT, CONFIG_USB_XHCI_HCD, CONFIG_USB_EHCI_HCD, CONFIG_USB_OHCI_HCD *-
*-
Device Drivers --->
     HID support --->
          -*- HID bus support
          <*> Generic HID driver
          [*] Battery level reporting for HID devices
              USB HID support --->
                   <*> USB HID transport layer
          [*] USB support --->
              <*> xHCI HCD (USB 3.0) support
              <*> EHCI HCD (USB 2.0) support
              < > OHCI HCD (USB 1.0) support
```

### **Configuraciones específicas**

```
-* Gráficos Intel *-
Device Drivers  --->
     Graphics support  --->
          <*> /dev/agpgart (AGP Support)  --->
          --- /dev/agpgart (AGP Support)
          -*-   Intel 440LX/BX/GX, I8xx and E7x05 chipset support
          <*> Direct Rendering Manager (XFree86 4.1.0 and higher DRI support)  --->
              --- Direct Rendering Manager (XFree86 4.1.0 and higher DRI support)
                   [*]   Enable legacy fbdev support for your modesetting driver
                   <*> Intel 8xx/9xx/G3x/G4x/HD Graphics
                   [ ]   Enable alpha quality support for new Intel hardware by default
                   ()    Force probe driver for selected new Intel hardware
                   [*]   Enable capturing GPU state following a hang
                   [*]     Compress GPU error state
                   [*]   Always enable userptr support
                   [ ]   Enable Intel GVT-g graphics virtualization host support
```

```
-* Tarjeta de Red Ethernet *-
Device Drivers  --->
     Networking device support  --->
          [*] Network device support --->
               [*]   Ethernet driver support  --->
	            < >     Intel(R) PRO/100+ support
                    < >     Intel(R) PRO/1000 Gigabit Ethernet support
                    <M>     Intel(R) PRO/1000 PCI-Express Gigabit Ethernet support
                    [*]     Support HW cross-timestamp on PCH devices
```

```
-* Tarjeta de WIFI *-
Device Drivers  --->
     [*] Network device support  --->
          [*] Wireless LAN  --->
	       [*]   Realtek devices
                    < >     Realtek 8180/8185/8187SE PCI support
                    < >     Realtek 8187 and 8187B USB support
                    <M>     Realtek rtlwifi family of devices  --->
                         <M>   Realtek RTL8192EE Wireless Network Adapter
```

```
-* Tarjeta de sonido *-
Device Drivers --->
     <*> Sound card support
          <*> Advanced Linux Sound Architecture --->
	      HD-Audio --->
               <*> HD Audio PCI
               [*] Build hwdep interface for HD-audio driver
               [ ] Allow dynamic codec reconfiguration
               [ ] Support digital beep via input layer
               [ ] Support initialization patch loading for HD-audio
               <*> Build Realtek HD-audio codec support
               < > Build Analog Devices HD-audio codec support
               < > Build IDT/Sigmatel HD-audio codec support
               < > Build VIA HD-audio codec support
               <*> Build HDMI/DisplayPort HD-audio codec support                         
```

```
-* Internet móvil (USB tethering) - Precione Shift + /  y escriba CONFIG_USB_USBNET *-
Device Drivers --->
     [*] Network device support --->
          <M> USB Network Adapters --->
               <M> Multi-purpose USB Networking Framework
                    <M>  CDC Ethernet support (smart devices such as cable modems)
                    <M>  CDC EEM support
		    <M>  Host for RNDIS and ActiveSync devices
                    <M>  Simple USB Network Links (CDC Ethernet subset)
                     [*] Embedded ARM Linux links (iPaq, ...)
```

```
-* Transferencia de archivos Android - Precione Shift + /  y escriba CONFIG_FUSE_FS *-
File systems  ---> 
     <*> FUSE (Filesystem in Userspace) support
```

`make && make modules_install`

`make install`

`echo "sys-kernel/linux-firmware @BINARY-REDISTRIBUTABLE" > /etc/portage/package.license`

`emerge --ask sys-kernel/linux-firmware`

### **☑ 7. Archivos de configuración**

`nano -w /etc/fstab`

```
-* archivo fstab *-
/dev/sda1  boot  vfat  noatime  0 0
/dev/sda2  /     ext4  noatime  0 1
/dev/sda3  /home ext4  noatime  0 0
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

### **☑ 8. Instalacion de GRUB EFI**

`emerge -av sys-boot/grub:2`

`grub-install --target=x86_64-efi --efi-directory=/boot`

`grub-mkconfig -o /boot/grub/grub.cfg`

### **9. Instalar Herramientas**

`emerge --ask --verbose --update --deep --newuse @world`

`emerge -av sys-fs/dosfstools`

`emerge -av net-misc/networkmanager`

`emerge -av gentoolkit`

### **☑ 10. Password ROOT y salir del sistema**

`passwd`

`exit`

`cd`

`umount -l /mnt/gentoo/dev{/shm,/pts,} `

`umount -R /mnt/gentoo`

`reboot`

**¡Felicidades, ya has instalado gentoo!**
