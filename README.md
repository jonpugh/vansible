vansible
========

Vagrant + Ansible

This repo is for experimentation with vagrant and ansible.

Goals
-----

The goal of this project is to create a standardized Vagrant-based development workflow for Drupal that:

- Requires little to no setup on the part of the developer.
- Installs as fast as possible.
- Has the smallest code footprint possible (which is why it uses Ansible.)

Features
--------

There are a number of tricks this Vagrantfile uses to streamline setup and development:

1. Uses your computer's SSH public/private key pair, removing the need to generate and send out your VM's SSH keys, and granting ssh access to the host automatically.
2. Uses project-specific YAML file to define settings for your project.  Simply add a name and git url to get started. Add settings overrides as needed here.
3. Automatically clones the project repo as defined in `settings.project.yml` on first `vagrant up`.
4. Vagrant handles saving the IP and domain name to your `/etc/hosts` file automatically.
5. Creates drush aliases for the site in the guest machine and *on the host* for easy access.


