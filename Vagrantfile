# -*- mode: ruby -*-
# vi: set ft=ruby :

module OS
    def OS.windows?
        (/cygwin|mswin|mingw|bccwin|wince|emx/ =~ RUBY_PLATFORM) != nil
    end

    def OS.mac?
        (/darwin/ =~ RUBY_PLATFORM) != nil
    end

    def OS.unix?
        !OS.windows?
    end

    def OS.linux?
        OS.unix? and not OS.mac?
    end
end

require 'yaml'
current_dir    = File.dirname(File.expand_path(__FILE__))
config     = YAML.load_file("#{current_dir}/app/config/agile.yml")

if File.exists? ("#{current_dir}/app/config/parameters.yml")
    params = YAML.load_file("#{current_dir}/app/config/parameters.yml")
    smb_username = params['parameters']['agile_vagrant_smb_user']
    smb_password = params['parameters']['agile_vagrant_smb_password']
else
    smb_username = ''
    smb_password = ''
end

ip = config['parameters']['agile.vagrant_ip']
name = config['parameters']['agile.project_name']
url = config['parameters']['agile.project_url']

Vagrant.configure(2) do |config|
  config.vm.box = "ubuntu/xenial64"
  config.vm.box_check_update = false

  config.vm.network "forwarded_port", guest: 80, host: 8002
  config.vm.network "forwarded_port", guest: 3306, host: 3302
  config.vm.network "forwarded_port", guest: 9200, host: 9202
  config.vm.network "private_network", ip: ip

   if Vagrant::Util::Platform.windows? then
     config.vm.synced_folder ".", "/var/www", type: "smb", smb_username: smb_username, smb_password: smb_password
   elsif OS.mac?
     config.vm.synced_folder ".", "/var/www", type: "nfs", :linux__nfs_options => ["rw","no_root_squash","no_subtree_check"]
   else
     config.vm.synced_folder ".", "/var/www", type: "nfs", :linux__nfs_options => ["rw", "no_root_squash", "no_subtree_check"], nfs_version: "4", nfs_udp: false
   end

   config.vm.provider "virtualbox" do |vb|
     vb.gui = false
     vb.memory = "2048"
   end

  config.vm.provision "file", source: "./deploy/apache/project.conf", destination: "/var/www/project.conf"
  config.vm.provision "file", source: "./deploy/apache/ssl/project.crt", destination: "/var/www/project.crt"
  config.vm.provision "file", source: "./deploy/apache/ssl/project.key", destination: "/var/www/project.key"
  config.vm.provision "file", source: "./deploy/phpmyadmin/config.inc.php", destination: "/var/www/config.inc.php"
  config.vm.provision "file", source: "./deploy/phpmyadmin/apache.conf", destination: "/var/www/apache.conf"
  config.vm.provision "file", source: "./deploy/rabbitmq/rabbitmq.config", destination: "/var/www/rabbitmq.config"
  config.vm.provision "shell", path: "./deploy/script.sh", args: [ip, name, url, '1234']
end
