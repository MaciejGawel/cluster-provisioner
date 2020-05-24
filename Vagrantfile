# -*- mode: ruby -*-
# vi: set ft=ruby :

ENV['VAGRANT_DEFAULT_PROVIDER'] = 'libvirt'
BOX_IMAGE                       = 'centos/7'.freeze
NODE_COUNT                      = 2

def deploy_kubernetes(machine)
  machine.vm.provision 'ansible' do |ansible|
    ansible.playbook = 'ansible/deploy.yml'
    ansible.limit = 'all'
    ansible.groups = {
      'masters' => ['node0'],
      'workers' => ["node[1:#{NODE_COUNT}"],
      'all_groups:children' => ['masters', 'workers']
    }
  end
end

def configure_ssh(machine)
  machine.vm.provision 'shell', inline: <<-SHELL
    sed -i 's/PasswordAuthentication no/PasswordAuthentication yes/g' /etc/ssh/sshd_config
    systemctl restart sshd.service
  SHELL
end

Vagrant.configure '2' do |config|
  config.vm.box = BOX_IMAGE

  config.vm.provider :libvirt do |libvirt|
    libvirt.cpus                  = 4
    libvirt.memory                = 16384
    libvirt.machine_virtual_size  = 50
    libvirt.storage_pool_path     = '/home/root/images-vagrant'
    libvirt.username              = 'vagrant'
    libvirt.password              = 'vagrant'
  end

  NODE_COUNT.times do |machine_id|
    config.vm.define "node#{machine_id}" do |machine|
      machine.vm.hostname = "node#{machine_id}"

      machine.vm.network :private_network, ip: "10.10.0.#{10 + machine_id}"
      configure_ssh machine

      if machine_id.zero?
        machine.vm.network :forwarded_port, guest: 6443, host: 6443
        deploy_kubernetes(machine)
      end
    end
  end
end
