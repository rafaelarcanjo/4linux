# -*- mode: ruby -*-
# vi: set ft=ruby :

VAGRANT_DISABLE_VBOXSYMLINKCREATE=1

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
      end
    end
  end
end
