# docker-registry-vagrant

Quick way to deploy a private docker repo using Vagrant/Virtualbox and Ansible.

Tested with Vagrant 2.2.6, Virtualbox 6.1 and Ansible 2.9.2.

This example will create a Virtualbox VM with IP set to 10.208.50.105.
The VM will run a nginx server with self-signed certificate, proxying traffic to docker registry container running at TCP/5000. The VM will also have an additional VDI hard disk of 16 GB attached, for docker registry storage.

Simply run:
vagrant up

Upon successful completion, a self-signed CA certificate file (named 10_208_20_105_CA.crt by default) will be copied to the local system inside docker. This file needs to be copied to all docker client hosts inside the location /etc/docker/certs.d/10.208.50.105.xip.io:5000/, for example.

See https://docs.docker.com/registry/insecure/#use-self-signed-certificates for details.

Once done with this procedure, test that docker can successfully connect to your newly launched registry with this command:

docker login 10.208.50.105.xip.io -u crab -p apple

Thanks to xip.io for hosting free DNS service for private IPv4 addresses.
