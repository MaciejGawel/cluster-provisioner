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
      'cluster_name' => CONFIG.dig('cluster_name'),
      'master_cc_ip' => CONFIG['masters'][0]['cc_ip'],
      'registry_ip' => CONFIG['registry_ip'],
      'pod_subnet' => CONFIG['pod_subnet'],
      'service_subnet' => CONFIG['service_subnet'],
      'ext_start' => CONFIG.dig('ext_start'),
      'ext_end' => CONFIG.dig('ext_end'),
      'calico_enabled' => CONFIG.dig('calico_enabled'),
      'metallb_enabled' => CONFIG.dig('metallb_enabled')
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

  machine.vm.network :forwarded_port, guest: 6443, host: machine_cfg['host_port'], host_ip: '0.0.0.0'
  deploy_kubernetes machine
end

Vagrant.configure '2' do |config|
  config.vm.box = BOX_IMAGE

  config.vm.provider :libvirt do |libvirt|
    libvirt.storage_pool_path = CONFIG.dig('storage_pool_path')
    libvirt.storage_pool_name = 'vagrant'
    libvirt.username          = 'vagrant'
    libvirt.password          = 'vagrant'
  end

  config.vm.provision 'shell', inline: <<-SHELL
    sed -i 's/PasswordAuthentication no/PasswordAuthentication yes/g' /etc/ssh/sshd_config
    systemctl restart sshd.service
    yum install -y cloud-utils-growpart
    growpart /dev/vda 1
    xfs_growfs -d /
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
