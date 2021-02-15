## Laboratorio Elasticsearch

### Laboratorio 1 - Introducción

#### Requisitos

  * Docker versión >= 19.03
  * jq

#### Objetivos

  * Levantar un entorno de desarrollo de Elasticsearch + kibana
  * Ingestar nuestro primer documento.
  * Consultar creandos en nuestro cluster.

##### Ejercicios
##### Instalación Elasticsarch/kibana. Parte 1

Realizaremos el despliegue del entorno de desarrollo de elasticsearch de tipo `single node` haciendo uso de contenedores docker. 

Para ello crearemos una `network` de forma que podamos asignarle una IP estática al nodo de elasticsearch para facilitarnos el trabajo en futuros ejercicios. 

A posteriori levantaremos kibana y comprobaremos que podemos acceder. 

1. Creamos una subred con docker que nos permite asignar direcciónes estáticas.

```bash
docker network create --subnet=172.18.1.0/27 elastic_network
```

2. Levantamos un nodo de elasticsearch indicandole:

  * --net: Nombre de la red docker
  * --ip: Dirección ip asignanda al contenedor. Debe ser una dirección válida para la red indicada previamente.
  * -e "discovery.type=single-node": Indicamos mediante variable de entorno que será un cluster de tipo single node.
  * --name: Nombre del contenedor

```bash
docker run -d --net elastic_network --ip 172.18.1.2 -e "discovery.type=single-node" --name elasticsearch docker.elastic.co/elasticsearch/elasticsearch:7.10.0
```

3. Levantamos kibana.

  * --net: Nombre de la red docker
  * -e SERVER_NAME=172.18.1.2:9200: Dirección contra la que debe resolver el cluster de elasticsearch

```bash
docker run -e SERVER_NAME=172.18.1.2:9200 --net elastic_network -d --name kibana -p 5601:5601 docker.elastic.co/kibana/kibana:7.10.0
```

4. Accedemos a la URL de kibana.

  Si lo pasos previos han ido bien ya tendremos accesible nuestro kibana en localhosts, puerto 5601 `http://localhost:5601/app/home#/`

  ![kibana](img/laboratorio_es/laboratorio1_kibana.png)


##### Ingestando nuestro primer documento. Parte 2

Vamos a ingestar nuestro primer documento haciendo uso de la api que elasticsearch.
A posteriori consultaremos el indice que se ha creado y borraremos el indice.

1. Vamos a ingestar nuestro primer documento en elasticsearch. 

  ```bash
  curl -XPUT "http://172.18.1.2:9200/mi_primer_indice/_doc/1" -H 'Content-Type: application/json' -d'{"title":"Miprimerdocumentoenelasticsearch","category":"DocumentoSimple","author":{"first_name":"Pepito","last_name":"Grillo"}}'
  ```

2. Consultamos los índices en el cluster

  Si todo ha ido bien ya tendremos nuestro primer documento ingestado en elasticsearch y un índice que contiene dicho documento.

  ```bash
  ➜  ~ curl -XGET "http://172.18.1.2:9200/_cat/indices"
  green  open .apm-custom-link                fVwvr3VHRdm1sKh09xYWPw 1 0  0   0    208b    208b
  green  open .kibana_task_manager_1          jwtENLSURf-VU-0JXL-G5Q 1 0  5 165 130.1kb 130.1kb
  green  open .apm-agent-configuration        2CYX9J0HRkCpsCmvDKkuDA 1 0  0   0    208b    208b
  green  open .kibana-event-log-7.10.0-000001 4XTyMYHGQty9TV_4aNJliQ 1 0  1   0   5.6kb   5.6kb
  green  open .kibana_1                       32s4-cwuQ7uWgpxmsMq1Qw 1 0 22   7  10.4mb  10.4mb
  yellow open mi_primer_indice                67sC4QBRQTWK5SX9pFqmqg 1 1  1   0   5.8kb   5.8kb
  ```

