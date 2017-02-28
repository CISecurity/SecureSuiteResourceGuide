![](http://i.imgur.com/5yZfZi5.jpg)

# CIS-CAT Pro Dashboard Deployment Guide #

----------
## Introduction ##
CIS-CAT Pro Dashboard is a web application built using the Grails Framework.  The grails framework uses a hibernate data model to remain DBMS agnostic.  Grails code compiles into java byte code and therefore runs on the JVM.  As such,  CIS-CAT Pro dashboard can be deployed in any RDBMS which supports JDBC, and on any web container that can handle java WAR's.  The documentation below describes how to deploy CIS-CAT Pro Dashboard on the following stack, which is officially supported by CIS:  MySQL 5.6 running on Ubuntu 16.04, a separate Ubuntu 16.04 server for Tomcat 8 and Apache 2.  Prerequisite to using this documentation is having two separate Ubuntu 16.04 instances running in your environment.

**Reiteration:**  Although this document is written assuming a MySQL database, Ubuntu servers, Tomcat and Apache HTTP Server, they are not required.  The Hibernate data model ensures a database-agnostic environment, so any relational database management system can be used (MySQL, Oracle, MS SQLServer, etc.).  Further, any operating system can host the application server, which can also utilize any software capable of hosting a java web application archive (.war file).

## Component Deployment ##
The following sections describe the installation and configuration of all components necessary to deploy CIS-CAT Pro Dashboard.

### Database ###
Install MySQL Server 5.6 onto the database Ubuntu server using [these instructions](https://www.digitalocean.com/community/tutorials/how-to-install-mysql-on-ubuntu-16-04).  
Once the MySQL database has been installed, note the following information:

- hostname/IP of database server
- port for connecting, by default this is `3306`
- username/password for connecting to the database

### Application Server ###
On a separate server from the database server, install Ubuntu 16.04 (or any operating system supported in the member's environment).  It is important that the database server be separated from the application server for security reasons.  The application server needs to be accessible over the network so that users can access the application, but if the application database were on the same server, it could also be exposed.  By having a separate server, the database can be configured to only communicate over the network via the MySQL port, and therefore be more secure.  Secondly, this configuration is more scalable.  Members can add application servers to the environment as load increases and have the applications communicate with the same database server.
Once Ubuntu 16.04 is installed, and a user can log in, run: sudo apt-get update, this will download the latest package lists from the known repositories so we can install some more software. 

### MySQL Client ###
A MySQL client should be installed in order to test connectivity with the database and create the schema for CIS-CAT Pro Dashboard.
run: sudo apt-get install mysql-client, this will give you access to the mysql command
From the MySQL database instructions above, note the hostname/ip of the database server as <hostname>, and the configured <username> and <password> 
run: mysql -h <hostname> -u <username> -p, you will then be prompted for the  <password>.
if you see a mysql> prompt, then you have successfully connected to the database. 
run: create schema <schemaname>; 
This will create a DB schema for CIS-CAT Dashboard Pro data.

### Java 8 ###
Because CIS-CAT Pro Dashboard is a java-based application, members will need to ensure that java is installed.

	# Install a JDK/JRE
	sudo apt-get install default-jdk
	
	# Ensure Java is installed at the proper version level (or higher)
	java -version

Ensure that the Java version is displayed as `1.8.0_111` (or higher)

### Application Server ###
Install Apache Tomcat 8 by following [this article](https://www.digitalocean.com/community/tutorials/how-to-install-apache-tomcat-8-on-ubuntu-16-04), through the end of Step 6.

Open `/opt/tomcat/bin/catalina.sh` and add the following lines to the top of the file, which will add some environment variables that CIS-CAT Pro Dashboard requires to run.

    export CCPD_CONFIG_FILE="/opt/tomcat/ccpd-config.yml"
    export CCPD_LOG_DIR="/opt/tomcat/logs"

**Note:**  The values of these environment variables can be configured to any location the administrator wishes, and must be to file system locations to which the Tomcat application server can access on startup.

#### Security Considerations ####
As a final step we want to remove the default applications available from the Tomcat install, including the examples and management applications.  These default sites can and will give away information about the environment and present an information security risk.

	# Navigate to the Tomcat applications directory
	cd /opt/tomcat/webapps
	
	# Remove the default applications
	sudo rm -R *  

This will remove all the default applications from the tomcat installation to help reduce the attack surface of the application server.

### Deploy WAR ###
At this point, it is prudent to deploy the CIS-CAT Pro Dashboard application (the "war") included in the downloadable bundle.  The initial deployment of the web application can then be tested via direct connection to the application server on port `8080`, without the web server.  This can help identify any application or application server misconfigurations prior to web server installation/configuration.

Establish a location for the 3 folders to enable legacy data imports:

    /opt/ccpd/Legacy
    /opt/ccpd/LegacyProcessed
    /opt/ccpd/LegacyError

The permissions on these files need to allow the tomcat user to read/write.  The user who will be managing the Legacy imports will need write access to the Legacy folder.

Create the CIS-CAT Pro Dashboard runtime configuration file: `/opt/tomcat/ccpd-config.yml`, including the following lines:

    vulnerability:
    	warningAgeInDays: 90
    ---
    environments:
	    production:
		    grails:
		    	serverURL: 'http://<public url of application server>/CCPD'
		    server:
		    	'contextPath': '/CCPD'
		    dataSource:
			    dbCreate: update
			    url: jdbc:mysql://<hostname/IP of database server>:<port>/ccpd
			    username: <database username>
			    password: <database password>
			    properties:
				    jmxEnabled: true
				    initialSize: 5
				    maxActive: 50
				    minIdle: 5
				    maxIdle: 25
				    maxWait: 10000
				    maxAge: 600000
				    timeBetweenEvictionRunsMillis: 5000
				    minEvictableIdleTimeMillis: 60000
				    validationQuery: SELECT 1
				    validationQueryTimeout: 3
				    validationInterval: 15000
				    testOnBorrow: true
				    testWhileIdle: true
				    testOnReturn: false
				    jdbcInterceptors: ConnectionState
				    defaultTransactionIsolation: 2 # TRANSACTION_READ_COMMITTED
    ---
    legacy:
	    sourceDir: "/opt/ccpd/Legacy"
	    processedDir: "/opt/ccpd/LegacyProcessed"
	    errorDir: "/opt/ccpd/LegacyError"

This file is also available in the CIS-CAT-Pro-Dashboard bundle, you will just need to replace the pertinent information, marked with <>, with the specifics of your environment.

### Mail Configuration ###
CIS-CAT Pro Dashboard utilizes the Grails `mail` plugin in order to send email messages from time to time, including password reset requests.  CIS-CAT Pro Dashboard must be able to connect to and utilize a valid SMTP server in order to send these email messages.  

Configuration of the mail plugin will be member-specific, and those configuration items will also be added to the `/opt/tomcat/ccpd-config.yml` file, noted above.

#### Default Sender Email Address ####
CIS-CAT Pro Dashboard can be configured with a default "sender" email address, indicating that the application has sent a "forgot password" message, allowing users to enter new logon credentials.  In the `/opt/tomcat/ccpd-config.yml` file (noted above), add the following section:

    ---
    grails:
	    plugin:
	    	springsecurity:
	    		ui:
	    			forgotPassword:
	    				emailFrom: "your-email@member-organization.com"

#### SMTP Configuration ####
Coupled with the default sender email address, CIS-CAT Pro Dashboard's mailing configuration must also include connection to a valid SMTP server, in order to correctly distribute the "forgot password" messages.  Numerous SMTP services exist, such as Gmail, Hotmail, Amazon SES, or in-house SMTP services available through corporate emailing technologies, such as Exchange.  CIS-CAT Pro Dashboard can support these SMTP servers, as long as the connection information is correct in the configuration file (the `/opt/tomcat/ccpd-config.yml` noted above).

By default the plugin assumes an unsecured mail server configured at `localhost` on port `25`. However you can change this via the application configuration file (the `/opt/tomcat/ccpd-config.yml` noted above). 

For example here is how you would configure the default sender to send with a Gmail account:
#### *Mail Configuration - Gmail* ####
    ---
    grails:
	    mail:
		    host: "smtp.gmail.com"
		    port: 465
		    username: "youracount@gmail.com"
		    password: "yourpassword"
		    props:
			    mail.smtp.auth: "true",
			    mail.smtp.socketFactory.port: "465"
			    mail.smtp.socketFactory.class: "javax.net.ssl.SSLSocketFactory"
			    mail.smtp.socketFactory.fallback: "false"

And the configuration for sending via a Hotmail/Live account:
#### *Mail Configuration - Hotmail/Live* ####
    ---
    grails:
	    mail:
		    host: "smtp.live.com"
		    port: 587
		    username: "youracount@live.com"
		    password: "yourpassword"
		    props:
			    mail.smtp.starttls.enable: "true"
			    mail.smtp.port: "587"

And the configuration for sending via a Yahoo account:
#### *Mail Configuration - Yahoo* ####
    ---
    grails:
	    mail:
		    host: "smtp.correo.yahoo.es"
		    port: 465
		    username: "myuser"
		    password: "mypassword"
		    props:
			    mail.smtp.auth: "true"
			    mail.smtp.socketFactory.port: "465"
			    mail.smtp.socketFactory.class: "javax.net.ssl.SSLSocketFactory"
			    mail.smtp.socketFactory.fallback: "false"

### Deploy WAR (Continued) ###
Connect to the application server and transfer the `CCPD.war` to the `/opt/tomcat/webapps` directory.  If Tomcat is running, it should automatically deploy the application.  If Tomcat is not running, starting it will deploy the application.

Once Tomcat is done with its deployment you should be able to access the application by entering `http://<public url of application server>:8080/CCPD`, into a browser.

CIS-CAT Pro Dashboard will bootstrap in a user with:
    username: admin, 
    password: @admin123

**Notes**:
Your network must allow traffic on 8080
(If deployed to AWS) Your security group in AWS must allow traffic on 8080

### Web Server ###
The final configuration is for a web server to proxy the Tomcat instance and to answer traffic on port `80/443`.

#### Apache HTTP Server ####
Install Apache 2.4 by running: 
    sudo apt-get install apache2

Create a file named `ccpd.conf` in the following directory: `/etc/apache2/sites-available` with the following lines:

    <VirtualHost *:80>
	    ServerName <public url of application server>
	    ProxyRequests Off
	    <Proxy *>
		    Order deny,allow
		    Allow from all
	    </Proxy>
	    ProxyPreserveHost on
	    ProxyPass / http://localhost:8080/
    </VirtualHost>

The `ServerName` should be the `<public url of application server>`

Execute the following commands to enable the proxy module:
    sudo a2enmod proxy
    sudo a2enmod proxy_ajp
    sudo a2enmod proxy_http
    sudo apache2 restart

Execute the following commands to enable the reverse proxy to Tomcat:
    sudo a2ensite shepherd.conf
    sudo service apache2 reload

The Apache HTTP server should now be configured to serve requests to CIS-CAT Pro Dashboard.  The application should be available at the URL entered into your `ccpd-config.yml`: `http://<public url http server>/CCPD`

### Securing Web Traffic ###
The steps above will have the CIS-CAT Pro Dashboard application running over normal HTTP on port `80`.  This presents a risk as data, including user credentials, will be transmitted in clear text.  It is recommended that traffic be secured using HTTPS.  The following article explains how to create a self-signed certificate and apply it to a web server in Ubuntu 16.04: [https://www.digitalocean.com/community/tutorials/how-to-create-a-self-signed-ssl-certificate-for-apache-in-ubuntu-16-04](https://www.digitalocean.com/community/tutorials/how-to-create-a-self-signed-ssl-certificate-for-apache-in-ubuntu-16-04)

### Ensuring Trust ###
In the browser's URL bar, navigate to the CIS-CAT Pro Dashboard application.  Click on the HTTPS certificate chain (next to URL address). In the Google Chrome browser, if the user sees a "Not Secure" label next to the URL, the certificate is not trusted.  Click on the "Not Secure" link to display the certificate information:

- Click the "Details" link to display the "Security Overview" information.
- Click on the "View Certificate" button to display the certificate details.

The certificate information needs to be exported, and added to the servers trust store.  On the "details" tab of the certificate information, click the "Copy to File..." button.  The "certificate export wizard" will be displayed.

- Click the "Next" button to navigate to the second screen, the export type selection:

Select the "DER encoded binary X.509 (.CER)" radio button.  This selection should be the default.  Click "Next" to continue.

Select or browse to a location to where the `.cer` file will be saved.  Clicking "Next" and finally "Finish" to complete the wizard and save the file to the selected location/filename.  This file will need to be transferred via SCP, S/FTP to the application server hosting the CIS-CAT Pro Dashboard application.  Note the location to which the file is transferred.  For the purposes of this User's Guide, assume the location of the `ccpd_cert.cer` file is `/home/ubuntu/ccpd_cert.cer`.

#### Importing the certificate into the trust store ####
Once the file has been transferred to the application server, locate the `$JAVA_HOME` file location, i.e. the filesystem location of the java executable being used to launch Tomcat and host CIS-CAT Pro Dashboard.  For the purposes of this User's Guide, assume the location of `$JAVA_HOME` can be found at `/usr/lib/jvm/java-8-openjdk-amd64`.  When using Java 8, the required `cacerts` file will be located at `$JAVA_HOME/lib/security/cacerts`.  The location of the `cacerts` file should be noted before importing the certificate using `keytool`.

Finally, using the java `keytool` application, import the certificate:

    # Navigate to the $JAVA_HOME folder's bin directory.
    cd /usr/lib/jvm/java-8-openjdk-amd64/bin
     
    # Execute keytool
    sudo keytool -import alias ccpd -keystore ../lib/security/cacerts -file /home/ubuntu/ccpd_cert.cer
The user should be prompted to enter the keystore credentials.  By default, this credential is `changeit`.

    # The user will be asked for confirmation
    Trust this certificate? [no]:

Answering `yes` to this confirmation prompt will import the certificate into the trust store.

**NOTE**:  If the application server is currently running, it must be restarted in order to incorporate the trust store.