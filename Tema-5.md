### Ejercicio 1: Instalar una máquina virtual Debian usando Vagrant y conectar con ella.

En primer lugar debemos elegir una box que contenga una imagen de Debian. En este caso, se ha elegido una box con Debian 8 de esta [página](http://www.vagrantbox.es/). A continuación, añadimos la box utilizando su enlace a nuestro gestor de máquinas virtuales mediante el comando.

~~~
$ vagrant box add Debian8 https://github.com/kraksoft/vagrant-box-debian/releases/download/8.0.0/debian-8.0.0-amd64.box
~~~

Este comando nos añade la box elegida a nuestro gestor de máquinas virtuales (Virtualbox en nuestro caso) a partir del enlace de dicha box. Por otra parte, vemos como antes del enlace a la caja hemos puesto Debian8. Esto es un alias para identificar posteriormente a la box.

Ahora nos situamos en un directorio vacío, y ejecutamos el comando siguiente.

~~~
$ vagrant init Debian8
~~~

Este comando nos creará en el directorio actual un fichero llamado "Vagrantfile", el cual está escrito en Ruby y contiene toda la configuración necesaria para que Vagrant pueda crear una MV con la box "Debian8" que añadimos previamente.

Antes de crear la MV, hemos tenido que instalar el siguiente plugin para que vagrant reconozca las guest additions en las boxes de virtualbox.

~~~
$ vagrant plugin install vagrant-vbguest
~~~

Tras esto, "levantamos" la MV definida en el vagrant file mediante el comando siguiente.

~~~
$ vagrant up
~~~

Una vez hecho esto, nos conectamos a dicha máquina virtual mediante el siguiente comando de forma directa.

~~~
$ vagrant ssh
~~~

### Ejercicio 3: Crear un script para provisionar de forma básica una máquina virtual para el proyecto que se esté llevando a cabo en la asignatura.

Para llevar a cabo este ejercicio se va a elegir una caja con debian 9, que fue SO elegido para nuestro microservicio de gestión de clientes en el hito 4. Esta caja se encuentra alojada en Vagrant Cloud bajo el alias generic/debian9, por tanto, no necesitamos añadirla explícitamente con $ vagrant box add, puesto que podemos indicar directamente que vamos a utilizarla en el vagrantfile y vagrant la buscará automáticamente en su nube, la descargará y la añadirá automáticamente cuando "levantemos" la MV.

Dicho esto, nos situamos en un directorio vacío y creamos en él un vagrantfile que defina una máquina virtual por defecto que utilice la caja generic/debian9. Esto lo hacemos con el siguiente comando.

~~~
$ vagrant init generic/debian9
~~~

Tras ejecutar el comando, se nos notificará de que en el directorio actual se ha creado un vagrantfile. Lo abrimos para añadir la configuración del provisionamiento. De esta forma el vagrantfile nos queda del siguiente modo.

 ~~~
 # -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|

  config.vm.box = "generic/debian9"

  config.vm.provision "shell", inline: <<-SHELL

     sudo su
     sudo echo "deb http://ftp.de.debian.org/debian testing main" >> /etc/apt/sources.list
     sudo apt-get update
     sudo apt-get -y install python-apt
     sudo apt-get -y install python
     sudo apt-get -y install python3.6
     sudo apt-get -y install python3-pip
     sudo apt-get -y install git
     sudo pip3 install pipenv
     sudo apt-get -y install python3-distutils
     cd ~
     sudo git clone https://github.com/mesagon/Proyecto-CC-MII.git
     cd Proyecto-CC-MII && sudo pipenv install

  SHELL

end

~~~

Vemos que en la orden config.vm.provison indicamos a vagrant que realice el provisionamiento mediante shell y tras esto ponemos inline: <<-SHELL. Con esto indicamos que a continuación vienen los comandos que debe de ejecutar vagrant en la MV para realizar su provisionamiento. Dichos comandos son los necesarios para montar la infraestructura virtual necesaria para desplegar nuestra aplicación en la MV, la cual ya se ha visto en los ejercicios de temas anteriores. Finalmente cerramos los comandos con la palabra SHELL.

Tras esto, ejecutamos

~~~
$ vagrant up
~~~

y se creará la MV con todo lo necesario para desplegar en ella nuestro
microservicio.

### Ejercicio 4: Configurar tu máquina virtual usando vagrant con el provisionador ansible.

Vamos a configurar el Vagrantfile del anterior ejercicio para que vagrant realice el provisionamiento de la misma MV utilizando Ansible. De esta forma, el vagrantfile es el siguiente.

~~~
# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|

  config.vm.box = "generic/debian9"

  config.vm.provision :ansible do |ansible|

    ansible.playbook = "playbook-Debian.yml"

  end
end
~~~

 Vemos como en la orden de provisionamiento indicamos a Vagrant que realice el provisionamiento con Ansible y tras esto, indicamos qué playbook de Ansible debe de ejecutar.

 En este caso, no tenemos que preocuparnos de poner en el inventario de Ansible los datos de la MV a provisionar, pues Vagrant crea su propio inventario específico para la MV que va a provisionar (la que crea el mismo) y se lo pasa a Ansible.

 Finalmente levantamos la MV.

 ~~~
 $ vagrant up
 ~~~

Y obtenemos como resultado

~~~
default: Running ansible-playbook...

PLAY [all] *********************************************************************

TASK [Gathering Facts] *********************************************************
ok: [default]

TASK [Añadir repositorio para python 3.6 y actualizar repositorios.] ***********
ok: [default]

TASK [Instalar python-apt.] ****************************************************
ok: [default]

TASK [Instalar python 2.7.] ****************************************************
ok: [default]

TASK [Instalar python 3.6.] ****************************************************
ok: [default]

TASK [Instalar pip3.] **********************************************************
ok: [default]

TASK [Instalar git.] ***********************************************************
ok: [default]

TASK [Instalar pipenv.] ********************************************************
changed: [default]

TASK [Instalar distutils.] *****************************************************
ok: [default]

TASK [Clonar proyecto.] ********************************************************
ok: [default]

TASK [Establecer entorno virtual.] *********************************************
changed: [default]

TASK [Instalar java jdk 1.8.] **************************************************
changed: [default]

TASK [Instalar paquetes necesarios.] *******************************************
changed: [default]

TASK [Añadir repositorio para logstash] ****************************************
changed: [default]

TASK [Instalar logstash.] ******************************************************
changed: [default]

PLAY RECAP *********************************************************************
default                    : ok=15   changed=6    unreachable=0    failed=0   
~~~

Vemos como cada una de las tareas del playbook se ha ejecutado correctamente, lo que significa que el provisionamiento se ha realizado con éxito.
