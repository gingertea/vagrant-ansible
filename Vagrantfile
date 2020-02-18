# -*- mode: ruby -*-
# vi: set ft=ruby :

require 'yaml'
require 'ipaddr'

settings = YAML.load_file 'vagrant.yaml'
ip_addr = IPAddr.new(settings['ip_address'])

IMAGE_NAME = "ubuntu/xenial64"
N = 2

host_system_properties = IO.popen(["VBoxManage", "list", "systemproperties"])
virtualbox_machine_folder = host_system_properties.select {|s| s.include?('Default machine folder')}[0].partition(':').last.strip

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
        master.vm.provider "virtualbox" do |vb|
          vb.name =  master.vm.hostname
        end
        master.vm.provision "ansible" do |ansible|
            ansible.playbook = "kubernetes-setup/master-playbook.yml"
            ansible.extra_vars = {
                node_ip: ip_addr.to_s,
                hostname: settings['master_name'],
                calico_url: settings['calico_url']
            }
        end
    end

    (1..N).each do |i|
        config.vm.define "node-#{i}" do |node|
            ip_addr = ip_addr.succ
            node.vm.box = IMAGE_NAME
            node.vm.network "private_network", ip: ip_addr.to_s
            node.vm.hostname = settings['node_name_prefix']+"-#{i}"
            node.vm.provider "virtualbox" do |vb|
              vb.name =  node.vm.hostname
              if settings['provision_local_volumes'] == true
                disk = node.vm.hostname + ".vdi"
                disk_filename = File.join(virtualbox_machine_folder, node.vm.hostname, disk)
                disk_size = settings['volume_size_gb']
                unless File.exist?(disk_filename)
                  vb.customize ["createmedium", "--filename", disk_filename, "--size", 1024*disk_size]
                end
                vb.customize ["storageattach", :id, "--storagectl", "SCSI", "--port", 2, "--device", 0, "--type", "hdd", "--medium", disk_filename]
              end
              vb.memory = settings['node_memory']
            end
            node.vm.provision "ansible" do |ansible|
                ansible.playbook = "kubernetes-setup/node-playbook.yml"
                ansible.extra_vars = {
                    node_ip: ip_addr.to_s,
                }
            end
        end
    end

end
