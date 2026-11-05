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







    
