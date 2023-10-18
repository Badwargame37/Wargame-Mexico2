Vagrant.configure("2") do |config|
    # Set the box to Ubuntu 20.04
  config.vm.box = "debian/bullseye64"

  # Set the IP address of the server
  config.vm.network "private_network", ip: "192.168.190.17"

  config.vm.provider "virtualbox" do |vb|
    vb.memory = "4096"
    vb.cpus = "4"
  end
  config.vm.synced_folder "./DockerCompose", "/home/vagrant/wargame1"
 
  # Script to provision the virtual machine
  config.vm.provision "shell", inline: <<-SHELL
    # Update the package repository and install dependencies
    sudo apt update
    sudo apt install -y apt-transport-https ca-certificates curl gnupg2 software-properties-common
	  sudo apt install python3-pip -y
	  pip install Flask
    # Install Ansible
    sudo apt update
    sudo apt install -y software-properties-common
    sudo apt-add-repository --yes --update ppa:ansible/ansible
    sudo apt install -y ansible
    sudo apt-get install gnupg -y
    # Install Docker
    curl -fsSL https://download.docker.com/linux/debian/gpg | sudo apt-key add -
    sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/debian $(lsb_release -cs) stable"
    sudo apt update
    sudo apt install -y docker-ce

    # Install Docker Compose
    sudo curl -L "https://github.com/docker/compose/releases/latest/download/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
    sudo chmod +x /usr/local/bin/docker-compose




    # Install GCC (GNU Compiler Collection)
    sudo apt install -y gcc

    # Add your user to the Docker group
    sudo usermod -aG docker ${USER}

    # Start Docker service
    sudo systemctl start docker
    sudo systemctl enable docker
sudo usermod -a ${USER} -G docker
    # Optionally, you can set up a Kubernetes cluster using kubeadm here
	cp /home/vagrant/wargame1/docker-stack /root -r


cd  ${HOME}/docker-stack/
docker network create --driver=bridge --subnet=192.168.191.0/24 pivot_net

docker network create --driver=bridge challenge_net
docker network create --driver=bridge guacamole_net
docker network create --driver=bridge haproxy_net
docker network create --driver=bridge wordpress_internal_net

chmod 644  ${HOME}/docker-stack/wordpress/dump.sql

cd wargame-mexico
docker compose -f ${HOME}/docker-stack/wargame-mexico/DockerCompose/network-config.yml up -d
docker compose -f ${HOME}/docker-stack/haproxy/docker-compose.yml up -d
docker compose -f ${HOME}/docker-stack/guacamole/docker-compose.yml up -d
docker compose -f ${HOME}/docker-stack/wordpress/docker-compose.yml up -d
docker compose -f ${HOME}/docker-stack/wordpress/docker-compose.yml up  db -d
docker compose -f ${HOME}/docker-stack/wargame-mexico/DockerCompose/Reverse/docker-compose.yml up -d
docker compose -f ${HOME}/docker-stack/wargame-mexico/DockerCompose/pivot/docker-compose.yml up -d
docker-compose -f ${HOME}/docker-stack/wargame-mexico/DockerCompose/pivot/docker-compose.yml down
docker-compose -f ${HOME}/docker-stack/wargame-mexico/DockerCompose/pivot/docker-compose.yml up -d

docker-compose -f ${HOME}/docker-stack/wordpress/docker-compose.yml exec db mysql -uroot -p7777777 thecooluncle < /root/docker-stack/wordpress/db.sql

docker compose -f ${HOME}/docker-stack/haproxy/docker-compose.yml ps
docker compose -f ${HOME}/docker-stack/guacamole/docker-compose.yml ps
docker compose -f ${HOME}/docker-stack/wordpress/docker-compose.yml ps
docker compose -f ${HOME}/docker-stack/wargame-mexico/DockerCompose/Reverse/docker-compose.yml ps
docker compose -f ${HOME}/docker-stack/wargame-mexico/DockerCompose/pivot/docker-compose.yml ps
echo "All installations completed."
############################reboot chec service #########
chmod +x ${HOME}/docker-stack/wargame-mexico/Service/script.sh
cp ${HOME}/docker-stack/wargame-mexico/Service/wargame.service /etc/systemd/system/wargame.service
sed -i 's/\r$//' ${HOME}/docker-stack/wordpress/script.sh
bash ${HOME}/docker-stack/wordpress/script.sh

sudo systemctl daemon-reload

sudo systemctl enable wargame.service
sudo systemctl start wargame.service

#create users and priv esc
useradd -m -s /usr/local/bin/rbash_custom Merl1n_Th3_W1z4rd
echo 'Merl1n_Th3_W1z4rd:Excal1bur&St4rDust' | chpasswd

# Allow SSH password authentication
sed -i 's/PasswordAuthentication no/PasswordAuthentication yes/g' /etc/ssh/sshd_config
systemctl restart sshd

  SHELL
end


