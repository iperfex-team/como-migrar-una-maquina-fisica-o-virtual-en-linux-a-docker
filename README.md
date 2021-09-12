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
