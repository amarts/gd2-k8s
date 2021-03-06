# -*- mode: ruby -*-
# vi: set ft=ruby :

driveletters = ('a'..'z').to_a

Vagrant.configure("2") do |config|
  config.vm.synced_folder ".", "/vagrant", disabled: true
  config.vm.synced_folder ".", "/home/vagrant/sync", disabled: true

  config.vm.box = "centos/atomic-host"
  (1..3).each do |i|
    config.vm.define "kube#{i}" do |ca|
      ca.vm.hostname = "kube#{i}"

      ca.vm.provider :libvirt do |lv|
        lv.cpus = 2
        lv.memory = 2048

        (1..3).each do |d|
          lv.storage :file, :device => "vd#{driveletters[d]}", :size => '1024G'
        end
      end
      if i == 3
        ca.vm.provision :ansible do |ansible|
          ansible.playbook = "vagrant-playbook.yml"
          ansible.become = true
          ansible.limit = "all"
          ansible.groups = {
            "etcd" => ["kube[1:3]"],
            "kube-master" => ["kube[1:2]"],
            "kube-node" => ["kube[1:3]"],
            "k8s-cluster:children" => ["kube-master", "kube-node"]
          }
        end
      end
    end
  end
end
