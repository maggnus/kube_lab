# -*- mode: ruby -*-
# vi: set ft=ruby :

# Require YAML module
require 'yaml'

# Vagrant API version
VAGRANTFILE_API_VERSION = "2"

# Read cluster settings
cluster_config = YAML.load_file('etc/config.yml')
servers = cluster_config["servers"]
providers = cluster_config["providers"]

count = 1

# Create boxes
Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|

  if Vagrant.has_plugin?("vagrant-hostmanager")
    config.hostmanager.enabled = true
    config.hostmanager.manage_host = false
    config.hostmanager.manage_guest = true
    config.hostmanager.ignore_private_ip = false
    config.hostmanager.include_offline = true
  end

  if Vagrant.has_plugin?("vagrant-vbguest")
    config.vbguest.auto_update = false
  end

  $ansible_groups = {}

  # Collect server groups
  servers.each do |server|
    role = server["role"]
    host = server["name"]
    if $ansible_groups.has_key?(role)
      $ansible_groups[role] << host
    else
      $ansible_groups[role] = [host]
    end
  end

  $host_vars = {}

  servers.each do |server|
    host = server["name"]
    $host_vars[host] = {}
    server.each do |key, value|
        $host_vars[host]["instance_" + key] = value
     end
  end
  
  servers.each do |server|
    config.vm.define server["name"] do |srv|
      srv.vm.box = cluster_config["box_name"]
      srv.vm.hostname = server["name"]
      # config.vm.network "private_network", type: "dhcp"
      srv.vm.network "private_network", ip: server["ip"] #, auto_config: false

      if providers["virtualbox"]["enable"]
        srv.vm.provider :virtualbox do |vb|
          vb.name = server["name"]
          vb.cpus = server["cpu"]
          vb.memory = server["ram"]
        end
      end

      ## Setting up static ip bug
      #srv.vm.provision "shell", inline: "sudo nmcli connection reload"
      #srv.vm.provision "shell", inline: "sudo systemctl restart network.service"


      if count == servers.length
        ## Ansible configuration      
        config.vm.provision :ansible do |ansible|
          #ansible.verbose = "vvvv"
          #ansible.playbook = server["role"]+".yml"
          #ansible.version = "latest"
          ansible.limit = "all"
          ansible.compatibility_mode = "2.0"
          ansible.playbook = 'ansible/playbook.yaml'
          ansible.host_key_checking = false
          ansible.groups = $ansible_groups
          ansible.host_vars = $host_vars
        end
      end

      count = count + 1

    end
  end
end

