Vagrant.configure("2") do |config|
  config.vm.box = "ubuntu/jammy64"
  config.vm.hostname = "webserver"
  config.vm.network "private_network", ip: "192.168.56.10"
  config.vm.provider "virtualbox" do |vb|
    vb.memory = "1024"
    vb.cpus = 1
  end
  config.vm.provision "ansible" do |ansible|
    ansible.playbook = "playbooks/webserver.yml"
    ansible.inventory_path = "inventory/hosts.ini"
  end
end
