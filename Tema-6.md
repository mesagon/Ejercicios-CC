### Ejercicio 1: Instala LXC en tu versión de Linux favorita. Normalmente la versión en desarrollo, disponible tanto en GitHub como en el sitio web está bastante más avanzada; para evitar problemas sobre todo con las herramientas que vamos a ver más adelante, conviene que te instales la última versión y si es posible una igual o mayor a la 2.0.

Vamos a instalar en Ubuntu 18.04 lxc con el siguiente comando.
~~~
$ sudo apt-get install lxc
~~~

Finalmente comprobamos que lxc se ha instalado correctamente con el comando

~~~
$ lxc-checkconfig
~~~

Nos tendrán que aparecer los distintos parámetros de configuración de lxc como "enabled" para poder crear contenedores.

Por último podemos ver la versión de lxc con el comando

~~~
lxc-info --version
~~~

En este caso hemos instalado la versión 3.1.0-devel.

### Ejercicio 2: Instalar una distro tal como Alpine y conectarse a ella usando el nombre de usuario y clave que indicará en su creación.

Vamos a crear un contenedor con una imagen de Alpine 3.8. Para ello ejecutamos el siguiente comando.

~~~
$ sudo lxc-create -t download -n contenedor-ejemplo
~~~

Con este comando vamos a crear un contenedor con la plantilla (o template) download (opción -t), mediante la cual se nos mostrará un listado de las distintas distribuciones con las que podemos crear el contenedor. Se nos pedirá que introduzcamos tres parámetros:

  - Distribución: El nombre de la distribución a instalar. En este caso escribimos alpine para instalar una imagen con alpine.
  - Release: Versión de la distribución. En este caso escribimos 3.8.
  - Arquitectura: Arquitectura en la que instalar la distribución. Hemos elegido amd64.

Tras esto se creará un contenedor con Alpine 3.8 amd64 llamado contenedor-ejemplo (opción -n).

Accedemos finalmente a dicho contenedor con la orden siguiente.

~~~
$ sudo lxc-console -n contenedor-ejemplo
~~~


### Ejercicio 3: Buscar alguna demo interesante de Docker y ejecutarla localmente, o en su defecto, ejecutar la imagen anterior y ver cómo funciona y los procesos que se llevan a cabo la primera vez que se ejecuta y las siguientes ocasiones.

Vamos a ejecutar el táper proporcionado en el ejemplo. Para ello ejecutamos el siguiente comando.

~~~
$ sudo docker run --rm jjmerelo/docker-daleksay -f smiling-octopus Uso argumentos, ea
~~~

Tras ejecutar el anterior comando por primera vez, obtenemos la siguiente salida en la consola.

~~~
Unable to find image 'jjmerelo/docker-daleksay:latest' locally
latest: Pulling from jjmerelo/docker-daleksay
0a8490d0dfd3: Pull complete
aac9fee8ebfb: Pull complete
c951ce7a8ac7: Pull complete
ab590fdbfbc0: Pull complete
08ad84f33a7b: Pull complete
Digest: sha256:9e2b66e742f80dd7422e48d69c0cf9a01afc9c2d50a6c2185fd37c3519a984ff
Status: Downloaded newer image for jjmerelo/docker-daleksay:latest
 ____________________