3. Vamos a consultar el documento que acabamos de ingestar.

  ```bash
  ➜  ~ curl -s -XGET "http://172.18.1.2:9200/mi_primer_indice/_doc/1?pretty=true" | jq
  {
    "_index": "mi_primer_indice",
    "_type": "_doc",
    "_id": "1",
    "_version": 1,
    "_seq_no": 0,
    "_primary_term": 1,
    "found": true,
    "_source": {
      "title": "Miprimerdocumentoenelasticsearch",
      "category": "DocumentoSimple",
      "author": {
        "first_name": "Pepito",
        "last_name": "Grillo"
      }
    }
  }
  ```

4. Borramos el indice y todos los documentos que contenga.

  ```bash
  ➜  ~ curl -s -XDELETE "http://172.18.1.2:9200/mi_primer_indice" | jq
  {
    "acknowledged": true
  }
  ```

### Laboratorio 2 - Introducción

#### Requisitos

  * Docker versión >= 19.03
  * jq

#### Objetivos

  * Levantar un cluster de 3 nodos elasticsearch + kibana

##### Ejercicios
##### Creando un cluster 3 nodos Elasticsarch/kibana. Parte 1

Para esta practica crearemos un cluster de 3 nodos sobre docker. Para ello iremos levantando un nodo tras otro e iremos viendo como se comporta el cluster.

Por último levantaremos kibana.

1. Levantamos el primero nodo de elasticsearch.

  Esta vez no usaremos el parametro `single.node` dado que queremos levantar un cluster funcional de 3 nodos.
  Usaremos los siguientes parametros:
    * --net: Nombre de la red docker
    * --ip: Dirección ip asignanda al contenedor. Debe ser una dirección válida para la red indicada previamente.
    * --name: Nombre del contenedor
    * --ulimit memlock=-1:-1 : Quitamos los limites para no tener problemas con la asignación de recursos.
    * "node.name=es01": Nombre del nodo.
    * "cluster.name=es-docker-cluster": nombre del cluster de elasticsearch.
    * "discovery.seed_hosts=es02,es03": nombre de los otros dos nodos que forman el cluster de elasticsearch.
    * "bootstrap.memory_lock=true": Activamos el memory lock.
    * "ES_JAVA_OPTS=-Xms512m -Xmx512m": Seteamos la jvm memory de proceso java.
  ```bash
  ➜  ~ docker run -d --net elastic_network --ip 172.18.1.2 --ulimit memlock=-1:-1  -e "node.name=es01" -e "cluster.name=es-docker-cluster" -e "discovery.seed_hosts=es02,es03" -e "cluster.initial_master_nodes=es01,es02,es03" -e "bootstrap.memory_lock=true" -e "ES_JAVA_OPTS=-Xms512m -Xmx512m" --name es01 docker.elastic.co/elasticsearch/elasticsearch:7.10.0
  ```

  Una vez levantado el log comenzará a mostrar errores dado que no encuentra los otros dos nodos que conforman el cluster.

  ```bash
  ➜  ~ docker logs --tail=30 es01
  {"type": "server", "timestamp": "2021-02-15T14:58:33,647Z", "level": "WARN", "component": "o.e.d.SeedHostsResolver", "cluster.name": "es-docker-cluster", "node.name": "es01", "message": "failed to resolve host [es03]", "cluster.uuid": "lqStUOrUSFS9OrZ5xUTiZg", "node.id": "MrHS8IFIQ1e-9FSFOvekBg" ,
  "stacktrace": ["java.net.UnknownHostException: es03",
  "at java.net.InetAddress$CachedAddresses.get(InetAddress.java:800) ~[?:?]",
  "at java.net.InetAddress.getAllByName0(InetAddress.java:1507) ~[?:?]",
  "at java.net.InetAddress.getAllByName(InetAddress.java:1366) ~[?:?]",
  "at java.net.InetAddress.getAllByName(InetAddress.java:1300) ~[?:?]",
  "at org.elasticsearch.transport.TcpTransport.parse(TcpTransport.java:556) ~[elasticsearch-7.10.0.jar:7.10.0]",
  "at org.elasticsearch.transport.TcpTransport.addressesFromString(TcpTransport.java:498) ~[elasticsearch-7.10.0.jar:7.10.0]",
  "at org.elasticsearch.transport.TransportService.addressesFromString(TransportService.java:864) ~[elasticsearch-7.10.0.jar:7.10.0]"

  ```

