# Examples of provisioning Vagrant Virtualbox boxes with Ansible

## kubeadm-vagrant - Set up a kubeadm kubernetes cluster with Virtualbox VMs.

This setup has been borrowed from here: https://kubernetes.io/blog/2019/03/15/kubernetes-setup-using-ansible-and-vagrant/

The only significant change here is the node network CIDR, which is in a different range (10.208.x.x). This is done to avoid any overlap with the Calico CNI managed pod network CIDR (192.168.0.0/16). Doing so saves us from problems like unable to open a shell in pod container, or getting a pod's logs from within a node. Having nodes and pods on different network addresses ensures that request sent to pods are not handled by the node's OS route table.
