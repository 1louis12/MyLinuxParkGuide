
Vagrant.configure("2") do |config|
  #box for ox
  #config.vm.box = "bento/almalinux-10"
  config.vm.box = "bento/ubuntu-26.04"
  config.vm.box_architecture = "arm64"
  #Virtual Network

  machines = {
    "serveur-central" => "192.168.56.10",
    "client-User1"  => "192.168.56.11",
    "client-User2"  => "192.168.56.12"
  }

  machines.each do |nom, ip|
    config.vm.define nom do |m|
      m.vm.hostname = nom
      m.vm.network "private_network", ip: ip

      m.vm.provider "vmware_desktop" do |v|
        v.gui = true
        v.vmx["displayname"] = nom
        v.vmx["memsize"] = "512"
        v.vmx["numvcpus"] = "1"
      end
    end
  end
  # add provisioner
  config.vm.provision "ansible" do |ansible|
    ansible.playbook = "ansible/playbook.yml"
    ansible.groups = {
          "clients" => ["client-User1", "client-User2"],
          "serveur-central" => ["serveur-central"]
        }
  end

end
