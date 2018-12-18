### Instanciando una máquina virtual.

Para realizar los ejercicios de este tema es necesario crear, en primer lugar, una máquina virtual con un determinado sistema operativo instalado en ella. En este caso, vamos a crear una máquina virtual con Ubuntu Server 18.04 LTS. Elegimos la última versión LTS de ubuntu server, pues incluye un mantenimiento durante 5 años por parte de Canonical.

Por otro lado, elegimos la versión "server" en lugar de "desktop", pues la primera ya trae instalado open-ssh, la versión de python 3.6.7 (necesaria para Ansible) y muchas otras utilidades. Además, no dispone de IU, que no es recomendable para MV en las cuales vamos a desplegar aplicaciones web.

Esta máquina virtual se ha creado en VirtualBox 4.3. Una vez creada, tendremos que configurar el adaptador de red utilizado para poder conectarnos a nuestra MV mediante ssh de cara a realizar el provisionamiento con Ansible. Para ello, en la configuración del la MV en VirtualBox vamos a Red -> Adaptador 1. Una vez allí ponemos seleccionamos el desplegable junto a "Conectado a:" y elegimos la opción adaptador puente. Tras esto, si estamos conectados al router por wifi, habrá que poner la opción "Nombre:" a wlan0 y si estamos conectados al router por cable habrá que ponerla a eth0.

Esto hace, que la máquina virtual se comporte en nuestra red local como un equipo físico más independiente del sistema anfitrión. Por tanto, tendrá su propia IP local, a través de la cual podremos conectarnos a la MV.


### Ejercicio 2: Instalar los prerrequisitos para ejecutar alguna aplicación propia (la del proyecto de la asignatura u otra) usando Ansible.

En primer lugar debemos de instalar Ansible en el sistema anfitrión mediante los siguientes comandos:

~~~
$ sudo apt-get update
$ sudo apt-get install software-properties-common
$ sudo apt-add-repository ppa:ansible/ansible
$ sudo apt-get update
$ sudo apt-get install ansible
~~~

A continuación debemos de añadir la MV creada anteriormente al inventario de Ansible. El inventario por defecto de Ansible se encuentra en /etc/ansible/hosts. Lo abrimos y añadimos la línea siguiente.

~~~
[webservers]
ubuntu_server ansible_host=192.168.1.130 ansible_port=22
~~~

De esta forma, ponemos bajo la gestión de Ansible a nuestra máquina virtual, a la cual la hemos llamado ubuntu_server y le hemos especificado a ansible la dirección IP y el puerto para que se pueda conectar con ella a través de ssh.

Antes de comprobar si funciona, Ansible necesita que en la MV esté instalado python 2.7. Por tanto, lo instalamos ejecutando el siguiente comando en la MV.

~~~
$ sudo apt-get install python
~~~

Además, en el fichero /etc/ansible/ansible.cfg hay que descomentar la línea "host_key_cheking=False" para evitar el chequeo de la MAC de la MV cada vez que hagamos ssh a ella.

Antes de acceder con ansible mediante ssh a la MV, debemos de crear un par clave pública/clave privada, de forma que la clave privada la ponemos en el sistema anfitrión en ~/.ssh/ en el home del usuario que vaya a ejecutar Ansible. Mientras que la clave publica tendremos que copiarla en la MV, en el archivo ~/.ssh/authorized_keys, donde ~ es el home del usuario con el que queremos iniciar sesión en la MV cuando nos conectemos mediante ssh desde el sistema anfitrión. En este caso, para el provisionamiento vamos iniciar sesión en la MV con el usuario root, el cual tiene permisos de superusuario. Por tanto, copiamos la clave pública en el archivo authorized_keys que se encuentra en el home del usuario root en la MV.

Para probar que Ansible puede acceder mediante ssh a la MV, ejecutamos desde la máquina anfitriona

~~~
$ ansible all -u root -m ping
~~~

Debemos de obtener el siguiente resultado si todo va bien.

~~~
ubuntu_server | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
~~~

Una vez configurado todo, vamos a instalar mediante comandos con ansible todo lo necesario para poder ejecutar la aplicación del proyecto de cloud computing.

En primer lugar instalamos pip.

~~~
ansible ubuntu_server -u root -m command -a "apt install -y python-pip"
~~~

Ahora instalamos git.

~~~
ansible ubuntu_server -u root -m command -a "apt install -y git"
~~~

Instalamos pipenv.

~~~
ansible ubuntu_server -u root -m command -a "pip install pipenv"
~~~

Instalamos la libreria de python3 distutils.

~~~
ansible ubuntu_server -u root -m command -a "apt install python3-distutils"
~~~

Clonamos el proyecto.

~~~
ansible ubuntu_server -u root -m git -a "repo=https://github.com/mesagon/Proyecto-CC-MII.git dest=/home/jesus/"
~~~

Instalamos el resto de dependencias con pipenv.

~~~
ansible ubuntu_server -u root -m shell -a "cd /home/jesus/prueba && pipenv install"
~~~

Lanzar la aplicación.

~~~
ansible ubuntu_server -u root -m shell -a "pipenv run gunicorn -D -b 0.0.0.0:8000 app:app chdir=/home/jesus/prueba"
~~~

### Ejercicio 3: Desplegar la aplicación que se haya usado anteriormente con todos los módulos necesarios usando un playbook de Ansible.

Vamos a crear un playbook de ansible que nos permita instalar en la máquina virtual todas las dependencias del ejercicio anterior con una sola orden. El playbook sería el siguiente.

~~~
---
 - hosts: webservers
   user: jesus
   tasks:

   - name: Instalar python 2.7.
     become: yes
     become_user: root		
     apt: pkg=python state=present

   - name: Instalar python 3.6.
     become: yes
     become_user: root
     apt: pkg=python3.6 state=present

   - name: Instalar pip.
     become: yes
     become_user: root		
     apt: pkg=python-pip state=present

   - name: Instalar git.
     become: yes
     become_user: root		
     apt: pkg=git state=present

   - name: Instalar pipenv.
     become: yes
     become_user: root		
     pip: name=pipenv

   - name: Instalar distutils.
     become: yes
     become_user: root		
     apt: pkg=python3-distutils state=present

   - name: Clonar proyecto.
     become: yes
     become_user: root		
     git: repo=https://github.com/mesagon/Proyecto-CC-MII.git dest=/home/jesus/prueba force=yes

   - name: Establecer entorno virtual.
     become: yes
     become_user: root
     shell: pipenv install
     args:
       chdir: /home/jesus/prueba

   - name: Desplegar aplicacion.
     become: yes
     become_user: root
     shell: pipenv run gunicorn -D -b 0.0.0.0:8000 app:app
     args:
       chdir: /home/jesus/prueba
~~~

Como vemos, el playbook consta de un conjunto de tareas las cuales consisten en la instalación de cada una de las dependencias necesarias para desplegar la aplicación. Además, vemos como hay una última tarea que se encarga de realizar el despliegue de la aplicación mediante gunicorn en la MV.       
