# chirpstack

Este tutorial describe los pasos necesarios para configurar la pila ChirpStack, incluidos todos los requisitos en una sola máquina.


Ubuntu 20.04 LTS

Dependencias

Agente MQTT : un protocolo de publicación / suscripción que permite a los usuarios publicar información sobre temas a los que otros pueden suscribirse. Una implementación popular del protocolo MQTT es Mosquitto .

Redis : una base de datos en memoria que se utiliza para almacenar datos relativamente transitorios.

PostgreSQL : la base de datos de almacenamiento a largo plazo que utilizan los paquetes de código abierto


Utilice el administrador de paquetes aptpara instalar estas dependencias:

sudo apt install mosquitto mosquitto-clients redis-server redis-tools postgresql 

#Configurar bases de datos y usuarios de PostgreSQL

Para ingresar a la utilidad de línea de comando para PostgreSQL:

sudo -u postgres psql


Dentro de este mensaje, ejecute las siguientes consultas para configurar las bases de datos que utilizan los componentes de la pila de ChirpStack. Se recomienda cambiar los nombres de usuario y las contraseñas. Solo recuerde usar estos otros valores al actualizar los archivos de configuración chirpstack-network-server.tomly chirpstack-application-server.toml. Dado que estas dos aplicaciones utilizan la misma tabla para realizar un seguimiento de las actualizaciones de la base de datos, deben tener bases de datos independientes.


-- set up the users and the passwords
-- (note that it is important to use single quotes and a semicolon at the end!)
create role chirpstack_as with login password 'dbpassword';
create role chirpstack_ns with login password 'dbpassword';

-- create the database for the servers
create database chirpstack_as with owner chirpstack_as;
create database chirpstack_ns with owner chirpstack_ns;

-- change to the ChirpStack Application Server database
\c chirpstack_as

-- enable the pq_trgm and hstore extensions
-- (this is needed to facilitate the search feature)
create extension pg_trgm;
-- (this is needed to store additional k/v meta-data)
create extension hstore;

-- exit psql
\q

#Configurar el repositorio de software ChirpStack

ChirpStack proporciona un repositorio que es compatible con el sistema de paquetes apt de Ubuntu.
Primero asegúrese de que ambos dirmngry apt-transport-httpsestén instalados:

sudo apt install apt-transport-https dirmngr


Configure la clave para este nuevo repositorio:

sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys 1CE2AFD36DBCCA00


Agregue el repositorio a la lista de repositorios creando un nuevo archivo:


sudo echo "deb https://artifacts.chirpstack.io/packages/3.x/deb stable main" | sudo tee /etc/apt/sources.list.d/chirpstack.list



Actualice la caché del paquete apt:

sudo apt update

Instale el puente de puerta de enlace de ChirpStack

Nota: Si tiene la intención de ejecutar el puente de puerta de enlace de ChirpStack solo en las puertas de enlace, puede omitir este paso.


sudo apt install chirpstack-gateway-bridge

El archivo de configuración se encuentra en /etc/chirpstack-gateway-bridge/chirpstack-gateway-bridge.toml. 
La configuración predeterminada es suficiente para esta guía.

# start chirpstack-gateway-bridge
sudo systemctl start chirpstack-gateway-bridge

# start chirpstack-gateway-bridge on boot
sudo systemctl enable chirpstack-gateway-bridge

Instalación del servidor de red ChirpStack
Instale el paquete usando apt:

sudo apt install chirpstack-network-server

El archivo de configuración se encuentra en /etc/chirpstack-network-server/chirpstack-network-server.tomly debe actualizarse para que coincida con la configuración de la base de datos y la banda. Vea a continuación dos ejemplos para las bandas EU868 y US915. Para obtener más información sobre todas las opciones de configuración del servidor de red ChirpStack, consulte Configuración del servidor de red ChirpStack .













































