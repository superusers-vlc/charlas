:Título: BackupPC
:Autores: José A. Salvador Vanaclocha, Access (Empleo Express ETT, S.L.) diciembre 2011
          Juanjo Ponz Vanaclocha, Access (Empleo Express ETT, S.L.) enero 2016
:Versión: 1.6 marzo 2016


.. raw:: pdf

    PageBreak 


.. contents:: Indice


.. section-numbering::


.. raw:: pdf

    PageBreak 


Sinopsis
========
Guía útil para la instalación y configuración de BackupPC en un sistema Ubuntu 14.04.X LTS.


Instalación
===========
Instalamos:

.. code-block:: console

    # apt-get install backuppc apache2-utils

*backuppc* tiene entre sus dependencias a samba. Por tanto, al instalar nos preguntara por el dominio de Windows. También reconfigurará el servidor web y creará un usuario *backuppc* para poder gestionar *BackupPC* a través de su interfaz web (http://nombre_de_equipo/backuppc/). para cambiar la contraseña que genera por defecto habrá que ejecutar

.. code-block:: console

    # htpasswd /etc/backuppc/htpasswd backuppc


Configuración
=============
La configuración general de *BackupPC* se encuentra en */etc/backuppc/config.pl*

Los hosts de los cuales se realizarán copias se encuentra en */etc/backuppc/hosts*

Si deseamos que algún host, p.e. host23, tuviera una configuración personalizada distinta de la general crearemos un fichero en *etc/backuppc* que se llame como el host acabado en pl. En este caso: host23.pl. Allí especificaremos los valores distintos de la configuración general para ese equipo.

Realizaremos la configuración desde la interfaz web http://nombre_de_maquina/backuppc/.

En caso de no poder acceder ejecutaremos:

.. code-block:: console

    # a2disconf serve-cgi-bin

y limpiamos la cache del navegador y lo reiniciamos.


La sección *Edit Config* determina la configuración general.

En *Edit Config>Xfer>* estableceremos:

- *XferMethod* como *rsync*.
- *BackupFilesExclude* ***** como *key* y las siguientes rutas:
  - */proc*
  - */sys*
  - */tmp*
  - */dev*
  - */run*
  - */mnt*
  - */media*
  - */lost+found*

 Con esto definimos que no copie esos directorios por defecto.

En *Edit Config>Hosts* añadimos el equipo al que queremos realizar las copias de seguridad. Al equipo lo llamaremos linuxhost. Establecemos pues como nombre linuxhost, desmarcamos DHCP y no especificamos ni usuarios ni más usuarios.

No marcar DHCP hace que *BackupPC* realice la búsqueda por DNS, en caso contrario la realiza con *nmblookup*.

La entrada para usuario o más usuarios concedería permiso a los usuarios de esas máquinas a loguearse en la interfaz web para realizar o restaurar copias de sus equipos.

La hora de inicio para el *nightly job* (cuando se realiza el proceso de agrupamiento, compresión y limpieza) viene determinada por el primer elemento de *WakeupSchedule*, es decir, si deseo que se verifique si se ha de realizar una copia de seguridad cada hora desde las 9 hasta las 14 y que además, el proceso *nightly job* se ejecute a las 12 debería especificarlo así: *12, 13, 14, 9, 10, 11*

Las copias por *rsync* se realizan tunelizadas por ssh desde *BackupPC* por el usuario *backuppc*. Por ese motivo tendremos que:

- Generar una clave para el usuario *backuppc* en el servidor *BackupPC*: 

.. code-block:: console

    # su backuppc
    # ssh-keygen -t rsa
    
Omitir la frase de paso.

- Transferir la clave pública al cliente (root@linuxhost): 

.. code-block:: console

    # ssh-copy-id -i /var/lib/backuppc/.ssh/id_rsa.pub root@linuxhost

- Obtener la clave publica del host root@linuxhost desde backuppc@servidor
- El cliente debe tener instalado tanto *rsync* como *openssh-server*
- El cliente debe tener al servidor *BackupPC* como host autorizado desde root

    Cuidado al utilizar *ssh-copy-id*, en ocasiones adjunta la clave pública sin insertar antes un retorno de carro. Esto hace que dos claves distintas puedan quedar unidas y por tanto la autenticación no funcione.

Añade al propio equipo (por su nombre y no como localhost) con *backuppc* como primer equipo a realizar las copias de seguridad. **Excluyendo el directorio donde se almacenan las copias de seguridad del resto de hosts** (*/var/lib/backuppc*).


Eliminar completamente el backup de un host
===========================================
Para eliminar completamente un host y sus backups debe eliminarse su entrada en *etc/hosts* y borrar el directorio */var/lib/backuppc/pc/$host*. Cada vez que haya un cambio en *conf/hosts* debe enviarse una señal HUP (-1) a backuppc para que relea la configuración.

	La eliminación del host puede hacerse desde la interfaz web seguida de una pulsación sobre *Opciones de administración - Actualizar configuración del servidor*. Los ficheros físicos de la copia de seguridad debe realizarse a mano como en el parrafo anterior se ha citado.

Cuando se eliminan los backups de un host inicialmente no se recuperará mucho espacio, esto es porque sus ficheros aún residen en el *pool* de backuppc. Cuando el siguiente proceso de limpieza nocturna ocurra este espacio será definitivamente liberado.


Forzar la ejecución de un BackupPC_nightly
==========================================
El proceso de BackupPC_nightly es el que ejecuta diariamente backuppc en el que se recorre el pool de datos para eliminar aquellos que ya no son necesarios.

Si deseas forzar la ejecución de este proceso:

.. code-block:: console

    # su backuppc
    backuppc$ /usr/share/backuppc/bin/BackupPC_nightly 0 255


Backup de un equipo Windows
===========================
Para realizar backup de una carpeta concreta de un equipo windows, crearemos un recurso compartido en Windows, dandole el nombre que queramos.

Una vez creado el recurso compartido, vamos a backuppc:

	Seleccionamos el host > Edit Config > Xfer > SmsShareName > Pulsamos en Add y escribimos el nombre del recurso compartido en windows
	
