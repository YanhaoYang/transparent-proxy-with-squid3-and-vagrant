# -*- mode: ruby -*-
# vi: set ft=ruby :
Vagrant.configure("2") do |config|
  config.vm.box = "bento/ubuntu-16.04"

  config.vm.provider "parallels" do |prl|
    prl.memory = 512
    prl.cpus = 1
  end

  config.vm.provider :virtualbox do |vb|
    vb.gui = false
    vb.memory = 512
    vb.cpus = 1
  end

  config.vm.define "squid" do |squid|
    squid.vm.hostname = "squid"
    squid.vm.network :private_network, ip: "172.28.128.200"

    squid.vm.provision "file", source: "files/squid.conf", destination: "/tmp/squid.conf"
    squid.vm.provision "file", source: "files/iptables.rules", destination: "/tmp/iptables.rules"
    squid.vm.provision "file", source: "files/restore-iptables",
      destination: "/tmp/restore-iptables"

    squid.vm.provision "shell", inline: <<-SHELL
      apt-get update
      apt-get install -y squid3
      mv /etc/squid3/squid.conf /etc/squid3/squid.conf.bak
      mv /tmp/squid.conf /etc/squid3/
      chown root:root /etc/squid3/
      restart squid3

      mv /tmp/iptables.rules /etc/iptables.rules
      chown root:root /etc/iptables.rules
      mv /tmp/restore-iptables /etc/network/if-up.d/restore-iptables
      chown root:root /etc/network/if-up.d/restore-iptables
      iptables-restore < /etc/iptables.rules
      echo 1 > /proc/sys/net/ipv4/ip_forward
    SHELL
  end

  config.vm.define "client" do |client|
    client.vm.hostname = "client"
    client.vm.network :private_network, ip: "172.28.128.202"

    client.vm.provision "file", source: "files/change-gateway", destination: "/tmp/change-gateway"
    client.vm.provision "shell", inline: <<-SHELL
      mv /tmp/change-gateway /usr/local/bin/change-gateway
      echo "dns-nameservers 8.8.8.8" >> /etc/network/interfaces
      /usr/local/bin/change-gateway
    SHELL
  end
end
