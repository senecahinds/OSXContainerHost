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

    node.vm.network "private_network", ip: "192.168.200.3"

    node.nfs.map_uid = Process.uid
    node.nfs.map_gid = Process.gid

    #sync the entire users folder for things like logs and whatever else.
    node.vm.synced_folder "/Users", "/Users.tmp", type: "nfs",
      mount_options: ['rw', 'vers=3', 'tcp', 'fsc' ,'actimeo=2']

    #Use bindfs to set the proper permissions.
    node.bindfs.bind_folder "/Users.tmp", "/Users",
      create_as_user: true,
      perms: "u=u:g=g:o=o"

    settings['rsync'].each do |k,v|
      item = v
      config.vm.synced_folder item["source"], item["dest"], type: "rsync",
        rsync__exclude: ".git/"
    end

    #use rsync for drupal. Non-rsync methods have proven too slow.
    #config.vm.synced_folder DRUPAL_DIR, "/www", type: "rsync",
    #  rsync__exclude: ".git/"

    node.vm.provision "ansible" do |ansible|
      ansible.playbook = "provisioning/ansible/container_host.yml"
    end
  end
end
