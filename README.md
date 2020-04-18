# Examples of provisioning server setups with Vagrant/Virtualbox and Ansible.

This repo contains some examples of setting up server environments using Vagrant boxes provisioned with Ansible. Virtualbox has been used as the Vagrant provider. To run , simply clone the repo and cd to the directory containing Vagrantfile and Ansible playbooks for that setup, and run `vagrant up`.

## kubeadm-vagrant - Set up a local kubeadm kubernetes cluster with a master and multiple worker nodes.

This setup has been borrowed from here: https://kubernetes.io/blog/2019/03/15/kubernetes-setup-using-ansible-and-vagrant/

The only significant change here is the node network CIDR or Virtualbox internal network address, which is in a different network (10.208.x.x). This is done to avoid any overlap with the Calico CNI managed pod network CIDR (192.168.0.0/16). Having nodes and pods on different networks ensures that requests sent to pods are not mistakenly handled by the node's OS route table. Some operations like getting a container's shell in pod, or getting pod logs, won't work until the node network is different from pod network.

To run:
```
cd kubeadm-vagrant
vagrant up
```

Once the provisioning is completed, you may log in to master node and check the cluster availability. Following is the output for a master node with two worker nodes.
```
vagrant ssh k8s-master

kubectl get nodes -o wide
NAME         STATUS   ROLES    AGE   VERSION   INTERNAL-IP    EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION      CONTAINER-RUNTIME
k8s-master   Ready    master   24h   v1.18.1   10.208.50.10   <none>        Ubuntu 18.04.4 LTS   4.15.0-96-generic   docker://19.3.8
node-1       Ready    <none>   24h   v1.18.1   10.208.50.11   <none>        Ubuntu 18.04.4 LTS   4.15.0-96-generic   docker://19.3.8
node-2       Ready    <none>   24h   v1.18.1   10.208.50.12   <none>        Ubuntu 18.04.4 LTS   4.15.0-96-generic   docker://19.3.8
```
