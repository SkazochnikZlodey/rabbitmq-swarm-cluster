======================
rabbitmq-swarm-cluster
======================

This docker image is built to get a lightweight rabbitmq cluster without any
external dependencies. It is designed to work in docker swarm mode.


Basic Concept
=============

Once the rabbitmq-server is up and running the join.sh script will try to join
this server instance to a reachable peer. Available peers are determined via
nslookup to `tasks.<SERVICE_NAME>`. If no peer is available but this container
is matches `SLOT == MASTER_SLOT` the rabbitmq app will be just started and no
further joining arithmetic will be executed on this container.


docker swarm yml file example:

```


version: "3.9"

networks:
  rmqnetwork:
    attachable: true

services:

  rabbitmq:
    image: skazochnikzlodey/rabbitmq-cluster-docker-swarm:latest
    hostname: "{{.Service.Name}}.{{.Task.Slot}}.{{.Task.ID}}"
    deploy:
      replicas: 2
      # placement:
      #   constraints: [node.role == worker]
#        max_replicas_per_node: 1
      restart_policy:
        condition: any
        delay: 5s
      update_config:
        parallelism: 1
        delay: 10s
        failure_action: rollback
        monitor: 1s
#        max_failure_ratio: 1
        order: start-first

    environment:
      - RABBITMQ_ERLANG_COOKIE=abc
#      - RABBITMQ_SERVER_ADDITIONAL_ERL_ARGS="-setcookie abc"
      - RABBITMQ_USE_LONGNAME=true
      - RABBITMQ_MNESIA_DIR=/var/lib/rabbitmq/mnesia
#      - RABBITMQ_PLUGINS_EXPAND_DIR=/var/lib/rabbitmq/mnesia/plugins-expand
      - RABBITMQ_PLUGINS_EXPAND_DIR=/var/lib/rabbitmq/plugins-expand
      - RABBITMQ_HIPE_COMPILE=1
      - RABBITMQ_DEFAULT_USER=admin
      - RABBITMQ_DEFAULT_PASS=P@ssw0rd
      - SERVICE_NAME={{.Service.Name}}
      - SLOT={{.Task.Slot}}
      - MASTER_SLOT=1
    ports:
      - "5672:5672"   # amqp
      - "15672:15672" # web ui
    # healthcheck:
    #   test: [ "CMD", "nc", "-z", "localhost", "5672" ]
    #   interval: 1m30s
    #   timeout: 15s
    #   retries: 3
    #   start_period: 45s

    networks:
      rmqnetwork:
#        ipv4_address: network_prefix_tpl.10
        aliases:
          - rabbitmq



```
