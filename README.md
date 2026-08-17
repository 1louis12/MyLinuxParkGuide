# This is my guidance for making a simple manageble linux machine Parc. 
# Note for Vagrant
[Vagrant Documentation](https://developer.hashicorp.com/vagrant/docs/vagrantfile) 

## Install Vagrant.
### MacOs 
```
brew tap hashicorp/tap
brew install hashicorp/tap/hashicorp-vagrant
```
### Linux  Ubuntu/Debian 

```
wget -O - https://apt.releases.hashicorp.com/gpg | sudo gpg --dearmor -o /usr/share/keyrings/hashicorp-archive-keyring.gpg
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com $(grep -oP '(?<=UBUNTU_CODENAME=).*' /etc/os-release || lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/hashicorp.list
sudo apt update && sudo apt install vagrant

```
## Get TinyOS (Not really use)

### Os option : 
* debian/bookworm64
```
config.vm.box = "debian/bookworm64"
```

* TinyCoreLinux
[Download Tiny OS](http://tinycorelinux.net/downloads.html)

#### Note for arm64 use
* ubuntu/focal64
```
config.vm.box = "ubuntu/focal64"
```

# Trouble Shoot 
- Allow the hypervisor access my local network
- ```v.gui = true``` forcing permission In VmwareOption
- Default login and password   *** vagrant ***
- choose [os](https://portal.cloud.hashicorp.com/vagrant/discover/bento/ubuntu-26.04)  right for Ip addr simple config (to avoir ==> serveur-central: Starting the VMware VM...
==> serveur-central: Waiting for the VM to receive an address...)
Sinon 
```
sudo /Applications/VMware\ Fusion.app/Contents/Library/vmnet-cli --stop
sudo /Applications/VMware\ Fusion.app/Contents/Library/vmnet-cli --configure
sudo /Applications/VMware\ Fusion.app/Contents/Library/vmnet-cli --start 
```

# Vagrant & Ansible
[doc]('https://developer.hashicorp.com/vagrant/docs/provisioning/ansible_intro')
## 