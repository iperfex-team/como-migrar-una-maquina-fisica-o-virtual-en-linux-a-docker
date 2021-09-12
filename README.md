## Máquina virtual

Para implementar nuestro entorno de desarrollo vamos a necesitar un sistema de virtualización.  

Usaremos **Virtualbox v6.1** que es Multiplataforma y es OpenSource. 

Los requisitos de la configuración de las dos máquina virtual son: 
 
 * **2 CPU**
 * **4 GB RAM** 
 * **20 GB Disco duro virtual** 
 * **1 PLACA modo Bridge** 


**Aclaración**: puede usar cualquier otro sistema de virtualización que desee sin afectar el resultado. Correrá por su cuenta la configuración para armar su entorno de desarrollo. Tiene que contemplar que dicha máquina virtual esté configurada en modo bridge para poder asignar una IP de su misma subnet y poder trabajar de forma remota. 

**VirtualBox** - [Descarga](https://www.virtualbox.org/wiki/Downloads)

**Debian** - [Descarga](https://www.debian.org/download)

**WordPress Virtual Machine** - [Descarga](https://bitnami.com/stack/wordpress/virtual-machine)


## Configuración VirtualBox Debian
 
Lo primero que debemos hacer es abrir VirtualBox y darle al botón **_Nueva_**. De esta forma comenzaremos el asistente de creación de la máquina virtual.



### Instalación Paquetes

Nos conectamos al servidor utilizando algún cliente SSH e instalamos los siguientes paquetes.

Lista de paquetes a instalar:

* vim
* curl
* screen
* mc
* git
* unzip
* net-tools
* links2
* sudo
* nmap
* make
* mycli
* rsync
* docker
* docker-compose

Ejecutamos los siguientes comandos.

```bash
su
apt update
apt -y install vim curl screen mc git unzip net-tools links2 sudo nmap make mycli rsync
```


### Acceso ssh con root (por default esta desactivado)

```bash
sed -i 's/#PermitRootLogin prohibit-password/PermitRootLogin yes/' /etc/ssh/sshd_config
/etc/init.d/ssh restart
```

### Personalizando el Prompt

```bash 
> /root/.bashrc && vim /root/.bashrc
````

Pegamos lo siguiente:

```bash
# ~/.bashrc: executed by bash(1) for non-login shells.

# Note: PS1 and umask are already set in /etc/profile. You should not
# need this unless you want different defaults for root.
PS1='\[\033[1;36m\]\u\[\033[1;31m\]@\[\033[1;32m\]\h:\[\033[1;35m\]\w\[\033[1;31m\]\$\[\033[0m\] '
# PS1='${debian_chroot:+($debian_chroot)}\h:\w\$ '
# umask 022

# You may uncomment the following lines if you want `ls' to be colorized:
export LS_OPTIONS='--color=auto'
eval "`dircolors`"
alias ls='ls $LS_OPTIONS'
alias ll='ls $LS_OPTIONS -l'
alias l='ls $LS_OPTIONS -lA'

#
# Some more alias to avoid making mistakes:
# alias rm='rm -i'
# alias cp='cp -i'
# alias mv='mv -i'

export HISTTIMEFORMAT="%d/%m/%y %T "

export LC_CTYPE=C
export LC_MESSAGES=C
export LC_ALL=C
export PATH=$PATH:/usr/local/sbin:/usr/sbin:/sbin
```
### Recargamos el /root/.bashrc

```bash
source /root/.bashrc
```
