A multi-machine Vagrant cluster to work with Spark, including:
- Spark 1.5
- spark-notebook

Includes code from [fredcons/vagrant-spark](https://github.com/fredcons/vagrant-spark) and [wangqiang8511/vagrant-salt-spark](https://github.com/wangqiang8511/vagrant-salt-spark).

## Usage

Configure the `Vagrantfile` to your liking with the number of compute nodes you want. If you want a single node, set `SPARK_NODES` to `0` and use the master node.

### Provisioning with Vagrant

The source code can be used to provision the virtual machine with Vagrant and Ansible, with a simple

```
vagrant up
```

which shall take a bit of time.

### Logging into the master node

Once the VM is launched, you can log in with:

```
vagrant ssh master
# gaining root access
sudo su -
```

Once connected as root, you can use the various libraries installed.

### Spark

```
/opt/spark/bin/spark-shell
```

### spark-notebook

```
cd /opt/spark-notebook/ && ./bin/spark-notebook
```

And then head to http://localhost:9000/
