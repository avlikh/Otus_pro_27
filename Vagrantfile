Vagrant.configure("2") do |config|
  # Base VM OS configuration.
  config.vm.box = "debian/bookworm64"
  config.vm.provider :virtualbox do |v|
    v.memory = 2048
    v.cpus = 2
  end

  # Define two VMs with static private IP addresses.
  boxes = [
    { :name => "backup",
      :ip => "192.168.56.11",
    },
    { :name => "client",
      :ip => "192.168.56.12",
    }
  ]

  # Provision each of the VMs.
  boxes.each do |opts|
    config.vm.define opts[:name] do |box|
      box.vm.hostname = opts[:name]
      box.vm.network "private_network", ip: opts[:ip]

      # Если это клиент - добавляем запуск shell перед ansible
      if opts[:name] == "client"
        box.vm.provision "shell", inline: <<-SHELL
          echo "Выполняем необходимые настройки перед Ansible"
          sudo apt update && sudo apt install -y python3 python3-pip
          echo "Готово!"
        SHELL

        box.vm.provision "ansible" do |ansible|
          ansible.playbook = "provision.yaml"
          ansible.inventory_path = "hosts.ini"
          ansible.host_key_checking = "false"
          ansible.become = "true"
          ansible.limit = "all"
        end
      end
    end
  end

  # Настройки backup
  config.vm.define "backup" do |server|
    # Создание диска
    server.vm.disk :disk, name: "backup", size: "2GB"
    # Инициализация диска
    server.vm.provision "shell", inline: <<-SHELL
      sudo apt update && apt install parted -y
      sudo parted /dev/sdb --script mklabel gpt
      sudo parted /dev/sdb --script mkpart primary ext4 0% 100%
      sudo mkfs.ext4 /dev/sdb1
      sudo mkdir -p /var/backup
      echo '/dev/sdb1 /var/backup ext4 defaults 0 0' | sudo tee -a /etc/fstab
      sudo mount -a
    SHELL
  end
end
