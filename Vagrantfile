# vim: set ft=ruby
#

#unless Vagrant.has_plugin?("vagrant-dnsmasq")
#      raise 'vagrant-dnsmasq is not installed!'
#end
#
#config.dnsmasq.domain = 'vagrant.fuck'
#
## overwrite default location for /etc/dnsmasq.conf
#brew_prefix = `brew --prefix`.strip
#config.dnsmasq.dnsmasqconf = brew_prefix + '/etc/dnsmasq.conf'
#
## command for reloading dnsmasq after config changes
#config.dnsmasq.reload_command = 'sudo launchctl unload /Library/LaunchDaemons/homebrew.mxcl.dnsmasq.plist; sudo launchctl load /Library/LaunchDaemons/homebrew.mxcl.dnsmasq.plist'
#

servers=[
  {
    :hostname => "graylog2-server",
    :ip => "192.168.33.22",
    :box => "bento/centos-7.3",
    :ram => 2048,
    :cpu => 2,
  },
  {
    :hostname => "mongodb1",
    :ip => "192.168.33.23",
    :box => "bento/centos-7.3",
    :ram => 256,
    :cpu => 1,
    :autostart => false
  },
  {
    :hostname => "mongodb2",
    :ip => "192.168.33.24",
    :box => "bento/centos-7.3",
    :ram => 256,
    :cpu => 1,
    :autostart => false
  },
  {
    :hostname => "mongodb3",
    :ip => "192.168.33.25",
    :box => "bento/centos-7.3",
    :ram => 256,
    :cpu => 1,
    :autostart => false
  },
  {
    :hostname => "elasticsearch1",
    :ip => "192.168.33.26",
    :box => "bento/centos-7.3",
    :ram => 1024,
    :cpu => 1,
    :autostart => false
  },
  {
    :hostname => "elasticsearch2",
    :ip => "192.168.33.27",
    :box => "bento/centos-7.3",
    :ram => 1024,
    :cpu => 1,
    :autostart => false
  },
  {
    :hostname => "elasticsearch3",
    :ip => "192.168.33.28",
    :box => "bento/centos-7.3",
    :ram => 1024,
    :cpu => 1,
    :autostart => false
  }
]

Vagrant.configure(2) do |config|

    if Vagrant.has_plugin?("vagrant-hostmanager")
        config.hostmanager.enabled           = true
        config.hostmanager.manage_guest      = true
    end

    config.ssh.insert_key = false
    servers.each do |machine|
        config.vm.define machine[:hostname] do |node|
            node.vm.box = machine[:box]
            #node.vm.hostname = machine[:hostname] + domain
            node.vm.hostname = machine[:hostname]
            node.vm.network "private_network", ip: machine[:ip]
            node.vm.provider "virtualbox" do |vb|
                vb.customize ["modifyvm", :id, "--memory", machine[:ram]]
            end
            node.vm.network "forwarded_port", guest: 9200, guest_ip: machine[:ip], host: 9201,
                auto_correct: true if machine[:hostname] == "elasticsearch1"
            node.vm.network "forwarded_port", guest: 9200, guest_ip: machine[:ip], host: 9202,
                auto_correct: true if machine[:hostname] == "elasticsearch2"
            node.vm.network "forwarded_port", guest: 9200,guest_ip: machine[:ip], host: 9203,
                auto_correct: true if machine[:hostname] == "elasticsearch3"
            node.vm.network "forwarded_port", guest: 9000,guest_ip: machine[:ip], host: 9000,
                auto_correct: true if machine[:hostname] == "graylog2-server"
        end
    end
    config.vm.provision "ansible" do |ansible|
    # ansible.limit = "all"
    ansible.config_file = "./ansible.cfg"
    ansible.verbose = "vv"
	ansible.groups = {
	  "mongodb" => ["mongodb1", "mongodb2", "mongodb3"],
      # "mongodbmaster" => ["mongodb1"],
      "elasticsearch" => ["elasticsearch1", "elasticsearch2", "elasticsearch3"],
	  "graylog2"  => ["graylog2-server"],
	  "replication_servers" => ["mongodb1", "mongodb2", "mongodb3"],
	  # "mongoc_servers" => ["mongodb1", "mongodb2", "mongodb3"],
	  # "mongos_servers" => ["mongodb1", "mongodb2"]
      #
	}
	ansible.host_vars = {
		 "mongodb1" => {"mongod_port" => 2700},
		 "mongodb2" => {"mongod_port" => 2701},
		 "mongodb3" => {"mongod_port" => 2702}
	   #"graylog" => { "graylog2-server" => "#{machine['graylog2-server'][:ip]}" }
	}
    #ansible.tags = ["provision","setup","elasticsearch"]
    #ansible.galaxy_role_file = "../fulhack.graylog2-ansible-role/requirements.yml"
    ansible.galaxy_roles_path = "./roles"
    ansible.tags = ["provision"]
    #ansible.playbook = "../task-setup.yml"
    ansible.playbook = "provision.yml"
    #ansible.playbook = "provision/site.yml"
    end
end
