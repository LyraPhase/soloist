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

  config.vm.provision 'shell' do |shell|
    shell.path = File.expand_path('../script/bootstrap.sh', __FILE__)
    shell.args = '$SUDO_USER'
  end
  # install .ruby-version @ .ruby-gemset
  ruby_version = File.open('.ruby-version', 'r').read.chomp
  ruby_gemset = File.open('.ruby-gemset', 'r').read.chomp
  config.vm.provision 'shell', inline: "bash -lc 'rvm use --install --default ruby-#{ruby_version}; rvm gemset create #{ruby_gemset}'"
  # Bundle install as user via rvmsudo
  config.vm.provision 'shell', inline: "bash -lc 'cd /vagrant/ && rvmsudo bundle install'", privileged: false
  # accept + persist chef license accept for non-interactive CI
  config.vm.provision 'shell', inline: "bash -lc 'cd /vagrant/ && rvmsudo bundle exec chef-solo --chef-license accept --local-mode --no-listen --why-run'", privileged: false
  # Run the ci script
  config.vm.provision 'shell', inline: "bash -lc 'cd /vagrant/ && ./script/ci.sh'", privileged: false
  # Run soloist integration test against fixtures
  config.vm.provision 'shell', inline: "bash -lc 'cd /vagrant/test/fixtures && bundle exec soloist'", privileged: false
end
