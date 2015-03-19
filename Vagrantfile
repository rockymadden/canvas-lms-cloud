VAGRANTFILE_API_VERSION = '2'
NETWORK_FILE_SYSTEM = false

box = 'ubuntu/trusty64'
box_url = 'http://cloud-images.ubuntu.com/vagrant/trusty/current/trusty-server-cloudimg-amd64-vagrant-disk1.box'

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
    config.vm.box = box
    config.vm.box_url = box_url

    config.vm.network :private_network, ip: '10.0.10.10'

    config.vm.provider :virtualbox do |virtualbox|
        virtualbox.memory = 4096
        virtualbox.cpus = 2
    end

    config.vm.provision :shell, inline: "echo 'Waiting 30 seconds for hosts to catch up...' && sleep 30"

    config.vm.provision :ansible do |ansible|
        ansible.limit = 'all'
        ansible.sudo = true
        ansible.extra_vars = { ansible_ssh_user: 'vagrant' }
        ansible.playbook = 'ansible/util_copy_ssh_id.yml'
        ansible.inventory_path = 'ansible/localhost_vagrant'
        ansible.raw_ssh_args = ['-o StrictHostKeyChecking=no', '-o UserKnownHostsFile=/dev/null']
    end

    config.vm.provision :shell, inline: "echo 'Waiting 30 seconds for hosts to catch up...' && sleep 30"

    config.vm.provision :ansible do |ansible|
        ansible.limit = 'all'
        ansible.playbook = 'ansible/localhost.yml'
        ansible.inventory_path = 'ansible/localhost_vagrant'
        ansible.raw_ssh_args = ['-o StrictHostKeyChecking=no', '-o UserKnownHostsFile=/dev/null']
    end

    config.vm.provision :shell, inline: "echo 'Waiting 30 seconds for hosts to catch up...' && sleep 30"

    config.vm.provision :ansible do |ansible|
        ansible.limit = 'all'
        ansible.playbook = 'ansible/util_initialize_rdb_localhost.yml'
        ansible.inventory_path = 'ansible/localhost_vagrant'
        ansible.raw_ssh_args = ['-o StrictHostKeyChecking=no', '-o UserKnownHostsFile=/dev/null']
    end

    config.vm.provision :shell, inline: "echo 'Waiting 30 seconds for hosts to catch up...' && sleep 30"

    config.vm.provision :ansible do |ansible|
        ansible.limit = 'all'
        ansible.playbook = 'ansible/util_initialize_canvas.yml'
        ansible.inventory_path = 'ansible/localhost_vagrant'
        ansible.raw_ssh_args = ['-o StrictHostKeyChecking=no', '-o UserKnownHostsFile=/dev/null']
    end

    config.vm.provision :shell, inline: "echo 'Waiting 30 seconds for hosts to catch up...' && sleep 30"

    config.vm.provision :ansible do |ansible|
        ansible.limit = 'all'
        ansible.playbook = 'ansible/util_test_install.yml'
        ansible.inventory_path = 'ansible/localhost_vagrant'
        ansible.raw_ssh_args = ['-o StrictHostKeyChecking=no', '-o UserKnownHostsFile=/dev/null']
    end
end
