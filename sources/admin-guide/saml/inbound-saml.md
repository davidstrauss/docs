# Inbound SAML in Gluu Server

For customers who need to support inbound SAML authentication for partners or
other organizations, we can deploy the Asimba SAML proxy.

The main use case for Asimba is to enable websites to use a single IDP for SSO,
even when the organization may have a number of IDPs that are trusted. For more
information, please contact us.

## Requirement to setup Inbound SAML 

* Metadata of authentication server
* Metadata of websites who will work as servcie provider
* SAML certificate of websites
* SSL certificate of authentication server 
* Required attributes

Above points are described breifly below. 

### Metadata of authentication server

Authentication server can be any remote / native Shibboleth IDP or Microsoft AD
FS. We need metadata of these servers to configure Asimba. After configuration
end user will be able to select their desired authentication server from
Asimba's discovery page. Or, we can configure the "selector" which will
automatically redirect user to desired IDP / ADFS. 

### Metadata of websites who will work as servcie provider

Just like authentication server metadata, Gluu Server - Asimba suite require
metadaa from all websites ( SPs ) which will be connected. 

### SAML certificate of websites

Base64 encoded certificates require to configure the trust store of Asimba
server as it can connect / allow the inbound SAML request from remote SP. 

### SSL certificate of authentication server

Base64 encoded certificates of authentication server is also a requirement. 

### Required attributes

Every organization has their own policy to release / pass few attributes. It can
be standard attribute like UID or email address or can be any custom attribute.
Gluu Engineers need a list of required attributes from organization which they
want to pass between their authentication server and target websites ( SP ). 


# Asimba configuration with Gluu Server


## Base installation of Asimba package

