# -*- mode: ruby -*-
# vi: set ft=ruby :

# All Vagrant configuration is done below. The "2" in Vagrant.configure
# configures the configuration version (we support older styles for
# backwards compatibility). Please don't change it unless you know what
# you're doing.
Vagrant.configure(2) do |config|
  # The most common configuration options are documented and commented below.
  # For a complete reference, please see the online documentation at
  # https://docs.vagrantup.com.

  # Every Vagrant development environment requires a box. You can search for
  # boxes at https://atlas.hashicorp.com/search.
  config.vm.box = "ubuntu/trusty64"

  # Provision script to install dependencies used by the esp-open-sdk and
  # micropython tools.  First install dependencies as root.
  config.vm.provision "shell", privileged: false, inline: <<-SHELL
    echo "Installing esp-open-sdk, Espressif ESP-IDF, and micropython dependencies..."
    sudo apt-get update
    sudo apt-get install -y build-essential git make unrar-free unzip \
                            autoconf automake libtool gcc g++ gperf \
                            flex bison texinfo gawk ncurses-dev libexpat-dev \
                            python sed libreadline-dev libffi-dev pkg-config \
                            help2man python-dev python-serial wget linux-image-extra-$(uname -r)
    echo "Installing Espressif ESP32 toolchain..."
    cd ~
    wget https://dl.espressif.com/dl/xtensa-esp32-elf-linux64-1.22.0-61-gab8375a-5.2.0.tar.gz 2> /dev/null
    tar xvfz xtensa-esp32-elf-linux64-1.22.0-61-gab8375a-5.2.0.tar.gz
    echo "PATH=/home/vagrant/xtensa-esp32-elf/bin:\$PATH" >> ~/.profile
    echo "Installing esp-open-sdk, Espressif ESP-IDF, and micropython source..."

    git clone --recursive https://github.com/pfalcon/esp-open-sdk.git
    cd esp-open-sdk
    make STANDALONE=y
    echo "PATH=$(pwd)/xtensa-lx106-elf/bin:\$PATH" >> ~/.profile
    source ~/.profile

    cd ..
    git clone --recursive https://github.com/espressif/esp-idf.git
    cd esp-idf
    make all

    cd ..
    git clone --recursive https://github.com/micropython/micropython.git
    cd micropython
    make -C mpy-cross
    make -C esp8266 axtls
    echo "Finished provisioning, now run 'vagrant ssh' to enter the virtual machine."
  SHELL

  # Virtualbox VM configuration.
  config.vm.provider "virtualbox" do |v|
    # Bump the memory allocated to the VM up to 1 gigabyte as the compilation of
    # the esp-open-sdk tools requires more memory to complete.
    v.memory = 1024
    # Enable USB.
    v.customize ["modifyvm", :id, "--usb", "on"]
  end

end
