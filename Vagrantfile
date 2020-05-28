# -*- mode: ruby -*-
# vi: set ft=ruby :

config_file = ENV['CONFIG'] || 'config.yml'
CONFIG = YAML::load_file(File.join(__dir__, config_file))

ENV['VAGRANT_DEFAULT_PROVIDER'] = 'libvirt'
BOX_IMAGE                       = 'centos/7'.freeze

def deploy_kubernetes(machine)
  machine.vm.provision 'ansible' do |ansible|
    ansible.playbook = 'ansible/deploy.yml'
    ansible.limit = 'all'
    ansible.verbose = CONFIG.dig('verbose') || ''
    ansible.groups = {
      'masters' => CONFIG['masters'].map { |vm| vm['name'] },
      'workers' => CONFIG['workers'].map { |vm| vm['name'] },
      'all_groups:children' => ['masters', 'workers']
    }
    ansible.extra_vars = {
      'k8s_version' => CONFIG.dig('k8s_version'),
      'master_cc_ip' => CONFIG['masters'][0]['cc_ip'],
      'pod_subnet' => CONFIG['pod_subnet'],
      'service_subnet' => CONFIG['service_subnet']
  }.compact
  end
end

def configure_worker(machine, machine_cfg)
  machine.vm.hostname = machine_cfg['name']
  machine.vm.network :private_network, ip: machine_cfg['cc_ip']

  machine.vm.provider :libvirt do |libvirt|
    libvirt.cpus                  = machine_cfg.dig('cpus') || 4
    libvirt.memory                = machine_cfg.dig('memory') || 16384
    libvirt.machine_virtual_size  = machine_cfg.dig('disk') || 50
  end
end

def configure_master(machine, machine_cfg)
  configure_worker machine, machine_cfg

  machine.vm.network :forwarded_port, guest: 6443, host: machine_cfg['host_port']
  deploy_kubernetes machine
end

Vagrant.configure '2' do |config|
  config.vm.box = BOX_IMAGE

  config.vm.provider :libvirt do |libvirt|
    libvirt.storage_pool_path = '/home/root/images-vagrant'
    libvirt.username          = 'vagrant'
    libvirt.password          = 'vagrant'
  end

  config.vm.provision 'shell', inline: <<-SHELL
    sed -i 's/PasswordAuthentication no/PasswordAuthentication yes/g' /etc/ssh/sshd_config
    systemctl restart sshd.service
  SHELL

  # Configure workers
  CONFIG['workers'].each do |worker|
    config.vm.define worker['name'] do |machine|
      configure_worker machine, worker
    end
  end

  # Configure masters
  CONFIG['masters'].each do |master|
    config.vm.define master['name'] do |machine|
      configure_master machine, master
    end
  end
end
