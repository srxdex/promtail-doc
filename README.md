# Table of Contents

1.  [Hosted logs](#org57975c8)
    1.  [Config.yaml](#orgef17b23)
    2.  [Árbol de directorio](#orgae22555)
    3.  [Docker compose](#org5151cf2)
2.  [crear dashboard](#orgcc88694)

<a id="org57975c8"></a>

# Hosted logs

Ir a Home -> Connections -> Add new connection -> Hosted logs

![img](asset/1.png)

Dar un nombre y crear el token para el nuevo dispositivo, esto genera una configuracion de ejemplo como la siguiente:

<a id="orgef17b23"></a>

## config.yaml

> [!WARNING]  
> Cada [dispositivo] tiene su propio TOKEN se debe sobreescribir la configuración de este repo.
> Es recomendable copiar desde la página de graphana ya que asi se copia el token del dispositivo junto a los demás datos
> usar esta config solo como base.

```
    server:
      http_listen_port: 0
      grpc_listen_port: 0

    positions:
      filename: /tmp/positions.yaml

    client:
      url: https://354058:<TOKEN>@logs-prod-017.grafana.net/api/prom/push

    scrape_configs:
    - job_name: system   #es recomendable poner nombre asociado al dispositivo
      static_configs:
      - targets:
          - localhost
        labels:
          job: varlogs   #poner mismo nombre que job_name para reconocer en graphana
          __path__: /var/log/*.log
```

Agregando una nueva fuente de logs:

```
    server:
      http_listen_port: 0
      grpc_listen_port: 0

    positions:
      filename: /tmp/positions.yaml

    client:
      url: https://354058:<TOKEN>@logs-prod-017.grafana.net/api/prom/push

    scrape_configs:
    - job_name: demo-123-elocker #nombre de job (identificador en graphana)
      static_configs:
      - targets:
    	  - localhost
      labels:
    	  job: demo-123-elocker
    	  __path__: /var/log/*.log
    
    #Nueva fuente de logs:
    - job_name: demo-123-mosquitto
      static_configs:
      - targets:
    	  - localhost
    labels:
      job: demo-123-mosquitto
      __path__: /var/log/mosquitto/*.log
```


<a id="orgae22555"></a>


## Árbol de directorio

Esta configuración debe ir dentro de un directorio llamado promtail.
Debe quedar algo asi:

    ├── docker-compose.yaml
    ├── promtail
    │       └── config.yaml

    /dir/promtail/config.yaml

Siendo dir el directorio donde ejecutaremos el compose o comando de docker


<a id="org5151cf2"></a>

## Docker compose

Configuración de docker-compose.yaml:

    version: "3"

    networks:
      loki:

    services:
      promtail:
        image: grafana/promtail:2.9.0
        volumes:
          - /home/pi/prod/logs:/var/log            #la primera parte es desde donde vienen nuestros logs
          - /var/log/mosquitto/:/var/log/mosquitto #var log mosquitto fue definido en config.yaml
          - ./promtail/config.yaml:/etc/promtail/config.yml

        command: -config.file=/etc/promtail/config.yml
        networks:
          - loki

<a id="orgcc88694"></a>

# crear dashboard

Para crear un nuevo dasboard nos dirigimos a

[new dashboard](https://innovacion1.grafana.net/dashboard/new)

Veremos esto:

![img](asset/2.png)

Presionamos en "add visualization" y se mostrará los siguiente

![img](asset/3.png)

Seleccionamos el recolector de logs por defecto en este caso grafanacloud-innovacion1-logs y despúes seleccionamos
dashboard, deberiamos ver algo asi:

![img](asset/4.png)

Donde dice time series seleccionamos que tipo de filtro requerimos en este caso
logs

![img](asset/5.png)

Una vez tenemos seleccionado la fuente de logs
podemos seleccionar una query desde un archivo como el este ejemplo
que apunta al log de mosquitto

![img](asset/6.png)

Para buscar los logs de un dispositivo los seleccionaremos por job

![img](asset/7.png)

Antes de salir dar un nombre al dashboard que identfique la máquina que está monitoreando, por ejemplo "demo-123-farmacia"
Finalmente pinchar en "Apply" y luego en "Save"