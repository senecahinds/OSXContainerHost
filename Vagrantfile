# -*- mode: ruby -*-
# vi: set ft=ruby :

# PLEASE NOTE:
# You will need the plugins listed below for this VM to function correctly.
# Please also note that the bindfs plugin does make performance a little slower
# but is necessary for permission in Docker containers to work as expected.
#
# $ vagrant plugin install vagrant-bindfs
require 'yaml'
settings = YAML.load_file 'settings.yml'
VAGRANTFILE_API_VERSION = "2"
Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  config.vm.define "container-host" do |node|
    node.vm.box = "ubuntu/trusty64"
    node.vm.hostname = "container-host"

    node.vm.provider "virtualbox" do |v|
      v.name = "container-host"
      v.cpus   = settings['vm']['cpus']
      v.memory = settings['vm']['memory']
    end

    node.vm.network "private_network", ip: settings['vm']['ip']

    node.nfs.map_uid = Process.uid
    node.nfs.map_gid = Process.gid

    #sync the entire users folder for things like logs and whatever else.
    node.vm.synced_folder settings['data'], "/data.tmp", type: "nfs",
      mount_options: ['rw', 'vers=3', 'tcp', 'fsc' ,'actimeo=2']

    #Use bindfs to set the proper permissions.
    node.bindfs.bind_folder "/data.tmp", "/data",
      create_as_user: true,
      perms: "u=u:g=g:o=o"

    settings['rsync'].each do |k,v|
      item = v
      config.vm.synced_folder item["source"], item["dest"], type: "rsync",
        rsync__exclude: ".git/"
    end

    node.vm.provision "ansible" do |ansible|
      ansible.playbook = "provisioning/ansible/container_host.yml"
    end

    # Configure the window for gatling to coalesce writes.
    if Vagrant.has_plugin?("vagrant-gatling-rsync")
      config.gatling.latency = 2.5
      config.gatling.time_format = "%H:%M:%S"
    end

  end
end

#
#
# Print dev information
puts "###############################\n"
puts "# NOTE: If this is a newly created vm you might need to run another vagrant reload to get docker working correctly\n\n"
puts "export DOCKER_HOST=tcp://#{settings['vm']['ip']}:2375"
puts "\n###############################\n"
