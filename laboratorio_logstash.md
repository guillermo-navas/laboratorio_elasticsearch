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

##### Creando la primera pipelines. Parte 2

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
