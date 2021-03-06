![iPERFEX](https://www.iperfex.com/wp-content/uploads/2019/01/iPerfex_logo_naranja-e1546949425459.png)

# Webinar - Como migrar una maquina fisica o virtual en Linux a Docker


[![Watch the video](https://github.com/iperfex-team/como-migrar-una-maquina-fisica-o-virtual-en-linux-a-docker/blob/main/como-migrar-una-maquina-fisica-o-virtual-en-linux-a-docker.png)](https://www.youtube.com/watch?v=V8Uxk5shwHc)

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

**IssabelPBX** - [Descarga](https://www.issabel.org/go/download)


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

instalando Docker

```
curl -fsSL get.docker.com -o get-docker.sh
sh get-docker.sh
```

Configuración 

```
mkdir -p /etc/docker
cat > /etc/docker/daemon.json <<ENDLINE
{
  "bip": "172.17.0.1/24"
}
ENDLINE

systemctl enable docker
systemctl start docker
```

Docker Compose

```
 curl -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose && chmod +x /usr/local/bin/docker-compose

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

## Rsync via SSH

Antes de hacer el rsync podemos eliminar unos paquetes en nuestro IssabelPBX que no veo la necesidad de llevarlo a docker.

Esto queda en cada uno.

```
yum remove -y issabel-email_admin
yum remove -y issabel-fax
yum remove -y issabel-addons
```

Desde el servidor de Debian (el que llamamos docker) seguimos los siguientes pasos:

```
mkdir -p /root/docker/issabelpbx
cd /root
rsync -aAXv --exclude={"/dev/*","/proc/*","/sys/*","/tmp/*","/run/*","/mnt/*","/media/*","/lost+found"} root@172.16.0.81:/ docker/issabelpbx
```

## FIX 

```
cd /root/docker/issabelpbx

#fix httpd
mkdir -p var/run/httpd
chown 0:48 -R var/run/httpd

#fix mariadb
mkdir -p var/run/mariadb
chown 27:27 -R var/run/mariadb

#fix fail2ban
mkdir -p var/run/fail2ban/
```

## Docker import

```
cd /root/docker/issabelpbx
tar -cf- . | docker import --change "EXPOSE 22 80 443 3306 5060/udp 5060/tcp 4569/udp" - issabelpbx
```

## Docker Compose

```
version: '3.1'

services:

  pbx:
   container_name: pbx
   image: issabelpbx:latest
   ports:
     - "4443:443"
     - "2222:22"
     - "3306:3306"
     - "5060:5060"
     - "4569:4569"
   command: bash -c "/usr/sbin/sshd -D & /usr/bin/mysqld_safe & /usr/sbin/httpd -DFOREGROUND & /usr/bin/python2 -s /usr/bin/fail2ban-server -s /var/run/fail2ban/fail2ban.sock -p /var/run/fail2ban/fail2ban.pid -x -b & /usr/sbin/asterisk -U asterisk -G asterisk -mqf -C /etc/asterisk/asterisk.conf"

#   volumes:
#     - ./user-data/mysql:/var/lib/mysql
#     - ./user-data/asterisk:/etc/asterisk
#     - ./user-data/log:/var/log
   restart: always
   privileged: true
```

### Docker persistencia archivos asterisk y mariadb.

```
cd /root/docker
mkdir -p /root/docker/user-data /root/docker/user-data/log

docker cp pbx:/var/lib/mysql ./user-data/mysql
chown 27:27 -R ./user-data/mysql

docker cp pbx:/etc/asterisk ./user-data/asterisk

docker cp pbx:/var/log ./user-data/log
chown 27:27 -R ./user-data/log/mariadb

```

```
version: '3.1'

services:

  pbx:
   container_name: pbx
   image: issabelpbx:latest
   ports:
     - "4443:443"
     - "2222:22"
     - "3306:3306"
     - "5060:5060"
     - "4569:4569"
   command: bash -c "/usr/sbin/sshd -D & /usr/bin/mysqld_safe & /usr/sbin/httpd -DFOREGROUND & /usr/bin/python2 -s /usr/bin/fail2ban-server -s /var/run/fail2ban/fail2ban.sock -p /var/run/fail2ban/fail2ban.pid -x -b & /usr/sbin/asterisk -U asterisk -G asterisk -mqf -C /etc/asterisk/asterisk.conf"

   volumes:
     - ./user-data/mysql:/var/lib/mysql
     - ./user-data/asterisk:/etc/asterisk
     - ./user-data/log:/var/log
   restart: always
   privileged: true
```


## Docker-compose microservicios

```
version: '3.1'

services:

  asterisk:
   container_name: asterisk
   image: issabelpbx:latest
   command: bash -c "/usr/bin/python2 -s /usr/bin/fail2ban-server -s /var/run/fail2ban/fail2ban.sock -p /var/run/fail2ban/fail2ban.pid & /usr/sbin/asterisk -U asterisk -G asterisk -mqf -C /etc/asterisk/asterisk.conf"
   volumes:
     - ./user-data/asterisk:/etc/asterisk
     - ./user-data/log:/var/log
   depends_on:
     - mariadb
   restart: always
   privileged: true
   network_mode: host

  mariadb:
   container_name: mariadb
   image: issabelpbx:latest
   command: bash -c "/usr/bin/mysqld_safe"
   volumes:
     - ./user-data/mysql:/var/lib/mysql
     - ./user-data/log:/var/log
   restart: always
   privileged: true
   network_mode: host

  apache:
   container_name: apache
   image: issabelpbx:latest
   command: bash -c "/usr/sbin/httpd -DFOREGROUND"
   volumes:
     - ./user-data/log:/var/log
   depends_on:
     - mariadb
     - asterisk
   restart: always
   privileged: true
   network_mode: host


#  ssh:
#   container_name: ssh
#   image: issabelpbx:latest
#   command: bash -c "/usr/sbin/sshd -D"
#   volumes:
#     - ./user-data/log:/var/log
#   restart: always
#   privileged: true
#   network_mode: host
```
