# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|

  config.vm.define "linux" do |linux|
    linux.vm.box = 'OEL7'
    linux.vm.box_url = 'http://view.artifacts.ttldev/artifactory/box-internal-master/packer/versioned/OEL7.json'
    #linux.vm.box_url = 'http://view.artifacts.ttldev/artifactory/box-internal-master/packer/versioned/P-OEL7.json'
  end

  config.vm.provider "virtualbox" do |v|
    v.gui = true
  end

  go = <<-EOF
    set -e
    # Install Go
    export GOPATH=/opt/go
    export PATH=$PATH:$GOPATH/bin

    echo "Installing Go $GOVERSION"
    if [ ! $(which go) ]; then
        echo "    Configuring GOPATH"
        sudo mkdir -p $GOPATH/src $GOPATH/bin $GOPATH/pkg
        sudo chown -R vagrant $GOPATH

        echo "    Configuring env vars"
        echo "export PATH=\$PATH:$GOPATH/bin" | sudo tee /etc/profile.d/golang.sh > /dev/null
        echo "export GOROOT=$GOROOT" | sudo tee --append /etc/profile.d/golang.sh > /dev/null
        echo "export GOPATH=$GOPATH" | sudo tee --append /etc/profile.d/golang.sh > /dev/null
    fi

    # Install drone
    echo "Building Drone"
    cd $GOPATH/src/github.com/drone/drone
    make deps
    chown -R vagrant:vagrant /opt/go

    # Auto cd to drone install dir
    echo "cd /opt/go/src/github.com/drone/drone" >> /home/vagrant/.bashrc
  EOF


  # Sync this repo into what will be $GOPATH
  config.vm.synced_folder ".", "/opt/go/src/github.com/drone/drone"

  config.vm.network :forwarded_port, guest: 8080, host: 8081, auto_correct: true

  config.vm.provision "shell", inline: <<-EOF
    curl -sL https://rpm.nodesource.com/setup | bash -
#      sudo yum install gcc gcc-c++ kernel-devel make bzr git mercurial vim zip ruby ruby-devel nodejs -y
    sudo yum install gcc gcc-c++ kernel-devel make bzr git mercurial vim zip ruby nodejs -y
    sudo yum install --enablerepo=ol7_addons docker -y
    sudo systemctl enable docker
    sudo systemctl start docker
    sudo yum install -y epel-release
    # EPEL key import
    sudo yum install -y golang golang-pkg-windows-amd64

  EOF
  
  config.vm.provision "shell", inline: go
end
