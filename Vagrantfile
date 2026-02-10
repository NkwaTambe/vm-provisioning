Vagrant.configure("2") do |config|
  config.vm.box = "generic/ubuntu2204"
  config.vm.provider "vmware_desktop" do |v|
    v.memory = 2048
    v.cpus = 2
    v.linked_clone = false
  end

  config.vm.network "forwarded_port", guest: 80, host: 8080

  config.vm.synced_folder ".", "/vagrant"

  config.vm.provision "shell", inline: <<-SHELL
    apt-get update
    apt-get install -y nginx
    sudo cp /vagrant/index.html /var/www/html/index.html
  SHELL
end