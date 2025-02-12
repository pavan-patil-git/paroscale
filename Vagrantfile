Vagrant.configure("2") do |config|
  config.vm.box = "ubuntu/bionic64"     # Base box for both nodes

  config.vm.define "node1" do |node|    # Define NFS Server node1
    node.vm.hostname = "node1"
    node.vm.network "private_network", type: "static", ip: "192.168.56.101"

    node.vm.provision "shell", inline: <<-SHELL
      sudo apt update   echo "Setting up NFS Server on node1"
      sudo apt install -y nfs-kernel-server
      sudo systemctl enable nfs-kernel-server

      sudo mkdir -p /srv/nfs_shared_drive       # Create NFS share directory
      sudo chown nobody:nogroup /srv/nfs_shared_drive
      sudo chmod 777 /srv/nfs_shared_drive      # Allow all access for simplicity

      echo "/srv/nfs_shared_drive 192.168.56.102(rw,sync,no_root_squash,no_subtree_check)" | sudo tee /etc/exports      # Configure NFS exports

      sudo exportfs -rav        # Restart NFS service
      sudo systemctl restart nfs-kernel-server
    SHELL
  end

  config.vm.define "node2" do |node|    # Define NFS Server node2
    node.vm.hostname = "node2"
    node.vm.network "private_network", type: "static", ip: "192.168.56.102"

    node.vm.provision "shell", inline: <<-SHELL
      echo "Setting up NFS Client on node2"
      sudo apt update
      sudo apt install -y nfs-common
      sudo systemctl enable nfs-common

      sudo mkdir -p /mnt/nfs_shared_drive     # Create mount point

      sudo mount 192.168.56.101:/srv/nfs_shared_drive /mnt/nfs_shared_drive     # Mount NFS share from node1

      df -h | grep nfs  # Verify mount

      echo "192.168.56.101:/srv/nfs_shared_drive /mnt/nfs_shared_drive nfs defaults 0 0" | sudo tee -a /etc/fstab       # Persist NFS mount across reboots
    SHELL
  end
end
