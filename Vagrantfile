# -*- mode: ruby -*-
# vi: set ft=ruby :

LIBRARY_NAME="Bibliothek"
LIBRARY_PATH=ENV['HOME'] + "/" + LIBRARY_NAME

Vagrant.configure(2) do |config|
  config.vm.box_url = "https://de.mirrors.cicku.me/fedora/linux/releases/41/Cloud/x86_64/images/Fedora-Cloud-Base-Vagrant-libvirt-41-1.4.x86_64.vagrant.libvirt.box"
  config.vm.box = "f41-cloud-libvirt"
  config.vm.provider "qemu" do |domain|
      domain.cpus = 1
      domain.arch = "x86_64"
      domain.machine = "type=q35,accel=hvf"
      domain.graphics_type = "spice"
      domain.memory = 1024
      domain.qemu_dir = "/usr/local/bin/"
      domain.net_device = "virtio-net-pci,netdev=mynet0"
      domain.extra_qemu_args = [ "-vnc", "unix:/tmp/koreader-vnc",
                                 "-vga", "virtio",
                                 "-virtfs", "local,path=#{LIBRARY_PATH},mount_tag=host0,security_model=mapped-xattr,id=host0,fmode=0644,dmode=0755" ]
      domain.other_default = []
  end
  config.vm.provision "shell", inline: "sudo dnf install -y SDL2 bindfs crond"
  config.vm.provision "shell", inline: "mkdir /home/vagrant/#{LIBRARY_NAME}"
  config.vm.provision "shell", inline: "sudo mkdir /mnt/#{LIBRARY_NAME}"
  config.vm.provision "shell", inline: "sudo echo 'host0   /mnt/#{LIBRARY_NAME}    9p      trans=virtio,version=9p2000.L,rw,_netdev   0 0' >> /etc/fstab"
  config.vm.provision "shell", inline: "sudo echo '/mnt/#{LIBRARY_NAME}    /home/vagrant/#{LIBRARY_NAME} fuse.bindfs map=501/1000:@20/@1000,x-systemd.requires=/mnt/#{LIBRARY_NAME},x-systemd.after=/mnt/#{LIBRARY_NAME} 0 0' >> /etc/fstab"
  config.vm.provision "shell", inline: "sudo curl -L https://github.com/koreader/koreader/releases/download/v2024.07/koreader-linux-x86_64-v2024.07.tar.xz -o koreader.tar.gz"
  config.vm.provision "shell", inline: "sudo tar xfv koreader.tar.gz"
  config.vm.provision "shell", inline: "sudo echo '[Unit]
Description=Koreader
After=home-vagrant-#{LIBRARY_NAME}.mount
Requires=home-vagrant-#{LIBRARY_NAME}.mount

[Service]
Type=simple
Environment=XDG_RUNTIME_DIR=/home/vagrant/#{LIBRARY_NAME}
Environment=HOME=/home/vagrant/#{LIBRARY_NAME}
ExecStart=/home/vagrant/bin/koreader /home/vagrant/#{LIBRARY_NAME}
#RemainAfterExit=yes
User=root

[Install]
WantedBy=default.target' > /etc/systemd/system/koreader.service"
  config.vm.provision "shell", inline: "sudo systemctl daemon-reload"
  config.vm.provision "shell", inline: "sudo systemctl enable koreader"
  config.vm.provision "shell", inline: "sudo systemctl enable crond"
  config.vm.provision "shell", inline: "sudo systemctl disable chronyd"
  config.vm.provision "shell", inline: "echo \"*/2 * * * * /sbin/hwclock --hctosys\" | sudo crontab -"
  config.vm.provision "shell", inline: "sudo grubby --update-kernel=/boot/vmlinuz-\$(uname -r) --args=tsc=unstable"
  config.vm.provision "shell", inline: "/sbin/hwclock --hctosys"
  config.vm.provision "shell", inline: "sudo reboot"
end
