# -*- mode: ruby -*-
# vi: set ft=ruby :

# Vagrantfile API/syntax version. Don't touch unless you know what you're doing!
VAGRANTFILE_API_VERSION = "2"


Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|

  # Look for project settings file.
  if !(File.exists?("settings.project.yml"))
    raise NoSettingsException
  end

  # Load the yml files
  require 'yaml'
  settings = YAML.load_file('settings.project.yml')

  # Clone project if not found
  if !(File.directory?("src"))
    system("git clone #{settings['project_repo']} src")

    if !(File.directory?("src"))
      raise NoSrcException
    end
  end

  # Config the VM
  config.vm.box = "hashicorp/precise64"
  config.vm.hostname = settings['server_hostname']

  # @TODO: Put these in settings.global.yml, merge into settings hash, and use here.
  VANSIBLE_TAGS = "common,drush"
  VANSIBLE_IP = "10.10.10.10"
  VANSIBLE_PLAYBOOK = "provision.yml"
  VANSIBLE_MEMORY = "2048"

  # Sets IP of the guest machine and allows it to connect to the internet.
  # @TODO: Add the adapter to settings.global.yml. Almost always wlan0
  config.vm.network :private_network, ip: VANSIBLE_IP
  config.vm.network :public_network

  # Sync .ssh folder to guest machine.
  config.vm.synced_folder "#{Dir.home}/.ssh", "/home/vagrant/.ssh_host"

  # Read this user's host machine's public ssh key to pass to ansible.
  if !(File.exists?("#{Dir.home}/.ssh/id_rsa.pub"))
     raise NoSshKeyException
  end
  ssh_public_key = IO.read("#{Dir.home}/.ssh/id_rsa.pub").strip!
  
  # ONLY WORKS if ansible is setup on the HOST machine.
  # See https://github.com/mitchellh/vagrant/issues/2103
  # @TODO: Uncomment once vagrant supports this.
  # config.vm.provision "ansible" do |ansible|
  #   ansible.playbook = VANSIBLE_PLAYBOOK
  #   ansible.tags = VANSIBLE_TAGS
  # end

  # Setup ansible and run the playbook.
  # @TODO: Remove once vagrant supports ansible on guest.
  config.vm.provision "shell", path: "tasks/setup-ansible.sh"
  
  # Run ansible Provisioner via shell.
  config.vm.provision "shell",
      inline: "cd /vagrant; ansible-playbook -c local  -i '#{settings['server_hostname']},' --tags='#{VANSIBLE_TAGS}' #{VANSIBLE_PLAYBOOK} --extra-vars 'authorized_keys=\"#{ssh_public_key}\"'"

  config.vm.provider :virtualbox do |vb|
    vb.customize ["modifyvm", :id, "--memory", VANSIBLE_MEMORY]
  end

  # Sync project folder to guest machine.
  config.vm.synced_folder "src/#{settings['path_to_drupal']}", "/var/www",
    owner: "www-data", group: "www-data"

  # Save a local alias for this project.
  DRUSH_ALIAS_FILE = "#{Dir.home}/.drush/#{settings['project']}.alias.drushrc.php"
  if (!File.exists?(DRUSH_ALIAS_FILE))
    // @TODO: Is is possible to load this from the ansible template?
    drush_alias = "
  <?php
  $aliases['#{settings['project']}'] = array(
    'uri' => '#{settings['server_hostname']}',
    'root' => '/var/www',
    'remote-host' => '#{settings['server_hostname']}',
    'remote-user' => 'vagrant',
  );
    "
    if (File.write("#{Dir.home}/.drush/#{settings['project']}.alias.drushrc.php", drush_alias))
      # @TODO: Replace with Vagrant::UI::Basic once we know how to use it :(
      puts "Drush alias created. You may access your site with `drush @#{settings['project']}`"
    end
  end

end

##
# Our Exceptions
#
class NoSettingsException < Vagrant::Errors::VagrantError
  error_message('Project settings file not found. Copy settings.project.example.yml to settings.project.yml, edit to match your project, then try again.')
end

class NoSrcException < Vagrant::Errors::VagrantError
  error_message('Project source does not exist.  Clone your project to ./src .')
end

class NoSshKeyException < Vagrant::Errors::VagrantError
  error_message('An ssh public key could not be found at ~/.ssh/id_rsa.pub. Please generate one and try again.')
end