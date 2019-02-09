# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.require_version ">= 1.6.0"

boxes = [
    {
        :name => "kafka1",
        :eth1 => "11.11.11.21",
        :mem => "2048",
        :cpu => "2"
    },
    {
        :name => "kafka2",
        :eth1 => "11.11.11.22",
        :mem => "2048",
        :cpu => "2"
    },
    {
        :name => "kafka3",
        :eth1 => "11.11.11.23",
        :mem => "2048",
        :cpu => "2"
    }
]

Vagrant.configure(2) do |config|

  config.vm.box = "bento/centos-7.3"
  boxes.each do |opts|
    config.vm.define opts[:name] do |config|
      config.vm.hostname = opts[:name]
      config.vm.provider "vmware_fusion" do |v|
        v.vmx["memsize"] = opts[:mem]
        v.vmx["numvcpus"] = opts[:cpu]
      end
      config.vm.provider "virtualbox" do |v|
        v.customize ["modifyvm", :id, "--memory", opts[:mem]]
        v.customize ["modifyvm", :id, "--cpus", opts[:cpu]]
      end
        config.vm.network :private_network, ip: opts[:eth1]
     
    end
  end
end
