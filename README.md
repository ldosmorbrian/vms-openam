
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



---------------------------------

Installing OpenAM with GUI

The vagrant init scripts will configure tomcat and deploy the OpenAM war.

Connect to [https://proxy.172.16.12.10.xip.io/openam]()
password
###Step 1: General

Enter password for admin user, example: Password1234

###Step 2: Server Settings

* Server URL: https://proxy.172.16.12.10.xip.io:443

* Cookie Domain: .xip.io

* Platform Locale: en_US

* Configuration Directory: /opt/am-config

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

* Site Name: proxy

* Load Balancer URL: https://proxy.172.16.12.10.xip.io/openam

###Step 6: Click Create Configuration

Sometimes the GUI doesn't finally say success, so if it seems to be hanging at Setting up monitoring processs,
it's probably actually done so just go to the AM URL and login.


###Sign in

Example: amadmin / Password12345


###Step 7: Click New Realm

* Name: marvel

Click Create

###Step 9: Configure Identity Store

* Click Identity Stores on left side panel

* Click "OpenDJ"

* Change "LDAP Organization DN" value to "dc=example,dc=com"

* Save Changes

* Click on "Identities" on left side panel

* Expect to see a couple users in there (jdoe and bjensen from the Example.ldif)


## Prepare AM with OIDC/OAuth2 for IDM

Reference: https://forum.forgerock.com/2018/05/forgerock-identity-platform-version-6-integrating-idm-ds/

We modified information from the reference guide to reflect being behind TLS proxy and using sub-realm.

###Step 1: Set up AM as an OIDC authorization server.

* Select Marvel Realm -> Configure OAuth Provider -> Configure OpenID Connect -> Create -> OK. 

### Step 2: Set up IDM as an OAuth 2.0 Client

* Select Applications -> OAuth 2.0. Choose Add Client. 
    
    In the New OAuth 2.0 Client window that appears, set openidm as a Client ID, set changeme as a Client Secret, 
    along with a Redirection URI of https://proxy.172.16.12.10.xip.io/idm/oauthReturn/.
    Also may need second Redirection URI of https://proxy.172.16.12.10.xip.io/idm/admin/oauthReturn/
    The scope is openid, which reflects the use of the OpenID Connect standard.
    
* Select Create, go to the Advanced Tab, scroll down. Activate the Implied Consent option.

* Save Changes

### Step 3: Go to the OpenID Connect tab

Enter the following information in the Post Logout Redirect URIs text box:

* https://proxy.172.16.12.10.xip.io/idm

* https://proxy.172.16.12.10.xip.io/idm/admin/

Save changes

###Step 3b: Go to Signing and Encryption tab

* Json Web Key URI: https://proxy.172.16.12.10.xip.io:443/openam/oauth2/marvel/connect/jwk_uri

### Step 4: Select Services -> OAuth2 Provider -> Advanced OpenID Connect:

* Scroll down and enter openidm in the "Authorized OIDC SSO Clients" text box.

Save Changes

### Step 5: Navigate to the Consent tab:

* Enable the Allow Clients to Skip Consent option.

Save Changes.


## Configure IDM to use AM for authentication.

Reference: https://forum.forgerock.com/2018/05/forgerock-identity-platform-version-6-integrating-idm-ds/

###Step 1:  Log in to IDM https://proxy.172.16.12.10.xip.io/idm/admin

* openidm-admin / openidm-admin

###Step 2: Configure Connector Password

The 'full-stack' configuration comes prepared to connect to OpenDJ LDap, 
but the default credentials don't match our configuration.

* Select Configure -> Ldap

* Click the LDAP box to open the config

* Set the password to our default configuration (Password12345)

###Step 3: Reconcile users from the common DS user store to IDM

* Select Configure -> Mappings

In the page that appears, find the mapping from System/Ldap/Account to Managed/User, and press Reconcile. 
That will populate the IDM Managed User store with users from the common DS user store.

### Step 4: Assign IDM Admin to a user

* Click Manage users (brings up a list of the two sample users)

* Choose 'bjensen' -> Authorization Roles -> Add Authorization Roles -> 'openidm-admin' -> choose

That means bjensen can perform admin activities in IDM.

###Step 4: Configure IDM Authentication

* Select Configure -> Authentication

Choose the ForgeRock Identity Provider option. 
In the window that appears, scroll down to the configuration details. 
Based on the instance of AM configured earlier, you’d change:

* Redirection URIs: https://proxy.172.16.12.10.xip.io/idm/oauthReturn/

* Well-Known Endpoint: https://proxy.172.16.12.10.xip.io/openam/oauth2/marvel/.well-known/openid-configuration

* Client ID: openidm

* Client Secret: changeme

Click submit.

* If submit button is disabled, check the Well-Known Endpoint URL, and also verify the IDM
truststore includes the CA signer for the proxy certificate.

###Step 5: Click to logout and re-authenticate.

The Info message appears after configuring authentication, click the 'Click Here'
and re-login as bjensen.


------------------------
Testing Email configuration
Since there's not a "test" button in the gui, the following can be done at commandline:

curl -vvk -H "Content-Type: application/json" \
    -H "X-OpenIDM-Username: openidm-admin"    \
    -H "X-OpenIDM-Password: openidm-admin" -X POST    \
    -d '{"from":"openidm@smtp.local.io", "to":"user1@smtp.local.io", "subject":"Test Email", "body":"Test body text"}'    \
    "https://proxy.172.16.12.10.xip.io/openidm/external/email?_action=send"

--------------------------
Next steps:

IDM Samples Guide:
C17: Using a Workflow to Provision User Accounts

* basically just approves an account to be mapped from one data source to another.

Need to further understand:

Mappings in general
Assignments (entitlements)
CH16: Provisioning with Roles


IDM Integrators Guide

CH11: Working with Managed Roles

CH22: Integrating Business Processes and Workflows


-------------------------------------
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

https://stackoverflow.com/questions/21488845/how-can-i-generate-a-self-signed-certificate-with-subjectaltname-using-openssl