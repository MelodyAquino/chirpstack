# chirpstack

Este tutorial describe los pasos necesarios para configurar la pila ChirpStack, incluidos todos los requisitos en una sola máquina.


Ubuntu 20.04 LTS

Dependencias

Agente MQTT : un protocolo de publicación / suscripción que permite a los usuarios publicar información sobre temas a los que otros pueden suscribirse. Una implementación popular del protocolo MQTT es Mosquitto .

Redis : una base de datos en memoria que se utiliza para almacenar datos relativamente transitorios.

PostgreSQL : la base de datos de almacenamiento a largo plazo que utilizan los paquetes de código abierto


Utilice el administrador de paquetes aptpara instalar estas dependencias:

sudo apt install mosquitto mosquitto-clients redis-server redis-tools postgresql 


Configurar bases de datos y usuarios de PostgreSQL



Para ingresar a la utilidad de línea de comando para PostgreSQL:






