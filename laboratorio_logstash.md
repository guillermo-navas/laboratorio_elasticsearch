## Laboratorio Logstash

### Laboratorio 1 - Instalación Introducción&Configuración

#### Requisitos


#### Objetivos

  * Realizar una instalación de logstash

#### Ejercicios
##### Instalación Logstash. Parte 1

Vamos a realizar las instalación de una instancia de logstash.

Recursos: https://www.elastic.co/guide/en/logstash/current/installing-logstash.html
Para otras distribuciones consultar el enlance anterior.


*Ejemplo instalación para Rhel&Centos*

1. Descargamos e instalamos la Public Key de elasticsearch

  ```bash
    sudo rpm --import https://artifacts.elastic.co/GPG-KEY-elasticsearch
  ```

2. Añadimos el respositorio yum al path `/etc/yum.repos.d/` como `logstash.repo`

  ```bash
    [logstash-7.x]
    name=Elastic repository for 7.x packages
    baseurl=https://artifacts.elastic.co/packages/7.x/yum
    gpgcheck=1
    gpgkey=https://artifacts.elastic.co/GPG-KEY-elasticsearch
    enabled=1
    autorefresh=1
    type=rpm-md
  ```

3. Instalamos Logstash

    ```bash
      sudo yum install logstash
    ```

4. Comprobamos la versión de logstash instalada.

  <details><summary>Solución</summary>

  ```bash
    logstash --version
  ```

  </details>

##### Creando la primera pipeline. Parte 2

En esta segunda parte vamos a crear nuestra primera pipeline logstash.

###### Ejercicio 1

Para esta primera parte queremos crear una pipelines con las siguientes premisas:

* **Input**: que permita al usuario introducir por pantalla el mensaje `Hola mundo $IP`, siendo la $IP una ip válida.
>https://www.elastic.co/guide/en/logstash/current/plugins-outputs-stdout.html
* **Filter**: Parsear el mensaje que se introduce en el input y geolocalizar la IP.
>https://www.elastic.co/guide/en/logstash/current/plugins-filters-grok.html
>https://grokdebug.herokuapp.com/
>https://www.elastic.co/guide/en/logstash/current/plugins-filters-geoip.html
* **Output**: Sacar la solución por la salida stdout.
>https://www.elastic.co/guide/en/logstash/current/plugins-outputs-stdout.html


  <details><summary>Solución</summary>

  El fichero de configuración de la pipeline `pipelines_geoip.conf`:

  ```bash
    input {
      stdin {}
    }
    filter {
      grok {
        match => {"message" => "%{IP:ip}"}
      }
      geoip {
        source => "ip"
      }
    }
    output {
      stdout {
        codec => rubydebug
      }
    }
  ```

  Para realizar la ejecución:

  ```bash
    ➜  laboratorio_elasticsearch git:(master) ✗ sudo logstash -f pipeline1.conf
    [sudo] contraseña para afortes:
    OpenJDK 64-Bit Server VM warning: Option UseConcMarkSweepGC was deprecated in version 9.0 and will likely be removed in a future release.
    WARNING: An illegal reflective access operation has occurred
    WARNING: Illegal reflective access by com.headius.backport9.modules.Modules (file:/usr/share/logstash/logstash-core/lib/jars/jruby-complete-9.2.8.0.jar) to field java.io.FileDescriptor.fd
    WARNING: Please consider reporting this to the maintainers of com.headius.backport9.modules.Modules
    WARNING: Use --illegal-access=warn to enable warnings of further illegal reflective access operations
    WARNING: All illegal access operations will be denied in a future release
    Thread.exclusive is deprecated, use Thread::Mutex
    WARNING: Could not find logstash.yml which is typically located in $LS_HOME/config or /etc/logstash. You can specify the path using --path.settings. Continuing using the defaults
    Could not find log4j2 configuration at path /usr/share/logstash/config/log4j2.properties. Using default config which logs errors to the console
    [WARN ] 2021-02-17 07:13:16.688 [LogStash::Runner] multilocal - Ignoring the 'pipelines.yml' file because modules or command line options are specified
    [INFO ] 2021-02-17 07:13:16.703 [LogStash::Runner] runner - Starting Logstash {"logstash.version"=>"7.4.2"}
    [INFO ] 2021-02-17 07:13:18.156 [Converge PipelineAction::Create<main>] Reflections - Reflections took 38 ms to scan 1 urls, producing 20 keys and 40 values
    [INFO ] 2021-02-17 07:13:18.761 [[main]-pipeline-manager] geoip - Using geoip database {:path=>"/usr/share/logstash/vendor/bundle/jruby/2.5.0/gems/logstash-filter-geoip-6.0.3-java/vendor/GeoLite2-City.mmdb"}
    [WARN ] 2021-02-17 07:13:18.933 [[main]-pipeline-manager] LazyDelegatingGauge - A gauge metric of an unknown type (org.jruby.RubyArray) has been create for key: cluster_uuids. This may result in invalid serialization.  It is recommended to log an issue to the responsible developer/development team.
    [INFO ] 2021-02-17 07:13:18.936 [[main]-pipeline-manager] javapipeline - Starting pipeline {:pipeline_id=>"main", "pipeline.workers"=>4, "pipeline.batch.size"=>125, "pipeline.batch.delay"=>50, "pipeline.max_inflight"=>500, :thread=>"#<Thread:0x4ec90181 run>"}
    [INFO ] 2021-02-17 07:13:19.010 [[main]-pipeline-manager] javapipeline - Pipeline started {"pipeline.id"=>"main"}
    The stdin plugin is now waiting for input:
    [INFO ] 2021-02-17 07:13:19.089 [Agent thread] agent - Pipelines running {:count=>1, :running_pipelines=>[:main], :non_running_pipelines=>[]}
    [INFO ] 2021-02-17 07:13:19.363 [Api Webserver] agent - Successfully started Logstash API endpoint {:port=>9600}
    HOLA 95.121.15.62
    /usr/share/logstash/vendor/bundle/jruby/2.5.0/gems/awesome_print-1.7.0/lib/awesome_print/formatters/base_formatter.rb:31: warning: constant ::Fixnum is deprecated
    {
              "host" => "blackbox",
                "ip" => "95.121.15.62",
             "geoip" => {
             "country_code3" => "ES",
             "country_code2" => "ES",
                  "location" => {
                "lon" => -3.7016,
                "lat" => 40.4143
            },
              "country_name" => "Spain",
               "region_code" => "M",
               "region_name" => "Madrid",
                        "ip" => "95.121.15.62",
            "continent_code" => "EU",
                  "latitude" => 40.4143,
                  "timezone" => "Europe/Madrid",
                 "longitude" => -3.7016,
               "postal_code" => "28039",
                 "city_name" => "Madrid"
        },
          "@version" => "1",
        "@timestamp" => 2021-02-17T06:14:05.468Z,
           "message" => "HOLA 95.121.15.62"
    }
    ^C[WARN ] 2021-02-17 07:15:15.218 [SIGINT handler] runner - SIGINT received. Shutting down.
    [INFO ] 2021-02-17 07:15:15.417 [Converge PipelineAction::Stop<main>] javapipeline - Pipeline terminated {"pipeline.id"=>"main"}
    [INFO ] 2021-02-17 07:15:15.455 [LogStash::Runner] runner - Logstash shut down.
  ```
  </details>


###### Ejercicio 2

Partiendo de la pipeline que acabamos de crear añadir:

* **Input**: Pasar en el mensaje `Hola mundo $IP $TIMESTAMP`, siendo $TIMESTAMP un formato fecha ISO8601. Ejemplo: `Hola mundo 95.121.15.62 2011-04-19T03:44:01.103Z`
>https://www.elastic.co/guide/en/logstash/current/plugins-filters-date.html
* **Filter**: Parsear la fecha introducida y setearla en el field `@timestamp`.
>https://www.elastic.co/guide/en/logstash/current/plugins-filters-grok.html
>https://grokdebug.herokuapp.com/
>https://www.elastic.co/guide/en/logstash/current/plugins-filters-geoip.html



  <details><summary>Solución</summary>

  El fichero de configuración de la pipeline `pipelines_geoip.conf`:

  ```bash
    input {
      stdin {}
    }
    filter {
      grok {
        match => {"message" => "%{IP:ip} %{TIMESTAMP_ISO8601:Fecha}"}
      }
      geoip {
        source => "ip"
      }
      date {
        match => ["Fecha", "ISO8601"]
      }
    }
    output {
      stdout {
        codec => rubydebug
      }
    }
  ```

  Ejecución

  ```bash
    ➜  laboratorio_elasticsearch git:(master) ✗ sudo logstash -f pipeline1.conf
    [INFO ] 2021-02-17 07:35:55.388 [Api Webserver] agent - Successfully started Logstash API endpoint {:port=>9600}
    Hola mundo 95.121.15.62 2011-04-19T03:44:01.103Z
    /usr/share/logstash/vendor/bundle/jruby/2.5.0/gems/awesome_print-1.7.0/lib/awesome_print/formatters/base_formatter.rb:31: warning: constant ::Fixnum is deprecated
    {
             "geoip" => {
               "region_name" => "Madrid",
                 "longitude" => -3.7016,
                        "ip" => "95.121.15.62",
            "continent_code" => "EU",
             "country_code3" => "ES",
                  "latitude" => 40.4143,
                 "city_name" => "Madrid",
              "country_name" => "Spain",
               "postal_code" => "28039",
               "region_code" => "M",
                  "timezone" => "Europe/Madrid",
                  "location" => {
                "lon" => -3.7016,
                "lat" => 40.4143
            },
             "country_code2" => "ES"
        },
              "host" => "blackbox",
                "ip" => "95.121.15.62",
             "Fecha" => "2011-04-19T03:44:01.103Z",
          "@version" => "1",
        "@timestamp" => 2011-04-19T03:44:01.103Z,
           "message" => "Hola mundo 95.121.15.62 2011-04-19T03:44:01.103Z"
    }
  ```
  </details>


### Laboratorio 2 - Trabajando con inputs, filters y outputs

#### Requisitos

  * Logstash

#### Objetivos

  * Conocer y trabajar con el input file de logstash.
  * Saber como procesar ficheros de texto vía logstash.
  * Aplicar patrones grok a ficheros de aplicación reales.
  * Configurar filter date.
  * Configurar filter geoip.
  * Aprender a procesar useragent

#### Preparación de set de datos

1. Lo primero que realizarmos es descargarnos los logs de ejemplso apache que tendremos que procesar

  ```bash
    wget https://raw.githubusercontent.com/elastic/examples/master/Common%20Data%20Formats/apache_logs/apache_logs --output-document apache.logs
  ```

#### Ejercicios
##### Ejercicio:1 Procesando logs de apache.

El objetivo de este ejercicio es procesar el set de datos de ejemplo, logs de apache previamente descargados, con logstash que separemos cada campo del log el fields distintos.

Una vez tengamos cada traza de logs debidamente categorizada debemos setear el timestamp de cada traza de log como @timestamp para que el tiempo se categorice correctamente.

Por otro lado debemos geolocalizar la ip que viene en cada traza del log.

Debemos parsear el user agent que viene en los logs de forma que podamos analizar sobre que navegador se ha ejecutado la petición.

Por último debemos ejecutar la pipelines y que se muestren todos los resultados por stdout.

  <details><summary>Solución</summary>

  El fichero de configuración de la pipeline `apache_workshow.conf`:

  ```bash
    input {
      file {
        path => ["/home/afortes/Documentos/laboratorio_elasticsearch/apache.logs"]
        start_position => "beginning"
        mode => "read"
        type => "apache_access"
        }
    }
    filter {
      grok {
        match => {
          "message" => '%{IPORHOST:clientip} %{USER:ident} %{USER:auth} \[%{HTTPDATE:timestamp}\] "%{WORD:verb} %{DATA:request} HTTP/%{NUMBER:httpversion}" %{NUMBER:response:int} (?:-|%{NUMBER:bytes:int}) %{QS:referrer} %{QS:agent}'
        }
      }
    
      date {
        match => [ "timestamp", "dd/MMM/YYYY:HH:mm:ss Z" ]
        locale => en
      }
    
      geoip {
        source => "clientip"
      }
    
      useragent {
        source => "agent"
        target => "useragent"
      }
    }
    output {
      stdout { codec => rubydebug }
    }
  ```

  Ejecución

  ```bash
    ➜  laboratorio_elasticsearch git:(master) ✗ sudo logstash -f apache_workshow.conf
    {
               "path" => "/home/afortes/Documentos/laboratorio_elasticsearch/apache.logs",
              "bytes" => 1015,
           "@version" => "1",
              "agent" => "\"Mozilla/5.0 (iPhone; CPU iPhone OS 7_0_4 like Mac OS X) AppleWebKit/537.51.1 (KHTML, like Gecko) Version/7.0 Mobile/11B554a Safari/9537.53\"",
           "response" => 200,
               "host" => "blackbox",
            "message" => "166.147.88.15 - - [17/May/2015:15:05:44 +0000] \"GET /reset.css HTTP/1.1\" 200 1015 \"http://www.semicomplete.com/\" \"Mozilla/5.0 (iPhone; CPU iPhone OS 7_0_4 like Mac OS X) AppleWebKit/537.51.1 (KHTML, like Gecko) Version/7.0 Mobile/11B554a Safari/9537.53\"",
            "request" => "/reset.css",
              "geoip" => {
                        "ip" => "166.147.88.15",
                  "latitude" => 37.751,
                  "timezone" => "America/Chicago",
             "country_code3" => "US",
                 "longitude" => -97.822,
            "continent_code" => "NA",
                  "location" => {
                "lat" => 37.751,
                "lon" => -97.822
            },
              "country_name" => "United States",
             "country_code2" => "US"
        },
          "timestamp" => "17/May/2015:15:05:44 +0000",
           "referrer" => "\"http://www.semicomplete.com/\"",
              "ident" => "-",
               "type" => "apache_access",
         "@timestamp" => 2015-05-17T15:05:44.000Z,
               "auth" => "-",
               "verb" => "GET",
        "httpversion" => "1.1",
           "clientip" => "166.147.88.15",
          "useragent" => {
                "name" => "Mobile Safari",
            "os_minor" => "0",
               "major" => "7",
              "device" => "iPhone",
            "os_major" => "7",
                  "os" => "iOS",
               "minor" => "0",
             "os_name" => "iOS",
               "build" => ""
        }
    }
    {
               "path" => "/home/afortes/Documentos/laboratorio_elasticsearch/apache.logs",
              "bytes" => 4877,
           "@version" => "1",
              "agent" => "\"Mozilla/5.0 (iPhone; CPU iPhone OS 7_0_4 like Mac OS X) AppleWebKit/537.51.1 (KHTML, like Gecko) Version/7.0 Mobile/11B554a Safari/9537.53\"",
           "response" => 200,
               "host" => "blackbox",
            "message" => "166.147.88.15 - - [17/May/2015:15:05:58 +0000] \"GET /style2.css HTTP/1.1\" 200 4877 \"http://www.semicomplete.com/\" \"Mozilla/5.0 (iPhone; CPU iPhone OS 7_0_4 like Mac OS X) AppleWebKit/537.51.1 (KHTML, like Gecko) Version/7.0 Mobile/11B554a Safari/9537.53\"",
            "request" => "/style2.css",
              "geoip" => {
                        "ip" => "166.147.88.15",
                  "latitude" => 37.751,
                  "timezone" => "America/Chicago",
             "country_code3" => "US",
                 "longitude" => -97.822,
            "continent_code" => "NA",
                  "location" => {
                "lat" => 37.751,
                "lon" => -97.822
            },
              "country_name" => "United States",
             "country_code2" => "US"
        },
          "timestamp" => "17/May/2015:15:05:58 +0000",
           "referrer" => "\"http://www.semicomplete.com/\"",
              "ident" => "-",
               "type" => "apache_access",
         "@timestamp" => 2015-05-17T15:05:58.000Z,
               "auth" => "-",
               "verb" => "GET",
        "httpversion" => "1.1",
           "clientip" => "166.147.88.15",
          "useragent" => {
                "name" => "Mobile Safari",
            "os_minor" => "0",
               "major" => "7",
              "device" => "iPhone",
            "os_major" => "7",
                  "os" => "iOS",
               "minor" => "0",
             "os_name" => "iOS",
               "build" => ""
        }
    }
    (...)
  ```
  </details>


##### Ejercicio:2 Procesando logs de apache.


Para este ejercicio partiremos de la pipeline anteriormente creada y tendremos que ingestar los documentos en elasticsearch en un indice que llamaremos `apache`.

* Debe crearse un índice apache por día, rotado dirario.

* Cuando realicemos la ingesta debemos mantener la salida por pantalla que hemos creado en el ejercicio anterior en aras de poder debuggar los resultados.

* Crearemos en kibana un index pattern para poder consultar el índice.

* Consultaremos desde kibana -> pestaña de discovery la información que hemos ingestado.

  <details><summary>Solución</summary>

  El fichero de configuración de la pipeline `apache_workshow.conf`:

  ```bash
    input {
      file {
        path => ["/home/afortes/Documentos/laboratorio_elasticsearch/apache.logs"]
        start_position => "beginning"
        mode => "read"
        type => "apache_access"
        }
    }
    filter {
      grok {
        match => {
          "message" => '%{IPORHOST:clientip} %{USER:ident} %{USER:auth} \[%{HTTPDATE:timestamp}\] "%{WORD:verb} %{DATA:request} HTTP/%{NUMBER:httpversion}" %{NUMBER:response:int} (?:-|%{NUMBER:bytes:int}) %{QS:referrer} %{QS:agent}'
        }
      }
    
      date {
        match => [ "timestamp", "dd/MMM/YYYY:HH:mm:ss Z" ]
        locale => en
      }
    
      geoip {
        source => "clientip"
      }
    
      useragent {
        source => "agent"
        target => "useragent"
      }
    }
    output {
      stdout { codec => rubydebug }
      elasticsearch {
          hosts => ["172.18.1.2:9200","172.18.1.3:9200","172.18.1.4:9200"]
          index => "apache-%{+YYYY.MM.dd}"
      }
    }
  ```

  Ejecución

  ```bash
    ➜  laboratorio_elasticsearch git:(master) ✗ sudo logstash -f apache_workshow.conf
    {
               "path" => "/home/afortes/Documentos/laboratorio_elasticsearch/apache.logs",
              "bytes" => 1015,
           "@version" => "1",
              "agent" => "\"Mozilla/5.0 (iPhone; CPU iPhone OS 7_0_4 like Mac OS X) AppleWebKit/537.51.1 (KHTML, like Gecko) Version/7.0 Mobile/11B554a Safari/9537.53\"",
           "response" => 200,
               "host" => "blackbox",
            "message" => "166.147.88.15 - - [17/May/2015:15:05:44 +0000] \"GET /reset.css HTTP/1.1\" 200 1015 \"http://www.semicomplete.com/\" \"Mozilla/5.0 (iPhone; CPU iPhone OS 7_0_4 like Mac OS X) AppleWebKit/537.51.1 (KHTML, like Gecko) Version/7.0 Mobile/11B554a Safari/9537.53\"",
            "request" => "/reset.css",
              "geoip" => {
                        "ip" => "166.147.88.15",
                  "latitude" => 37.751,
                  "timezone" => "America/Chicago",
             "country_code3" => "US",
                 "longitude" => -97.822,
            "continent_code" => "NA",
                  "location" => {
                "lat" => 37.751,
                "lon" => -97.822
            },
              "country_name" => "United States",
             "country_code2" => "US"
        },
          "timestamp" => "17/May/2015:15:05:44 +0000",
           "referrer" => "\"http://www.semicomplete.com/\"",
              "ident" => "-",
               "type" => "apache_access",
         "@timestamp" => 2015-05-17T15:05:44.000Z,
               "auth" => "-",
               "verb" => "GET",
        "httpversion" => "1.1",
           "clientip" => "166.147.88.15",
          "useragent" => {
                "name" => "Mobile Safari",
            "os_minor" => "0",
               "major" => "7",
              "device" => "iPhone",
            "os_major" => "7",
                  "os" => "iOS",
               "minor" => "0",
             "os_name" => "iOS",
               "build" => ""
        }
    }
    {
               "path" => "/home/afortes/Documentos/laboratorio_elasticsearch/apache.logs",
              "bytes" => 4877,
           "@version" => "1",
              "agent" => "\"Mozilla/5.0 (iPhone; CPU iPhone OS 7_0_4 like Mac OS X) AppleWebKit/537.51.1 (KHTML, like Gecko) Version/7.0 Mobile/11B554a Safari/9537.53\"",
           "response" => 200,
               "host" => "blackbox",
            "message" => "166.147.88.15 - - [17/May/2015:15:05:58 +0000] \"GET /style2.css HTTP/1.1\" 200 4877 \"http://www.semicomplete.com/\" \"Mozilla/5.0 (iPhone; CPU iPhone OS 7_0_4 like Mac OS X) AppleWebKit/537.51.1 (KHTML, like Gecko) Version/7.0 Mobile/11B554a Safari/9537.53\"",
            "request" => "/style2.css",
              "geoip" => {
                        "ip" => "166.147.88.15",
                  "latitude" => 37.751,
                  "timezone" => "America/Chicago",
             "country_code3" => "US",
                 "longitude" => -97.822,
            "continent_code" => "NA",
                  "location" => {
                "lat" => 37.751,
                "lon" => -97.822
            },
              "country_name" => "United States",
             "country_code2" => "US"
        },
          "timestamp" => "17/May/2015:15:05:58 +0000",
           "referrer" => "\"http://www.semicomplete.com/\"",
              "ident" => "-",
               "type" => "apache_access",
         "@timestamp" => 2015-05-17T15:05:58.000Z,
               "auth" => "-",
               "verb" => "GET",
        "httpversion" => "1.1",
           "clientip" => "166.147.88.15",
          "useragent" => {
                "name" => "Mobile Safari",
            "os_minor" => "0",
               "major" => "7",
              "device" => "iPhone",
            "os_major" => "7",
                  "os" => "iOS",
               "minor" => "0",
             "os_name" => "iOS",
               "build" => ""
        }
    }
    (...)
  ```

  Vemos que se ha creado el indice apache con rotación diaría.

  ```bash
    ➜  laboratorio_elasticsearch git:(master) ✗ curl -s -XGET "http://172.18.1.2:9200/_cat/indices"
    green open apache-2015.05.19               ovITiXTbTpuew7aMSvQFmA 1 1  272  0 637.7kb 319.1kb
    green open apache-2015.05.18               oai7N97AQ2uOqlXB9EVWIA 1 1 2893  0   4.7mb   2.1mb
    green open apache-2015.05.17               ctqqPtATSrKSSJ75Q2qxuQ 1 1 1632  0   3.2mb   1.6mb
  ```
  Creamos el index pattern en kibana

  ![index pattern](img/laboratorio_logstash/index_pattern.png)

  Consultamos la pestaña de discovery

  ![discovery](img/laboratorio_logstash/discovery.png)
  </details>
