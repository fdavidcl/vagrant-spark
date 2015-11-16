A simple VM to work with Spark, including : 
- Spark 1.5
- Zeppelin
- spark-notebook
- datastax community edition (with datastax agent and opscenter)

## Provisioning with Vagrant 

The source code can be used to provision the virtual machine with Vagrant and Ansible, with a simple :

```
vagrant up
```

which shall take a bit of time

## Using a pre-build VM

The resulting VM is available on Hashicorp's [Atlas platform](https://atlas.hashicorp.com/fredcons/boxes/spark-handson) (formerly Vagrant Cloud).
Here are a few commands for a faster start up time (with a bigger download upfront) :

```
mkdir spark-handson && cd spark-handson
cat <<EOT >> Vagrantfile
VAGRANTFILE_API_VERSION = "2"
Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  config.vm.box = "fredcons/spark-handson"
  config.vm.network "private_network", ip: "192.168.33.10"
  config.vm.network "forwarded_port", guest: 8080, host: 8080
  config.vm.network "forwarded_port", guest: 8081, host: 8081
  config.vm.network "forwarded_port", guest: 9000, host: 9000
  config.vm.network "forwarded_port", guest: 4040, host: 4040
  config.vm.network "forwarded_port", guest: 8888, host: 8888
  config.vm.network "forwarded_port", guest: 8989, host: 8989
  config.vm.network "forwarded_port", guest: 2181, host: 2181
  config.vm.network "forwarded_port", guest: 9092, host: 9092
  # port conflict with spark
  #config.vm.network "forwarded_port", guest: 8081, host: 8081
  config.vm.provider "virtualbox" do |vb|
    vb.customize ["modifyvm", :id, "--cpus", "2"]
    vb.customize ["modifyvm", :id, "--memory", "2048"]
  end
end
EOT
vagrant up
vagrant ssh
```

You may want to adjust the CPU / memory parameters.

## Usage

Once the VM is launched, you can log in with :

```
vagrant ssh
# gaining root access
sudo su -
```

Once connected as root, you can use the various libraries installed :

### Spark

```
/opt/spark/bin/spark-shell
```

### Zeppelin

```
/opt/zeppelin/bin/zeppelin.sh start
```

And then head to http://localhost:8080/

### spark-notebook

```
cd /opt/spark-notebook/ && ./bin/spark-notebook
```

And then head to http://localhost:9000/

### cassandra

Cassandra services are started by default.
The datastax opscenter can be reached at `http://localhost:8888/opscenter/`

### confluent.io

```
sudo /usr/bin/zookeeper-server-start /etc/kafka/zookeeper.properties
sudo /usr/bin/kafka-server-start /etc/kafka/server.properties
sudo /usr/bin/schema-registry-start /etc/schema-registry/schema-registry.properties
sudo /usr/bin/kafka-rest-start /etc/kafka-rest/kafka-rest.properties
```

### spark / kafka integration

```
sudo /usr/bin/kafka-topics --create --zookeeper localhost:2181 --replication-factor 1 --partition 1 --topic spark-streaming-test
# vanilla kafka version
sudo /usr/bin/kafka-create-topic.sh --zookeeper localhost:2181 --replica 1 --partition 1 --topic spark-streaming-test
```
