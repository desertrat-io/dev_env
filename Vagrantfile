# -*- mode: ruby -*-
# vi: set ft=ruby :

PRIVATE_NETWORK_IP = '192.168.50.4'.freeze
MAILHOG_OUT_PORT = 1025.freeze
MAILHOG_UI_PORT = 8025.freeze
REDIS_PORT = 6379.freeze
# something local is messing with this
CASSANDRA_EXT_PORT = 7001.freeze
CASSANDRA_INT_PORT = 7000.freeze
KAFKA_BROKER_PORT = 9094.freeze
KAFKA_BROKER_INTERNAL_PORT = 9092.freeze
KAFKA_SCHEMA_REGISTRY_PORT_1 = 8081.freeze
KAFKA_SCHEMA_REGISTRY_PORT_2 = 8082.freeze
MONGO_DB_PORT = 27017.freeze
EVENTSTORE_DB_PORT = 2113.freeze
KAFKA_UI_INT_PORT = 8080.freeze
KAFKA_UI_EXT_PORT = 9080.freeze

Vagrant.configure('2') do |config|

  config.vm.box = 'bento/ubuntu-22.04'

  config.vm.network :private_network, ip: PRIVATE_NETWORK_IP

  # mailhog
  config.vm.network 'forwarded_port', guest: MAILHOG_OUT_PORT, host: MAILHOG_OUT_PORT
  config.vm.network 'forwarded_port', guest: MAILHOG_UI_PORT, host: MAILHOG_UI_PORT
  
  # redis
  config.vm.network 'forwarded_port', guest: REDIS_PORT, host: REDIS_PORT

  # cassandra
  config.vm.network 'forwarded_port', guest: CASSANDRA_INT_PORT, host: CASSANDRA_EXT_PORT

  # kafka
  config.vm.network 'forwarded_port', guest: KAFKA_BROKER_PORT, host: KAFKA_BROKER_PORT
  config.vm.network 'forwarded_port', guest: KAFKA_BROKER_INTERNAL_PORT, host: KAFKA_BROKER_INTERNAL_PORT

  # kafka schema registry
  config.vm.network 'forwarded_port', guest: KAFKA_SCHEMA_REGISTRY_PORT_1, host: KAFKA_SCHEMA_REGISTRY_PORT_1
  config.vm.network 'forwarded_port', guest: KAFKA_SCHEMA_REGISTRY_PORT_2, host: KAFKA_SCHEMA_REGISTRY_PORT_2

  # mongo
  config.vm.network 'forwarded_port', guest: MONGO_DB_PORT, host: MONGO_DB_PORT

  # EventStoreDB
  config.vm.network 'forwarded_port', guest: EVENTSTORE_DB_PORT, host: EVENTSTORE_DB_PORT

  # kafka UI
  config.vm.network 'forwarded_port', guest: KAFKA_UI_INT_PORT, host: KAFKA_UI_EXT_PORT

  config.vm.provider 'virtualbox' do |vb|
     vb.memory = '3072'
     vb.cpus = 4
  end

  # easy to run containers
  config.vm.provision :docker do |d|
    d.run 'mailhog/mailhog', args: "-p #{MAILHOG_OUT_PORT}:#{MAILHOG_OUT_PORT} -p #{MAILHOG_UI_PORT}:#{MAILHOG_UI_PORT}"
    d.run 'redis', args: "-p #{REDIS_PORT}:#{REDIS_PORT}"
    d.run 'cassandra', args: "-p #{CASSANDRA_INT_PORT}:#{CASSANDRA_INT_PORT}"
    d.run 'mongo', args: "-p #{MONGO_DB_PORT}:#{MONGO_DB_PORT}"
    d.run 'bitnami/schema-registry', args: "-p #{KAFKA_SCHEMA_REGISTRY_PORT_1}:#{KAFKA_SCHEMA_REGISTRY_PORT_1} -e SCHEMA_REGISTRY_KAFKA_BROKERS=PLAINTEXT://#{PRIVATE_NETWORK_IP}:#{KAFKA_BROKER_INTERNAL_PORT} --env-file /vagrant/schema-registry/vars"
  end

  config.vm.provision :shell, inline: <<-BASH
      apt-get update
      apt-get install docker-compose-plugin docker-compose -y 
    BASH

  config.trigger.before [:halt, :reload] do |trigger|
    trigger.run_remote = {inline: <<-BASH
      cd /vagrant
      docker compose down
    BASH
  }
  end
  # kafka without zookeeper is a pain, and the docs suck and this will start kafka for us
  # however the only examples are in a docker compose file, so instead of translating that to a docker file let's
  # just easymode it because i'm lazy
  config.trigger.after [:up, :reload] do |trigger|
   trigger.run_remote = {inline: <<-BASH 
       cd /vagrant
       docker compose up -d
       docker network connect vagrant_kafka bitnami-schema-registry
     BASH
   }
  end
end
