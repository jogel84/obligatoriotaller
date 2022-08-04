# **Obligatorio Taller Linux - ORT 2022**

Instalación de Tomcat9, MariaDB en Rocky Linux y Ubuntu Server

---

&nbsp;
## **Solución**

La solución está compuesta por un playbook de Ansible. 
El playbook `main.yml` realiza la instalación y configuración inicial de la ultima versión disponible del motor mariadb (10.x), mas la creación de una base de datos para la aplicación, tambien realiza la instalación de tomcat 9 en la ubicación `/opt/tomcat9` y la descarga de jdk 1.8 necesario para que funcione el aplicativo.

El playbook ejecutará tareas (archivos yml configurados en la solución) para los equipos con sistema operativo Rocky Linux y Ubuntu Server.

&nbsp;
## **tasks**

### **Tareas para mariadb**

`actualizarPaquetes.yml` 
- Actualiza y descarga los paquetes de los servidores Ubuntu Server y Rocky Linux

`configuracionInicialMariaDB.yml`

- Realiza la configuracion inicial del motor de base de datos, configura la contraseña root y el acceso solamente mediante localhost.
- Realiza el equivalente al mysql_secure_instalation con las siguientes configuraciones
    - Remueve usuarios anonimos
    - Deshabilita el login remoto con usuario root
    - Remueve la BD de test

`configurarServicioMariaDB.yml`

- Realiza instalación de PyMySQL e inicia y habilita el servicio mariadb

`creacionDatabaseApp.yml`

- Creacion de la BD `todo` junto con la creacion de las tablas `users` y `todos`
- Se crea usuario `todo` en el servidor de la BD con permisos totales a todas las tablas de la BD `todo`

`instalarMariaDB.yml`

- Instalacion de mariadb junto con todas las dependencias necesarias.

&nbsp;
### **Tareas para tomcat**

`configuracionApp.yml`

- Crea directorio y copia archivos necesarios para la configuración de la aplicación

`firewallTomcat.yml`

- Crea regla de firewall para Rock Linux para permitir el trafico en el puerto 8080/TCP

`instalacionjdk-Tomcat.yml`

- Creacion de directorios para aplicativo tomcat9 y jdk 1.8
- Descarga, instalacion, creacion de usuario para de tomcat9 con privilegios necesarios.
- Descarga e instalacion de jdk 1.8

&nbsp;
### **files**

`app.properties`

- Archivo necesario para la conexión de la BD

`my.cnf`

- Credenciales de acceso para root en el servidor de BD

`todo.war`

- Aplicativo

`tomcat.service`

- Archivo de servicio para tomcat

&nbsp;
### **inventario**

- Contiene los hosts para la administración de Ansible

&nbsp;
### **ansible.cfg**

- Archivo de configuración de Ansible el cual indica la ubicación del inventario

&nbsp;
## **Requermimientos**

- Servidor bastion con Ansible instalado para la ejeción de playbooks
- Se requiere de intercambio de claves SSH entre el servidor bastion y los demas servidores.
- El usuario Ansible creado en todos los servidores y con privilegios de sudo sin especificar contraseña `ansible ALL=(ALL:ALL) NOPASSWD: ALL`
- Comunicación a nivel de red entre todos los servidores al servidor bastion
- Acceso a internet en todos los equipos para actualizar y descargar los archivos de instalación.

&nbsp;
## **Configuración de los sevidores**

Se instala 3 servidores linux de tal forma:

- 1 servidor bastion con sistema operativo Rocky Linux, el cual es el controlador para el despliegue de playbooks de Ansible
- 1 servidor cliente con sistema operativo Rocky Linux
- 1 servidor cliente con sistema operativo Ubuntu Server

&nbsp;
### **Esquema de particiones de todos los servidores:**

Se utiliza LVM para realizar el esquema de particiones en todos los servidores.

&nbsp; `df -h`

| Filesystem |   Size   |  Used  |   Avail   |  Use%  | Mounted on |
| ---------- | -------- | ------ | --------- | ------ | ---------- |
| /dev/mapper/rl_cliente1-root|  5.0G|  2.3G|  2.7G|  46%| /|
| /dev/sda2| 1014M|  250M|  765M|  25%| /boot|
| /dev/sda1| 1022M|  5.7M| 1017M|   1%| /boot/efi|
| /dev/mapper/rl_cliente1-var|   5.0G|  583M|  4.5G|  12%| /var|
| /dev/mapper/rl_cliente1-home| 1014M|   40M|  975M|   4%| /home|
| /dev/mapper/rl_cliente1-opt|   5.0G  |542M  |4.5G  |11%| /opt|

&nbsp; `swapon --show`

|NAME      |TYPE      |SIZE |USED |PRIO|
|----      |----      |---- |---- |----|
/dev/dm-1| partition|   2G|   0B|   -2|

&nbsp;
### **Servidores destino de Ansible**

- Crear usuario Ansible y establecer contraseña 
    ```
    $ sudo useradd ansible
    $ sudo passwd <contraseña>
    ```

- Agregar la siguiente linea al final del archivo, en el caso de Rocky `ansible ALL=(ALL) NOPASSWD: ALL`, en el caso de Ubuntu `ansible ALL=(ALL:ALL) NOPASSWD: ALL`.

    ```
    $ sudo visudo
    ```
&nbsp;
### **Servidor bastion**

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

&nbsp;
## **Ejecución de playbooks**

Para ejecutar los comandos como están descritos mas abajo, es necesario posicionarse en la ruta principal del proyecto.

Ejecutar playbook `main.yml`

Se deben configurar los valores de las variables `mariadb_root_password=<CONTRASEÑA_BD>` por la contraseña para el usuario root que desee.

```
$ ansible-playbook playbooks/main.yml --extra-vars "mariadb_root_password=<CONTRASEÑA_BD>"
```