# practica-cluster-mongodb
"Cluster distribuido de MongoDB con Docker y Sharding".

# Practica: Cluster Distribuido de MongoDB con Docker (Sharding)

**Alumno:** Luis Eduardo Salas Escapite  
**Matricula:** 353302  
**Materia:** Bases de Datos Avanzadas  
**Universidad Autonoma de Chihuahua (UACH)**

---

## 1. Introduccion y Arquitectura del Cluster
El objetivo de esta practica es implementar un entorno de base de datos distribuida utilizando **MongoDB** sobre **Docker**, configurando una arquitectura de fragmentacion (*sharding*) para repartir grandes volumenes de datos de manera horizontal en multiples nodos.

La arquitectura desplegada consta de 5 contenedores interconectados mediante una red puente (`cluster-net`):
* **`configSrv` (Config Server):** Almacena los metadatos y la configuracion del closter (ubicacion de los datos en los shards).
* **`shard1`, `shard2`, `shard3` (Shards):** Nodos de almacenamiento independientes configurados como Replica Sets que contienen los fragmentos de los datos reales.
* **`router` (Mongos Router):** El punto de entrada unificado. Las aplicaciones o consultas se conectan a este nodo, el cual redirige transparentemente las operaciones al shard correspondiente.

---

## 2. Archivo Docker Compose (`docker-compose.yml`)
La infraestructura fue levantada utilizando el siguiente archivo de orquestacion en Docker:

```yaml
version: '3.8'

networks:
  cluster-net:
    driver: bridge

services:
  configSrv:
    image: mongo:latest
    container_name: configSrv
    command: ["mongod", "--configsvr", "--replSet", "configReplSet", "--port", "27019"]
    ports:
      - "27019:27019"
    networks:
      - cluster-net

  shard1:
    image: mongo:latest
    container_name: shard1
    command: ["mongod", "--shardsvr", "--replSet", "shard1ReplSet", "--port", "27018"]
    ports:
      - "27018:27018"
    networks:
      - cluster-net

  shard2:
    image: mongo:latest
    container_name: shard2
    command: ["mongod", "--shardsvr", "--replSet", "shard2ReplSet", "--port", "27018"]
    ports:
      - "27020:27018"
    networks:
      - cluster-net

  shard3:
    image: mongo:latest
    container_name: shard3
    command: ["mongod", "--shardsvr", "--replSet", "shard3ReplSet", "--port", "27018"]
    ports:
      - "27021:27018"
    networks:
      - cluster-net

  router:
    image: mongo:latest
    container_name: router
    command: ["mongos", "--configdb", "configReplSet/configSrv:27019", "--bind_ip_all", "--port", "27017"]
    ports:
      - "27017:27017"
    networks:
      - cluster-net
    depends_on:
      - configSrv
      - shard1
      - shard2
      - shard3
```
<img width="1600" height="908" alt="WhatsApp Image 2026-11-05 at 1 00 58 AM" src="https://github.com/user-attachments/assets/5ce5dbcb-d747-4cc1-b05b-46c1c2a952fb" />

# Operaciones de Configuracion Inicial #

Se inicializaron los conjuntos de replicas en cada contenedor y se registraron los shards en el router mongos mediante los siguientes comandos:

    Inicializacion del Config Server y Shards (rs.initiate)

    Adicion de Shards al Router:
    ```
    sh.addShard("shard1ReplSet/shard1:27018")
    sh.addShard("shard2ReplSet/shard2:27018")
    sh.addShard("shard3ReplSet/shard3:27018")

    ```
<img width="603" height="627" alt="WhatsApp Image 2026-11-05 at 1AM" src="https://github.com/user-attachments/assets/bf3f75cd-fba3-4765-bf72-7133a1c8de96" />

# Habilitacion de Sharding y Llave de Fragmentacion (Shard Key): #
````
sh.enableSharding("BDAutos")
db.cars.createIndex({ "maker_id": "hashed" })
sh.shardCollection("BDAutos.cars", { "maker_id": "hashed" })

````
<img width="730" height="444" alt="WhatsApp Image 2026-11-05 at 1 18 34 AM" src="https://github.com/user-attachments/assets/9da51694-3315-4131-842f-f930c22906e7" />

# Operaciones CRUD en el Cluster Distribuido #

Se realizaron pruebas de insercion, consulta y gestion de datos a traves del puerto del router (27017):

Insercion de Datos (Create)

````
db.cars.insertMany([
  { maker_id: 1, brand: "Volkswagen", model: "Jetta 1.8T", year: 2004, color: "Negro" },
  { maker_id: 2, brand: "Chevrolet", model: "Camaro V6", year: 2010, color: "Rojo" },
  { maker_id: 1, brand: "Volkswagen", model: "Golf GTI", year: 2012, color: "Blanco" },
  { maker_id: 3, brand: "Ford", model: "Mustang", year: 2018, color: "Azul" }
])
````
<img width="1177" height="607" alt="WhatsApp Image 2026-11-05 at 1 24 42 AM" src="https://github.com/user-attachments/assets/f2e626de-93e0-4f4a-9c7f-8ca02f94f1e1" />

# Consulta de datos(read) # 
````db.cars.find().pretty()````

<img width="734" height="803" alt="WhatsApp Image 2026-11-05 at 1 27 47 AM" src="https://github.com/user-attachments/assets/309e47f3-1276-48fa-b2d1-a15983575dd1" />

# Evidencia de Distribución de Datos (Sharding) #

**Para comprobar que el clúster distribuyó la información de manera efectiva entre los fragmentos definidos, se ejecutó el comando de distribución:**

````
db.cars.getShardDistribution()
````
<img width="754" height="738" alt="WhatsApp Image 2026-11-05 at 1 29 50 AM" src="https://github.com/user-attachments/assets/925a5ffb-6192-42be-97e4-ab55877d7b6a" />

# Conclusiones #

````
La implementación de este clúster demostró la potencia de las bases de datos distribuidas al permitir
escalar el almacenamiento de forma horizontal. El uso de un router mongos simplifica por completo
las operaciones CRUD para los clientes, mientras que el motor de MongoDB gestiona de manera
transparente y automatizada la fragmentación de los documentos utilizando una llave
hash (hashed shard key), garantizando un balance eficiente de la carga de
trabajo entre los diferentes nodos del sistema.
````






    
