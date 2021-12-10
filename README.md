# Guia Personal Gentoo
Guía personal e informal de instalación de Gentoo.

Hardware:

* Tarjeta Madre B460M Aorus Elite
* Procesador: Intel core i3 10100
* Memoria RAM: 8GB
* SSD: Western Digital M.2 240 GB
* Tarjeta de red inalámbrica PCI-Express (RTL8192EE)

Configuración Gentoo:
* Perfil: Gnome
* Init: OpenRC (Por defecto)
* Formateo del Disco: GPT
* Arranque GRUB: EFI

## **☑ 0. Conexión a Wifi**

Esta guía supone que la tarjeta de red se encuentra dentro de los drivers disponibles en el CD de intalación de Gentoo.

Tenga en cuenta que para conectarse a una red Wi-Fi necesita conocer al menos 3 datos:

1. Nombre del dispositivo de red inalámbrica (INTERFAZ)
2. El nombre de nuestra red Wi-Fi (SSID)
3. La contraseña de red (PASSWOWRD)

El  nombre de la red y la contraseña son proporcionados por su proveedor de internet. Sin embargo, para obtener el nombre de la interfaz de red con el siguiente comando:

`iwconfig` o `ifconfig -a`

Una vez que se dispone del nombre de la interfaz puede conectarse a la red inalámbrica de la siguiente manera:

`wpa_supplicant -B -i INTERFAZ -c<(wpa_passphrase SSID PASSWORD)`

**NOTA:** Recuerde cambiar la palabra INTERFAZ por el nombre del dispositivo de red que obtuvo con el comando ifconfig. No olvide también cambiar el nombre de la red (SSID) y su respectiva contraseña.

Por último, verifique la conexión a internet de la siguiente manera:

`ping c3 www.gentoo.org`

## **☑ 1. Preparación el disco para alojar la intalación Gentoo**

Particionamiento mediante la aplicaci

`cfdisk /dev/sda`

**NOTA:** Si el disco no cuenta con tabla de particiones seleccione GPT (GUID Partition Table).

```
dev/sda1; Tipo: Efi system; Tamaño: 150M
dev/sda2; Tipo: Linux file system; Tamaño: 15G
dev/sda3; Tipo: Linux file system; Resto del disco
```


### **Creación y montaje sistemas de archivos**

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

`tar -xvpf stage3-amd64-openrc-*.tar.xz --numeric-owner --xattrs-include="*.*"`

**NOTA:** Sustituya el * por el nombre completo del stage.

### **Modificación del archivo de opciones de compilación**

`nano -w /mnt/gentoo/etc/portage/make.conf`

```
-* archivo make.conf *-
COMMON_CFLAGS="-march=skylake -O2 -pipe"
CPU_FLAGS="aes avx avx2 f16c fma3 mmx mmxext pclmul popcnt sse sse2 sse3 sse4_1 sse4_2 ssse3"
MAKEOPTS="-j4"

GRUB_PLATFORMS="efi-64"
VIDEO_CARDS="intel i965 iris"

L10N="es-MX es"
USE="X alsa elogind networkmanager -systemd"
```

## **☑ 3. Enjaulamiento**

Antes de montar los sistemas de archivos necesarios, copie la información de los DNS para asegurar que la red siga funcionando después de entrar al nuevo entorno.

`cp --dereference /etc/resolv.conf /mnt/gentoo/etc/`

### ** Montaje los sistemas de archivos necesarios**

`mount --types proc /proc /mnt/gentoo/proc`

`mount --rbind /sys /mnt/gentoo/sys`

`mount --make-rslave /mnt/gentoo/sys`

`mount --rbind /dev /mnt/gentoo/dev`

`mount --make-rslave /mnt/gentoo/dev`

### ** Entrar al nuevo entorno**

`chroot /mnt/gentoo /bin/bash`

`source /etc/profile`

`export PS1="(chroot) ${PS1}"`

`mount /dev/sda1 /boot`

## **☑ 4. Actualización del repositorio y selección de un perfil Gentoo**

`emerge-webrsync -v`

`emerge --ask --sync --quiet`

`eselect profile list`

`eselect profile set X`

**NOTA:** Sustituya la X por el número del perfil deseado.

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

## **6. Configuración del Nucleo Linux**

`emerge --ask sys-kernel/gentoo-sources`

Cree un enlace simbólico que apunte a las fuentes del núcleo instaladas:

`eselect kernel list`

`eselect kernel set 1`

### **Configuraciones necesarias**

`cd /usr/src/linux`

`make menuconfig`

Véa la configuración del kernel aquhttps://github.com/kings-73/Kernel-Linux

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

`emerge -av sys-boot/grub:2 emerge -av sys-fs/dosfstools`

`grub-install --target=x86_64-efi --efi-directory=/boot`

`grub-mkconfig -o /boot/grub/grub.cfg`

### **9. Instalar Herramientas Adicionales**

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
