Vagrant.configure("2") do |config|
        config.vm.define "ubuntu1" do | ubuntu1 |
          ubuntu1.vm.hostname = "ubuntu1"
          ubuntu1.vm.box = "ubuntu/xenial64"
          ubuntu1.vm.network "private_network", ip: "192.168.50.11"
          ubuntu1.vm.network :forwarded_port, guest: 22, host: 2211, host_ip: "localhost", id: "ssh", auto_correct: true
# Enable ssh forward agent
          ubuntu1.ssh.forward_agent = true
          ubuntu1.vm.provision "shell", inline: <<-SHELL
                        #setenforce permissive
                        sudo apt-get update -y
                        sudo sed -i 's/PasswordAuthentication no/PasswordAuthentication yes/g' /etc/ssh/sshd_config
                        sudo systemctl restart sshd
           SHELL
        end

        config.vm.define "ubuntu2" do | ubuntu2 |
          ubuntu2.vm.hostname = "ubuntu2"
          ubuntu2.vm.box = "ubuntu/xenial64"
          ubuntu2.vm.network "private_network", ip: "192.168.50.12"
          ubuntu2.vm.network :forwarded_port, guest: 22, host: 2212, host_ip: "localhost", id: "ssh", auto_correct: true
          ubuntu2.ssh.forward_agent = true
          ubuntu2.vm.provision "shell", inline: <<-SHELL
                        #setenforce permissive
                        sudo apt-get update -y
                        sudo sed -i 's/PasswordAuthentication no/PasswordAuthentication yes/g' /etc/ssh/sshd_config
                        sudo systemctl restart sshd
           SHELL
        end

        config.vm.define "ubuntu3" do | ubuntu3 |
          ubuntu3.vm.hostname = "ubuntu3"
          ubuntu3.vm.box = "ubuntu/xenial64"
          ubuntu3.vm.network "private_network", ip: "192.168.50.13"
          ubuntu3.vm.network :forwarded_port, guest: 22, host: 2213, host_ip: "localhost", id: "ssh", auto_correct: true
          ubuntu3.ssh.forward_agent = true
          ubuntu3.vm.provision "shell", inline: <<-SHELL
                        #setenforce permissive
                        sudo apt-get update -y
                        sudo sed -i 's/PasswordAuthentication no/PasswordAuthentication yes/g' /etc/ssh/sshd_config
                        sudo systemctl restart sshd
           SHELL
        end


         config.vm.define "ubuntuc" do | ubuntuc |
          ubuntuc.vm.hostname = "ubuntuc"
          ubuntuc.vm.box = "ubuntu/xenial64"
          ubuntuc.vm.network "private_network", ip: "192.168.50.254"
          ubuntuc.vm.network :forwarded_port, guest: 22, host: 2254, host_ip: "localhost", id: "ssh", auto_correct: true
          ubuntuc.ssh.forward_agent = true
          ubuntuc.vm.provision "shell", inline: <<-SHELL
                        #setenforce permissive
                        sudo apt-get update -y
                        sudo sed -i 's/PasswordAuthentication no/PasswordAuthentication yes/g' /etc/ssh/sshd_config
                        sudo systemctl restart sshd
           SHELL
        end
end
