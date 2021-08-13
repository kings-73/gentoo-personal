# gentoo-personal
Guía de instalación personal desde una distro existente. Algunas características son las siguientes:

Modo UEFI

Gestor de servicios: OpenRC

Perfil estándar


## **1. Preparando el disco**

Para mi intalación personal usaré el disco: /dev/sda. Para particionarlo usaré siguiente comando:

`cfdisk /dev/sda`

**NOTA:** Si el disco no cuenta con tabla de particiones seleccione GPT (GUID Partition Table).

```
dev/sda1; Tipo: Efi system; Tamaño: 150M
dev/sda2; Tipo: Linux file system; Tamaño: Resto del disco
```


### **1.1. Creación y monteje sistemas de archivos**

`mkfs.vfat -F 32 /dev/sda1`

`mkfs.ext4 /dev/sda2`

### **1.2. Montaje de sistemas de archivos**

`mkdir /mnt/gentoo`

`mount /dev/sda2 /mnt/gentoo`

`mkdir /mnt/gentoo/boot`

`mount /dev/sda1 /mnt/gentoo/boot`
