# -*- mode: ruby -*-
# vi: set ft=ruby :

# All Vagrant configuration is done below. The "2" in Vagrant.configure
# configures the configuration version (we support older styles for
# backwards compatibility). Please don't change it unless you know what
# you're doing.
Vagrant.configure(2) do |config|
  # The most common configuration options are documented and commented below.
  # For a complete reference, please see the online documentation at
  # https://docs.vagrantup.com.

  # do NOT automatically update guest additions
  # (if you have installed vagrant-vbguest from https://github.com/dotless-de/vagrant-vbguest) 
  # we will build the default ubuntu versions in order to enable
  # virtualbox guest addition features like screen resize & cut/paste
#  config.vbguest.auto_update = false

  # Every Vagrant development environment requires a box. You can search for
  # boxes at https://atlas.hashicorp.com/search.
  config.vm.box = "http://speechkitchen.org/boxes/mario-kaldi.box"
  config.ssh.forward_x11 = true

  # Disable automatic box update checking. If you disable this, then
  # boxes will only be checked for updates when the user runs
  # `vagrant box outdated`. This is not recommended.
  # config.vm.box_check_update = false

  # Create a forwarded port mapping which allows access to a specific port
  # within the machine from a port on the host machine. In the example below,
  # accessing "localhost:8080" will access port 80 on the guest machine.
  # config.vm.network "forwarded_port", guest: 80, host: 8080

  # Create a private network, which allows host-only access to the machine
  # using a specific IP.
  config.vm.network "private_network", ip: "192.168.56.101"

  # Create a public network, which generally matched to bridged network.
  # Bridged networks make the machine appear as another physical device on
  # your network.
  # config.vm.network "public_network"

  # Share an additional folder to the guest VM. The first argument is
  # the path on the host to the actual folder. The second argument is
  # the path on the guest to mount the folder. And the optional third
  # argument is a set of non-required options.
  # config.vm.synced_folder "../data", "/vagrant_data"

  # Provider-specific configuration so you can fine-tune various
  # backing providers for Vagrant. These expose provider-specific options.
  # Example for VirtualBox:
  #
  config.vm.provider "virtualbox" do |vb|
    # Display the VirtualBox GUI when booting the machine
    vb.gui = true

    # Customize the amount of memory on the VM:
    vb.memory = "4096"

    # Enable bidirectional clipboard
    vb.customize ['modifyvm', :id, '--clipboard', 'bidirectional']
  end
  #
  # View the documentation for the provider you are using for more
  # information on available options.

  # Define a Vagrant Push strategy for pushing to Atlas. Other push strategies
  # such as FTP and Heroku are also available. See the documentation at
  # https://docs.vagrantup.com/v2/push/atlas.html for more information.
  # config.push.define "atlas" do |push|
  #   push.app = "YOUR_ATLAS_USERNAME/YOUR_APPLICATION_NAME"
  # end

  # Enable provisioning with a shell script. Additional provisioners such as
  # Puppet, Chef, Ansible, Salt, and Docker are also available. Please see the
  # documentation for more information about their specific syntax and use.
  config.vm.provision "shell", inline: <<-SHELL
    # prepare for newer mono
    apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv-keys 3FA7E0328081BFF6A14DA29AA6A19B38D3D831EF
    apt-add-repository 'deb http://download.mono-project.com/repo/debian beta main'
#    apt-add-repository 'deb http://download.mono-project.com/repo/debian wheezy main'
#    apt-add-repository 'deb http://download.mono-project.com/repo/debian wheezy/snapshots/3.12.0 main'
    apt-get update
    apt-get install -y --force-yes mono-runtime=4.0.4.1-0xamarin1
    apt-get install -y --force-yes monodevelop=5.9.6.20-0xamarin1

    apt-get install -y --no-install-recommends ubuntu-desktop unity-lens-applications indicator-session
    apt-get install -y gnome-icon-theme gedit flite python-pexpect python-nltk python-unidecode openjdk-6-jre python-pip
    pip install http://www.cfilt.iitb.ac.in/biplab/practNLPTools-1.0.tar.gz
    apt-get install -y xterm gnome-terminal firefox gedit
    apt-get install -y gstreamer1.0-plugins-bad  gstreamer1.0-plugins-base gstreamer1.0-plugins-good\
      gstreamer1.0-pulseaudio  gstreamer1.0-plugins-ugly  gstreamer1.0-tools libgstreamer1.0-dev

    apt-get upgrade -y
    apt-get dist-upgrade -y

    # rebuild VirtualBox kernel additions?
    apt-get install -y virtualbox-guest-dkms # brings in virtualbox-guest-utils virtualbox-guest-x11
