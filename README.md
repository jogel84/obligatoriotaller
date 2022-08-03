# Obligatorio Taller Linux - ORT 2022

Instalación de Tomcat9, MariaDB en Rocky Linux y Ubuntu Server

## Solución

La solución está compuesta por dos playbooks de Ansible. 
El playbook `mariadb-playbook.yml` realiza la instalación y configuración inicial de la ultima versión disponible del motor mariadb, mas la creación de una base de datos para la aplicación.
El playbook `tomcat9-playbook.yml` realiza la instalación de tomcat 9 en la ubicación `/opt/tomcat9` y la descarga de jdk 11 necesario para que funcione el aplicativo.

Cada playbook ejecutará tareas (archivos yml configurados en la solución) para los equipos con sistema operativo Rocky Linux y Ubuntu Server.

### tasks

sarasa

### files

sarasa

### inventario

sarasa

### ansible.cfg

sarasa

## Ejecución de playbooks

Para ejecutar los comandos como están descritos mas abajo, es necesario posicionarse en la ruta principal del proyecto.

Ejecutar playbook `mariadb-playbook.yml`

Se deben configurar los valores de las variables `mariadb_root_password=<CONTRASEÑA_BD>` por la contraseña para el usuario root que desee y la variable `mariadb_database_name=<NOMBRE_BD>` por el nombre de la base de datos a crear en la ejecucion del playbook.

```
$ ansible-playbook playbooks/mariadb-playbook.yml --extra-vars "mariadb_root_password=<CONTRASEÑA_BD> mariadb_database_name=<NOMBRE_BD>"
```


Ejecutar playbook `tomcat9-playbook.yml`

```
$ ansible-playbook playbooks/tomcat9-playbook.yaml
```

## Requermimientos

- Servidor bastion con Ansible instalado para la ejeción de playbooks
- Se requiere de intercambio de claves SSH entre el servidor bastion y los demas servidores.
- El usuario Ansible creado en todos los servidores y con privilegios de sudo sin especificar contraseña `ansible ALL=(ALL:ALL) NOPASSW: ALL`
- Comunicación a nivel de red entre todos los servidores al servidor bastion
- Acceso a internet en todos los equipos para actualizar y descargar los archivos de instalación.


## Configuración de los sevidores

Se instala 3 servidores linux de tal forma:

- 1 servidor bastion con sistema operativo Rocky Linux, el cual es el controlador para el despliegue de playbooks de Ansible
- 1 servidor cliente con sistema operativo Rocky Linux
- 1 servidor cliente con sistema operativo Ubuntu Server

### Esquema de particiones de todos los servidores:

Se utiliza LVM para realizar el esquema de particiones en todos los servidores.

`df -h`

| Filesystem |   Size   |  Used  |   Avail   |  Use%  | Mounted on |
| ---------- | -------- | ------ | --------- | ------ | ---------- |
| /dev/mapper/rl_cliente1-root|  5.0G|  2.3G|  2.7G|  46%| /|
| /dev/sda2| 1014M|  250M|  765M|  25%| /boot|
| /dev/sda1| 1022M|  5.7M| 1017M|   1%| /boot/efi|
| /dev/mapper/rl_cliente1-var|   5.0G|  583M|  4.5G|  12%| /var|
| /dev/mapper/rl_cliente1-home| 1014M|   40M|  975M|   4%| /home|
| /dev/mapper/rl_cliente1-opt|   5.0G  |542M  |4.5G  |11%| /opt|

`swapon --show`

|NAME      |TYPE      |SIZE |USED |PRIO|
|----      |----      |---- |---- |----|
/dev/dm-1| partition|   2G|   0B|   -2|


### Servidores destino de Ansible

- Crear usuario Ansible y establecer contraseña 
    ```
    $ sudo useradd ansible
    $ sudo passwd <contraseña>
    ```

- Agregar la siguiente linea al final del archivo, en el caso de Rocky `ansible ALL=(ALL) NOPASSWD: ALL`, en el caso de Ubuntu `ansible ALL=(ALL:ALL) NOPASSWD: ALL`.

    ```
    $ sudo visudo
    ```

### Servidor bastion

- En caso que el servidor bastion sea de la familia de RedHat, se debe agregar el siguiente paquete extra

    ```
    $ sudo yum install epel-release
    ```
- Instalar ansible 
    ```
    Rocky:  $ sudo yum install ansible
    Ubuntu: $ sudo apt-get install ansible
     ```

- Crear usuario Ansible y establecer contraseña 
    ```
    $ sudo useradd ansible
    $ sudo passwd <contraseña>
    ```

- Generar par de clave SSH desde el usuario ansible
    ```
    $ ssh-keygen
    ```

- Copiar la clave ssh desde el usuario ansible al resto de los servidores
    ```
    $ ssh-copy-id <IP servidor destino>
    ```

- Copiar los archivos del repositorio a la ubicacion de preferencia, ejemplo `/opt/obligatoriotaller`