<div id="table-of-contents">
<h2>Table of Contents</h2>
<div id="text-table-of-contents">
<ul>
<li><a href="#sec-1">1. Info</a></li>
<li><a href="#sec-2">2. Host names / A records</a>
<ul>
<li><a href="#sec-2-1">2.1. ifbma.org</a></li>
<li><a href="#sec-2-2">2.2. dev.ifbma.org</a></li>
<li><a href="#sec-2-3">2.3. www.ifbma.org</a></li>
<li><a href="#sec-2-4">2.4. bikesyndikat-shop.com</a></li>
<li><a href="#sec-2-5">2.5. www.bikesyndikat-shop.com</a></li>
</ul>
</li>
<li><a href="#sec-3">3. Host names / port forwards</a>
<ul>
<li><a href="#sec-3-1">3.1. planlibre.de</a></li>
<li><a href="#sec-3-2">3.2. plan-libre.de</a></li>
</ul>
</li>
<li><a href="#sec-4">4. let's encrypt</a>
<ul>
<li><a href="#sec-4-1">4.1. Installation</a></li>
<li><a href="#sec-4-2">4.2. Configuration</a></li>
<li><a href="#sec-4-3">4.3. Running</a>
<ul>
<li><a href="#sec-4-3-1">4.3.1. Apache</a></li>
<li><a href="#sec-4-3-2">4.3.2. wildfly</a></li>
<li><a href="#sec-4-3-3">4.3.3. openfire</a></li>
</ul>
</li>
</ul>
</li>
</ul>
</div>
</div>

# Info<a id="sec-1" name="sec-1"></a>

Debian Linux server

# Host names / A records<a id="sec-2" name="sec-2"></a>

## ifbma.org<a id="sec-2-1" name="sec-2-1"></a>

## dev.ifbma.org<a id="sec-2-2" name="sec-2-2"></a>

## www.ifbma.org<a id="sec-2-3" name="sec-2-3"></a>

## bikesyndikat-shop.com<a id="sec-2-4" name="sec-2-4"></a>

## www.bikesyndikat-shop.com<a id="sec-2-5" name="sec-2-5"></a>

# Host names / port forwards<a id="sec-3" name="sec-3"></a>

## planlibre.de<a id="sec-3-1" name="sec-3-1"></a>

## plan-libre.de<a id="sec-3-2" name="sec-3-2"></a>

# let's encrypt<a id="sec-4" name="sec-4"></a>

## Installation<a id="sec-4-1" name="sec-4-1"></a>

    git clone https://github.com/certbot/certbot
    cd certbot

## Configuration<a id="sec-4-2" name="sec-4-2"></a>

    /etc/init.d/apache2 stop
    # kill the port forward ssh
    nohup ssh -f -p 24 -R :80:localhost:80 -R :443:localhost:443 root@planlibre.de 'echo test;while true;do sleep 3;done;' &
    ./certbot-auto certonly -a standalone -d ifbma.org -d www.ifbma.org -d dev.ifbma.org -d planlibre.de -d plan-libre.de

## Running<a id="sec-4-3" name="sec-4-3"></a>

### Apache<a id="sec-4-3-1" name="sec-4-3-1"></a>

Add to each virtual host a 

    NameVirtualHost *:443
    
    <VirtualHost *:80>
      ServerName www.ifbma.org
      ServerAlias ifbma.org
      ServerAlias dev.ifbma.org
      Redirect / https://ifbma.org/
    </VirtualHost>

