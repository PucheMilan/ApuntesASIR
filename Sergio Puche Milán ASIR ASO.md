Sergio Puche Milán ASIR ASO



Instalación de double boot Kali Linux y Arch Linux


Índice instalación del  S.O
Kali Linux	3
Captura del S.O totalmente operativo	4
Captura adjunta	5
Arch Linux	6
Instalación por línea de comandos:	6
Creación de particiones de disco	7
Configuración de la localización	10
Configuración de la red	11
Instalar GRUB en el directorio efi	11
Crear el archivo de configuración	11
Habilitar los servicios	11
Creación de usuario	12
Salir de la instalación y reiniciar la máquina	12
Instalación completamente funcional de Arch Linux junto con Kali Linux en un doble bootloader	12


Kali Linux


Kali Linux, también conocida como "hacker Linux", es una distribución basada en Debian GNU/Linux, diseñada principalmente para la auditoría y seguridad informática en general
Kali Linux, trae más de 600 programas preinstalados, entre otros:
    • Maltego, un programa para recopilar datos sobre personas o negocios en Internet. 
    • Nmap, un escáner de puertos. 
    • Kismet, un sniffer pasivo para detectar redes inalámbricas. 
    • Wireshark para analizar el tráfico de datos. 
    • John the Ripper, una herramienta para desencriptar contraseñas.
    • Ingeniería inversa, un tema muy interesante que detallo a continuación:
La ingeniería inversa sirve para obtener información de un producto, cómo ha sido construido o cómo funciona por dentro.

El objetivo de la ingeniería inversa es obtener información o un diseño a partir de un producto accesible al público, con el fin de determinar de qué está hecho, qué lo hace funcionar y cómo fue fabricado. Hoy en día los productos más comúnmente sometidos a ingeniería inversa son los programas. 

Sirve para:
    • Clasificar Malware
    • Liberar drivers/hardware 
    • Analizar vulnerabilidades 
    • Hacker crackmes/ctf (Capture the Flag) 
    • Contalidad con software 
    • Espionaje industrial 


Captura del S.O totalmente operativo




Es una instalación sencilla ya que el propio S.O viene preparado con todos los paquetes para trabajar con él.
Ahora procedemos a instalar los dos S.O en una DualBoot. Para ello instalamos antes Kali Linux ya que es más sencillo de instalar que Arch Linux y siempre se puede recuperar mejor que el propio Arch Linux.

Captura adjunta


Arch Linux

Archlinux es un modelo de lanzamiento continuo respaldado por pacman, un gestor de paquetes ligero, sencillo y rápido, que permite actualizar de forma continuada todo el sistema con una orden lo cual es muy cómodo si se sabe utilizar adecuadamente.

Para instalar este S.O hemos utilizado una guía ejecutando paso a paso la línea de comandos para conformar el S.O, desde asignar particiones, montar tabla de particiones GPT...etc, todo por línea de comandos.

Instalación por línea de comandos:
Configuración del teclado
	loadkeys es

Verificación de  UEFI en la máquina virtual
	ls /sys/firmware/efi/efivars

Sincronización del reloj

	timedatectl set-ntp true

Selección del servidor más cercano  instala el paquete reflector
	sudo pacman -Syy reflector

Buscar el servidor más cercano
	reflector --list-countries | more

Identificación de país, pondremos en este caso España

	reflector -c "United States" -a 6 --sort rate --save /etc/pacman.d/mirrorlist

Actualización de los servidores de paquetes
	pacman -Syyy

Creación de particiones de disco
	lsblk

Para crear una tabla de particiones se utiliza el comando cfdisk y seleccionaremos gpt
La primera partición será de 250M dónde cargaremos la EFI y asignaremos EFI System
Los gigas restantes se asignan a Linux filesystem

Verificación y formateo de las particiones:
La primera que tiene 250M será tipo FAT32
	mkfs.fat -F32 /dev/sda1

La siguiente será ext4
	mkfs.ext4 /dev/sda2

Para montar las particiones
	mount /dev/sda2 /mnt

Creación  del directorio dónde estará la partición de inicio.
	mkdir -p /mnt/boot/efi

Montar el directorio
	mount /dev/sda1 /mnt/boot/efi

 Verificación con lsblk e instalación de los paquetes base
	pacstrap /mnt base linux linux-firmware vim

Cuando la instalación base termine se instalará el archivo fstab
	genfstab -U /mnt >> /mnt/etc/fstab
Y después el archivo swap, pero primero se entra en la instalación base 
	arch-chroot /mnt

Creación del archivo swap utilizando el comando dd, el archivo que se creará será de 2GB que estará dividido en dos bloques de 1GB
	dd if=/dev/zero of=/swapfile bs=1G count=2 status=progress

Cambiar los permisos del archivo
	chmod 600 /swapfile

Y cambiar el tipo de archivo a swap
	mkswap /swapfile

Se habilita con
	swapon /swapfile

Se agrega el archivo al sistema de archivos es decir al fstab (utilizando vim)
	vim /etc/fstab

Justo al final de archivo agregar lo siguiente:
	/swapfile none swap defaults 0 0

Para salir de la consola pulsa esc y ctrl+z para volver la instalación.
Configuración de zona horaria
	timedatectl list-timezones | grep España

Creación de un enlace simbólico en etc/localtime

	ln -sf /usr/share/zoneinfo/America/Chicago /etc/localtime

Para sincronizar con el reloj del hardware

	hwclock --systohc

Configuración de la localización

Entrar de nuevo en vim

	vim /etc/locale.gen
	
Comentar la línea en la que aparezca es_MX.UTF-8 

Generar una localización del sistema

	locale-gen

Creación del archivo de configuración en la cuenta de instalación

	echo LANG=es_MX.UTF-8  >> /etc/locale.conf



Configuración de la red

Creación del archivo hostname  y le dará un nombre a la máquina, archlinux

	vim /etc/hostname

Justo al final se lo siguiente:

	vim /etc/hosts

	127.0.0.1	localhost
	::1		localhost
	127.0.1.1	archlinx.localdomain	archlinux

Crear un password para root

Declar passwd, y escribir el nuevo password

Instalando los paquetes finales

Utilización GRUB para que sea  boot-loader

Agregar los siguientes paquetes:

pacman -S grub efibootmgr networkmanager network-manager-applet dialog os-prober mtools dosfstools base-devel linux-headers cups reflector openssh git xdg-utils xdg-user-dirs virtualbox-guest-utils

Instalar GRUB en el directorio efi

	grub-install --target=x86_64-efi --efi-directory=/boot/efi --botloader-id=GRUB

Crear el archivo de configuración
	grub-mkconfig -o /boot/grub/grub.cfg

Habilitar los servicios
	systemctl enable NetworkManager
	systemctl enable sshd
	systemctl enable org.cups.cupsd


Creación de usuario
	useradd -mG wheel sergio
	passwd sergio

Creación permisos de los super usuario, para ello entrar otra vez en la consola y escribir la siguiente línea de comando:

	EDITOR=vim visudo
Salir de la instalación y reiniciar la máquina
	exit
	umount -a
	reboot

Instalación completamente funcional de Arch Linux junto con Kali Linux en un doble bootloader
