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
| Vagrant bindfs | Once vagrant is installed run: ``` $ vagrant plugin install vagrant-bindfs ```
| Ansible        | https://devopsu.com/guides/ansible-mac-osx.html |


## Usage

#### Setup
```
$ git clone git@github.com:SeerUK/OSXContainerHost.git
$ cp default.settings.yml settings.yml
```
Edit the settings.yml as necessary

#### Running
```
$ vagrant up
$ export DOCKER_HOST=tcp://192.168.200.3:2375
```
(Or alternatively, add: `export DOCKER_HOST=tcp://192.168.200.3:2375` to whatever shell rc is relevent to your system)

#### Rsync
```
$ vagrant rsync-auto
```

## Things to be wary of

* NFS is configured to map all activity in `/Users/` to the uid and gid you run Vagrant as. This will also affect volumes in containers if they are mounted from the `/Users/` folder (which I imagine most will be...). Realistically, for most use-cases this shouldn't be a problem, but it is something to be aware of. Please suggest a better solution if there is one.
* Docker containers are created inside the VM, the images are downloaded inside the VM. Everything happens in there; therefore, if you remove and recreate the VM you will lose all of your existing containers.
