# -*- mode: ruby -*-
# vi: set ft=ruby :

REQURED_PLUGINS = %w{
  vagrant-vbguest
  vagrant-omnibus
  vagrant-berkshelf
  vagrant-cachier
}
unless REQURED_PLUGINS.all? {|e| Vagrant.has_plugin?(e) }
  raise "This Vagrantfile requires the following plugins to be installed: #{REQURED_PLUGINS * ' '}"
end

PROJECT_RUBY_VERSION = File.read(".ruby-version") rescue "2.2.3"

Vagrant.configure(2) do |config|
  config.vm.box = "bento/centos-7.1"

  # config.vm.network "forwarded_port", guest: 80, host: 8080
  # config.vm.network "private_network", ip: "192.168.33.10"
  # config.vm.network "public_network"

  config.vm.provider "virtualbox" do |vb|
    vb.memory = "1024"
    vb.cpus = 2
  end

  # config.cache.scope = :box if Vagrant.has_plugin?("vagrant-cachier")
  config.berkshelf.enabled = true if Vagrant.has_plugin?("vagrant-berkshelf")
  config.omnibus.chef_version = :latest if Vagrant.has_plugin?("vagrant-omnibus")

  config.vm.provision :chef_solo do |chef|
    chef.json = {
      rbenv: {
        user_installs: [
          {
            user: 'vagrant',
            rubies: [
              {
                name: PROJECT_RUBY_VERSION,
                environment: { 'CFLAGS' => '-march=native -O2 -pipe', 'MAKEFLAGS' => '-j2' }
              }
            ],
            global: PROJECT_RUBY_VERSION,
            gems: {
              PROJECT_RUBY_VERSION => [
                { name: 'bundler' },
                { name: 'rake' }
              ]
            },
            plugins: [
              {
                name: "ruby-build",
                git_url: "https://github.com/sstephenson/ruby-build.git",
              },
              {
                name: "rbenv-vars",
                git_url: "https://github.com/sstephenson/rbenv-vars.git",
              },
              {
                name: "rbenv-gem-rehash",
                git_url: "https://github.com/sstephenson/rbenv-gem-rehash.git"
              }
            ]
          }
        ]
      },
      postgresql: {
        password: {
          # Required for installing postgresql server when not using chef server
          # Actual password: vagrant
          # To generate: echo -n 'vagrant''postgres' | openssl md5 | sed -e 's/.* /md5/'
          postgres: "md5100cf32c205cbe9f41af3c738d12a4ee"
        }
      },
    }

    [
      "ruby_rbenv::user",
      "ruby_build", # required by ruby_rbenv
      "nodejs",
      "vim",
      "git",
      "postgresql::client",
      "postgresql::ruby",
    ].each { |e| chef.add_recipe e }
  end
end
