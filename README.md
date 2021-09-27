# chirpstack


La pila del servidor de red LoRaWAN de código abierto ChirpStack proporciona componentes de código abierto para redes LoRaWAN. Juntos forman una solución lista para usar que incluye una interfaz web fácil de usar para la administración de dispositivos y API para la integración. La arquitectura modular permite integrarse dentro de las infraestructuras existentes. Todos los componentes están sujetos a la licencia MIT y pueden utilizarse con fines comerciales.

Instalación

# Requerimientos

 *Ubuntu 20.04 LTS

*Agente MQTT: un protocolo de publicación / suscripción que permite a los usuarios publicar información sobre temas a los que otros pueden suscribirse. Una implementación popular del protocolo MQTT es Mosquitto .

*Redis: una base de datos en memoria que se utiliza para almacenar datos relativamente transitorios.

*PostgreSQL: la base de datos de almacenamiento a largo plazo que utilizan los paquetes de código abierto

*Mosquitto*

Utilice el administrador de paquetes *apt* para instalar estas dependencias:

    sudo apt install mosquitto mosquitto-clients redis-server redis-tools postgresql 

*PostgreSQL*

Configurar bases de datos y usuarios de PostgreSQL 
Para ingresar a la utilidad de línea de comando para PostgreSQL:
sudo -u postgres psql


Dentro de este mensaje, ejecute las siguientes consultas para configurar las bases de datos que utilizan los componentes de la pila de ChirpStack. Se recomienda cambiar los nombres de usuario y las contraseñas. Solo recuerde usar estos otros valores al actualizar los archivos de configuración chirpstack-network-server.toml y chirpstack-application-server.toml. 
Dado que estas dos aplicaciones utilizan la misma tabla para realizar un seguimiento de las actualizaciones de la base de datos, deben tener bases de datos independientes.


-- configurar los usuarios y las contraseñas
-- Es importante usar comillas simples y un punto y coma al final 

    create role chirpstack_as with login password 'dbpassword';
    create role chirpstack_ns with login password 'dbpassword';

-- crear la base de datos para los servidores

    create database chirpstack_as with owner chirpstack_as;
    create database chirpstack_ns with owner chirpstack_ns;

-- cambio a la base de datos del servidor de aplicaciones ChirpStack
    \c chirpstack_as

-- habilitar las extensiones pq_trgm y hstore
-- (esto es necesario para facilitar la función de búsqueda)

    create extension pg_trgm;

-- (esto es necesario para almacenar metadatos k / v adicionales)

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

-- start chirpstack-gateway-bridge
    sudo systemctl start chirpstack-gateway-bridge

-- start chirpstack-gateway-bridge on boot
    sudo systemctl enable chirpstack-gateway-bridge

# Instalación del servidor de red ChirpStack

Instale el paquete usando apt:

    sudo apt install chirpstack-network-server

El archivo de configuración se encuentra en /etc/chirpstack-network-server/chirpstack-network-server.tomly debe actualizarse para que coincida con la configuración de la base de datos y la banda. Para obtener más información sobre todas las opciones de configuración del servidor de red ChirpStack, consulte Configuración del servidor de red ChirpStack.

Para la Configuracion de AU915 

Modificar  el archivo chirpstack-network-server.toml

ruta : cd/etc/chirpstack-network-server/chirpstack-network-server.toml


    sudo -s 
    nano chirpstack-network-server.toml


#modificar la linea dns con 

    dsn="postgres://chirpstack_ns:dbpassword@localhost/chirpstack_ns?sslmode=disable"

Cambiar al plan de frecuencia utilizado:
    name=" AU915"


Modificar para que los pings slot se mantengan en una frecuencia y no hagan frequency hopping.

    ping_slot_frequency=923300000

guardar cambios 

y luego  
    exit 


Verificación de errores : 
Para iniciar ejecutar el siguiente comando:

    sudo systemctl start chirpstack-network-server

Para ver el estado ejecutar el siguiente comando:

    sudo systemctl status chirpstack-network-server




# Instalar el servidor de aplicaciones ChirpStack

Para instalar el servidor de aplicaciones ChirpStack, ejecute el siguiente comando:

    sudo apt-get install chirpstack-application-server

Después de la instalación, modifique el archivo de configuración que se encuentra en /etc/chirpstack-application-server/chirpstack-application-server.toml. 
Dado que usó la contraseña dbpassword al crear la base de datos PostgreSQL, desea cambiar la variable de configuración postgresql.dsn a:
postgres: // chirpstack_as: dbpassword @ localhost / chirpstack_as? sslmode = disable
Otra configuración obligatoria que debe cambiar es 
application_server.external_api.jwt_secret.

Modificar el archivo  chirpstack-application-server.toml
ruta :cd/etc/chirpchirpstack-application-server/stack-application-server.toml
Linea: 
[general]
log_level=4

[postgresql]
dsn="postgres://chirpstack_as:dbpassword@localhost/chirpstack_as?sslmode=disable"

[application_server.external_api]
  jwt_secret="verysecret"



Verificación de errores : 

    sudo systemctl start chirpstack-application-server

    sudo systemctl status chirpstack-application-server





Imprima el resultado del registro del servidor de aplicaciones ChirpStack:

    sudo journalctl -f -n 100 -u chirpstack-application-server




















































