# -*- mode: ruby -*-
# vi: set ft=ruby :

require 'yaml'
require 'ipaddr'

IMAGE_NAME = "ubuntu/xenial64"

settings = YAML.load_file 'vagrant.yaml'
ip_addr = IPAddr.new(settings['ip_address'])
host_system_properties = IO.popen(["VBoxManage", "list", "systemproperties"])
virtualbox_machine_folder = host_system_properties.select {|s| s.include?('Default machine folder')}[0].partition(':').last.strip

Vagrant.configure("2") do |config|
    config.ssh.insert_key = false

    config.vm.provider "virtualbox" do |v|
	      v.customize ["modifyvm", :id, "--natdnshostresolver1", "on"]
        v.memory = 1536
        v.cpus = 1
    end

    config.vm.define "docker-registry" do |registry|
        registry.vm.box = IMAGE_NAME
        registry.vm.network "private_network", ip:  ip_addr.to_s
        registry.vm.hostname = settings['hostname']
        registry.vm.provider "virtualbox" do |vb|
          vb.name =  registry.vm.hostname
          if settings['provision_local_volumes'] == true
            disk = registry.vm.hostname + ".vdi"
            disk_filename = File.join(virtualbox_machine_folder, registry.vm.hostname, disk)
            disk_size = settings['volume_size_gb']
            unless File.exist?(disk_filename)
              vb.customize ["createmedium", "--filename", disk_filename, "--size", 1024*disk_size]
            end
            vb.customize ["storageattach", :id, "--storagectl", "SCSI", "--port", 2, "--device", 0, "--type", "hdd", "--medium", disk_filename]
          end
        end
        registry.vm.provision "ansible" do |ansible|
            ansible.playbook = "docker-private-registry-setup/playbook.yml"
            ansible.extra_vars = {
                nginx_ip: ip_addr.to_s,
                nginx_ip_with_underscores: ip_addr.to_s.gsub('.','_'),
                basic_auth_user: settings['basic_auth_user'],
                basic_auth_pass: settings['basic_auth_pass'],
            }
        end
    end

end
