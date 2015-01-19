# -*- mode: ruby -*-
# vi: set ft=ruby :

$nat = <<SCRIPT
sudo apt-get -qq -y install iptables
echo "1" > /proc/sys/net/ipv4/ip_forward
iptables -t nat -A POSTROUTING -p all -s 172.20.1.2 -j SNAT --to-source 4.4.4.4
iptables -t nat -A PREROUTING -p all -d 4.4.4.4 -j DNAT --to-destination 172.20.1.2
iptables -t nat -A POSTROUTING -p all -s 172.22.1.2 -j SNAT --to-source 5.5.5.5
iptables -t nat -A PREROUTING -p all -d 5.5.5.5 -j DNAT --to-destination 172.22.1.2
iptables -A FORWARD -i eth1 -o eth2
iptables -A FORWARD -i eth2 -o eth1
ip route add 4.4.4.4 dev eth1
ip route add 5.5.5.5 dev eth2
arp -s 4.4.4.4 08:00:27:11:e1:4b
arp -s 5.5.5.5 08:00:27:69:ec:8d
SCRIPT

$consul = <<SCRIPT
sudo apt-get install -qq -y unzip
cd /usr/bin
wget --quiet https://dl.bintray.com/mitchellh/consul/0.4.1_linux_amd64.zip
unzip *.zip
rm *.zip
SCRIPT

Vagrant.configure(2) do |config|
  config.vm.box = "socketplane/ubuntu-14.10"
  
  config.vm.define "c1"  do |c1|
    c1.vm.network "private_network", ip: "172.20.1.2", mac: "08002711e14b" #public 4.4.4.4
    c1.vm.provision "shell", inline: "ip route add 5.5.5.5 via 172.20.1.254"
    c1.vm.provision "shell", inline: $consul
  end
  config.vm.define "c2"  do |c2|
    c2.vm.network "private_network", ip: "172.20.1.3"
    c2.vm.provision "shell", inline: $consul
  end 
  config.vm.define "c3"  do |c3|
    c3.vm.network "private_network", ip: "172.22.1.2", mac: "08002769ec8d" #public 5.5.5.5
    c3.vm.provision "shell", inline: "ip route add 4.4.4.4 via 172.22.1.254"
    c3.vm.provision "shell", inline: $consul
  end 
  config.vm.define "r1"  do |r1|
    r1.vm.network "private_network", ip: "172.20.1.254"
    r1.vm.network "private_network", ip: "172.22.1.254"
    r1.vm.provision "shell", inline: $nat
  end 
end
