OSXContainerHost
================

A Vagrant provisioned VM for OSX to run Docker containers in.
* Ubuntu 14.04
* NSF support
* Rsync support for your project directory
* Per-dev settings file with YAML.

## Requirements

| Requirement    | Notes        |
| :-------------- | :------------ |
| Virtualbox     | https://www.virtualbox.org/wiki/Downloads |
| Docker         | https://github.com/boot2docker/osx-installer/releases/latest Note: Boot2docker itself isn't used, but it comes with docker. You can probably install docker another way if you do not want boot2docker itself on your system. |
| Vagrant        | https://www.vagrantup.com/downloads.html (Requires 1.7+) |
| Vagrant bindfs | ``` $ vagrant plugin install vagrant-bindfs ``` |
| Vagrant rsync-back | ``` $ vagrant plugin install vagrant-rsync-back ``` |
| Vagrant gatling-rsync | ``` $ vagrant plugin install vagrant-gatling-rsync ``` |
| Ansible        | https://devopsu.com/guides/ansible-mac-osx.html |


## Usage

#### Setup
```
$ git clone git@github.com:mwisner/OSXContainerHost.git
$ cp default.settings.yml settings.yml
```
Edit the settings.yml as necessary

#### Running
```
$ vagrant up
$ export DOCKER_HOST=tcp://192.168.200.3:2375
```
(Or alternatively, add: `export DOCKER_HOST=tcp://192.168.200.3:2375` to whatever shell rc is relevent to your system)

Verify docker is running correctly with ```docker ps```. If this command returns an error try running `vagrant reload`. It seems to take a reload to get docker to run correctly for some reason after a newly created vm.


#### Rsync
Why Rsync? This vagrant setup is used to develop Drupal sites. During testing vboxfs folder syncing was ~12 seconds, NFS was ~3 seconds but using rsync was 0.5 seconds.

The problem with this is that it's currently only one-way. Please see https://github.com/SeerUK/OSXContainerHost/tree/development-2way_rsync for some information regarding two-way sync using lsync.

**File watcher:**
```
$ vagrant gatling-rsync-auto
```

**Update host with changes made on the vm:**
```
$ vagrant rsync-back
```
Note: Not automatic. you'll have do this anytime a change is made on the vm.


## Things to be wary of

* NFS is configured to map all activity in `/Users/` to the uid and gid you run Vagrant as. This will also affect volumes in containers if they are mounted from the `/Users/` folder (which I imagine most will be...). Realistically, for most use-cases this shouldn't be a problem, but it is something to be aware of. Please suggest a better solution if there is one.
* Docker containers are created inside the VM, the images are downloaded inside the VM. Everything happens in there; therefore, if you remove and recreate the VM you will lose all of your existing containers.
