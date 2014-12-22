VAGRANTFILE_API_VERSION = "2"

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  config.vm.box = "chef/centos-7.0"
  config.berkshelf.enabled = true
  config.omnibus.chef_version = :latest
  config.cache.scope = :box
  config.cache.auto_detect = true

  Dir.foreach('nodes') do |node|
    if node.index('.json')
      name = node[0, node.index('.json')]
      config.vm.define name do |node_config|
        json = JSON.parse(Pathname(__FILE__).dirname.join('nodes', node).read)
        vagrantJson = json.delete('vagrant')
        node_config.dns.tld = "dev"
        node_config.dns.patterns = vagrantJson['dns']
        node_config.vm.host_name = name
        node_config.vm.network :private_network, :ip => vagrantJson['ip']
        node_config.vm.provider :virtualbox do |vb|
          vb.customize [
                           'modifyvm', :id,
                           '--memory', vagrantJson['memory'],
                           '--cpus', vagrantJson['cpus'],
                       ]
        end
        node_config.vm.provision :chef_solo do |chef|
          chef.cookbooks_path = ["site-cookbooks", "cookbooks"]
          chef.roles_path = "roles"
          chef.data_bags_path = "data_bags"
          chef.provisioning_path = "/tmp/vagrant-chef"
          chef.run_list = json.delete('run_list')
          chef.json = json
        end
      end
    end
  end
end

