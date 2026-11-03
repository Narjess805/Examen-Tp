Vagrant.configure("2") do |config|
  config.vm.box = "ubuntu/focal64"
  config.vm.box_check_update = false

  # ---------- Noeud de controle (demarre en premier) ----------
  config.vm.define "ansible-control" do |c|
    c.vm.hostname = "control-node"
    c.vm.network "private_network", ip: "192.168.56.10"
    c.vm.provider "virtualbox" do |vb|
      vb.name   = "ansible-control"
      vb.memory = 1024
      vb.cpus   = 1
    end

    c.vm.provision "shell", inline: <<-SHELL
      set -e
      export DEBIAN_FRONTEND=noninteractive
      apt-get update -y
      apt-get install -y ansible

      # Generation de la cle SSH (une seule fois)
      if [ ! -f /home/vagrant/.ssh/id_rsa ]; then
        sudo -u vagrant ssh-keygen -t rsa -b 2048 -N "" -f /home/vagrant/.ssh/id_rsa
      fi
      # Distribution de la cle publique via le dossier partage
      mkdir -p /vagrant/keys
      cp /home/vagrant/.ssh/id_rsa.pub /vagrant/keys/id_rsa.pub
    SHELL
  end

  # ---------- Noeuds manages ----------
  nodes = {
    "web1" => ["webserver-01", "192.168.56.11"],
    "web2" => ["webserver-02", "192.168.56.12"],
    "db1"  => ["dbserver-01",  "192.168.56.20"]
  }

  nodes.each do |name, (host, ip)|
    config.vm.define name do |n|
      n.vm.hostname = host
      n.vm.network "private_network", ip: ip
      n.vm.provider "virtualbox" do |vb|
        vb.name   = name
        vb.memory = (name == "db1" ? 1024 : 512)
        vb.cpus   = 1
      end

      n.vm.provision "shell", privileged: false, inline: <<-SHELL
        mkdir -p ~/.ssh && chmod 700 ~/.ssh
        touch ~/.ssh/authorized_keys
        KEY="$(cat /vagrant/keys/id_rsa.pub)"
        grep -qF "$KEY" ~/.ssh/authorized_keys || echo "$KEY" >> ~/.ssh/authorized_keys
        chmod 600 ~/.ssh/authorized_keys
      SHELL
    end
  end
end