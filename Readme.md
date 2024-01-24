
# Table of Contents

1.  [hosted logs](#org1ba29e0)


<a id="org1ba29e0"></a>

# hosted logs

nos dirigimos a Home -> Connections -> Add new connection -> Hosted logs

![img](asset/1.png)

y creamos un token para el nuevos dispositivo nos dara una
configuracion agregando el token recien creado ejemplo:

    
    server:
      http_listen_port: 0
      grpc_listen_port: 0
    
    positions:
      filename: /tmp/positions.yaml
    
    client:
      url: https://354058:glc_eyJvIjoiNzY4NDE4IiwibiI6InN0YWNrLTUwNDY5Ni1pbnRlZ3JhdGlvbi1hcnR1cml0byIsImsiOiIzY2ZNWjVoOHY0MjF2M3NaNTZxbjZQdWwiLCJtIjp7InIiOiJ1cyJ9fQ==@logs-prod-017.grafana.net/api/prom/push
    
    scrape_configs:
    - job_name: system
      static_configs:
      - targets:
          - localhost
        labels:
          job: varlogs
          __path__: /var/log/*.log

para agregar una nueva fuente de logs debe quedar asi

    server:
      http_listen_port: 0
      grpc_listen_port: 0
    
    positions:
      filename: /tmp/positions.yaml
    
    client:
      url: https://354058:glc_eyJvIjoiNzY4NDE4IiwibiI6InN0YWNrLTUwNDY5Ni1pbnRlZ3JhdGlvbi1hcnR1cml0byIsImsiOiIzY2ZNWjVoOHY0MjF2M3NaNTZxbjZQdWwiLCJtIjp7InIiOiJ1cyJ9fQ==@logs-prod-017.grafana.net/api/prom/push
    
    scrape_configs:
    - job_name: ar-locker
      static_configs:
      - targets:
          - localhost
        labels:
          job: ar-locker
          __path__: /var/log/*.log
    - job_name: ar-mosquito
      static_configs:
      - targets:
          - localhost
        labels:
          job: ar-mosquito
          __path__: /var/log/mosquito 

esta configuracion debe ir dentro de un directorio llamado promtail

debe quedar algo asi:

    /dir/promtail/config.yaml

siendo dir el directorio donde ejecutaremos el compose o comando de docker

ejemplo de comando docker para ejecutar promtail con esta configuracion

    
    docker run --name promtail --volume "$PWD/promtail:/etc/promtail" --volume "/var/log:/var/log" grafana/promtail:main -config.file=/etc/promtail/config.yaml

donde  &#x2013;volume "/var/log:/var/log"   el primer var log es la ruta desde dondo provienen nuestros logs

tambien es posible ejecutarlo desde docker compose en este caso
seria as:

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