2. Levantamos el segundo nodo `es02`. Usamos el mismo procedimiento anteriormente descripto cambiando las variables oportunas

  Necesitamos modificar la ip, el node.name, discovery.seed_hosts y --name del contenedor

  ```bash
  ➜  ~ sudo docker run -d --net elastic_network --ip 172.18.1.3 --ulimit memlock=-1:-1  -e "node.name=es02" -e "cluster.name=es-docker-cluster" -e "discovery.seed_hosts=es01,es03" -e "cluster.initial_master_nodes=es01,es02,es03" -e "bootstrap.memory_lock=true" -e "ES_JAVA_OPTS=-Xms512m -Xmx512m" --name es02 docker.elastic.co/elasticsearch/elasticsearch:7.10.0
  ```

  Vemos que una vez desplegado el nodo 2 los logs de es01 deberían dejar de mostrar warning e indicar que ha descubierto el nodo es02

  ```bash
  ➜  ~ docker logs --tail=30 es01
  {"type": "server", "timestamp": "2021-02-15T15:10:41,156Z", "level": "INFO", "component": "o.e.c.c.CoordinationState", "cluster.name": "es-docker-cluster", "node.name": "es01", "message": "cluster UUID set to [rnUF_FKRS42ni1DmX7x36Q]" }
  {"type": "server", "timestamp": "2021-02-15T15:10:41,245Z", "level": "INFO", "component": "o.e.c.s.ClusterApplierService", "cluster.name": "es-docker-cluster", "node.name": "es01", "message": "master node changed {previous [], current [{es02}{le9oqAsIQA-qpyDY5Di6hA}{BIdQvgvpShathzFm_7Tz4A}{172.18.1.3}{172.18.1.3:9300}{cdhilmrstw}{ml.machine_memory=16702734336, ml.max_open_jobs=20, xpack.installed=true, transform.node=true}]}, added {{es02}{le9oqAsIQA-qpyDY5Di6hA}{BIdQvgvpShathzFm_7Tz4A}{172.18.1.3}{172.18.1.3:9300}{cdhilmrstw}{ml.machine_memory=16702734336, ml.max_open_jobs=20, xpack.installed=true, transform.node=true}}, term: 1, version: 1, reason: ApplyCommitRequest{term=1, version=1, sourceNode={es02}{le9oqAsIQA-qpyDY5Di6hA}{BIdQvgvpShathzFm_7Tz4A}{172.18.1.3}{172.18.1.3:9300}{cdhilmrstw}{ml.machine_memory=16702734336, ml.max_open_jobs=20, xpack.installed=true, transform.node=true}}" }
  
  ```

3. Mismo procedimiento anterior seteando las variables para el nodo3.

  Necesitamos modificar la ip, el node.name, discovery.seed_hosts y --name del contenedor

  ```bash
  ➜  ~ sudo docker run -d --net elastic_network --ip 172.18.1.4 --ulimit memlock=-1:-1  -e "node.name=es03" -e "cluster.name=es-docker-cluster" -e "discovery.seed_hosts=es01,es02" -e "cluster.initial_master_nodes=es01,es02,es03" -e "bootstrap.memory_lock=true" -e "ES_JAVA_OPTS=-Xms512m -Xmx512m" --name es03 docker.elastic.co/elasticsearch/elasticsearch:7.10.0
  ```

  Una vez levantado el tercer nodo veremos un mensaje en los logs del nodo 1 y 2 que el 3er nodo se ha incorporado al cluster.

  ```bash
  ➜  ~ docker logs --tail=30 es01
  {"type": "server", "timestamp": "2021-02-15T15:15:09,899Z", "level": "INFO", "component": "o.e.c.s.ClusterApplierService", "cluster.name": "es-docker-cluster", "node.name": "es01", "message": "added {{es03}{xZEkA7zkSLqtJo3n2h9k3g}{i6OI5rM7RN2C9R89-qtUrQ}{172.18.1.4}{172.18.1.4:9300}{cdhilmrstw}{ml.machine_memory=16702734336, ml.max_open_jobs=20, xpack.installed=true, transform.node=true}}, term: 1, version: 41, reason: ApplyCommitRequest{term=1, version=41, sourceNode={es02}{le9oqAsIQA-qpyDY5Di6hA}{BIdQvgvpShathzFm_7Tz4A}{172.18.1.3}{172.18.1.3:9300}{cdhilmrstw}{ml.machine_memory=16702734336, ml.max_open_jobs=20, xpack.installed=true, transform.node=true}}", "cluster.uuid": "rnUF_FKRS42ni1DmX7x36Q", "node.id": "RbWseV8SQmSpgdqRUfyafQ"  }
  ```

