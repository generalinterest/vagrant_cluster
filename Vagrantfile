IP_NETWORK = "192.168.50."
NUM_MASTERS = 1 #max 9
NUM_WORKERS = 3 #starting from .11 up until you collide with the proxy's.
NUM_PROXYS = 0  #starting from .254 down



$setMasterPost = <<-MasterPostSCRIPT
apt-get install nfs-kernel-server -y
MasterPostSCRIPT

$setWorkerPost = <<-WorkerPostSCRIPT
sudo apt-get install nfs-common -y
WorkerPostSCRIPT


$setNode = <<-SCRIPT
echo hello from node with private ip address #{IP_NETWORK}$1

useradd kadmin -d /home/kadmin -G admin -G sudo -m
echo kadmin:changeme | /usr/sbin/chpasswd
sed -i 's/PasswordAuthentication no/PasswordAuthentication yes/' /etc/ssh/sshd_config
service sshd restart

# Set up an Environment Variable to specify the IP address the kubelet will use to work with the cluster.
echo "export NODEIP=#{IP_NETWORK}$1" >> ~vagrant/.bash_profile

#disable IPv6
cat >> /etc/sysctl.conf << EOFSYSCTL
net.ipv6.conf.all.disable_ipv6 = 1
net.ipv6.conf.default.disable_ipv6 = 1
net.ipv6.conf.lo.disable_ipv6 = 1
EOFSYSCTL
sysctl -p

SCRIPT

Vagrant.configure("2") do |config|

  (1..NUM_MASTERS).each do |i|
    config.vm.define "master-#{i}" do |master|
      master.vm.network "private_network", ip: IP_NETWORK+"#{i+1}"
      master.vm.hostname = "master-#{i}" 
      master.vm.box = "ubuntu/xenial64"
      master.disksize.size = '20GB'
      master.vm.provider "virtualbox" do |v|
        v.name = "master-#{i}"
        v.memory = 2048
        v.cpus = 2
      end
      master.vm.provision "shell" do |s|
        s.inline = $setNode
        s.args   = "#{i+1}"
      end
      master.vm.provision "shell" do |s|
        s.inline = $setMasterPost
        s.args   = "#{i+1}"
      end
      master.vm.provision "shell", inline: "sh /vagrant/k8s_setup"
    end
  end

  (1..NUM_PROXYS).each do |i|
    config.vm.define "proxy-#{i}" do |proxy|
      proxy.vm.network :forwarded_port, guest: 6443, host: 6443
      proxy.vm.network "private_network", ip: IP_NETWORK+"#{255-i}"
      proxy.vm.hostname = "proxy-#{i}"
      proxy.vm.box = "ubuntu/xenial64"
      proxy.vm.provider "virtualbox" do |v|
        v.name = "proxy-#{i}"
        v.memory = 1024
        v.cpus = 2
      end
      proxy.vm.provision "shell", inline: "sh /vagrant/haproxy_setup"
      proxy.vm.provision "shell",
        inline: "echo hello from proxy with private ip address #{IP_NETWORK}#{255-i}"
    end
  end



  (1..NUM_WORKERS).each do |i|
    config.vm.define "worker-#{i}" do |worker|
      worker.vm.network "private_network", ip: IP_NETWORK+"#{i+10}"
      worker.vm.hostname = "worker-#{i}"
      worker.vm.provider "virtualbox" do |v|
        v.name = "worker-#{i}"
        v.memory = 2048
        v.cpus = 2
      end
      worker.vm.box = "ubuntu/xenial64"
      worker.disksize.size = '20GB'
      worker.vm.provision "shell" do |s|
        s.inline = $setNode
        s.args   = "#{i+10}"
      end
      worker.vm.provision "shell" do |s|
        s.inline = $setWorkerPost
        s.args   = "#{i+10}"
      end
      worker.vm.provision "shell", inline: "sh /vagrant/k8s_setup"
    end
  end

end
