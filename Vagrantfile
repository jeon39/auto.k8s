Vagrant.configure("2") do |config|
  common = <<-SCRIPT
    if ! grep -q ansible /etc/hosts; then sudo echo "192.168.2.21    ansible" >> /etc/hosts; fi
    if ! grep -q node01 /etc/hosts; then sudo echo "192.168.2.22    node01" >> /etc/hosts; fi
    if ! grep -q node02 /etc/hosts; then sudo echo "192.168.2.23    node02" >> /etc/hosts; fi
    if ! grep -q node03 /etc/hosts; then sudo echo "192.168.2.24    node03" >> /etc/hosts; fi
    if ! grep -q node04 /etc/hosts; then sudo echo "192.168.2.25    node04" >> /etc/hosts; fi
    if ! id vagrant >  /dev/null 2>&1 ; then sudo useradd -G 10 -m -p $(openssl passwd -1 vagrant) vagrant; fi
    if [ ! -f /etc/sudoers.d/vagrant ] ; then sudo echo "vagrant    ALL=(ALL)    NOPASSWD: ALL" > /etc/sudoers.d/vagrant ; fi
  
    sudo apt-get -y install vim tree net-tools telnet git python3-pip
    sudo echo "autocmd filetype yaml setlocal ai ts=2 sw=2 et" > /home/devops/.vimrc
    sudo ufw status
    sudo ufw disable
  SCRIPT

  controlz_setup = <<-SCRIPT
    sudo apt-get install -y ansible
    KUB_PATH=/home/vagrant/kubespray
  
    if [ ! -f "$KUB_PATH/ansible.cfg" ]; then git clone https://github.com/kubernetes-sigs/kubespray.git; fi
    if ! grep -q remote_user /home/vagrant/kubespray/ansible.cfg; then LINE=$(grep -n defaults /home/vagrant/kubespray/ansible.cfg | cut -d: -f1); echo 'LINE===>'$LINE; sudo perl -p -i -e '$.=='$LINE+1' and print "remote_user = vagrant\nprivate_key_file = /home/vagrant/.ssh/id_rsa\n"' /home/vagrant/kubespray/ansible.cfg; echo "[privilege_escalation]" >> $KUB_PATH/ansible.cfg; echo "become=true" >> $KUB_PATH/ansible.cfg; echo "become_method=sudo" >> $KUB_PATH/ansible.cfg; echo "become_user=root" >> $KUB_PATH/ansible.cfg; echo "become_ask_pass=false" >> $KUB_PATH/ansible.cfg; cd $KUB_PATH && pip3 install -r requirements.txt; fi
  SCRIPT

  nodes_setup = <<-SCRIPT
    sed -i -e 's/PasswordAuthentication no/PasswordAuthentication yes/g' /etc/ssh/sshd_config
    sed -i -e 's/#PubkeyAuthentication yes/PubkeyAuthentication yes/g' /etc/ssh/sshd_config
    service ssh restart
  SCRIPT

  ubuntu_gui_setup = <<-SCRIPT
    # Google Chrome Repository
    wget -q -O - https://dl-ssl.google.com/linux/linux_signing_key.pub|sudo apt-key add -
    sudo sh -c 'echo \"deb [arch=amd64] http://dl.google.com/linux/chrome/deb/ stable main\" > /etc/apt/sources.list.d/google.list'
    sudo apt update -y
    sudo apt upgrade -y
  
    # Install ubuntu-desktop
    sudo apt install -y --no-install-recommends ubuntu-desktop
  
    # Install Chrome
    sudo apt install -y google-chrome-stable

    # Install Chromium
    # sudo install -y chromium-browser

    # Install Firefox
    # sudo install -y firefox

    # Jpaness support
    # sudo apt install -y fcitx-mozc
    # sudo apt install -y fonts-noto
  SCRIPT

  node_restart = <<-SCRIPT
    sudo shutdown -r now
  SCRIPT

	config.vm.box = "ubuntu/bionic64"

	config.vm.define "node01" do |node1|
		node1.vm.hostname = "node01"
		node1.vm.network "private_network", ip: "192.168.2.22"
		node1.vm.provider "virtualbox" do |v|
			v.customize [ "modifyvm", :id, "--cpus", "2" ]
			v.customize [ "modifyvm", :id, "--memory", "4096" ]
		end
                node1.vm.provision:shell, :inline => nodes_setup
                node1.vm.provision:shell, :inline => ubuntu_gui_setup
                node1.vm.provision:shell, :inline => node_restart
	end
	config.vm.define "node02" do |node2|
		node2.vm.hostname = "node02"
		node2.vm.network "private_network", ip: "192.168.2.23"
		node2.vm.provider "virtualbox" do |v|
			v.customize [ "modifyvm", :id, "--cpus", "2" ]
			v.customize [ "modifyvm", :id, "--memory", "4096" ]
		end
		node2.vm.provision:shell, :inline => nodes_setup
	end
        config.vm.define "node03" do |node3|
                node3.vm.hostname = "node03"
                node3.vm.network "private_network", ip: "192.168.2.24"
                node3.vm.provider "virtualbox" do |v|
                        v.customize [ "modifyvm", :id, "--cpus", "2" ]
                        v.customize [ "modifyvm", :id, "--memory", "4096" ]
                end
                node3.vm.provision:shell, :inline => nodes_setup
        end
        config.vm.define "node04" do |node4|
                node4.vm.hostname = "node04"
                node4.vm.network "private_network", ip: "192.168.2.25"
                node4.vm.provider "virtualbox" do |v|
                        v.customize [ "modifyvm", :id, "--cpus", "2" ]
                        v.customize [ "modifyvm", :id, "--memory", "4096" ]
                end
                node4.vm.provision:shell, :inline => nodes_setup
        end
        config.vm.define "ansible" do |ansible|
                ansible.vm.hostname = "ansible"
                ansible.vm.network "private_network", ip: "192.168.2.21"
                ansible.vm.provision:shell, :inline => common
                ansible.vm.provision:shell, :inline => controlz_setup
                ansible.vm.provision:file, source: "ansible-control-play.yml", destination: "ansible-control-play.yml"
                ansible.vm.provision:shell, inline: "ansible-playbook -i /home/vagrant/kubespray/inventory/my-k8s/hosts ansible-control-play.yml"
                ansible.vm.provision:file, source: "ansible-all-play.yml", destination: "ansible-all-play.yml"
                ansible.vm.provision:shell, inline: "ansible-playbook -i /home/vagrant/kubespray/inventory/my-k8s/hosts ansible-all-play.yml"
                ansible.vm.provision:file, source: "k8s-control-play.yml", destination: "k8s-control-play.yml"
                ansible.vm.provision:shell, inline: "ansible-playbook -i /home/vagrant/kubespray/inventory/my-k8s/hosts k8s-control-play.yml"
                ansible.vm.provision:file, source: "k8s-all-play.yml", destination: "k8s-all-play.yml"
                ansible.vm.provision:shell, inline: "cd ./kubespray && ansible-playbook -i ./inventory/my-k8s/hosts ../k8s-all-play.yml"
                ansible.vm.provision:shell, inline: "cd /home/vagrant/kubespray && ansible-playbook --flush-cache -i /home/vagrant/kubespray/inventory/my-k8s/hosts cluster.yml -v"
                ansible.vm.provision:file, source: "k8s-kubectl-play.yml", destination: "k8s-kubectl-play.yml"
                ansible.vm.provision:shell, inline: "cd ./kubespray && ansible-playbook -i ./inventory/my-k8s/hosts ../k8s-kubectl-play.yml"
        end

end
