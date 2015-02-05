# -*- mode: ruby -*-
# vi: set ft=ruby :

# Some features used in this configuration file require specific version of
# Vagrant.
Vagrant.require_version ">= 1.5.0"

# Vagrant API version.
VAGRANTFILE_API_VERSION = "2"

# Configuration
#
# Load configuration settings from YAML file(s).
#
# Default configuration is stored in provisioning/default.settings.yml.
# Project specific overrides should be placed in provisioning/settings.yml.
#
# Settings are being merged recursively. Values from project settings file
# overwrites ones from default settings; missing values are not being touch;
# new values will be added to the resulting settings hash.

require 'yaml'

default_settings = YAML::load_file("provisioning/default.settings.yml")
if File.exist?("provisioning/settings.yml")
  require_relative 'ruby/utility'
  project_settings = YAML::load_file("provisioning/settings.yml")
  unless project_settings.nil?
    settings = merge_recursively(default_settings, project_settings)
  else
    settings = default_settings
  end
end

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|

  # Boxes
  #
  # Box is a package format for Vagrant environments. See more at
  # http://docs.vagrantup.com/v2/boxes.html
  #
  # HashiCorp provides publicly available list of Vagrant boxes at
  # https://atlas.hashicorp.com/boxes/search

  # Official Ubuntu Server 14.04 LTS (Trusty Tahr).
  config.vm.box = "ubuntu/trusty64"
  config.vm.box_version = "14.04"

  # Networking
  #
  # By default Vagrant doesn't require explicit network setup. However, in
  # various situations this action is mandatory.
  # See https://docs.vagrantup.com/v2/networking/index.html

  # Set machine's hostname.
  config.vm.hostname = settings["box"]["name"]

  # Network File System (NFS) requires private network to be specified when
  # VirtualBox is used (due to a limitation of VirtualBox's built-in networking)
  # See http://docs.vagrantup.com/v2/synced-folders/nfs.html
  config.vm.network "private_network", ip: settings["box"]["ip_address"]

  # SSH settings
  #
  # See https://docs.vagrantup.com/v2/vagrantfile/ssh_settings.html

  # Enable SSH agent forwarding
  config.ssh.forward_agent = true

  # Fix annoying "stdin: is not a tty" error.
  # See https://github.com/mitchellh/vagrant/issues/1673#issuecomment-40278692
  config.ssh.shell = "bash -c 'BASH_ENV=/etc/profile exec bash'"

  # VirtualBox configuration
  #
  # VirtualBox allows for some additional virtual machine tuning. List of
  # available options can be found here: http://www.virtualbox.org/manual/ch08.html
  # See https://docs.vagrantup.com/v2/virtualbox/configuration.html

  # Tune VirtualBox powered machine.
  config.vm.provider :virtualbox do |v|
    # Set CPUs count.
    v.customize ["modifyvm", :id, "--cpus", settings["virtualbox"]["cpus"]]
    # Set memory limit (in MB).
    v.customize ["modifyvm", :id, "--memory", settings["virtualbox"]["memory"]]
    # Set CPU execution cap (in %).
    v.customize ["modifyvm", :id, "--cpuexecutioncap", settings["virtualbox"]["cpuexecutioncap"]]
    # Use host's resolver mechanisms to handle DNS requests.
    v.customize ["modifyvm", :id, "--natdnshostresolver1", "on"]
    # Set VM name.
    v.name = settings["box"]["name"]
  end

  # Synced Folders
  #
  # See https://docs.vagrantup.com/v2/synced-folders/index.html

  # NFS sync method is much faster than others. It's not supported on Windows
  # hosts by Vagrant itself, but there is a Vagrant plugin entitled "Vagrant
  # WinNFSd" aimed to resolve this issue. However, at the moment of writing this
  # comment (27.01.2015), plugin has significant issue with some VM OS (like
  # Ubuntu). See https://github.com/GM-Alex/vagrant-winnfsd/issues/27. When the
  # aforementioned issue will be resolved, installation of the plugin would be
  # enforced on Windows hosts. Without WinNFSd plugin Vagrant will fall back to
  # the default VirtualBox folder sync.
  # See https://docs.vagrantup.com/v2/synced-folders/nfs.html

  # Configure synched folders.
  config.vm.synced_folder ".", "/vagrant", type: "nfs"
  config.vm.synced_folder ".", "/var/www/vhosts/default", type: "nfs"

  # Provisioning
  #
  # See https://docs.vagrantup.com/v2/provisioning/index.html

  # IMPORTANT. Vagrant has an issue with Shell provisioner (described here
  # https://github.com/mitchellh/vagrant/issues/1673). To avoid annoying
  # "stdin: is not a tty" and/or "dpkg-reconfigure: unable to re-open stdin: No
  # file or directory" error messages, stdout and sterr have been redirected
  # to /dev/null. See provisioning/windows.sh

  require "rbconfig"

  if (RbConfig::CONFIG["host_os"] =~ /mswin|mingw|cygwin/)
    # Run Shell provisioner for Windows hosts.
    config.vm.provision "shell" do |shell|
      shell.path = "provisioning/windows.sh"
      shell.args = "provisioning/playbooks/main.yml"
    end
  else
    # Run Ansible provisioner for Mac/Linux hosts.
    config.vm.provision "ansible" do |ansible|
      ansible.playbook = "provisioning/playbooks/main.yml"
    end
  end

end