and a 

    SSLEngine on
    SSLCertificateFile      /etc/letsencrypt/live/ifbma.org/cert.pem
    SSLCertificateKeyFile   /etc/letsencrypt/live/ifbma.org/privkey.pem
    SSLCertificateChainFile /etc/letsencrypt/live/ifbma.org/fullchain.pem
    #SSLCARevocationPath /etc/apache2/ssl.crl/
    #SSLCARevocationFile /etc/apache2/ssl.crl/ca-bundle.crl
    SSLProtocol all -SSLv2 -SSLv3
    SSLCipherSuite ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-AES256-GCM-SHA384:DHE-RSA-AES128-GCM-SHA256:DHE-DSS-AES128-GCM-SHA256:kEDH+AESGCM:ECDHE-RSA-AES128-SHA256:ECDHE-ECDSA-AES128-SHA256:ECDHE-RSA-AES128-SHA:ECDHE-ECDSA-AES128-SHA:ECDHE-RSA-AES256-SHA384:ECDHE-ECDSA-AES256-SHA384:ECDHE-RSA-AES256-SHA:ECDHE-ECDSA-AES256-SHA:DHE-RSA-AES128-SHA256:DHE-RSA-AES128-SHA:DHE-DSS-AES128-SHA256:DHE-RSA-AES256-SHA256:DHE-DSS-AES256-SHA:DHE-RSA-AES256-SHA:AES128-GCM-SHA256:AES256-GCM-SHA384:AES128-SHA256:AES256-SHA256:AES128-SHA:AES256-SHA:AES:CAMELLIA:DES-CBC3-SHA:!aNULL:!eNULL:!EXPORT:!DES:!RC4:!MD5:!PSK:!aECDH:!EDH-DSS-DES-CBC3-SHA:!EDH-RSA-DES-CBC3-SHA:!KRB5-DES-CBC3-SHA
    SSLOptions +StrictRequire
    SSLHonorCipherOrder on
    SSLCompression off

### wildfly<a id="sec-4-3-2" name="sec-4-3-2"></a>

1.  Import into `/home/wildfly/.keystore`

    There has to be a corresponing line in the server configuration, like
    the ssl/keystore one in:
    
         <security-realm name="ApplicationRealm">
            <server-identities>
                <ssl>
                    <keystore path=".keystore" relative-to="user.home" keystore-password="changeit" alias="s1as" key-password="changeit"/>
                </ssl>
            </server-identities>
            <authentication>
                <local default-user="$local" allowed-users="*" skip-group-loading="true"/>
                <properties path="application-users.properties" relative-to="jboss.server.config.dir"/>
            </authentication>
            <authorization>
                <properties path="application-roles.properties" relative-to="jboss.server.config.dir"/>
            </authorization>
        </security-realm>

2.  convert

        openssl pkcs12 -inkey /etc/letsencrypt/live/ifbma.org/privkey.pem \
         -in /etc/letsencrypt/live/ifbma.org/cert.pem -export -name s1as \
         -out /home/wildfly/device-ip.pkcs12

3.  import

        keytool -delete -alias s1as -keystore /home/wildfly/.keystore
        keytool -importkeystore -srckeystore /home/wildfly/device-ip.pkcs12 \
         -srcstoretype PKCS12 -keystore /home/wildfly/.keystore
        
        su - wildfly
        jboss-cli.sh --connect <<<EOF
        :reload
        EOF
        tail -f ${WILDFLYDIR}/standalone/log/server.log

### openfire<a id="sec-4-3-3" name="sec-4-3-3"></a>

1.  short version

    It is also possible to run this analogous to the wildfly procedure. Use
    that openssl conversion then.
    
        keytool -delete -alias dev.ifbma.org -storepass changeit \
         -keystore /etc/openfire/security/keystore
        keytool -importkeystore -srckeystore /home/wildfly/device-ip.pkcs12 \
         -srcstoretype PKCS12 -keystore /etc/openfire/security/keystore
        service openfire restart

2.  convert as full chain

    Different to wildfly, export the full chain, use the password used in
    openfire and as a name the jabber domain.
    
        openssl pkcs12 -inkey /etc/letsencrypt/live/ifbma.org/privkey.pem \
         -in /etc/letsencrypt/live/ifbma.org/fullchain.pem -export -name dev.ifbma.org \
         -out /etc/openfire/fullchain.pkcs12 -passout pass:changeit

3.  import from full chain

        keytool -delete -alias dev.ifbma.org -storepass changeit \
         -keystore /etc/openfire/security/keystore
        keytool -importkeystore -srckeystore /etc/openfire/fullchain.pkcs12 \
         -srcstoretype PKCS12 -destkeystore /etc/openfire/security/keystore \
         -deststorepass changeit -destkeypass changeit -srcstorepass changeit
         -alias dev.ifbma.org
        service openfire restart
