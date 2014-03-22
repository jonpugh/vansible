# -*- mode: ruby -*-
# vi: set ft=ruby :

# Vagrantfile API/syntax version. Don't touch unless you know what you're doing!
VAGRANTFILE_API_VERSION = "2"

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|

  config.vm.box = "hashicorp/precise64"
  config.vm.hostname = "local.engageny.org"

  # Sets IP of the guest machine and allows it to connect to the internet.
  # config.vm.network :private_network, ip: "10.10.10.10"
  config.vm.network :public_network
  config.vm.network :forwarded_port, guest: 80, host: 8888

  # Sync .ssh folder to guest machine.
  config.vm.synced_folder "#{Dir.home}/.ssh", "/home/vagrant/.ssh_host"

  # Read hosts public ssh key to pass to ansible.
  if !(File.exists?("#{Dir.home}/.ssh/id_rsa.pub"))
      warn "We could not find your SSH public key at ~/.ssh/id_rsa.pub. Please generate a key there and try again."
      exit
  end
  ssh_public_key = IO.read("#{Dir.home}/.ssh/id_rsa.pub")
  
  # ONLY WORKS if ansible is setup on the HOST machine.
  # See https://github.com/mitchellh/vagrant/issues/2103
  # @TODO: Uncomment once vagrant supports this.
  # config.vm.provision "ansible" do |ansible|
  #   ansible.playbook = "setup.yml"
  #   ansible.tags = "common,drush"
  # end

  # Setup ansible and run the playbook.
  # @TODO: Remove once vagrant supports ansible on guest.
  config.vm.provision "shell", path: "setup-ansible.sh"
  
  # @TODO: Read tags from settings.yml, or local.yml or something. trying to make this extensible.
  config.vm.provision "shell",
      inline: "cd /vagrant; ansible-playbook -c local  -i 'localhost,' --tags='common,drush' setup.yml  --extra-vars 'authorized_keys=\"#{ssh_public_key}\"'"

  config.vm.provider :virtualbox do |vb|
    vb.customize ["modifyvm", :id, "--memory", "2048"]
  end
end
