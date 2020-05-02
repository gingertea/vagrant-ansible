# Examples of provisioning server setups with Vagrant/Virtualbox and Ansible.

This repo contains some examples of setting up server environments using Vagrant boxes provisioned with Ansible. Virtualbox has been used as the Vagrant provider. To run , simply clone the repo and cd to the directory containing Vagrantfile and Ansible playbooks for that setup, and run `vagrant up`.

## kubeadm-vagrant - Set up a local kubeadm kubernetes cluster with a master and multiple worker nodes.

This setup has been borrowed from here: https://kubernetes.io/blog/2019/03/15/kubernetes-setup-using-ansible-and-vagrant/. Calico has been used as the CNI plugin.

The only significant change here is the node network CIDR or Virtualbox internal network address, which is in a different network (10.208.x.x). This is done to avoid any overlap with the Calico CNI managed pod network CIDR (192.168.0.0/16). Having nodes and pods on different networks ensures that requests sent to pods are not mistakenly handled by the node's OS route table. Some operations like getting a container's shell in pod, or getting pod logs, won't work until the node network is different from pod network.

Settings like node network addresses, number of nodes, calico version etc can be configured from the vagrant.yaml file.

To run:
```
cd kubeadm-vagrant
vagrant up
```

Once the provisioning is completed, you may log in to master node and check the cluster availability. Following is an example output for a master node with two worker nodes.
```
vagrant ssh k8s-master

kubectl get nodes -o wide
NAME         STATUS   ROLES    AGE   VERSION   INTERNAL-IP    EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION      CONTAINER-RUNTIME
k8s-master   Ready    master   24h   v1.18.1   10.208.50.10   <none>        Ubuntu 18.04.4 LTS   4.15.0-96-generic   docker://19.3.8
node-1       Ready    <none>   24h   v1.18.1   10.208.50.11   <none>        Ubuntu 18.04.4 LTS   4.15.0-96-generic   docker://19.3.8
node-2       Ready    <none>   24h   v1.18.1   10.208.50.12   <none>        Ubuntu 18.04.4 LTS   4.15.0-96-generic   docker://19.3.8
```

## docker-registry-vagrant - Deploy a private docker registry.

This example will create a private docker registry on a Virtualbox VM on your host. The VM will run a nginx server on port 8443, proxy-ing traffic to docker registry container running at TCP/5000 The server uses a certifcate issued by a self-signed CA, which we create during the provisioning process.

 The VM will have an additional VDI hard disk of 16 GB attached to it, for docker storage volume. Parameters like server IP, docker registry authentication credentials and virtual hard disk size need to be configured from the vagrant.yaml file before run.

Once you have changed the defaults in the vagrant.yaml inside the docker-registry-vagrant folder as per your requirements, type the following in a shell to run:
```
cd docker-registry-vagrant
vagrant up
```
To test if the server is up, check if you can access the server URL (https:// _server_ip_:8443.

You should see a CA certificate file named _server\_ip_ CA.crt inside the docker-private-registry-setup directory after the provisioning is complete. This certificate needs to be copied to all your docker clients in a directory named _server_ip_:8443 inside the docker config directory.

For instance, to access the registry from your host machine with external IP 192.168.1.10, do the following:
```
sudo mkdir /etc/docker/certs.d/192.168.1.10:8443
cd docker-private-registry-setup/192.168.1.10_CA.crt \
/etc/docker/certs.d/192.168.1.10:8443

```
See https://docs.docker.com/registry/insecure/#use-self-signed-certificates for details.

Once done with this procedure, test that docker can successfully connect to your newly launched registry with this command:
```
docker login 192.168.1.10:8443 -u crab -p apple
```

If the command succeeds you should be able to pull/push to the registry, like this:
```
docker pull nginx
docker tag nginx 192.168.1.10:8443/nginx
docker push 192.168.1.10:8443/nginx
docker pull 192.168.1.10:8443/nginx
```

## openldap-vagrant - Set up a local openldap server with some example users and groups.

This example will create a openldap server listening on your host's external IP on port 1389. The server will accept simple binds over TLS and non-TLS connections. The server IP needs to be changed to the external IP of your host from the [vagrant.yaml](openldap-vagrant/vagrant.yaml) file, to expose the server to external networks. Additionally, you host firewall should allow connections to TCP port 1389.

Once you have changed the server IP and other defaults as per your requirements, type the following in a shell to run:
```
cd openldap-vagrant
vagrant up
```
After the provisioning is complete, you should find a  ldap_ca_cert.pem file inside the openldap-setup directory. This certificate needs to be copied/imported to ldap clients to avoid TLS trust related errors. See the 'Known Issues' section below though, to enable TLS first.

Assuming that you have run this example with the defaults with the server IP set to 192.168.1.10, run this command to test that the server is running and accepting connections:
```
ldapsearch -x -b "dc=directory,dc=example,dc=com" -H ldap://192.168.1.11:1389 -D "cn=admin,dc=directory,dc=example,dc=com" -w secret 
```

Known issues:
 - The olc attributes required to enable TLS connections to the server need to be added manually after the provisioning is completed. To do so, ssh to the box, and run the `ldapmodify` command with the ldif file containing the certificate and key file paths as the command parameter, explained as below:
 ```
 vagrant ssh
 sudo -i
 ldapmodify -Y EXTERNAL -H ldapi:/// -f cert_info.ldif
 ```



.
