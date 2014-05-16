VAGRANTFILE_API_VERSION = '2'
NETWORK_FILE_SYSTEM = false

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
    config.vm.box = 'ubuntu/trusty64'
    config.vm.box_url = 'http://cloud-images.ubuntu.com/vagrant/trusty/current/trusty-server-cloudimg-amd64-vagrant-disk1.box'

    config.vm.network :public_network
    config.vm.network :forwarded_port, guest: 22, host: 2200

    config.vm.provider :virtualbox do |virtualbox|
      virtualbox.memory = 4096
      virtualbox.cpus = 2
    end

    config.vm.provision :ansible do |ansible|
        ansible.sudo = true
        ansible.extra_vars = { ansible_ssh_user: 'vagrant' }
        ansible.playbook = 'provisioning/localhost.yml'
        ansible.inventory_path = 'provisioning/localhost_vagrant'
    end
end
