IP_NETWORK = "192.168.50."
NUM_MASTERS = 3 #max 9
NUM_WORKERS = 1 #starting from .11 up until you collide with the proxy's.
NUM_PROXYS = 1  #starting from .254 down
#etcd_cluster_string = "initial-cluster: "
etcd_cluster_string = " "
(1..NUM_MASTERS).each do |i|
  etcd_cluster_string="#{etcd_cluster_string}master-#{i}=https://#{IP_NETWORK}#{i+1}:2380,"
end  
etcd_cluster_string="#{etcd_cluster_string}".chop
#puts ("etcd_cluster_hosts:" + etcd_cluster_string )


$setMasterPost = <<-MasterPostSCRIPT
apt-get install nfs-kernel-server -y

mkdir -p  /tmp/#{IP_NETWORK}$1
cat << EOF > /tmp/kubeadmcfg.yaml
apiVersion: "kubeadm.k8s.io/v1beta1"
kind: ClusterConfiguration
etcd:
    local:
        serverCertSANs:
        - #{IP_NETWORK}$1
        peerCertSANs:
        - #{IP_NETWORK}$1
        extraArgs:
            initial-cluster:#{etcd_cluster_string}
            initial-cluster-state: new
            name: "master-$2"
            listen-peer-urls: https://#{IP_NETWORK}$1:2380
            listen-client-urls: https://#{IP_NETWORK}$1:2379
            advertise-client-urls: https://#{IP_NETWORK}$1:2379
            initial-advertise-peer-urls: https://#{IP_NETWORK}$1:2380
EOF

# create etcd certiificates

#if this is the first node only do this and copy this to /vagrant for the other masters to pick it up.

if [ -f /vagrant/.etcd/ca.crt ]
then
  echo "/vagrant/.etcd/ca.crt exists so copy it"
  sudo mkdir -p  /etc/kubernetes/pki/etcd
  sudo cp /vagrant/.etcd/ca.*  /etc/kubernetes/pki/etcd
 else
  echo "/vagrant/.etcd/ca.crt needs to be created"
  sudo kubeadm init phase certs etcd-ca
  sudo mkdir /vagrant/.etcd
  sudo cp  /etc/kubernetes/pki/etcd/ca.*  /vagrant/.etcd
fi




sudo kubeadm init phase certs etcd-server --config=/tmp/kubeadmcfg.yaml
sudo kubeadm init phase certs etcd-peer --config=/tmp/kubeadmcfg.yaml
sudo kubeadm init phase certs etcd-healthcheck-client --config=/tmp/kubeadmcfg.yaml
sudo kubeadm init phase certs apiserver-etcd-client --config=/tmp/kubeadmcfg.yaml
#sudo cp -R /etc/kubernetes/pki /tmp/#{IP_NETWORK}$1/
# cleanup non-reusable certificates
#sudo find /etc/kubernetes/pki -not -name ca.crt -not -name ca.key -type f -delete

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
      master.vm.provision "shell", inline: "sh /vagrant/k8s_setup"
      master.vm.provision "shell" do |s|
        s.inline = $setMasterPost
        s.args   = "#{i+1}", "#{i}"
      end
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
