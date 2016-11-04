A multi-machine Vagrant cluster to work with Spark, including:
- Spark 1.5
- spark-notebook

Includes code from [fredcons/vagrant-spark](https://github.com/fredcons/vagrant-spark), [wangqiang8511/vagrant-salt-spark](https://github.com/wangqiang8511/vagrant-salt-spark) and [H4ml3t/spark-scala-maven-boilerplate-project](https://github.com/H4ml3t/spark-scala-maven-boilerplate-project).

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

```sh
vagrant ssh master
# gaining root access (required to launch the cluster)
sudo su -
```

Once connected as root, you can use the various libraries installed or execute an example program.

### Example

In this example we will provide a text file to Spark in order to count the ocurrences of each word. First, start up the cluster with

```sh
spark_start
```

Then, compile the project with Maven and submit it to the cluster:

```sh
cd /vagrant/examples/count_words
mvn clean package # This can take a while
spark-submit --class com.examples.MainExample \
  --master spark://spark-cluster-master:7077 \
  /vagrant/examples/count_words/target/spark-scala-maven-project-0.0.1-SNAPSHOT-jar-with-dependencies.jar \
  /vagrant/examples/quijote.txt \
  /vagrant/examples/quijote_out
```

When the task is completed, you will be able to see the results inside the `examples/quijote_out` directory.

### Spark

```
/opt/spark/bin/spark-shell
```

### spark-notebook

```
cd /opt/spark-notebook/ && ./bin/spark-notebook
```

And then head to http://localhost:9000/

## Other resources

For a real Data Analytics-oriented cluster, maybe check out [vagrant-cluster](https://github.com/irifed/vagrant-cluster)