#    DEBIAN_FRONTEND=noninteractive apt-get install -y -o Dpkg::Options::="--force-confdef" -o Dpkg::Options::="--force-confold" virtualbox-guest-dkms
#    dpkg-reconfigure virtualbox-guest-dkms
#    dpkg-reconfigure virtualbox

    cd /home/vagrant
    #sometimes really slow
    #wget -qO- http://speechkitchen.org/vms/Data/IVW3home.tar.gz | tar zxv
    wget -nv http://speechkitchen.org/vms/Data/IVW3home.tar.gz
    tar zxvf IVW3home.tar.gz
    chown -R vagrant:vagrant Desktop .config .bashrc .local
    rm IVW3home.tar.gz

    ln -fs /home/vagrant /home/mario
    chown vagrant:vagrant /home/mario

    # prerequisites for newest gst demo: jansson
    wget http://security.ubuntu.com/ubuntu/pool/main/j/jansson/libjansson4_2.7-1ubuntu1_amd64.deb
    wget http://security.ubuntu.com/ubuntu/pool/main/j/jansson/libjansson-dev_2.7-1ubuntu1_amd64.deb
    dpkg -i libjansson4_2.7-1ubuntu1_amd64.deb
    dpkg -i libjansson-dev_2.7-1ubuntu1_amd64.deb
    
    # rebuild & link the gst demo against Kaldi libraries
    cd /home/vagrant/Desktop/gst-kaldi-nnet2-online/src
    rm *.so
    make

    ln -fs /home/vagrant /home/mario
    chown vagrant:vagrant /home/mario

    # update gnome panel menu icons

    # create args for gconf command in /tmp/arg
    cat <<'END' >/tmp/arg
['application:///home/mario/startServices.desktop', 'application://gnome-terminal.desktop', 'application://nautilus.desktop', 'application://monodevelop.desktop', 'unity://running-apps', 'unity://expo-icon', 'unity://devices']
END

    # create dbus-launch command for vagrant to run in /tmp/dbus.sh
    cat <<'END' >/tmp/dbus.sh
dbus-launch --exit-with-session gsettings set com.canonical.Unity.Launcher favorites "`cat /tmp/arg`"
END
chmod +x /tmp/dbus.sh

    # run it
    su - vagrant -c /tmp/dbus.sh
    # did it work?
    #su -c 'dbus-launch --exit-with-session gsettings get com.canonical.Unity.Launcher favorites' vagrant

    # enable automatic login
    sudo sed -i 's|exec /sbin/getty -8 38400 tty1|exec /bin/login -f vagrant < /dev/tty1 > /dev/tty1 2>\&1|' /etc/init/tty1.conf

    echo "[SeatDefaults]"           | sudo tee -a /usr/share/lightdm/lightdm.conf.d/50-autologin-vagrant.conf
    echo "autologin-user=vagrant"   | sudo tee -a /usr/share/lightdm/lightdm.conf.d/50-autologin-vagrant.conf
    echo "autologin-user-timeout=0" | sudo tee -a /usr/share/lightdm/lightdm.conf.d/50-autologin-vagrant.conf

    # disable password lock
    su - vagrant -c "dbus-launch --exit-with-session gsettings set org.gnome.desktop.screensaver lock-enabled        false"
    su - vagrant -c "dbus-launch --exit-with-session gsettings set org.gnome.desktop.session     idle-delay          0"
    su - vagrant -c "dbus-launch --exit-with-session gsettings set org.gnome.desktop.lockdown    disable-lock-screen 'true'"

    # fix: volume is sometimes zeroed at startup
    amixer set 'Master' 100% on
    amixer set 'PCM' 100% on
    amixer set 'Mic' 100% on

    # kick into desktop mode from terminal mode
    service lightdm start
  SHELL
end