* Get the war for Asimba from [http://asimba.org](http://asimba.org)
* Copy the war file in `/opt/tomcat/webapps`
* Restart tomcat, it will extract the Asimba
* Get the asimba.conf template from Gluu
* Generate the keystore for your Asimba server:
    * Command: `keytool -genkeypair -keyalg RSA -alias "<ALIAS_OF_KEYSTORE>" -keypass <PASSWORD> -keystore <NAME_OF_JKS>.jks -storepass <PASSWORD>`
        * What is your first and last name?: Provide the hostname of Asimba server
        * What is the name of your organizational unit?: IT
        * What is the name of your organization?: Provide name
        * What is the name of your City or Locality?: city name
        * What is the name of your State or Province?: State name
        * What is the two-letter country code for this unit?: US
* Adding IDP / ADFS in Asimba: 
    * Gather metadata of IDP / ADFS and keep them in some place under /tomcat/webapps/asimba-saml-proxy/WEB-INF/ 
    * Collect certificate of IDP / ADFS and import them in Asimba truststore JKS

* Adding SP in Asimba: 
    * Gather metadata of SP and keep them in some place under /tomcat/webapps/asimba-saml-proxy/WEB-INF/
    * Collect certificate of SP and import them in Asimba truststore JKS

* Restart tomcat

## Apache configuration

* Configure "idp.conf": 

            <Location /asimba-saml-proxy>
                ProxyPass ajp://<ASIMBA_HOSTNAME>:8009/asimba-saml-proxy retry=5 disablereuse=On
                ProxyPassReverse ajp://<ASIMBA_HOSTNAME>:8009/asimba-saml-proxy
    
                Order allow,deny
                Allow from all
            </Location>

* Restart httpd 

## Test Asimba setup

* Try to download the metadata of Asimba server with: `wget -c https://<HOSTNAME>/asimba-saml-proxy/profiles/saml2`
 

## Adding new Service Provider or Identity Provider in Asimba server

### A new IDP / ADFS configuration in Asimba

All our configurations are based on one Asimba configuration file named
"asimba.xml". It's also possible to configure Asimba with JDBC. For more info
Asimba [wiki](http://sourceforge.net/p/asimba/wiki/Home/) is availalbe.

* Required tools
    * Metadata of remote IDP
    * SAML certificate of remote IDP

* Configuring asimba.xml: 
    * Grab the metadata of remote IDP and save it in some place. Make sure that user tomcat can read this xml copy. 
        * Specify this metadata in  `<websso>\<method>\<idps>` section. 
            * `idp id` must follow the entityID of this metadata.
            * `scoping` should be "false"
            * Add the static path of metadata inside `<file>` section. 
    
A sample configuration should look like below:

<code>

            <idp id="https://idp.gluu.org/idp/shibboleth" friendlyname="Gluu IDP" scoping="false" 
                avoid_subjectconfirmation="false">
                    <nameidpolicy enabled="false" />
                        <metadata>
                            <file>${webapp.root}/WEB-INF/metadata/idp/idp_gluu_org.xml</file>
                        <metadata>
            </idp>

</code>


## Preparation for configuring Asimba SAML Proxy inside Gluu Server

* Collect the metadata of IDP / AD FS which will be connected with your Asimba Server. 
* Collect SAML certificate of that IDP / AD FS server which will be connected with Asimba Proxy Server. 
    * Certificate must have to be a base64 encoded ASCII files which contain `-----BEGIN CERTIFICATE-----` and `-----END CERTIFICATE-----`. 
    * Certificate cannot be password protected. 
* Collect the metadata of SP which will be connected with Asimba SAML Proxy. 
* Collect the SAML certificate of SP. 
    * Certificate must have to be a base64 encoded ASCII files which contain `-----BEGIN CERTIFICATE-----` and `-----END CERTIFICATE-----`.
    * Certificate cannot be password protected.

## Base installation of Asimba SAML Proxy

* Get Asimba from [Asimba repository](http://sourceforge.net/projects/asimba/files/release/) 
* Send the war file in ~/tomcat/webapps/
* Restart tomcat, it will extract the asimba 
* Generate the keystore for your SAML Proxy Server:
    * Command: `keytool -genkeypair -keyalg RSA -alias "<ALIAS_OF_KEYSTORE>" -keypass <PASSWORD> -keystore <NAME_OF_JKS>.jks -storepass <PASSWORD>`
        * What is your first and last name?: Provide the hostname of Asimba server
        * What is the name of your organizational unit?: IT
        * What is the name of your organization?: Provide name
        * What is the name of your City or Locality?: city name
        * What is the name of your State or Province?: State name
        * What is the two-letter country code for this unit?: US


### Apache configuration

#### Configure "idp.conf": 

            <Location /asimba-saml-proxy>
                ProxyPass ajp://<ASIMBA_HOSTNAME>:8009/asimba-saml-proxy retry=5 disablereuse=On
                ProxyPassReverse ajp://<ASIMBA_HOSTNAME>:8009/asimba-saml-proxy
    
                Order allow,deny
                Allow from all
            </Location>


 
## Adding new Service Provider or Identity Provider in Asimba server

### A new IDP / ADFS configuration in Asimba

All our configurations are based on one Asimba configuration file named
"asimba.xml". It's also possible to configure Asimba with JDBC. For more info
Asimba [wiki](http://sourceforge.net/p/asimba/wiki/Home/) is availalbe.

#### Required files

* Metadata of remote IDP
* SAML certificate of remote IDP

#### Configuring asimba.xml: 

* Grab the metadata of remote IDP and save it in some place. Make sure that user tomcat can read this xml copy. 
    * Specify this metadata in  `<websso>\<method>\<idps>` section. 
        * `idp id` must follow the entityID of this metadata.
        * `scoping` should be "false"
        * Add the static path of metadata inside `<file>` section. 
    
A sample configuration should look like below:
<code>

            <idp id="https://idp.gluu.org/idp/shibboleth" friendlyname="Gluu IDP" scoping="false" 
                avoid_subjectconfirmation="false">
                    <nameidpolicy enabled="false" />
                        <metadata>
                            <file>${webapp.root}/WEB-INF/metadata/idp/idp_gluu_org.xml</file>
                        <metadata>
            </idp>

</code>

#### Work on keystore: 

* Import the SAML certificate of IDP into Asimba's JKS
    * Convert certificate into DER format
    * Import this der formatted certificate into Asimba's keystore
    * Please note that it's a good practice to follow the IDP's `entityID` as `alias` of this certificate.

Sample command would be: 

        keytool -import -trustcacerts -alias https://idp.gluu.org/idp/shibboleth \ 
            -file idp_gluu_org.der -keystore asimba-keystore.jks
    


#### Restart tomcat. If everything goes well, your IDP is configured with Asimba


### A new SP configuration in Asimba

#### Required files: 
    
* Metadata of SP
* SAML certificate of SP


#### Configuring asimba.xml for new SP:

* Collect the metadata of SP and place in some location of your file system. Make sure that the metadata is in xml format and user tomcat readable.  
* Add Requestor: Every SP is known as `requestor` to Asimba. There are couple of place where we need to configure this SP inside Asimba's asimba.xml file. 
    * Requestor should be configured in <requestorpoolfactory\<requestors>\<requestor
    * `requestor id` should be the `entityID` of SP
    * `friendlyname` can be anything which is unique

A sample configuration should look like below: 
<code>

            <requestor id="http://sptest2.gluu.org/secure"
                    friendlyname="Gluu Test SP"
                    enabled="true" />


</code>


* Add profile of requestor: In this section the location of SP's metadata and some other configurations are done. 
    * This configuration is configured in <profiles>\<websso\<requestors\<requestor
    * The `requestor id` is the entityID of SP
    * `metadata` location is a the absolute path

A sample configuration of requestor's profile configuration: 
<code>


            <requestor id="http://sptest2.gluu.org/secure"
                signing="FALSE">
                    <metadata>
                        <file>${webapp.root}/WEB-INF/metadata/sp/sptest2_gluu_org.xml</file>
                    </metadata>
            </requestor>


</code>


#### Work on keystore:

* Import the SAML certificate of SP into Asimba's JKS
    * Convert certificate into DER format
    * Import this der formatted certificate into Asimba's keystore
    * Please note that it's a good practice to follow the SP's `entityID` as `alias` of this certificate.

Sample command would be:

        keytool -import -trustcacerts -alias https://sptest2.gluu.org/secure \
            -file sp_gluu_org.der -keystore asimba-keystore.jks


#### Restart tomcat. SP is configured in Asimba. 

### Configure SP for Asimba server

Now, it's time to configure your SP as it can send the request to Asimba server.
The primary item to configure Asimba server in SP's configuration is `the
metadata of Asimba`. Which can be grabbed easily with 

    wget -c https://<HOSTNAME>/asimba-saml-proxy/profiles/saml2

### Configure remote IDP / AD FS for Asimba server

Remote IDP / AD FS also need to be configured for Asimba server as they can talk
together. 

If the remote authentication server is a Shibboleth IDP, it just require three
things: 

* Metadata of Asimba - which can be grabbed easily. 
* Name identifier attribute, which will be shared with Asimba - `persistentID`
* `SAML2SSO` relying party configuration for this trust relationship with Asimba. Values should be: 
    * _includeAttributeStatement_ : true
    * _assertionLifetime_ : 300000
    * _assertionProxyCount_ : 0
    * _signResponses_ : conditional
    * _signAssertions_ : never
    * _signRequests_ : conditional
    * _encryptAssertions_ : never
    * _encryptNameIds_ : never 



### How to Test

The workflow of SAML Proxy is:

End user hit the SP --> SP will take user to Asimba's discovery page to select IDP --> User will select IDP for authentication --> After successful
authentication user will be logged into SP

Gluu has an auto selector mechanism which automatically redirect user from
specified SP to desired IDP for authentication. If you want to know more, please
talk to us. 

