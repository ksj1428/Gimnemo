# -*- mode: ruby -*-
# vi: set ft=ruby :
Vagrant.configure("2") do |config|
  config.vm.box = "ubuntu/focal64"

  config.vm.define "docker" do |docker|
    docker.vm.hostname = "docker-server"
    docker.vm.provider "virtualbox" do |vb|
      vb.name = "docker-server"
      vb.cpus = 2
      vb.memory = 8192
    end
    docker.vm.network "private_network", ip: "192.168.33.200"
    docker.vm.provision "shell", inline: <<-SCRIPT
     sudo useradd -m -s /bin/bash sshfs
     echo sshfs:qwe@123 | sudo chpasswd
    SCRIPT
  end

  config.vm.define "sshfs" do |sshfs|
    sshfs.vm.hostname = "sshfs-server"
    sshfs.vm.provider "virtualbox" do |vb|
      vb.name = "sshfs-server"
      vb.cpus = 1
      vb.memory = 4096
    end
    sshfs.vm.network "forwarded_port", guest: 8080, host: 80
    sshfs.vm.network "private_network", ip: "192.168.33.250"
    ubuntu.vm.provision "shell", inline: <<-SCRIPT
      sudo apt-get update -y
      sudo apt-get install -y ca-certificates curl gnupg
      sudo install -m 0755 -d /etc/apt/keyrings
      curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
      sudo chmod a+r /etc/apt/keyrings/docker.gpg
      echo \
        "deb [arch="$(dpkg --print-architecture)" signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
        "$(. /etc/os-release && echo "$VERSION_CODENAME")" stable" | \
        sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
      sudo apt-get update -y
      sudo apt-get install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
      sudo usermod -a -G docker vagrant
      docker login -u gimnemo --password-stdin < /vagrant/env/docker_token


      docker volume create --label service=web --label creater=nemo web_src_vol
      sudo cp -r /vagrant/htdocs/* /var/lib/docker/volumes/web_src_vol/_data/
      docker run -d -p 8080:80 -v web_src_vol:/usr/share/nginx/html --name web-server nginx
    SCRIPT
  end
end
