# -*- mode: ruby -*-
# vi: set ft=ruby :

# All Vagrant configuration is done below. The "2" in Vagrant.configure
# configures the configuration version (we support older styles for
# backwards compatibility). Please don't change it unless you know what
# you're doing.
Vagrant.configure("2") do |config|
  # The most common configuration options are documented and commented below.
  # For a complete reference, please see the online documentation at
  # https://docs.vagrantup.com.

  # Every Vagrant development environment requires a box. You can search for
  # boxes at https://vagrantcloud.com/search.
  config.vm.box = "bento/centos-8"

  # Disable automatic box update checking. If you disable this, then
  # boxes will only be checked for updates when the user runs
  # `vagrant box outdated`. This is not recommended.
  # config.vm.box_check_update = false

  # Create a forwarded port mapping which allows access to a specific port
  # within the machine from a port on the host machine. In the example below,
  # accessing "localhost:8080" will access port 80 on the guest machine.
  # NOTE: This will enable public access to the opened port
  # config.vm.network "forwarded_port", guest: 80, host: 8080

  # Create a forwarded port mapping which allows access to a specific port
  # within the machine from a port on the host machine and only allow access
  # via 127.0.0.1 to disable public access
  
  # For Cockpit
  config.vm.network "forwarded_port", guest: 9090, host: 29090, host_ip: "127.0.0.1"
  
  # For SMTP
  config.vm.network "forwarded_port", guest: 25, host: 1025
  
  # For IMAP
  config.vm.network "forwarded_port", guest: 143, host: 1143

  # For POP3
  config.vm.network "forwarded_port", guest: 110, host: 1110

  # For submission
  config.vm.network "forwarded_port", guest: 587, host: 1587
  
  #for Apache
  config.vm.network "forwarded_port", guest: 80, host: 8080
  
  # dnsauth 
  config.vm.network "forwarded_port", guest: 54, host: 154, host_ip: "127.0.0.1"
  
  # dnsrecurs
  config.vm.network "forwarded_port", guest: 53, host: 153, host_ip: "127.0.0.1"

  # Create a private network, which allows host-only access to the machine
  # using a specific IP.
  # config.vm.network "private_network", ip: "192.168.33.10"

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
  # config.vm.provider "virtualbox" do |vb|
  #   # Display the VirtualBox GUI when booting the machine
  #   vb.gui = trueconfig.vm.provision "shell", path: "configure_mail_server.sh", run: 'always'
  # end
  #
  # View the documentation for the provider you are using for more
  # information on available options.

  # Enable provisioning with a shell script. Additional provisioners such as
  # Puppet, Chef, Ansible, Salt, and Docker are also available. Please see the
  # documentation for more information about their specific syntax and use.
   config.vm.provision "Package Installation", type: "shell", inline: <<-SHELL
     #Installation block
     yum -y install epel-release https://rpms.remirepo.net/enterprise/remi-release-8.rpm https://yum.puppetlabs.com/puppet-release-el-8.noarch.rpm
     yum -y install vim cockpit bash-completion postfix dovecot telnet nc \
     cyrus-sasl cyrus-sasl-plain \
     pdns pdns-recursor \
     bind-utils \
     puppet \
  	 httpd  \
  	 mysql-server mysql
  	 # Reset PHP module streams.
     dnf -y module reset php
  	 # Enable the php:remi-7.4 module stream.
     dnf -y module enable php:remi-7.4
  	 # Install PHP modules required or recommended by Roundcube.
  	 yum -y install php-fpm.x86_64 php-ldap php-imagick php-common php-gd php-imap php-json php-curl php-zip php-xml php-mbstring php-bz2 php-intl php-gmp php-mysql
   SHELL
   config.vm.provision "Bulk service enablement", type: "shell", inline: <<-SHELL
 	   # Service configuration block
     systemctl enable --now postfix 
     systemctl enable --now dovecot
     systemctl enable --now cockpit.socket
     systemctl enable pdns-recursor
     systemctl enable pdns
  	 systemctl enable --now mysqld
  	 systemctl enable --now httpd
  	 systemctl enable --now php-fpm.service
     systemctl enable --now saslauthd.service
   SHELL
 
   config.vm.provision "Hostname selection", type: "shell", inline: <<-SHELL
	 # OS configuration block
     hostnamectl set-hostname allinone-by.localhost
   SHELL
	 
   config.vm.provision "RoundCube Installation and Configuration", type: "shell", inline: <<-SHELL
	 #Installation of Roundcube Webmail
	 #Creation /var/www
	 mkdir -p /var/www
	 # Downloading RoundCube
     wget https://github.com/roundcube/roundcubemail/releases/download/1.4.2/roundcubemail-1.4.2-complete.tar.gz
     # Extracting RoundCube to /var/www
     tar -xf roundcubemail-1.4.2-complete.tar.gz -C /var/www
     # Renaming RoundCube Directory
     mv /var/www/roundcubemail-1.4.2 /var/www/roundcube
	 
    # Create a MySQL Database and User for Roundcube
     mysql -u root --execute="CREATE DATABASE roundcube DEFAULT CHARACTER SET utf8 COLLATE utf8_general_ci;"
     mysql -u root --execute="CREATE USER roundcubeuser@localhost IDENTIFIED BY 'password';"
     mysql -u root --execute="GRANT ALL PRIVILEGES ON roundcube.* TO roundcubeuser@localhost;"
     mysql -u root --execute="flush privileges;"
     mysql -u root roundcube < /var/www/roundcube/SQL/mysql.initial.sql
	 
	 # Create Apache Virtual Host Config File for Roundcube
     cat > /etc/httpd/conf.d/roundcube.conf << EOF
