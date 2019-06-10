# vagrant_cluster

Work in progress

For K8S install's - kubectl logs not working yet - need to investigate.

install virtualbox and vagrant


And get the following vagrant pluggins..

vagrant plugin install vagrant-disksize

Currently have a problem with vagrant ssh timeing out.  Have a network issue to figure out when the vm has hostonly networking interface as well.

Note that these Vagrant vm are multi-homed in that they have two network interfaces.  Becuase of that, the kubadm comands must specify the local network IP address for cluster communications.

For the defauklt network on this project, the first master is at 192.168.50.2, so the kubeadm option is ...
--apiserver-advertise-address 192.168.50.2

The first worker is at 192.168.50.11, so when running kubeadm join, you need to specify
--apiserver-advertise-address 192.168.50.11


vagrant up


vagrant ssh master-1

sh /vagrant/k8s_setup

vagrant ssh worker-1

sh /vagrant/k8s_setup

Repeat for all workers.



https://metallb.universe.tf/
https://metallb.universe.tf/tutorial/layer2/

kubectl apply -f https://raw.githubusercontent.com/google/metallb/v0.7.3/manifests/metallb.yaml
kubectl apply -f /vagrant/metallb.config

kubectl apply -f https://raw.githubusercontent.com/google/metallb/v0.7.3/manifests/tutorial-2.yaml

kubectl get services
NAME         TYPE           CLUSTER-IP      EXTERNAL-IP      PORT(S)        AGE
:
nginx        LoadBalancer   10.101.161.61   192.168.50.240   80:32564/TCP   32s




on haproxy at .254 enable ip forwarding.
vi /etc/sysctl.conf
net.ipv4.ip_forward=1

sudo sysctl -p

Find and take note of the exernal IP address the haproxy has.

ip address 

On your mac, add a route to the same interface you selected for the haproxy external interface.

sudo route add -net 192.168.50.0/24 ip.address.of.proxy -ifscope interface_you_selected


curl 192.168.50.240

Tear it all down


vagrant destroy

sudo route delete -net 192.168.50.0/24 ip.address.of.proxy -ifscope interface_you_selected
