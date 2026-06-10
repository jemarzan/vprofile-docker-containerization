# -*- mode: ruby -*-
# vi: set ft=ruby :

# -------------------------------------------------------
# Vprofile Stack — Auto-Provisioned Vagrant Machine
# Pulls images from Docker Hub and starts all services
# Author: Jem
# -------------------------------------------------------

Vagrant.configure("2") do |config|

  config.vm.box = "ubuntu/jammy64"

  # Private network — access app at http://192.168.56.16
  config.vm.network "private_network", ip: "192.168.56.16"

  # Forward port 80 → host so you can also use http://localhost
  config.vm.network "forwarded_port", guest: 80, host: 80

  # VirtualBox settings
  config.vm.provider "virtualbox" do |vb|
    vb.name   = "vprofile-docker"
    vb.memory = "2048"
    vb.cpus   = 2
  end

  # Sync the project folder so docker-compose.yml is available inside the VM
  config.vm.synced_folder ".", "/vagrant"

  # -------------------------------------------------------
  # Provisioning — runs once on first `vagrant up`
  # Re-run anytime with: vagrant provision
  # -------------------------------------------------------
  config.vm.provision "shell", inline: <<-SHELL

    echo "========================================"
    echo " Step 1: Install Docker"
    echo "========================================"
    sudo apt update -y
    sudo apt install -y ca-certificates curl

    sudo install -m 0755 -d /etc/apt/keyrings
    sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg \
      -o /etc/apt/keyrings/docker.asc
    sudo chmod a+r /etc/apt/keyrings/docker.asc

    sudo tee /etc/apt/sources.list.d/docker.sources <<EOF
Types: deb
URIs: https://download.docker.com/linux/ubuntu
Suites: $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}")
Components: stable
Architectures: $(dpkg --print-architecture)
Signed-By: /etc/apt/keyrings/docker.asc
EOF

    sudo apt update -y
    sudo apt install -y \
      docker-ce \
      docker-ce-cli \
      containerd.io \
      docker-buildx-plugin \
      docker-compose-plugin

    # Allow vagrant user to run docker without sudo
    sudo usermod -aG docker vagrant
    sudo systemctl enable docker
    sudo systemctl start docker

    echo "========================================"
    echo " Step 2: Pull Images from Docker Hub"
    echo "========================================"
    sudo docker pull jemdevops/vprofiledb:latest
    sudo docker pull jemdevops/vprofileapp:latest
    sudo docker pull jemdevops/vprofileweb:latest
    sudo docker pull memcached
    sudo docker pull rabbitmq

    echo "========================================"
    echo " Step 3: Start the Vprofile Stack"
    echo "========================================"
    cd /vagrant
    sudo docker compose up -d

    echo "========================================"
    echo " Done! Vprofile is running."
    echo " App:  http://192.168.56.16"
    echo " App:  http://localhost"
    echo "========================================"

  SHELL

end
