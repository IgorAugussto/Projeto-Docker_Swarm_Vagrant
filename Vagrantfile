Vagrant.configure("2") do |config|

  # Box base (Ubuntu leve)
  config.vm.box = "ubuntu/focal64"

  # Desabilita pasta sincronizada (evita lentidão)
  config.vm.synced_folder ".", "/vagrant"

  # =========================
  # MASTER NODE
  # =========================
  config.vm.define "master" do |master|
    master.vm.hostname = "master"
    master.vm.network "private_network", ip: "192.168.56.10"

    master.vm.provider "virtualbox" do |vb|
      vb.memory = 2048
      vb.cpus = 2
    end

    master.vm.provision "shell", inline: <<-SHELL
      apt-get update
      apt-get install -y docker.io

      systemctl enable docker
      systemctl start docker

      # Inicializa o Swarm
      docker swarm init --advertise-addr 192.168.56.10

      # Salva token para workers
      docker swarm join-token worker -q > /vagrant/token.txt
    SHELL
  end

  # =========================
  # WORKER NODES
  # =========================

  nodes = [
    { name: "node01", ip: "192.168.56.11" },
    { name: "node02", ip: "192.168.56.12" },
    { name: "node03", ip: "192.168.56.13" }
  ]

  nodes.each do |node|
    config.vm.define node[:name] do |n|
      n.vm.hostname = node[:name]
      n.vm.network "private_network", ip: node[:ip]

      n.vm.provider "virtualbox" do |vb|
        vb.memory = 1024
        vb.cpus = 1
      end

      n.vm.provision "shell", inline: <<-SHELL
        apt-get update
        apt-get install -y docker.io

        systemctl enable docker
        systemctl start docker

        # Aguarda o token do master
        while [ ! -f /vagrant/token.txt ]; do
          sleep 5
        done

        TOKEN=$(cat /vagrant/token.txt)

        # Entra no cluster como worker
        docker swarm join --token $TOKEN 192.168.56.10:2377
      SHELL
    end
  end

end