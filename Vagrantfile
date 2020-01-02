Vagrant.configure("2") do |config|
  config.vm.box = "ubuntu/bionic64"
config.vm.provider "virtualbox" do |vb|
  vb.memory = "2048"
end
config.vm.provision "build",
  type: "shell", 
  privileged: false, 
  inline: <<-SHELL
  # Update the OS.
  sudo apt-get update
  sudo apt-get -y upgrade

  # Install needed applications
  sudo apt-get -y install make unrar-free autoconf automake libtool gcc g++ gperf \
    flex bison texinfo gawk ncurses-dev libexpat-dev python-dev python python-serial \
    sed git unzip bash help2man wget bzip2 libtool-bin python3-pip cmake
  
  # setup alternatives for pyhon and install needed apps. 
  sudo update-alternatives --install /usr/bin/python python /usr/bin/python3 1
  sudo pip3 install pyserial 'pyparsing<2.4'
  
  #get and install the esp sdk for esp8266
  git clone --recursive https://github.com/pfalcon/esp-open-sdk.git
  cd esp-open-sdk/
  make STANDALONE=y
  echo "export PATH=/home/vagrant/esp-open-sdk/xtensa-lx106-elf/bin:$PATH" >> ~/.bashrc
  export PATH=/home/vagrant/esp-open-sdk/xtensa-lx106-elf/bin:$PATH

  # Install the esp-idf
  cd ~
  git clone https://github.com/espressif/esp-idf.git
  cd esp-idf
  git checkout 6ccb4cf5b7d1fdddb8c2492f9cbc926abaf230df
  git submodule update --init --recursive

  # install the esp32 build tools. 
  cd ~
  wget https://dl.espressif.com/dl/xtensa-esp32-elf-linux64-1.22.0-80-g6c4433a-5.2.0.tar.gz
  mkdir -p ~/esp
  cd ~/esp
  tar -xzf ~/xtensa-esp32-elf-linux64-1.22.0-80-g6c4433a-5.2.0.tar.gz

  # Clone the micropython and setup the cross compiler. 
  cd ~
  git clone https://github.com/micropython/micropython.git
  cd micropython
  git submodule update --init
  make -C mpy-cross

SHELL
config.vm.provision "update",
  run: 'always',
  type: "shell", 
  privileged: false, 
  inline: <<-SHELL
  # Delete existin bin files
  rm -f /vagrant/*.bin
  #set variables
  export PATH=~/esp/xtensa-esp32-elf/bin:/home/vagrant/esp-open-sdk/xtensa-lx106-elf/bin:$PATH
  export ESPIDF=/home/vagrant/esp-idf
  
  # update micropython
  cd ~
  cd micropython
  git pull

  #compile esp8266
  cd ports/esp8266
  sudo chown -r vagrant /vagrant/modules
  cp /vagrant/modules/* ./modules
  make clean all
  make
  cp build-GENERIC/firmware-combined.bin /vagrant/micropython-esp8266.bin
  
  #compile esp32. 
  cd ~/micropython/ports/esp32
  cp /vagrant/modules/* ./modules
  make clean all
  make
  cp build-GENERIC/firmware.bin /vagrant/micropython-esp32.bin
  

SHELL
end
