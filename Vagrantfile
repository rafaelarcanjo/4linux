# -*- mode: ruby -*-
# vi: set ft=ruby :

VAGRANT_DISABLE_VBOXSYMLINKCREATE=1

require 'yaml'

current_dir     = File.dirname(File.expand_path(__FILE__))
secrets         = YAML.load_file("#{current_dir}/secrets.yaml")
vagrant_secrets = secrets['secrets'][secrets['secrets']['use']]

vms = {
  'server' => {
    'memory'    => '2048', 
    'cpus'      => 2, 
    'ip'        => '175', 
    'box'       => 'debian/buster64', 
    'provision' => 'provision/playbook.yaml'
  },
  'client' => {
    'memory'    => '2048', 
    'cpus'      => 2, 
    'ip'        => '254', 
    'box'       => 'debian/buster64', 
    'provision' => 'provision/playbook.yaml'
  }
}

Vagrant.configure('2') do |config|

  config.vm.box_check_update = false

  vms.each do |name, conf|
    config.vm.define "#{name}" do |k|
      k.ssh.insert_key=false
      k.vm.box = "#{conf['box']}"
      k.vm.hostname = "#{name}"
      k.vm.network 'private_network', ip: "192.168.63.#{conf['ip']}"
      k.vm.synced_folder "./app-devops", "/opt/app-devops"
      
      k.vm.provider 'virtualbox' do |vb|
        vb.memory = conf['memory']
        vb.cpus = conf['cpus']
      end
      
      k.vm.provision 'ansible_local' do |ansible|
        ansible.playbook = "#{conf['provision']}"
        ansible.compatibility_mode = '2.0'
        ansible.extra_vars = {
          GRAFANA_PASS: vagrant_secrets['GRAFANA_PASS'],
          MYSQL_ROOT_PASSWORD: vagrant_secrets['MYSQL_ROOT_PASSWORD'],
          PROMETHEUS_PASSWORD: vagrant_secrets['PROMETHEUS_PASSWORD'],
          CLOUDFLARE_TOKEN: vagrant_secrets['CLOUDFLARE_TOKEN']
        }
      end
    end
  end
end