<VirtualHost *:80>
  ServerName 127.0.0.1
  DocumentRoot /var/www/roundcube/
  ErrorLog /var/log/httpd/roundcube_error.log
  CustomLog /var/log/httpd/roundcube_access.log combined
  <Directory />
    Options FollowSymLinks
    AllowOverride All
  </Directory>
  <Directory /var/www/roundcube/>
    Options FollowSymLinks MultiViews
    AllowOverride All
    Order allow,deny
    allow from all
  </Directory>
</VirtualHost>
EOF

  cat > /var/www/roundcube/config/config.inc.php << EOF
<?php
/* Local configuration for Roundcube Webmail */
// ----------------------------------
// SQL DATABASE
// ----------------------------------
// Database connection string (DSN) for read+write operations
// Format (compatible with PEAR MDB2): db_provider://user:password@host/database
// Currently supported db_providers: mysql, pgsql, sqlite, mssql, sqlsrv, oracle
// For examples see http://pear.php.net/manual/en/package.database.mdb2.intro-dsn.php
// Note: for SQLite use absolute path (Linux): 'sqlite:////full/path/to/sqlite.db?mode=0646'
//       or (Windows): 'sqlite:///C:/full/path/to/sqlite.db'
// Note: Various drivers support various additional arguments for connection,
//       for Mysql: key, cipher, cert, capath, ca, verify_server_cert,
//       for Postgres: application_name, sslmode, sslcert, sslkey, sslrootcert, sslcrl, sslcompression, service.
//       e.g. 'mysql://roundcube:@localhost/roundcubemail?verify_server_cert=false'
\\$config['db_dsnw'] = 'mysql://roundcubeuser:password@localhost/roundcube';
// ----------------------------------
// IMAP
// ----------------------------------
// The IMAP host chosen to perform the log-in.
// Leave blank to show a textbox at login, give a list of hosts
// to display a pulldown menu or set one host as string.
// Enter hostname with prefix ssl:// to use Implicit TLS, or use
// prefix tls:// to use STARTTLS.
// Supported replacement variables:
// %n - hostname ($_SERVER['SERVER_NAME'])
// %t - hostname without the first part
// %d - domain (http hostname $_SERVER['HTTP_HOST'] without the first part)
// %s - domain name after the '@' from e-mail address provided at login screen
// For example %n = mail.domain.tld, %t = domain.tld
// WARNING: After hostname change update of mail_host column in users table is
//          required to match old user data records with the new host.
\\$config['default_host'] = 'localhost';
// SMTP port. Use 25 for cleartext, 465 for Implicit TLS, or 587 for STARTTLS (default)
\\$config['smtp_port'] = 25;
// provide an URL where a user can get support for this Roundcube installation
// PLEASE DO NOT LINK TO THE ROUNDCUBE.NET WEBSITE HERE!
\\$config['support_url'] = '';
// This key is used for encrypting purposes, like storing of imap password
// in the session. For historical reasons it's called DES_key, but it's used
// with any configured cipher_method (see below).
\\$config['des_key'] = 'bmvFmezqIpUEbO4NOGnyVC08';
// Name your service. This is displayed on the login screen and in the window title
\\$config['product_name'] = 'Roundcube MH';
// ----------------------------------
// PLUGINS
// ----------------------------------
// List of active plugins (in plugins/ directory)
\\$config['plugins'] = array();
\\$config['enable_installer'] = true;
?>
EOF
     # SE Linux permission configuration
     chcon -t httpd_sys_content_t /var/www/roundcube/ -R
     chcon -t httpd_sys_rw_content_t /var/www/roundcube/temp/ /var/www/roundcube/logs/ -R
     setfacl -R -m u:apache:rwx /var/www/roundcube/temp/ /var/www/roundcube/logs/
     setsebool -P httpd_can_network_connect 1

	
	 # Setting up Timezone for PHP
     sed -i '/date.timezone =/s/^;//' /etc/php.ini
     sed -i "s/^\(date.timezone\s*=\s*\).*$/\1 'UTC'/" /etc/php.ini

  SHELL

   config.vm.provision "Puppet manifests for future", type: "shell", inline: <<-SHELL
	 #Puppet Manifests
	 mkdir -p /home/vagrant/puppet_manifest
     cat > /home/vagrant/puppet_manifest init.pp << EOF
     
