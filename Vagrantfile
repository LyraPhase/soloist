#!/usr/bin/env ruby

Vagrant.configure("2") do |config|
  ssh_key = File.read(File.expand_path("~/.ssh/identity.lyra.pub"))

  config.vm.box = "ubuntu/focal64"
  config.vm.provider :virtualbox do |p|
    p.name = "ubuntu-focal64"
  end
  config.vm.network 'private_network', ip: "192.168.6.66"

  config.vm.provision 'shell', inline: 'test -d /etc/skel/.ssh || mkdir /etc/skel/.ssh'
  config.vm.provision 'shell' do |shell|
    shell.inline = "echo $@ | tee /etc/skel/.ssh/authorized_keys"
    shell.args = ssh_key
  end

  config.vm.provision 'shell', inline: 'curl -sSL https://rvm.io/mpapis.asc | sudo gpg --import -'
  config.vm.provision 'shell', inline: 'curl -sSL https://rvm.io/pkuczynski.asc | sudo gpg --import -'

  config.vm.provision 'shell' do |shell|
    shell.path = File.expand_path('../script/bootstrap.sh', __FILE__)
    shell.args = `whoami`.chomp
  end
  config.vm.provision 'shell', inline: "bash -lc 'rvm use --install --default ruby-3.0.3'"
  config.vm.provision 'shell', inline: "bash -lc 'gem install chef --no-document'"
end
