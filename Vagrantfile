Vagrant.configure("2") do |config|
  config.vm.box = "ubuntu/bionic64"
config.vm.provider "virtualbox" do |vb|
  vb.memory = "2048"
end
config.vm.provision "build",
  type: "shell", 
  privileged: false, 
  inline: <<-SHELL
  sudo apt-get update
  sudo apt-get -y upgrade
  sudo apt-get -y install make unrar-free autoconf automake libtool gcc g++ gperf \
    flex bison texinfo gawk ncurses-dev libexpat-dev python-dev python python-serial \
    sed git unzip bash help2man wget bzip2 libtool-bin

  git clone --recursive https://github.com/pfalcon/esp-open-sdk.git
  cd esp-open-sdk/
  make STANDALONE=y
  echo "export PATH=$PWD/xtensa-lx106-elf/bin:$PATH" >> ~/.bashrc
  export PATH=$PWD/xtensa-lx106-elf/bin:$PATH

  cd ~
  git clone https://github.com/micropython/micropython.git
  cd micropython
  git submodule update --init
  make -C mpy-cross
SHELL
config.vm.provision "update",
  type: "shell", 
  privileged: false, 
  inline: <<-SHELL
  cd ~
  cd micropython
  git pull
  cd ports/esp8266
  sudo chown -r vagrant /vagrant/modules
  cp /vagrant/modules/* ./modules
  make clean all
  make

SHELL
end
