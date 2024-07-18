Vagrant.configure("2") do |config|

  ###################################################################################
  config.vm.define "openbao-client" do |node|

    # which image to use
    node.vm.box = "opensuse/Leap-15.6.x86_64"

    # sizing of the VMs
    node.vm.provider "libvirt" do |lv|
      lv.random_hostname = true
      lv.memory = 1024
      lv.cpus = 2
    end

    # set the hostname
    node.vm.hostname = "openbao-client"

    # disable shared folders
    node.vm.synced_folder ".", "/vagrant", disabled: true

  end # config.vm.define agents

  ###################################################################################
  config.vm.define "openbao" do |node|

    # which image to use
    node.vm.box = "opensuse/Leap-15.6.x86_64"

    # sizing of the VMs
    node.vm.provider "libvirt" do |lv|
      lv.random_hostname = false
      lv.memory = 8196
      lv.cpus = 2
    end

    # set the hostname
    node.vm.hostname = "openbao"

    # disable shared folders
    node.vm.synced_folder ".", "/vagrant", disabled: true

    # Ansible
    node.vm.provision "ansible" do |ansible|
      ansible.compatibility_mode = "2.0"
          ansible.limit = "all"
      ansible.playbook = "ansible/playbook-vagrant.yml"
    end # node.vm.provision

    node.trigger.after :destroy do |trigger|
      trigger.warn = "Removing ansible/openbao_root_token"
      trigger.run = {inline: "rm -vf ansible/openbao_root_token"}
    end

  end # config.vm.define

end # Vagrant.configure
