BOX_NAME = ENV['VAGRANT_BOX_NAME'] || 'bento/centos-7.2'

$num_instances = 4
$instance_name_prefix = "nebula"
$vm_gui = false
$vm_memory = 512
$vm_cpus = 1
$forwarded_ports = { 9869 => 9869, 29876 => 29876 }
# Example $forwarded_ports = { 8080 => 8080, 9000 => 9000, 6080 => 6080, 6500 => 8500}
#Size of attached disk space in GB
$attached_disk_space = 10

def vm_gui
  $vb_gui.nil? ? $vm_gui : $vb_gui
end

def vm_memory
  $vb_memory.nil? ? $vm_memory : $vb_memory
end

def vm_cpus
  $vb_cpus.nil? ? $vm_cpus : $vb_cpus
end


Vagrant.configure(2) do |config|
  config.vm.box = BOX_NAME

  $num_instances.downto(1) do |i|
    config.vm.define vm_name = "%s%02d" % [$instance_name_prefix, i] do |config|
      config.vm.hostname = vm_name


      $forwarded_ports.each do |guest, host|
        config.vm.network "forwarded_port", guest: guest, host: host, auto_correct: true
      end

      config.ssh.insert_key = false
      config.ssh.private_key_path = ["keys/insecure_private_key"]
      config.vm.provision "file", source: "keys/insecure_private_key", destination: "~/.ssh/id_rsa"

      ext_ip = "192.168.33.#{i+10}"
      config.vm.network :private_network, ip: ext_ip, nic_type: "virtio"

      config.vm.provider :virtualbox do |vb|
        vb.gui = vm_gui
        vb.memory = vm_memory
        vb.cpus = vm_cpus
        vb.customize ["modifyvm", :id, "--nicpromisc2", "allow-all"]
      end

       vagrant_root = File.dirname(File.expand_path(__FILE__))
         config.vm.provider "virtualbox" do |vb|
           file_to_disk = File.join(vagrant_root, ".vagrant/%s_ceph_osd.vdi" % vm_name)
           unless File.exist?(file_to_disk)
           vb.customize ['createhd', '--filename', file_to_disk, '--size', $attached_disk_space * 1024]
         end
         vb.customize ['storageattach', :id, '--storagectl', 'SATA Controller', '--port', 1, '--device', 0, '--type', 'hdd', '--medium', file_to_disk]
	   end

        config.vm.provision "shell", inline: <<-SHELL
          cp /vagrant/playbooks/files/dot_ssh/config .ssh/
          cat .ssh/authorized_keys > .ssh/id_rsa.pub
          chmod 600 .ssh/id_rsa
          chmod 600 .ssh/config
          chown -R vagrant .ssh/
        SHELL

      if i == 1
        config.vm.provision "shell", inline: <<-SHELL
          sudo yum -y install python-setuptools
          sudo easy_install pip
          sudo yum -y install python-devel
          sudo yum -y install gcc
          sudo pip install ansible --upgrade
          sudo -u vagrant ansible-playbook -i /vagrant/inventory/3x1 --extra-vars "deploy_username=vagrant" /vagrant/playbooks/one_start.yml
        SHELL
      end
    end
  end
end
