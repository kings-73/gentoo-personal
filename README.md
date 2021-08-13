# gentoo-personal
Guía de instalación.
Modo UEFI
Gestor de servicios: OpenRC

## **1. Preparando el disco**

Antes de particionar verifique el disco correcto a particionar.
`lsblk`

En este caso seleccionaremos el disco /dev/sda. Para particionarlo escriba el siguiente comando:
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
