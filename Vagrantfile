# -*- mode: ruby -*-
# vi: set ft=ruby :

VAGRANTFILE_API_VERSION = "2"

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|

  # Look for project variables file.
  if !(File.exists?(File.dirname(__FILE__) + "/vars.project.yml"))
    raise NoVarsException
  end

  # Load the yml files
  require 'yaml'
  vars = YAML.load_file(File.dirname(__FILE__) + "/vars.global.yml")
  vars.merge!(YAML.load_file(File.dirname(__FILE__) + "/vars.project.yml"))

  # Clone project repo to `./src` if the folder doesn't exist yet, and the vars exists.
  if !(File.directory?(File.dirname(__FILE__) + "/src"))

    # If a project_repo is set, clone it.
    if vars['project_repo']
      system("git clone #{vars['project_repo']} #{File.dirname(__FILE__)}/src")
    # otherwise, create the src directory so we can share it to the guest.
    else
      system("mkdir #{File.dirname(__FILE__)}/src");
    end

    if !(File.directory?(File.dirname(__FILE__) + "/src"))
      raise NoSrcException
    end
  end

  # Config the VM
  config.vm.box = "hashicorp/precise64"
  config.vm.hostname = vars['server_hostname']

  # Sets IP of the guest machine and allows it to connect to the internet.
  # @TODO: Add the adapter to vars.global.yml. Almost always wlan0
  config.vm.network :private_network, ip:  vars['vansible_ip']
  #config.vm.network :public_network, bridge: vars['vansible_adapter']

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
   config.vm.provision "ansible" do |ansible|
     ansible.playbook = vars['vansible_playbook']
     ansible.tags = vars['vansible_tags']
     ansible.extra_vars = {
       ansible_ssh_user: 'vagrant',
       authorized_keys: ssh_public_key
     }
     ansible.sudo = true
   end

  # Setup ansible and run the playbook.
  # @TODO: Remove once vagrant supports ansible on guest.
  # config.vm.provision "shell", path: "tasks/setup-ansible.sh"

  # Run ansible Provisioner via shell.
  #config.vm.provision "shell",
  #    inline: "cd /vagrant; ansible-playbook -c local  -i '#{vars['server_hostname']},' --tags='#{vars['vansible_tags']}' #{vars['vansible_playbook']} --extra-vars 'authorized_keys=\"#{ssh_public_key}\"'"

  config.vm.provider :virtualbox do |vb|
    vb.customize ["modifyvm", :id, "--memory", vars['vansible_memory']]
  end

  # Sync project folder to guest machine.
  config.vm.synced_folder "src/#{vars['path_to_drupal']}", "/var/www",
    owner: "www-data", group: "www-data"

  # Save a local alias for this project.
  if (!File.exists?("#{Dir.home}/.drush"))
    Dir.mkdir("#{Dir.home}/.drush")
  end

  DRUSH_ALIAS_FILE = "#{Dir.home}/.drush/#{vars['project']}.alias.drushrc.php"
  if (!File.exists?(DRUSH_ALIAS_FILE))
    # @TODO: Is is possible to load this from the ansible template?
    drush_alias = "
  <?php
  $aliases['#{vars['project']}'] = array(
    'uri' => '#{vars['server_hostname']}',
    'root' => '/var/www',
    'remote-host' => '#{vars['server_hostname']}',
    'remote-user' => 'vagrant',
  );
    "
    if (File.write("#{Dir.home}/.drush/#{vars['project']}.alias.drushrc.php", drush_alias))
      # @TODO: Replace with Vagrant::UI::Basic once we know how to use it :(
      puts "Drush alias created. You may access your site with `drush @#{vars['project']}`"
    end
  end

end

##
# Our Exceptions
#
class NoVarsException < Vagrant::Errors::VagrantError
  error_message('Project variables file not found. Copy vars.project.example.yml to vars.project.yml, edit to match your project, then try again.')
end

class NoSrcException < Vagrant::Errors::VagrantError
  error_message('Could not create ./src folder. Run as the owner of this folder. ')
end

class NoSshKeyException < Vagrant::Errors::VagrantError
  error_message('An ssh public key could not be found at ~/.ssh/id_rsa.pub. Please generate one and try again.')
end
