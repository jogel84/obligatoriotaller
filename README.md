# Obligatorio Taller Linux - ORT 2022

Instalación de Tomcat9, MariaDB en Rocky Linux y Ubuntu Server

## Requermimientos

- Servidor bastion con Ansible instalado para la ejeción de playbooks
- Se requiere de intercambio de claves SSH entre el servidor bastion y los demas servidores.
- El usuario Ansible creado en todos los servidores y con privilegios de sudo sin especificar contraseña `ansible ALL=(ALL:ALL) NOPASSW: ALL`
- Comunicación a nivel de red entre todos los servidores al servidor bastion
- Acceso a internet en todos los equipos para actualizar y descargar los archivos de instalación.

## Configuración de los sevidores

### Servidores destino de Ansible

- Se crea usuario Ansible y se establece contraseña 
    ```
    sudo useradd ansible
    sudo passwd <contraseña>
    ```
- Agregar la siguiente linea al final del archivo, en el caso de Rocky `ansible ALL=(ALL) NOPASSWD: ALL`, en el caso de Ubuntu `ansible ALL=(ALL:ALL) NOPASSWD: ALL`.
    ```
    sudo visudo
    ```

### Servidor bastion

Instalaremos ansible en el servidor bastion

- En caso que el servidor bastion sea de la familia de RedHat, se debe agregar el siguiente paquete extra
    ```
    sudo yum install epel-release
    ```
- Instalar ansible 
    Rocky:  `sudo yum install ansible`
    Ubuntu: `sudo apt-get install ansible`
- Crear usuario Ansible y establecer contraseña 
    ```
    sudo useradd ansible
    sudo passwd <contraseña>
    ```
- Generar par de clave SSH desde el usuario ansible
    ```
    ssh-keygen
    ```
- Copiar la clave ssh desde el usuario ansible al resto de los servidores
    ```
    ssh-copy-id <IP servidor destino>
    ```
- Copiar los archivos del repositorio a la ubicacion de preferencia, ejemplo `/opt/obligatoriotaller`