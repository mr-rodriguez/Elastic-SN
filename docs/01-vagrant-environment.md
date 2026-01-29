# Install Vagrant
*Before installing and running vagrant boxes, ensure you have the provider you're planning to use correctly installed first (e.g. virtualbox, qemu, libvirtd, etc.)*

I will use virtualbox for this build on my Pop! OS linux PC
- install virtualbox
```bash
sudo apt update && sudo apt install virtualbox
```

Install vagrant
```bash
# download archived keyring(s)
wget -O - https://apt.releases.hashicorp.com/gpg | sudo gpg --dearmor -o /usr/share/keyrings/hashicorp-archive-keyring.gpg

# updating source.d
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com $(grep -oP '(?<=UBUNTU_CODENAME=).*' /etc/os-release || lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/hashicorp.list

# update repo and install vagrant
sudo apt update && sudo apt install vagrant
```

# Creating Vagrantfile
*You can paste this config into your Vagrantfile or modify it how you wish. Change hostnames, ip addresses, memory, etc. as you see fit*
- I'm creating 3 ubuntu VMs
	- elasticsearch node
	- kibana node
	- ubuntu test workstation
```ruby
# -*- mode: ruby -*-
# vi: set ft=ruby :
Vagrant.configure("2") do |config|
  config.vm.define "es01" do |ubuntu|
    ubuntu.vm.box = "ubuntu/focal64"
    ubuntu.vm.network "private_network", ip: "x.x.x.x"
    ubuntu.vm.hostname = "es01"
    ubuntu.vm.provision "shell", inline: <<-SHELL
      echo "vm.max_map_count=262144" | sudo tee -a /etc/sysctl.conf
      sudo sysctl -p
    SHELL
    ubuntu.vm.provider "virtualbox" do |vb|
      vb.memory = "8000"
      vb.cpus = 2
    end
  end
  config.vm.define "kb01" do |ubuntu|
    ubuntu.vm.box = "ubuntu/focal64"
    ubuntu.vm.network "private_network", ip: "x.x.x.x"
    ubuntu.vm.hostname = "kb01"
    ubuntu.vm.provision "shell", inline: <<-SHELL
      echo "vm.max_map_count=262144" | sudo tee -a /etc/sysctl.conf
      sudo sysctl -p
    SHELL
    ubuntu.vm.provider "virtualbox" do |vb|
      vb.memory = "4096"
      vb.cpus = 2
    end
  end
  config.vm.define "ubuntu" do |ubuntu|
    ubuntu.vm.box = "ubuntu/focal64"
    ubuntu.vm.network "private_network", ip: "x.x.x.x"
    ubuntu.vm.hostname = "ubuntu"
    ubuntu.vm.disk :disk, name: "extra_storage", size: "8GB"
    ubuntu.vm.provider "virtualbox" do |vb|
      vb.memory = "2000"
    end
  end
 end
```

# Create file structure and the Vagrantfile
```bash
mkdir -p ~/vagrant/elastic
vim ~/vagrant/elastic/Vagrantfile
cd ~/vagrant/elastic
```

# Running Vagrant boxes
*Before running `vagrant up` for virtualbox, ensure you have your desired ip ranges set in `/etc/vbox/networks.conf`*

Sample config (does not have to be this range)
```text
* 192.168.30.0/24
```

Sample editing `networks.conf`
```bash
sudo echo "* 192.168.30.0/24" >> /etc/vbox/networks.conf
```

Start and run vagrant
- ensure you are in the folder where your `Vagrantfile` is located
```bash
vagrant up
```
Check status
```bash
vagrant status
```
Example output
```text
Current machine states:

elasticsearch01           running (virtualbox)
kibana01                  running (virtualbox)
ulinux                    running (virtualbox)
```

SSH into your vagrant box
```bash
vagrant ssh <machine_name>
```

# KVM errors after running `vagrant up`
*If you get some kind of kvm error, read the error to give you indication of what kvm module you'll need to disable*

Check with this command
```bash
lsmod | grep kvm
```

Commands to temporarily disable kvm modules
```bash
sudo modprobe -r kvm_amd
# or
sudo modprobe -r kvm
```
