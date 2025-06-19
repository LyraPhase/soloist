#!/usr/bin/env ruby

Vagrant.configure("2") do |config|
  #ssh_key = File.read(File.expand_path("~/.ssh/identity.lyra.pub"))

  config.vm.box = "bento/ubuntu-22.04"
  # config.vm.provider :virtualbox do |p|
  config.vm.provider :libvirt do |libvirt|
    libvirt.driver = 'kvm' if File.exist?('/dev/kvm')
    libvirt.uri = 'qemu:///system'
    libvirt.system_uri = 'qemu:///system'
    # libvirt.uri = 'qemu+tls://saturn.internal/system'
    # libvirt.system_uri = 'qemu+tls://saturn.internal/system'
    libvirt.memory = '1024'
    libvirt.disk_driver_opts = { cache:'writeback', io:'threads', discard:'unmap', detect_zeroes:'unmap' }
    # UEFI boot w/Tianocore edk2
    libvirt.loader = '/usr/share/edk2/x64/OVMF_CODE.4m.fd'
    libvirt.nvram_template = '/usr/share/edk2/x64/OVMF_VARS.4m.fd'
    # p.name = "ubuntu-22-04"
  end
  config.vm.network :private_network, :type => 'dhcp', :autostart => true
  config.vm.synced_folder ".", "/vagrant", type: 'nfs', nfs_version: 4, nfs_udp: false, nfs_export: true
  config.vm.communicator = 'ssh'

  config.vm.provision 'shell', inline: 'test -d /etc/skel/.ssh || mkdir /etc/skel/.ssh'
  #config.vm.provision 'shell' do |shell|
  #  shell.inline = "echo $@ | tee /etc/skel/.ssh/authorized_keys"
  #  shell.args = ssh_key
  #end

  config.vm.provision 'shell' do |shell|
    shell.path = File.expand_path('../script/bootstrap.sh', __FILE__)
    shell.args = '$SUDO_USER'
  end
  # install .ruby-version @ .ruby-gemset
  ruby_version = File.open('.ruby-version', 'r').read.chomp
  ruby_gemset = File.open('.ruby-gemset', 'r').read.chomp
  config.vm.provision 'shell', inline: "bash -lc 'rvm use --install --default ruby-#{ruby_version}; rvm gemset create #{ruby_gemset}'", privileged: false
  config.vm.provision 'shell', inline: "bash -lc 'cd /vagrant/ && gem install \"bundler:$(grep -A 1 \"BUNDLED WITH\" Gemfile.lock | tail -n 1)\"'", privileged: false

  # Use separate bundler config inside Vagrant VM
  config.vm.provision 'shell' do |shell|
    shell.path = File.expand_path('../script/ci-bundle-config.sh', __FILE__)
    shell.args = '$SUDO_USER'
  end

  # Bundle install as user via rvm
  config.vm.provision 'shell', inline: "bash -lc 'cd /vagrant/ && bundle config set --local frozen true'", privileged: false
  config.vm.provision 'shell', inline: "bash -lc 'cd /vagrant/ && bundle install'", privileged: false
  # accept + persist chef license accept for non-interactive CI
  config.vm.provision 'shell', inline: "bash -lc 'cd /vagrant/ && rvmsudo bundle exec chef-solo --chef-license accept --local-mode --no-listen --why-run'", privileged: false
  # Run the ci script
  config.vm.provision 'shell', inline: "bash -lc 'cd /vagrant/ && ./script/ci.sh'", privileged: false
  # Install no-op test fixture cookbook ckbk
  # NOTE: Librarian::Chef::Cli.new.install() suffers from a race condition
  #       Thus, cookbook files may still be in the process of writing
  #       while soloist tries to access them
  # So, we must manually run it first... redundantly
  config.vm.provision 'shell', inline: "bash -lc 'cd /vagrant/test/fixtures && bundle exec berks install'", privileged: false
  # Run soloist integration test against fixtures
  config.vm.provision 'shell', inline: "bash -lc 'cd /vagrant/test/fixtures && bundle exec soloist'", privileged: false
end
