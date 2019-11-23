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

Clonanos esta maquina 2 veces para tener otros dos nodos y les configuramos el hostname y las ipsstaticas así:

# Hostname      IP
Nodo2           10.2.0.2
Nodo3           10.2.0.3
netmask         255.255.0.0
gateway         10.2.0.1

# Configuración del archivo hosts en cada nodo
cloud.nodo1     10.2.0.1
cloud.nodo1     10.2.0.2
cloud.nodo1     10.2.0.3

Antes de proceder a configurar el cluster hay que destruir el cluster inicial, en ocaciones el archivo de configuración inicial presenta problemas, esto hay que hacerlo en cada nodo:

- sudo pcs cluster destroy

# Configuración del cluster

- Logueo en para cada nodo:
sudo pcs cluster auth cloud.nodo1 cloud.nodo2 -u hacluster -p "contraseña_colocada" --force
- La respuesta para cada nodo debe ser:
nombre_nodo: authorized

- Configuramos nombre del cluster y añadimos los nodos.
sudo pcs cluster setup --name mycluster cloud.node1 cloud.nodo2
# Si no hicieron el comando para destruir el cluster antes de todo, este comando les tirara error, por que el sistema no puede acceder a un archivo de configuración que solo tiene permisos de lectura. 

- Configuraciones de los nodos:
sudo pcs cluster enable --all //habilitación del nodo con el inicio del sistema operativo
sudo pcs cluster start --all //iniciar todos los nodos

- Verificamos el estado del cluster
sudo pcs status

- Configuraciones extras:
sudo pcs property set stonith-enabled=false
sudo pcs property set no-quorum-policy=ignore

- Verificamos que si queden como queremos:
sudo pcs property list

# Configuración de los servicios:

- IPVirtual: 10.2.0.20 que es la que recibira todas las peticiones http
sudo pcs resource create fl_ip ocf:heartbeat:IPaddr2 ip=10.2.0.20 cidr_netmask=16 op monitor interval=5s

- Nginx: Servidor http
sudo pcs resource create web_server ocf:'heart'beat:nginx configfile="/etc/nginx/nginx.conf" op monitor timeout="5s" interval="5s"

- En ocaciones hay que reiniciar el servicio del nginx
sudo systemctl start nginx

- Verificamos el estado de los servicios
sudo pcs status resources

- Ahora verificamos en un navegador 10.2.0.20

# Damos de baja a cualquiera de los nodos y el servidor sigue en pie

- Dar de baja nodo1
sudo pcs cluster stop cloud.nodo1

- Dar de baja nodo2
sudo pcs cluster start cloud.nodo1
sudo pcs cluster stop cloud.nodo2

Hasta aca siempre hay respuesta de la pagina pero si ejecutamos:

sudo pcs cluster stop --all

Ya no abra respuesta.
