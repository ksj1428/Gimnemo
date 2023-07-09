# -*- mode: ruby -*-
# vi: set ft=ruby :
Vagrant.configure("2") do |config|
  config.vm.box = "ubuntu/focal64"

  config.vm.define "sshfs" do |sshfs|
    sshfs.vm.hostname = "sshfs-server"
    sshfs.vm.provider "virtualbox" do |vb|
      vb.name = "sshfs-server"
      vb.cpus = 1
      vb.memory = 2048
    end
    sshfs.vm.network "private_network", ip: "192.168.33.250"
    sshfs.vm.provision "shell", inline: <<-SCRIPT
      sudo useradd -m -s /bin/bash sshfs
      echo "sshfs:qwe@123" | sudo chpasswd
      git clone https://github.com/ksj1428/static-web-template.git
      sudo mkdir /home/sshfs/htdocs
      sudo cp -r static-web-template/* /home/sshfs/htdocs/
      sudo sed -i 's/PasswordAuthentication no/PasswordAuthentication yes/g' \
        /etc/ssh/sshd_config
      sudo systemctl restart sshd
    SCRIPT
  end
  config.vm.define "docker" do |docker|
    docker.vm.hostname = "docker-server"
    docker.vm.provider "virtualbox" do |vb|
      vb.name = "docker-server"
      vb.cpus = 4
      vb.memory = 8192
    end
    docker.vm.network "forwarded_port", guest: 80, host: 80
    docker.vm.network "private_network", ip: "192.168.33.200"
    docker.vm.provision "shell", inline: <<-SCRIPT
      sudo apt-get update -yqq
      sudo apt-get install -yqq ca-certificates curl gnupg 
      sudo install -m 0755 -d /etc/apt/keyrings
      curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
      sudo chmod a+r /etc/apt/keyrings/docker.gpg
      echo "deb [arch="$(dpkg --print-architecture)" signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
        "$(. /etc/os-release && echo "$VERSION_CODENAME")" stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
      sudo apt-get update -yqq
      sudo apt-get install -yqq docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
      sudo usermod -a -G docker vagrant
      docker login -u gimnemo --password-stdin < /vagrant/env/docker_token
      docker pull httpd
      docker plugin install --grant-all-permissions vieux/sshfs
      docker volume create --driver vieux/sshfs \
        -o sshcmd=sshfs@192.168.33.250:/home/sshfs/htdocs \
        -o password=qwe@123 -o allow_other sshfs_vol1
      docker run -d -p 80:80 -v sshfs_vol1:/usr/local/apache2/htdocs --name web-server httpd
    SCRIPT
  end
end
  