< Uso argumentos, ea >
 --------------------
      \
       \
        \                                     ,
                                            ,o
                                            :o
                   _....._                  `:o
                 .'       ``-.                \o
                /  _      _   \                \o
               :  /*\    /*\   )                ;o
               |  \_/    \_/   /                ;o
               (       U      /                 ;o
                \  (\_____/) /                  /o
                 \   \_m_/  (                  /o
                  \         (                ,o:
                  )          \,           .o;o'           ,o'o'o.
                ./          /\o;o,,,,,;o;o;''         _,-o,-'''-o:o.
 .             ./o./)        \    'o'o'o''         _,-'o,o'         o
 o           ./o./ /       .o \.              __,-o o,o'
 \o.       ,/o /  /o/)     | o o'-..____,,-o'o o_o-'
 `o:o...-o,o-' ,o,/ |     \   'o.o_o_o_o,o--''
 .,  ``o-o'  ,.oo/   'o /\.o`.
 `o`o-....o'o,-'   /o /   \o \.                       ,o..         o
   ``o-o.o--      /o /      \o.o--..          ,,,o-o'o.--o:o:o,,..:o
                 (oo(          `--o.o`o---o'o'o,o,-'''        o'o'o
                  \ o\              ``-o-o''''
   ,-o;o           \o \
  /o/               )o )  Carl Pilcher
 (o(               /o /                |
  \o.       ...-o'o /              \   |
    \o`o`-o'o o,o,--'       ~~~~~~~~\~~|~~~~~~~~~~~~~~~~~~~~~~~~~~~~
      ```o--'''                       \| /
                                       |/
 ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~|~~~~~~~~~~~~~~~~~~~~~~~~~~~~
                                       |
 ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Vemos que en primer lugar Docker busca la imagen jjmerelo/docker-daleksay entre las imágenes locales de Docker en nuestro sistema y al ver que no existe, la busca y la descarga desde el Hub de Docker. Una vez descargada la imagen, crea un contenedor con ella y ejecuta en el un programa (smilling-octopus) que nos muestra un pulpo sonriente repitiendo los argumentos que le hemos pasado al contenedor con la orden -f. Tras esto, Docker elimina el contenedor de nuestro sistema debido a que lo hemos ejecutado con la orden --rm.

Cuando volvemos a ejecutar el contenedor con la msima orden, se repite el proceso anterior, con la diferencia de que la imagen jjmerelo/docker-dalecksay no es descargada por Docker, pues ya la descargó en la primera ejecución y la almacenó entre sus imágenes locales.

### Ejercicio 4: Comparar el tamaño de las imágenes de diferentes sistemas operativos base, Fedora, CentOS y Alpine, por ejemplo.

En primer lugar nos descargamos e instalamos las tres imágenes con los siguientes comandos.

~~~
$ sudo docker pull fedora
$ sudo docker pull centos
$ sudo docker pull alpine
~~~

Una vez instaladas las imágenes las visualizamos junto a su tamaño con la siguiente orden, la cual lista todas las imágenes disponibles localmente.

~~~
$ sudo docker images
~~~

Con este comando obtenemos la siguiente lista de imágenes.

~~~
REPOSITORY                 TAG                 IMAGE ID            CREATED             SIZE
alpine                     latest              caf27325b298        4 days ago          5.53MB
fedora                     latest              26ffec5b4a8a        2 weeks ago         275MB
hello-world                latest              fce289e99eb9        4 weeks ago         1.84kB
centos                     latest              1e1148e4cc2c        2 months ago        202MB
jjmerelo/docker-daleksay   latest              5bf18c53ecd5        2 years ago         72.1MB
~~~

Podemos ver todas las imágenes instaladas en nuestro sistema actualmente. Vemos como las imágenes de CentOS y Fedora tienen un tamaño aproximadamente 40 veces mayor que la imagen de Alpine. Esto se debe a que Alpine es una imagen ligera pensada especialmente para la creación de contenedores.

### Ejercicio 5: Crear a partir del contenedor anterior una imagen persistente con commit.

En primer lugar listamos los contenedores que hemos creado con el siguiente comando.

~~~
$ sudo docker ps -l
~~~

Con esto obtenemos la siguiente lista.

~~~
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS                      PORTS               NAMES
2a77af1522b8        alpine              "sh"                40 minutes ago      Exited (0) 40 minutes ago                       stoic_wiles
~~~

Vemos que tenemos solo un contenedor y se encuentra en estado "exited". Este es un contenedor con una imagen de Alpine. Lo que vamos a hacer a continuación es guardar el estado actual de este contenedor a través de su ID con el siguiente comando.

~~~
$ sudo docker commit 2a77af1522b8 imagen-alpine
~~~

El comando anterior guarda el estado actual del contenedor cuya ID se le ha proporcionado en una imagen llamada "imagen-alpine". tras esto, podemos consultar la nueva imagen que contiene el estado del contenedor con el comando siguiente.

~~~
$ sudo docker images
~~~

Y en la lista obtenida podemos visualizar la nueva imagen en la primera fila.

~~~
REPOSITORY                 TAG                 IMAGE ID            CREATED              SIZE
imagen-alpine              latest              57c426be3f8e        47 seconds ago       5.53MB
alpine                     latest              caf27325b298        6 days ago           5.53MB
fedora                     latest              26ffec5b4a8a        2 weeks ago          275MB
hello-world                latest              fce289e99eb9        5 weeks ago          1.84kB
centos                     latest              1e1148e4cc2c        2 months ago         202MB
jjmerelo/docker-daleksay   latest              5bf18c53ecd5        2 years ago          72.1MB
~~~

Con esta nueva imagen podemos crear contenedores con el mismo estado que el contenedor a partir del cual hemos creado dicha imagen.

### Ejercicio 6: Examinar la estructura de capas que se forma al crear imágenes nuevas a partir de contenedores que se hayan estado ejecutando.

Cuando creamos la imagen "imagen-alpine" a partir del contenedor original con Alpine, se añadió una nueva capa identificada por un sha256. Dicho sha256 se nos devolvió como resultado del commit.

En este caso, una imagen Docker está formada por una capa de lectura y escritura situada en la "cima" de todas sus capas y por una serie de capas de solo lectura, también denominadas imágenes intermedias. Cada imagen intermedia se corresponde con la ejecución de un determinado comando sobre la imagen. Esto se ha consultado [aquí](https://medium.com/@jessgreb01/digging-into-docker-layers-c22f948ed612).

Podemos ver las capas de la imagen "imagen-alpine" pasándole su ID al siguiente comando.

~~~
$ sudo docker history 57c426be3f8e
~~~

Y obtenemos una tabla, en la que en cada fila aparece una capa identificada por un ID, cuando fue creada y el comando que la creó.

~~~
IMAGE               CREATED             CREATED BY                                      SIZE                COMMENT
57c426be3f8e        33 minutes ago      sh                                              0B                  
caf27325b298        6 days ago          /bin/sh -c #(nop)  CMD ["/bin/sh"]              0B                  
<missing>           6 days ago          /bin/sh -c #(nop) ADD file:2a1fc9351afe35698…   5.53MB      
~~~
