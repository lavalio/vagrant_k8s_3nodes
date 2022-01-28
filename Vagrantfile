
ENV['VAGRANT_NO_PARALLEL'] = 'yes'
IMAGE = "ubuntu/impish64"


Vagrant.configure(2) do |config|

  config.vm.provision "shell", inline: <<-SHELL
    apt-get update -y
    echo "192.168.56.10  master-node" >> /etc/hosts
    echo "192.168.56.11  worker-node01" >> /etc/hosts
    echo "192.168.56.12  worker-node02" >> /etc/hosts
  SHELL

  # Kubernetes Master Server
  config.vm.define "master" do |node|
 
    node.vm.box = IMAGE
    node.vm.box_check_update = false
    node.vm.hostname = "master-node"
    node.vm.network "private_network", ip: "192.168.56.10"
    node.vm.network "public_network", ip: "192.168.1.100", :bridge => "eno1"

    # Config namespace: config.ssh
    # Make it as a regular ssh-login
    node.ssh.insert_key = false
    node.ssh.private_key_path = ['~/.vagrant.d/insecure_private_key', '~/.ssh/id_rsa']
    node.vm.provision "file", source: "~/.ssh/id_rsa.pub", destination: "~/.ssh/authorized_keys"
    
    node.vm.provision "shell", inline: <<-EOC
      sudo sed -i -e "\\#PasswordAuthentication yes# s#PasswordAuthentication yes#PasswordAuthentication no#g" /etc/ssh/sshd_config
      sudo systemctl restart sshd.service
      echo "finished"
    EOC

    
    node.vm.provider "virtualbox" do |v|
      v.memory = 2048
      v.cpus = 2
    end

    #install kubernetes in master node
    node.vm.provision "shell", path: "scripts/tools.sh"
    node.vm.provision "shell", path: "scripts/master.sh"
  
  end


  # Kubernetes Worker Nodes
  NodeCount = 2

  (1..NodeCount).each do |i|

    config.vm.define "node0#{i}" do |node|

      node.vm.box = IMAGE
      node.vm.box_check_update = false
   
      node.vm.hostname = "worker-node0#{i}"
      node.vm.network "private_network", ip: "192.168.56.1#{i}"
      node.vm.network "public_network", ip: "192.168.1.10#{i}", :bridge => "eno1"

      # Config namespace: config.ssh
      node.ssh.insert_key = false 
      node.ssh.private_key_path = ['~/.vagrant.d/insecure_private_key', '~/.ssh/id_rsa'] 
      node.vm.provision "file", source: "~/.ssh/id_rsa.pub", destination: "~/.ssh/authorized_keys" 
      
      node.vm.provision "shell", inline: <<-EOC
        sudo sed -i -e "\\#PasswordAuthentication yes# s#PasswordAuthentication yes#PasswordAuthentication no#g" /etc/ssh/sshd_config
        sudo systemctl restart sshd.service
        echo "finished"
      EOC
 
      node.vm.provider "virtualbox" do |v|
        v.memory = 2048
        v.cpus = 2
      end

      #install kubernetes in worker node
      node.vm.provision "shell", path: "scripts/tools.sh"
      node.vm.provision "shell", path: "scripts/worker.sh"
            
    end

  end

end