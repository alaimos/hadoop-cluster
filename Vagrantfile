#############################################
## INIZIO SCRIPT DI INSTALLAZIONE SOFTWARE ##
#############################################
$master_script = <<SCRIPT
#!/bin/bash
apt-get -q -y install curl
wget 'http://archive.cloudera.com/cm5/ubuntu/xenial/amd64/cm/cloudera.list' -O /etc/apt/sources.list.d/cloudera-cm5.list
wget 'http://archive.cloudera.com/cm5/ubuntu/xenial/amd64/cm/archive.key' -O archive.key
apt-key add archive.key
rm archive.key
apt-get update
export DEBIAN_FRONTEND=noninteractive
export JAVA_HOME="/usr/lib/jvm/java-8-oracle"
apt-get -q -y install cloudera-manager-server-db cloudera-manager-server cloudera-manager-daemons
service cloudera-scm-server-db initdb
service cloudera-scm-server-db start
service cloudera-scm-server start
SCRIPT

$hosts_script = <<SCRIPT
#!/bin/bash
sed -i 's/PasswordAuthentication no/PasswordAuthentication yes/g' /etc/ssh/sshd_config
service sshd reload
cat >> /etc/sysctl.conf <<EOF
vm.swappiness = 10
EOF
sysctl -p
sed -i 's/archive.ubuntu.com/it.archive.ubuntu.com/' /etc/apt/sources.list
apt-get update
export DEBIAN_FRONTEND=noninteractive
apt-get dist-upgrade -y 
apt-get -y install virtualbox-guest-dkms software-properties-common python-software-properties
service ntp stop ; ntpdate -s time.nist.gov ; service ntp start
cat > /etc/hosts <<EOF
127.0.0.1       localhost

# The following lines are desirable for IPv6 capable hosts
::1     ip6-localhost ip6-loopback
fe00::0 ip6-localnet
ff00::0 ip6-mcastprefix
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters

EOF
add-apt-repository -y ppa:webupd8team/java
echo debconf shared/accepted-oracle-license-v1-1 select true | debconf-set-selections
echo debconf shared/accepted-oracle-license-v1-1 seen true | debconf-set-selections
apt-get update
apt-get -q -y install oracle-java8-installer oracle-java8-set-default #oracle-j2sdk1.7 
echo >> /etc/environment <<EOF
    JAVA_HOME="/usr/lib/jvm/java-8-oracle"
EOF
export JAVA_HOME="/usr/lib/jvm/java-8-oracle"
echo >> /etc/profile <<EOF
	export HADOOP_CLASSPATH=$(hadoop classpath)
EOF
SCRIPT
#############################################
## FINE   SCRIPT DI INSTALLAZIONE SOFTWARE ##
#############################################

Vagrant.configure("2") do |config|

  # Install required plugins
  required_plugins = %w( vagrant-hostmanager vagrant-disksize )
  _retry = false
  required_plugins.each do |plugin|
    unless Vagrant.has_plugin? plugin
      system "vagrant plugin install #{plugin}"
      _retry=true
    end
  end

  if (_retry)
    exec "vagrant " + ARGV.join(' ')
  end

  # Define base image
  config.vm.box = "ubuntu/xenial64"
  config.disksize.size = "25GB"

  # Manage /etc/hosts on host and VMs
  config.hostmanager.enabled = false
  config.hostmanager.manage_host = true
  config.hostmanager.include_offline = true
  config.hostmanager.ignore_private_ip = false
  
  #############################################
  ## INIZIO VM MASTER                        ##
  #############################################
  config.vm.define :master do |master|
    master.vm.provider :virtualbox do |v|
      v.name = "vm-cluster-node1"
	  v.cpus = 4
	  v.memory = 4096
    end
    master.vm.network :private_network, ip: "10.211.55.100"
    master.vm.hostname = "vm-cluster-node1"
    master.vm.provision :shell, :inline => $hosts_script
    master.vm.provision :hostmanager
    master.vm.provision :shell, :inline => $master_script
  end
  #############################################
  ## FINE   VM MASTER                        ##
  #############################################
  
  #############################################
  ## INIZIO VM SLAVE1                        ##
  #############################################
  config.vm.define :slave1 do |slave1|
    slave1.vm.provider :virtualbox do |v|
      v.name = "vm-cluster-node2"
	  v.cpus = 2
	  v.memory = 2048
    end
    slave1.vm.network :private_network, ip: "10.211.55.101"
    slave1.vm.hostname = "vm-cluster-node2"
    slave1.vm.provision :shell, :inline => $hosts_script
    slave1.vm.provision :hostmanager
  end
  #############################################
  ## FINE   VM SLAVE1                        ##
  #############################################
  
  #############################################
  ## INIZIO VM SLAVE2                        ##
  #############################################
  config.vm.define :slave2 do |slave2|
    slave2.vm.provider :virtualbox do |v|
      v.name = "vm-cluster-node3"
	  v.cpus = 2
	  v.memory = 2048
    end
    slave2.vm.network :private_network, ip: "10.211.55.102"
    slave2.vm.hostname = "vm-cluster-node3"
    slave2.vm.provision :shell, :inline => $hosts_script
    slave2.vm.provision :hostmanager
  end
  #############################################
  ## FINE   VM SLAVE2                        ##
  #############################################
  
  #############################################
  ## INIZIO VM SLAVE3                        ##
  #############################################
  # config.vm.define :slave3 do |slave3|
  #   slave3.vm.provider :virtualbox do |v|
  #     v.name = "vm-cluster-node4"
  # 	  v.cpus = 2
  # 	  v.memory = 2048
  # end
  #  slave3.vm.network :private_network, ip: "10.211.55.103"
  #  slave3.vm.hostname = "vm-cluster-node4"
  #  slave3.vm.provision :shell, :inline => $hosts_script
  #  slave3.vm.provision :hostmanager
  #end
  #############################################
  ## FINE   VM SLAVE3                        ##
  #############################################
  
end
