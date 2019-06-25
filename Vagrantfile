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

  # Every Vagrant development environment requires a box. You can search for
  # boxes at https://atlas.hashicorp.com/search.
  config.vm.box = "centos/7"

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
  config.vm.network "private_network", ip: "172.16.12.10"

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
    # vb.gui = true
  
    # Customize the amount of memory on the VM:
    vb.memory = "8192"
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
    # hostname
    sed -i s/localhost\.localdomain/proxy.172.16.12.10.xip.io/ /etc/sysconfig/network
    sysctl kernel.hostname=proxy.172.16.12.10.xip.io
    # firewall
    iptables -I INPUT -i lo -j ACCEPT
    firewall-cmd --zone=public --permanent --add-port=443/tcp
    firewall-cmd --zone=public --permanent --add-port=80/tcp
    firewall-cmd --zone=public --permanent --add-port=22/tcp
    firewall-cmd --reload
    # os updates
    yum update -y
    # os config
    sed -i '$ i openam soft nofile 65536' /etc/security/limits.conf
    sed -i '$ i openam hard nofile 131072' /etc/security/limits.conf
    # httpd
    yum install httpd -y
    systemctl enable httpd.service
    echo 'ProxyPass /openam               ajp://localhost:8009/openam' >> /etc/httpd/conf/httpd.conf
    echo 'ProxyPassReverse /openam        ajp://localhost:8009/openam' >> /etc/httpd/conf/httpd.conf
    echo '#' >> /etc/httpd/conf/httpd.conf
    echo 'ProxyPass /examples             ajp://localhost:8009/examples' >> /etc/httpd/conf/httpd.conf
    echo 'ProxyPassReverse /examples      ajp://localhost:8009/examples' >> /etc/httpd/conf/httpd.conf
    systemctl restart httpd.service
    # java
    yum install java-1.8.0-openjdk-devel
    # tomcat
    mkdir -p /opt/tomcat
    tar xvzf /vagrant/apache-tomcat-8.5.42.tar.gz -C /opt/tomcat
    ln -s /opt/tomcat/apache-tomcat-8.5.42 /opt/tomcat/latest
    useradd tomcat
    chown -R tomcat:tomcat /opt/tomcat
    chmod +x /opt/tomcat/latest/bin/*.sh
    cp /vagrant/tomcat-service.sysctl /etc/systemd/system/tomcat.service
    systemctl daemon-reload
    systemctl enable tomcat
    systemctl start tomcat
#    ln -s /usr/local/tomcat/conf /etc/tomcat
#    cp /vagrant/tomcat.conf /etc/tomcat
#    cp /vagrant/tomcat-service /etc/init.d/tomcat
#    sed -i 's/Connector port/Connector URIEncoding="UTF-8" port/' /etc/tomcat/server.xml
#    chown -R tomcat:tomcat /usr/local/apache-tomcat-8.0.42
#    chmod 755 /etc/init.d/tomcat
#    chkconfig tomcat on
#    # openam
#    service tomcat stop

#    unzip /vagrant/AM-eval-6.5.1.zip -d /vagrant/
#    cp /vagrant/openam/AM-eval-6.5.1.war /usr/local/tomcat/webapps/openam.war
#    chown tomcat:tomcat /usr/local/tomcat/webapps/openam.war
#    # start tomcat
#    service tomcat start
#    # Extract web policy agent
#    unzip /vagrant/Apache_v22_Linux_64bit_4.0.0.zip -d /opt
#    chown -R apache:apache /opt/web_agents
  SHELL
end
