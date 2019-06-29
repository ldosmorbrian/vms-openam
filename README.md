
# OpenAM on Wildfly behind Apache httpd proxy

Getting started with local development:

Install Vagrant

        sudo apt install vagrant

Install VirtualBox

       sudo apt-get install virtualbox-qt

Initial vagrant init:

        vagrant init centos/7

Start vm

        vagrant up --provider virtualbox

These application bundles are expected in this base dir:

        AM-eval-6.5.1.zip # download from ForgeRock backstage
        web-agent-5.6.0-Apache_v24_Linux_64bit.zip  # download from ForgeRock backstage
        wildfly-11.0.0.Final.tar.gz # download from wildfly.org (supposedly this is closest match to EAP 7.1)


## HTTPD Notes

yum install mod_ssl
copy certs to some directory, for example here's one potential file configuration (from /etc/httpd/conf/ssl.conf):
SSLEngine on
SSLCertificateFile /etc/pki/tls/certs/proxy.172.16.12.10.xip.io.crt
SSLCertificateKeyFile /etc/pki/tls/private/proxy.172.16.12.10.xip.io.key
SSLVerifyClient require
SSLCACertificateFile /etc/pki/tls/certs/ca-bundle.trust.crt
SSLVerifyDepth 0

Needed to define ServerName globally
TODO: update certgen tool to include pkcs12 version of certs, example:
openssl pkcs12 -in MORIARTY.BRIAN.C.crt -inkey MORIARTY.BRIAN.C.key -export -out MORIARTY.BRIAN.C.pkcs12

## Wildfly

Enbable AJP:
[root@localhost wildfly]# bin/jboss-cli.sh 
You are disconnected at the moment. Type 'connect' to connect to the server or 'help' for the list of supported commands.
[disconnected /] connect
[standalone@localhost:9990 /] /subsystem=undertow/server=default-server/ajp-listener=myListener:add(socket-binding=ajp, scheme=http, enabled=true)
{"outcome" => "success"}
[standalone@localhost:9990 /] exit
[

## DS Notes

sysctl --write fs.inotify.max_user_watches=524288


## IDM Notes




Installing OpenAM with GUI

The vagrant init scripts will configure tomcat and deploy the OpenAM war.

For this test scenario, stop the firewall first and then connect to [https://proxy.172.16.12.10.xip.io/openam]()

###Step 1: General

Enter password for admin user, example: Password1234

###Step 2: Server Settings

* Server URL: https://proxy.172.16.12.10.xip.io:443

* Cookie Domain: .xip.io

* Platform Locale: en_US

* Configuration Directory: /opt/openamcfg

###Step 3: Configuration Data Store Settings

* First Instance

* Configuration Data Store: Embedded DS

* Host Name: localhost (can't edit)

* Port: 50389
 
* Admin Port: 5444

* JMX Port: 1689

* Encryption Key: hiXJmelhruZ62WqG7ga2dhMZgb30g0Ri

* Root Suffix: dc=openam,dc=forgerock,dc=org

###Step 4: User Data Store Settings

* Extenral User Data Store

User Data Store Type: ForgeRock Directory Services (DS)

SSL/TLS Enabled: no

Directory Name: proxy.172.16.12.10.xip.io

Root Suffix: dc=openam,dc=forgerock,dc=org

Login ID: cn=Directory Manager

Port: 1389

###Step 5: Site Configuration

* Yes (just say yes, it's behind httpd)

* Site Name: sandbox

* Load Balancer URL: https://proxy.172.16.12.10.xip.io/openam


###Step 6: Default Policy Agent User

Enter password, example: adminpa1235813

###Configurator Summary Details

Should look like what was done so far, expect it to complete and click proceed to login.

###Sign in

Example: amAdmin / admin1235813

###Next Steps

Click Top Level Realm

Click Subjects

Delete "demo"

Click group tab, create new group, with name SysAdmin

Click User tab, create new user, with your user name.

Click Privileges tab, click SysAdmin, check all boxes, click save.

###Agents

Click Agents on OpenAM top level realm

New Agent

* Name: proxypa

* Password: well, you know...

* Configuration: Centralized

* Server URL: http://openam.172.16.12.10.xip.io:8080/openam

* Agent URL: http://proxy.172.16.12.10.xip.io:80

##Install Web Policy Agent

It should already be extracted to /opt/web_agents

        cd /opt/web_agents/apache22_agent/bin
        echo adminpa1235813 > /tmp/pwd.txt
        ./agentadmin --i
    
* Configuration file: /etc/httpd/conf/httpd.conf

* Change ownership: yes

* OpenSSOAgentBootstrap.properties: hit enter

* OpenAM server URL: http://openam.172.16.12.10.xip.io:8080/openam

* Agent URL: http://proxy.172.16.12.10.xip.io:80

* Agent Profile name: proxypa

* Agent Real: default [/], hit enter

* Path to password: /tmp/pwd.txt

* Confirm: yes (hopefully, right?)

Perform an ordered restart of services

        service httpd stop
        service tomcat stop
        service tomcat start
        service httpd start
        

### When it didn't work...

Shutdown firewall, connected to OpenAM directly at http://proxy.172.16.12.10.xip.io:8080

Edited the Web Policy Agent configuration in the OpenAM web page.

* Top Level Realm --> Agents tab --> proxypa

        Added http://proxy.172.16.12.10.xip.io:8080/ as another Agent Root URL for CDSSO
        
* Agents tab --> proxypa --> Application tab

        Added several new Not Enforced URLs
        http://proxy.172.16.12.10.xip.io:8080/openam*
        http://proxy.172.16.12.10.xip.io:8080/openam*?*
        http://proxy.172.16.12.10.xip.io:8080/openam/*
        http://proxy.172.16.12.10.xip.io:8080/openam/*?*
        http://proxy.172.16.12.10.xip.io:80/UpdateAgentCacheServlet?*
        http://proxy.172.16.12.10.xip.io:80/UpdateAgentCacheServlet
        http://proxy.172.16.12.10.xip.io:80/amagent
        http://proxy.172.16.12.10.xip.io:80/amagent*
        http://proxy.172.16.12.10.xip.io:80/amagent*?*

* Stop apache, clean out the web policy agent to try again

        service httpd stop
        remove the few lines that were added to /etc/httpd/conf/httpd.conf
        rm -Rf /opt/web_agents
        unzip /vagrant/Apache_v22_Linux_64bit_4.0.0.zip -d /opt
        chown -R apache:apache /opt/web_agents
        


## References

### Tomcat

https://linuxize.com/post/how-to-install-tomcat-8-5-on-centos-7/

### Java

https://backstage.forgerock.com/docs/am/6.5/install-guide/#prepare-java-openjdk

### HTTPD

http://dev.antoinesolutions.com/apache-server/mod_ssl

https://httpd.apache.org/docs/2.4/ssl/ssl_howto.html

https://www.techrepublic.com/article/how-to-enable-https-on-apache-centos/