EOF
  SHELL
	 
   config.vm.provision "user config", type: "shell", inline: <<-SHELL
     # User configuration block
     useradd engineer 
     usermod -p '$6$xyz$.UccqMWqX8VK4PRzmKTR1woU2y5IgDas9n.XPkhgK8M62yVqI4sLx.Yw2AC5z7t4Ke3NiU7aq7i3Su5QdrRcF1' engineer
     useradd manager
     usermod -p '$6$xyz$PcPt/h72LIQm.YoxBmDLqfpbX1w3vhcJ1LwyYjOaslRr67l0g3ZkE5nKN0c4Ed98wYTvMWvhlGcV7NZorCE2i/' manager
     useradd contractor
     usermod -p '$6$xyz$tlQI91A01E6TWfFL6jqBSSLdzLKJtFyF2aWfdTZyOBUn56UjQbMyecGla5IMGqX./neusxkBsr3IwUGZhTnel0' contractor
	SHELL

   config.vm.provision "email service config", type: "shell", inline: <<-SHELL
	   chmod 0600 /var/mail/*
     # Setting up inet_interfaces to all(listening all addresses)
     sed -i 's/^\(inet_interfaces\s*=\s*\).*$/\1all/' /etc/postfix/main.cf
     # Enabling sasl authentication
     sed -i "\$a# Custom added options\nsmtpd_sasl_auth_enable = yes" /etc/postfix/main.cf
     # Setting up mbox path
     sed -i "/mail_location\ =\ mbox:~\/mail:INBOX=\/var\/mail\/%u/s/^#//" /etc/dovecot/conf.d/10-mail.conf
     # Removing manager from alias list
     sed -i '/^manager/d' /etc/aliases && newaliases
     systemctl restart postfix
     systemctl restart dovecot
  SHELL

   config.vm.provision "pdns server config", type: "shell", inline: <<-SHELL
     # Uncomment local-port at pdns config file and changing port value to 54
     sed -i '/local-port=/s/^# //' /etc/pdns/pdns.conf
     sed -i 's/^\(local-port\s*=\s*\).*$/\154/' /etc/pdns/pdns.conf
	 
     # Creating directory for zone file and copying it to his directory
     mkdir -p /var/lib/pdns
	   cp /vagrant/config_files/youdidnotevenimaginethisdomainexists.com.db /var/lib/pdns
	 
     #cat > /var/www/roundcube/config/config.inc.php << EOF
     # Add zone file for domain
     cat > /var/lib/pdns/youdidnotevenimaginethisdomainexists.com.db << EOF
\\$ORIGIN youdidnotevenimaginethisdomainexists.com.
@                      3600 SOA   ns1.allinone-mh.localhost. (
                              postmaster.allinone-mh.localhost.     ; address of responsible party
                              2016072701                 ; serial number
                              3600                       ; refresh period
                              600                        ; retry period
                              604800                     ; expire time
                              1800                     ) ; minimum ttl
                      86400 NS    ns1.p30.dynect.net.
                      86400 NS    ns2.p30.dynect.net.
                      86400 NS    ns3.p30.dynect.net.
                      86400 NS    ns4.p30.dynect.net.
                       3600 MX    10 mx1.allinone-mh.localhost.
                       3600 MX    20 mx2.allinone-mh.localhost.
                       3600 MX    30 mx3.allinone-mh.localhost.
                         60 A     10.0.2.15
EOF
     # Creating custom named config and adding path to him to recursor.conf
     cat > /etc/pdns/named.conf << EOF
zone "youdidnotevenimaginethisdomainexists.com" {
  file "/var/lib/pdns/youdidnotevenimaginethisdomainexists.com.db";
  type master;
};
EOF
    systemctl restart pdns
  SHELL

   # Moving sed commands into separate shell provisioner
   config.vm.provision "pdns recursor config",type: "shell", inline: <<-SHELL
     # Uncomment local-port at pdns-recursor config file
	 cp /vagrant/config_files/named.conf /etc/pdns
     sed -i '/local-port=/s/^# //' /etc/pdns-recursor/recursor.conf
     sed -i \"\\$abind-config=/etc/pdns/named.conf\" /etc/pdns/pdns.conf
     # Configuring forward-zones
     sed -i '/forward-zones=/s/^# //' /etc/pdns-recursor/recursor.conf
     sed -i 's/^\\(forward-zones\s*=\s*\\).*$/\\1youdidnotevenimaginethisdomainexists.com=127.0.0.1:54/' /etc/pdns-recursor/recursor.conf
    systemctl restart pdns-recursor
  SHELL
	 
	 
#   config.vm.provision "shell", path: "configure_mail_server.sh"
#   config.vm.provision "shell", inline: <<-SHELL
#     chmod 0600 /var/mail/*
#   SHELL
end
