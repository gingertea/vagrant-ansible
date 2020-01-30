# -*- mode: ruby -*-
# vi: set ft=ruby :

require 'yaml'
require 'ipaddr'

settings = YAML.load_file 'vagrant.yaml'
ip_addr = IPAddr.new(settings['ip_address'])

IMAGE_NAME = "ubuntu/xenial64"
N = 2

Vagrant.configure("2") do |config|
    config.ssh.insert_key = false

    config.vm.provider "virtualbox" do |v|
	v.customize ["modifyvm", :id, "--natdnshostresolver1", "on"]
        v.memory = 1024
        v.cpus = 2
    end
      
    config.vm.define "k8s-master" do |master|
        master.vm.box = IMAGE_NAME
        master.vm.network "private_network", ip:  ip_addr.to_s
        master.vm.hostname = settings['master_name']
        master.vm.provision "ansible" do |ansible|
            ansible.playbook = "kubernetes-setup/master-playbook.yml"
            ansible.extra_vars = {
                node_ip: ip_addr.to_s,
                hostname: settings['master_name'],
            }
        end
    end

    (1..N).each do |i|
        config.vm.define "node-#{i}" do |node|
            ip_addr = ip_addr.succ
            node.vm.box = IMAGE_NAME
            node.vm.network "private_network", ip: ip_addr.to_s
            node.vm.hostname = settings['node_name_prefix']+"-#{i}"
            node.vm.provision "ansible" do |ansible|
                ansible.playbook = "kubernetes-setup/node-playbook.yml"
                ansible.extra_vars = {
                    node_ip: ip_addr.to_s,
                }
            end
        end
    end
    
end    
