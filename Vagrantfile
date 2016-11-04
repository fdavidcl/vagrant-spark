# coding: utf-8
# -*- mode: ruby -*-
# vi: set ft=ruby :

# Vagrantfile API/syntax version. Don't touch unless you know what you're doing!
VAGRANTFILE_API_VERSION = "2"

# Configure the number of compute nodes (one extra node will be used as master)
SPARK_NODES             = 2
IP_PREFIX               = "10.211.55."
HOSTNAME_PREFIX         = "spark-cluster-"

# Master public/private keys
# Important: generate your own keys with `ssh-keygen -t rsa`
master_private = File.read("provisioning/master_keys/id_rsa")
master_public = File.read("provisioning/master_keys/id_rsa.pub")

# Calculate the IPs of master and compute nodes
master_ip   = "#{IP_PREFIX}100"
master_host = "#{HOSTNAME_PREFIX}master"
node_ip     = (1..SPARK_NODES).map { |n| IP_PREFIX + (100 + n).to_s }
node_host   = (1..SPARK_NODES).map { |n| "#{HOSTNAME_PREFIX}node#{n}" }

hosts_script = <<SCRIPT
cat > /etc/hosts <<EOF
127.0.0.1       localhost

# The following lines are desirable for IPv6 capable hosts
::1     ip6-localhost ip6-loopback
fe00::0 ip6-localnet
ff00::0 ip6-mcastprefix
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters

#{master_ip} #{master_host}
#{node_ip.zip(node_host).map{|e| e.join " "}.join "\n"}
EOF
SCRIPT

master_script = <<SCRIPT
#{hosts_script}

cat > ~/.profile <<EOF
export SPARK_HOME=/opt/spark
export PATH=\$SPARK_HOME/bin:/opt/apache-maven-3.3.9/bin/:$PATH
export SPARK_SLAVES=\$HOME/spark_slaves
EOF

cat > ~/spark_slaves <<EOF
#{node_host.join "\n"}
EOF

cat > ~/.bashrc <<EOF
spark_start() {
  \$SPARK_HOME/sbin/start-all.sh
}
spark_stop() {
  \$SPARK_HOME/sbin/stop-all.sh
}
EOF

mkdir -p ~/.ssh
echo "#{master_private}" > ~/.ssh/id_rsa
chmod 0600 ~/.ssh/id_rsa
echo "#{master_public}" > ~/.ssh/id_rsa.pub
SCRIPT

slave_script = <<SCRIPT
#{hosts_script}

mkdir -p ~/.ssh
echo "#{master_public}" > ~/.ssh/authorized_keys
SCRIPT

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  config.vm.box = "precise64"

  config.vm.provider "virtualbox" do |vb|
    vb.customize ["modifyvm", :id, "--cpus", "2"]
    vb.customize ["modifyvm", :id, "--memory", "1024"]
  end

  # If true, then any SSH connections made will enable agent forwarding.
  # Default value: false
  config.ssh.forward_agent = true

  config.vm.provision "fix-no-tty", type: "shell" do |s|
    s.privileged = false
    s.inline = "sudo sed -i '/tty/!s/mesg n/tty -s \\&\\& mesg n/' /root/.profile"
  end
  
  config.vm.provision "ansible" do |ansible|
    ansible.sudo = true 
    ansible.playbook = "provisioning/playbook.yml"
  end

  # Multi-machine configuration:
  config.vm.define :master do |master|
    master.vm.network :private_network, ip: master_ip
    master.vm.hostname = master_host
    
    master.vm.provision :shell, :inline => master_script
    
    # spark
    master.vm.network "forwarded_port", guest: 8080, host: 8080
    master.vm.network "forwarded_port", guest: 8081, host: 8081
    # spark notebook
    master.vm.network "forwarded_port", guest: 9000, host: 9000  
  end

  SPARK_NODES.times do |n|
    config.vm.define "node#{n}" do |node|
      node.vm.network :private_network, ip: node_ip[n]
      node.vm.hostname = node_host[n]
      
      node.vm.provision :shell, :inline => slave_script
    end
  end
end
