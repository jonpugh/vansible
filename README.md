vansible
========

Vagrant + Ansible

This repo is for experimentation with vagrant and ansible.  I don't know where this is going, but it's going somewhere, as it works really well.

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


Setup
-----
This repo is designed to be cloned for each Drupal site you are working on.

To start a new project:

0. Install Git, Vagrant, and VirtualBox (or another provider like VMware).
0.1. Install the vagrant-hosts plugin:
  ```
  vagrant plugin install vagrant-hosts
  ```

1. Clone this repo to a folder named after the project.

    ```
    $ git clone git@github.com:jonpugh/vansible.git projectname
    ```
2. Copy and edit settings.project.yml.

    ```
    $ cd projectname
    $ cp settings.project.example.yml settings.project.yml
    $ vim settings.project.yml
    ```

    Put your project's name and the URL of your project's git repository in
    `settings.project.yml`.

3. Vagrant up.

    ```
    $ vagrant up
    ```

    The first time you vagrant up, vagrant will:

    1. Download the base box if needed.
      Vansible uses hashicorp/precise64 as it's base box.
    2. Clone your git repo to `./src`.
    3. Provision your virtual server using provision.yml.
    4. Save a record to your `/etc/hosts` file so you can access the virtual machine.

4. Install your Drupal site.

    Your site will now be available at http://projectname/.

    You can access the site with drush from your computer (the host) using the alias `@projectname`.

    Currently you must use the web-based installer because the sites/projectname folder is owned by www-data, so visit http://projectname/install.php in a web browser.

    Alternately you can import a db manually with drush sql-sync, sqlc, etc.

5. Develop!

    Edit the code in the ./src folder of this repo to work on your drupal site.

@TODO
-----

1. Accept input during `vagrant up` to write settings.project.yml file automatically. 
2. Setup "remote master" alias to setup automatic syncing. (look at drush fetch).
3. ...
