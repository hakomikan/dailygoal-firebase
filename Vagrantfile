Vagrant::configure("2") do |config|
    config.vm.provider "virtualbox" do |v|
        v.linked_clone = true
        v.customize ["modifyvm", :id, "--paravirtprovider", "kvm"]
        v.memory = 4096
    end

    config.vm.hostname = "vagrant"
    config.vm.box = "ubuntu/trusty64"
    config.vm.synced_folder ".", "/home/vagrant/app"

    config.vm.network "private_network", ip: "192.168.33.10"

    # default port
    config.vm.network "forwarded_port", guest: 3000, host: 3000
    # debug port
    config.vm.network "forwarded_port", guest: 9229, host: 9229

    config.vm.provision "install tools",
        type: "shell",
        privileged: false,
        inline: <<-SHELL
            export DEBIAN_FRONTEND=noninteractive

            sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv EA312927
            echo "deb http://repo.mongodb.org/apt/ubuntu trusty/mongodb-org/3.2 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-3.2.list
            curl -sL https://deb.nodesource.com/setup_7.x | sudo -E bash -

            curl -sS https://dl.yarnpkg.com/debian/pubkey.gpg | sudo apt-key add -
            echo "deb https://dl.yarnpkg.com/debian/ stable main" | sudo tee /etc/apt/sources.list.d/yarn.list
            
            sudo apt-get update
            
            sudo apt-get install -y yarn
            sudo apt-get install -y wget curl 

            sudo apt-get install -y git zsh whois nodejs emacs lv bindfs
            sudo apt-get install -y mongodb-org=3.2.9 mongodb-org-server=3.2.9 mongodb-org-shell=3.2.9 mongodb-org-mongos=3.2.9 mongodb-org-tools=3.2.9
            wget -O- https://toolbelt.heroku.com/install-ubuntu.sh | sh

            wget -O- https://bootstrap.pypa.io/get-pip.py | sudo python
            sudo pip install percol
        SHELL

    config.vm.provision "export env",
        type: "shell",
        run: "always",
        privileged: true,
        inline: <<-SHELL
            echo "# vagrant script for every boot" > /etc/profile.d/vagrant.sh
            echo export CLIENT_SECRET=#{ENV['CLIENT_SECRET']} >> /etc/profile.d/vagrant.sh 
            echo export CLIENT_ID=#{ENV['CLIENT_ID']} >> /etc/profile.d/vagrant.sh            
            chmod +x /etc/profile.d/vagrant.sh
        SHELL

    config.vm.provision "bind directories",
        type: "shell",
        run: "always",
        privileged: false,
        inline: <<-SHELL
            mkdir -p ~/tmp_node_modules
            mkdir -p ~/app/node_modules
            mkdir -p ~/tmp_dist
            mkdir -p ~/app/dist
            sudo bindfs /home/vagrant/app /home/vagrant/app -p 0755 -u vagrant -o nonempty
            # sudo bindfs /home/vagrant/tmp_node_modules /home/vagrant/app/node_modules -p 0755 -u vagrant -o nonempty
            sudo bindfs /home/vagrant/tmp_dist /home/vagrant/app/dist -p 0755 -u vagrant -o nonempty
        SHELL
    
    if ENV['USE_DOTFILES'] == "1" then
        config.vm.synced_folder ENV['DOTFILES_DIR'], "/home/vagrant/dotfiles"
        config.vm.provision "setup dotfiles",
            type: "shell",
            privileged: false,
            inline: <<-SHELL
                script=/home/vagrant/dotfiles/#{ENV['DOTFILES_SETUPSCRIPT_NAME']}
                echo dotfiles source dir: #{ENV['DOTFILES_DIR']}
                if [ -f $script ]; then
                    echo setup dotfiles by $script
                    $script
                else
                    echo $script is not found
                fi
            SHELL
    end
end
