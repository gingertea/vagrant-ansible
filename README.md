The setup of this project has been borrowed from here: https://kubernetes.io/blog/2019/03/15/kubernetes-setup-using-ansible-and-vagrant/

The only significant change here is the node network CIDR, which I have put in a different range (10.208.x.x). This is done to avoid any overlap with the Calico CNI managed pod network CIDR (192.168.0.0/16). Doing so saves us from problems like unable to open a shell in, or getting a pod's logs from within a node (overlapping CIDRs cause the packets directed to pod to get routed from the node's OS networking).
