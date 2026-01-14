Vagrant.configure("2") do |config|
  # Используем указанный образ
  config.vm.box = "generic/ubuntu2204"

  # Настройка сети: проброс 80 порта гостя на 8080 хоста
  config.vm.network "forwarded_port", guest: 80, host: 8080, host_ip: "127.0.0.1"

  # Настройки провайдера VirtualBox
  config.vm.provider "virtualbox" do |vb|
    # Настройка оперативной памяти
    vb.memory = "1024"

    # Создание и подключение первого дополнительного диска (1 ГБ)
    file_to_disk1 = 'disk1.vdi'
    unless File.exist?(file_to_disk1)
      vb.customize ['createhd', '--filename', file_to_disk1, '--size', 1024]
    end
    vb.customize ['storageattach', :id, '--storagectl', 'SATA Controller', '--port', 1, '--device', 0, '--type', 'hdd', '--medium', file_to_disk1]

    # Создание и подключение второго дополнительного диска (1 ГБ)
    file_to_disk2 = 'disk2.vdi'
    unless File.exist?(file_to_disk2)
      vb.customize ['createhd', '--filename', file_to_disk2, '--size', 1024]
    end
    vb.customize ['storageattach', :id, '--storagectl', 'SATA Controller', '--port', 2, '--device', 0, '--type', 'hdd', '--medium', file_to_disk2]
  end

  # Провижининг: разметка, форматирование и монтирование
  config.vm.provision "shell", inline: <<-SHELL
    set -e

    # Функция для подготовки диска
    # На Ubuntu в VirtualBox доп. диски обычно определяются как /dev/sdb, /dev/sdc и т.д.
    prepare_disk() {
      DEVICE=$1
      MOUNT_POINT=$2

      if [ -b "$DEVICE" ]; then
        echo "Processing $DEVICE..."
        # Форматирование в ext4
        mkfs.ext4 -F "$DEVICE"
        
        # Создание точки монтирования
        mkdir -p "$MOUNT_POINT"
        
        # Монтирование
        mount "$DEVICE" "$MOUNT_POINT"
        
        # Добавление в /etc/fstab по UUID для надежности
        UUID=$(blkid -s UUID -o value "$DEVICE")
        echo "UUID=$UUID $MOUNT_POINT ext4 defaults 0 2" >> /etc/fstab
        echo "Disk $DEVICE mounted to $MOUNT_POINT and added to fstab."
      else
        echo "Device $DEVICE not found!"
      fi
    }

    # В данной конфигурации:
    # /dev/sda - системный
    # /dev/sdb - диск 1
    # /dev/sdc - диск 2
    
    prepare_disk "/dev/sdb" "/mnt/disk1"
    prepare_disk "/dev/sdc" "/mnt/disk2"
  SHELL
end