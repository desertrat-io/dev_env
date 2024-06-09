# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure('2') do |config|

  config.vm.box = 'bento/ubuntu-22.04'
  
  # mailhog
  config.vm.network 'forwarded_port', guest: 1025, host: 1025
  config.vm.network 'forwarded_port', guest: 8025, host: 8025
  
  # redis
  config.vm.network 'forwarded_port', guest: 6379, host: 6379

  # cassandra
  config.vm.network 'forwarded_port', guest: 7000, host: 7001

  # kafka
  config.vm.network 'forwarded_port', guest: 9094, host: 9094

  # mongo
  config.vm.network 'forwarded_port', guest: 27017, host: 27017

  config.vm.provider 'virtualbox' do |vb|
     vb.memory = '3072'
     vb.cpus = 4
  end

  # easy to run containers
  config.vm.provision :docker do |d|
    d.run 'mailhog/mailhog', args: '-p 1025:1025 -p 8025:8025'
    d.run 'redis', args: '-p 6379:6379'
    d.run 'cassandra', args: '-p 7000:7000'
    d.run 'mongo', args: '-p 27017:27017'
  end

  # but kafka without zookeeper is a pain, and the docs suck and this will start kafka for us
  # however the only examples are in a docker compose file, so instead of translating that to a docker file let's
  # just easymode it because i'm lazy
  config.trigger.after [:up, :reload] do |trigger|
    # remove the pip update when the lib is fixed upstream
    # easiest fix is to just downgrade the ubuntu version from 24.04 to 22.04
    # https://stackoverflow.com/questions/64952238/docker-errors-dockerexception-error-while-fetching-server-api-version
    trigger.run_remote = {inline: <<-BASH
        apt-get update
        apt-get install docker-compose-plugin docker-compose -y
        cd /vagrant
        docker-compose up -d    
      BASH
    }
  end
end
