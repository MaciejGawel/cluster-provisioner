# Vagrant-based Kubernetes cluster provisioner

## Usage

### Host prerequisites

Install dependencies

```sh
yum install libvirt-devel ruby-devel
```

Install Vagrant

```sh
wget https://releases.hashicorp.com/vagrant/2.2.9/vagrant_2.2.9_x86_64.rpm
yum install vagrant_2.2.9_x86_64.rpm
vagrant plugin install vagrant-libvirt
```

Add Centos box

```sh
vagrant box add centos/7 --provider=libvirt
```

Install Ansible

```sh
yum install epel-release
yum install ansible
```

### Deploy an environment

The simple deployment can be done with the following command

```sh
vagrant up
```

To deploy multiple environments create multiple config files

```sh
CONFIG=config.yml vagrant up
```
