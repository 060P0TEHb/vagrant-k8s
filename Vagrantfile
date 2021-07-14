# -*- mode: ruby -*-
# vim: set ft=ruby :

IMAGE_NAME = "debian/contrib-buster64"
N = 2
home = ENV['HOME']

Vagrant.configure("2") do |config|
    config.ssh.insert_key = false
    config.vm.provision "shell", inline: "swapoff -a"
    config.vm.synced_folder ".", "/vagrant", create: true
    #   v.customize [ "modifyvm", :id, "--uartmode1", "disconnected" ]
      
    config.vm.define "k8s-master" do |master|
        master.vm.box = IMAGE_NAME
        master.vm.provider "virtualbox" do |vm|
            vm.memory = 4098
            vm.cpus = 2        
        end
        master.vm.network "private_network", ip: "192.168.50.10"
        master.vm.hostname = "k8s-master"
        master.vm.provision "ansible" do |ansible|
          config.vm.provision "shell", inline: "swapoff -a"
            ansible.playbook = "ansible/kube-initial.yml"
            ansible.extra_vars = {
              node_ip: "192.168.50.10",
            }
        end
    end

    (1..N).each do |i|
        config.vm.define "k8s-node-#{i}" do |node|
            node.vm.box = IMAGE_NAME
            node.vm.provider "virtualbox" do |vm|
                vm.memory = 4098
                vm.cpus = 4
            end
            node.vm.network "private_network", ip: "192.168.50.#{i + 10}"
            node.vm.hostname = "k8s-node-#{i}"
            node.vm.provision "ansible" do |ansible|
                ansible.playbook = "ansible/node-playbook.yml"
                ansible.extra_vars = {
                  node_ip: "192.168.50.#{i + 10}",
                }
            end

            node.vm.provider "virtualbox" do |vb|
                needsController = false
                unless File.exist?(home + "/VirtualBox VMs/disks/sata1-k8s#{i}.vdi")
                  vb.customize ['createhd', '--filename', home + "/VirtualBox VMs/disks/sata1-k8s#{i}.vdi", '--size', 30720]
                  needsController = true
                end
                if needsController == true
                    vb.customize ['storageattach', :id, '--storagectl', 'SATA Controller', '--port', '2', '--device', 0, '--type', 'hdd', '--medium', home + "/VirtualBox VMs/disks/sata1-k8s#{i}.vdi"]
                end
            end
        end
    end
end