4. Comprobamos el estado del cluster.

  Para ello vamos a hacer uso de la api `_cluster/health` preguntando contra la ip de cualquier nodo.

  ```bash
  ➜  ~ curl -s -XGET "http://172.18.1.2:9200/_cluster/health" | jq
  {
    "cluster_name": "es-docker-cluster",
    "status": "green",
    "timed_out": false,
    "number_of_nodes": 3,
    "number_of_data_nodes": 3,
    "active_primary_shards": 0,
    "active_shards": 0,
    "relocating_shards": 0,
    "initializing_shards": 0,
    "unassigned_shards": 0,
    "delayed_unassigned_shards": 0,
    "number_of_pending_tasks": 0,
    "number_of_in_flight_fetch": 0,
    "task_max_waiting_in_queue_millis": 0,
    "active_shards_percent_as_number": 100
  }
  ```

5. Consultamos que nodo está ejerciendo como master.

  Usamos la `api _cat/nodes` para consultar estado de nodos y quién es el master

  ```bash
  ➜  ~ curl -s -XGET "http://172.18.1.2:9200/_cat/nodes?v"
    ip         heap.percent ram.percent cpu load_1m load_5m load_15m node.role  master name
    172.18.1.3           40          79  29    1.67    1.73     1.72 cdhilmrstw *      es02
    172.18.1.2           43          79  29    1.67    1.73     1.72 cdhilmrstw -      es01
    172.18.1.4           23          79  29    1.67    1.73     1.72 cdhilmrstw -      es03
  ```
  Vemos que el nodo es02 está ejerciendo el rol de master.


6. Vamos a parar el nodo que estar ejerciciendo y ver si el cluster asigna un nuevo master.

  Paramos el nodo master

  ```bash
  ➜  ~ docker stop es02
  es02

  ```
  Consultamos que nodo está ejerciendo como master

  ```bash
  ➜  ~ curl -s -XGET "http://172.18.1.2:9200/_cat/nodes?v"
    ip         heap.percent ram.percent cpu load_1m load_5m load_15m node.role  master name
    172.18.1.2           59          73  30    1.67    1.70     1.71 cdhilmrstw *      es01
    172.18.1.4           33          73  30    1.67    1.70     1.71 cdhilmrstw -      es03
  ```
  Vemos que ahora aparece un nodo menos y el nuevo master es es01

  Levantamos el nodo de nuevo

  ```bash
  ➜  ~ docker start es02
  es02
  ```
  Consultamos estado de los nodos

  ```bash
  ➜  ~ curl -s -XGET "http://172.18.1.2:9200/_cat/nodes?v"
    ip         heap.percent ram.percent cpu load_1m load_5m load_15m node.role  master name
    172.18.1.2           30          78  36    2.14    1.66     1.67 cdhilmrstw *      es01
    172.18.1.3           45          79  75    2.14    1.66     1.67 cdhilmrstw -      es02
    172.18.1.4           49          78  36    2.14    1.66     1.67 cdhilmrstw -      es03
  ```
