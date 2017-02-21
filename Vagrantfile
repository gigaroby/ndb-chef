ENV["LC_ALL"] = "en_US.UTF-8"

Vagrant.configure("2") do |c|
  if Vagrant.has_plugin?("vagrant-omnibus")
#    require 'vagrant-omnibus'
    c.omnibus.chef_version = "12.4.3"
  end
  if Vagrant.has_plugin?("vagrant-cachier")
    c.omnibus.cache_packages = true        
    c.cache.scope = :machine
    c.cache.auto_detect = false
    c.cache.enable :apt
    c.cache.enable :gem    
  end

  (1 .. 2).each do |i|
    c.vm.define "node-#{i}" do |node|
      #  c.vm.synced_folder "/srv/hops-downloads", "/srv/hops-downloads"
      node.vm.box = "opscode-ubuntu-14.04"
      node.vm.box_url = "https://atlas.hashicorp.com/ubuntu/boxes/trusty64/versions/20150924.0.0/providers/virtualbox.box"
      node.vm.hostname = "ndb-node-#{i}"

      # MySQL Server
      local_port = 13000 + i
      node.vm.network(:forwarded_port, {:guest=>3306, :host=>local_port})

      local_ip = "192.168.198.#{10 + i}"

      node.vm.network "private_network", ip: local_ip, :adapter => 2

      node.vm.provider :virtualbox do |p|
        p.customize ["modifyvm", :id, "--memory", "4096"]
        p.customize ["modifyvm", :id, "--natdnshostresolver1", "on"]
        p.customize ["modifyvm", :id, "--natdnsproxy1", "on"]
        p.customize ["modifyvm", :id, "--nictype1", "virtio"]
        p.customize ["modifyvm", :id, "--cpus", "2"]
      end

      node.vm.provision :chef_solo do |chef|
        chef.cookbooks_path = "cookbooks"
        chef.json = {
          "ntp" => {
            "install" => "true"
          },
          "ndb" => {
            "mgmd" => {
              "private_ips" => [local_ip]
            },
            "ndbd" => {
              "private_ips" => [local_ip]
            },
            "mysqld" => {
              "private_ips" => [local_ip]
            },
            "memcached" => {
              "private_ips" => [local_ip]
            },
            "public_ips" => [local_ip],
            "private_ips" => [local_ip],
            "enabled" => "true",
          },
          "public_ips" => [local_ip],
          "private_ips" => [local_ip],
          "vagrant" => "true",
        }
        chef.add_recipe "kagent::install"
        chef.add_recipe "ndb::install"
        chef.add_recipe "ndb::mgmd"
        chef.add_recipe "ndb::ndbd"
        chef.add_recipe "ndb::mysqld"
      end
    end
  end
end

