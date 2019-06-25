
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
        apache-tomcat-8.5.42.tar.gz # download from apache-tomcat

Installing OpenAM with GUI

The vagrant init scripts will configure tomcat and deploy the OpenAM war.

For this test scenario, stop the firewall first and then connect to [http://openam.172.16.12.10.xip.io:8080/openam]()

###Step 1: General

Enter password for admin user, example: admin1235813

###Step 2: Server Settings

* Server URL: http://openam.172.16.12.10.xip.io:8080

* Cookie Domain: .xip.io

* Platform Locale: en_US

* Configuration Directory: /home/tomcat/openam

###Step 3: Configuration Data Store Settings

* First Instance

* Configuration Data Store: OpenAM

* Host Name: localhost (not editable)

* Port: 50389
 
* Admin Port: 4444

* JMX Port: 1689

* Encryption Key: IGU70v0LWLnKErlNowTO6t/nATTfxXcG

* Root Suffix: dc=openam,dc=forgerock,dc=org

###Step 4: User Data Store Settings

* OpenAM User Data Store

###Step 5: Site Configuration

* No

* But we decide yes, below might be ok:

* Site Name: sandbox

* Load Balancer URL: http://proxy.172.16.12.10.xip.io:80/openam

* Enable Session HA Persistence: Check/Yes

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

### 