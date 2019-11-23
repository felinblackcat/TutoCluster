# High Availability

Se configurara un Cluster High Availability en Ubuntu Server 16.


# Objetivos:

- Configurar un servidor cluster HA.
- Administrar nodos para el cluster, agregar y quitar nodos.
- Agregar servicios.


# Espacio de trabajo:

4 Maquinas virtuales utilizando virtualbox. 


# Configuracion Hardware de cada nodo:

- 1 procesador.
- 1gb de RAM.
- 8mb memoria de video.
- 10gb Disco duro.
- En cuanto a la red se trabajara en red interna.

# Procedimiento

Omitiendo la instalación de Ubuntu Server se procedera con la instalación de las herramientas para configurar el cluster.
Configurando el primer nodo, se haran clonaciones para los siguientes nodos, de esta manera ahorrar trabajo, para cada nodo
se les configurara su respectiva direccion ip estatica y el nombre del host.


# Instalaciones:

Como el servicio que se pondra a prueba es un servidor http se utilizara nginx.

# Instalación y configuración nginx:

- sudo apt install nginx -y
- sudo systemctl enable nginx
- sudo systemctl start nginx
- sudo systemctl status nginx

# Instalación y configuración de Corosync and Pacemaker:

- sudo apt install corosync pacemaker pcs
- sudo systemctl enable pcsd
- sudo systemctl start pcsd
- sudo systemctl status pcsd
Cambiamos la contraseña del usuario hacluster
- sudo passwd hacluster

# Configuración adaptador de red con ip estatica:
- sudo nano /etc/network/interfaces
- Se deja como en la imagen:











