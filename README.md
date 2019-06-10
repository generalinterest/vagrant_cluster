# vagrant_cluster

Work in progress

For K8S install's - kubectl logs not working yet - need to investigate.

install virtualbox and vagrant


And get the following vagrant pluggins..

vagrant plugin install vagrant-disksize


Note that these Vagrant vm are multi-homed in that they have two network interfaces.  Becuase of that, the kubeadm comands must specify the local network IP address for cluster communications.

vagrant up


vagrant ssh master-1

sh /vagrant/k8s_setup

vagrant ssh worker-1

sh /vagrant/k8s_setup

Repeat for all workers.



To have a load-balancer option for pods on this bare-matal installation, see ...
https://metallb.universe.tf/

We will use the layer2 method for simplicity where all you need is an IP pool to allocate for services that have the kind load-balancer...
https://metallb.universe.tf/tutorial/layer2/

kubectl apply -f https://raw.githubusercontent.com/google/metallb/v0.7.3/manifests/metallb.yaml

This is a sample config taking a block of IP's from the clusters network...

kubectl apply -f /vagrant/metallb.config

Run the example nginx app...
kubectl apply -f https://raw.githubusercontent.com/google/metallb/v0.7.3/manifests/tutorial-2.yaml

kubectl get services
NAME         TYPE           CLUSTER-IP      EXTERNAL-IP      PORT(S)        AGE
:
nginx        LoadBalancer   10.101.161.61   192.168.50.240   80:32564/TCP   32s

On your mac ...

curl 192.168.50.240

Add notes here for HA master configuration with haproxy

Tear it all down


vagrant destroy

sudo route delete -net 192.168.50.0/24 ip.address.of.proxy -ifscope interface_you_selected
