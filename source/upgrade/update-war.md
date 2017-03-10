# Updating a war file

A [war](https://en.wikipedia.org/wiki/WAR_(file_format)) file is a 
zipped up java web application. Because they make deployment a breeze,
even if you hate violence you can learn to love .war's!

When you install a new version of Gluu, it always comes with the latest
war files. However, sometimes you might make a customization, or 
Gluu might send you a war file as a temporary fix between
service pack or version releases. These instructions walk you through
how to put it to use. 

Keep in mind that a new version of code may also require updates to
the LDAP schema or to the application JSON properties. Make sure 
you are aware of any requirements before you start, because missing
data can cause the Gluu Server to malfunction.

In the following example, we assume the service is oxTrust (which
uses the warfile "identity.war"). But you can utilize this process 
for any of other Gluu Server's modules like oxAuth, Asimba, 
Shibboleth IDP or oxAuth-RP by simply changing name of warfile in
provided commands accordingly (you can find exact locations of
corresponding archives at the end of this page).

1. Login to chroot container 

    `# service gluu-server-3.0.1 login`
    
2. Stop the respective service. 

    `# service identity stop`
    
3. Navigate to the /opt/gluu/jetty folder and back up the current app
in root's home directory (just in case you need to restore!)

    `# cd /opt/gluu/jetty/`
    
    `# tar -czf ~/identity.tar.gz identity`
    

4. Download and install the latest release war (assuming the 
URL was in the $WAR_URL environment variable).

    `# cd /opt/gluu/jetty/identity/webapps/`
    
    `# rm identity.war`
    
    `# wget $WAR_URL`
    

5. Start the service (for example oxTrust)
    
    `# service identity start`
    
Latest release of war files can be downloaded from locations below:

- [oxTrust](https://ox.gluu.org/maven/org/xdi/oxtrust-server/)
- [oxAuth](https://ox.gluu.org/maven/org/xdi/oxauth-server/)
- [Shibboleth IdP](https://ox.gluu.org/maven/org/xdi/oxshibbolethIdp/)
- [Asimba SAML proxy](https://ox.gluu.org/maven/org/xdi/oxasimba-proxy/)
- [oxAuth RP](https://ox.gluu.org/maven/org/xdi/oxauth-rp/)

Next list provides exact locations of different Gluu CE 3.0 components which
can be updated this way inside container:

- oxTrust: `/opt/gluu/jetty/identity/webapps/identity.war`
- oxAuth: `/opt/gluu/jetty/oxauth/webapps/oxauth.war`
- Shibboleth IdP: `/opt/gluu/jetty/idp/webapps/idp.war`
- Asimba SAML proxy: `/opt/gluu/jetty/asimba/webapps/asimba.war`
- oxAuth RP: `/opt/gluu/jetty/oxauth-rp/webapps/oxauth-rp.war`
