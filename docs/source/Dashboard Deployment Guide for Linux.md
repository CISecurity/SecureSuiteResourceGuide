![](http://i.imgur.com/5yZfZi5.jpg)

# CIS-CAT Pro Dashboard Deployment Guide for Linux#

----------
## Introduction ##
CIS-CAT Pro Dashboard is a web application built using the Grails Framework.  The grails framework uses a hibernate data model to remain DBMS agnostic.  Grails code compiles into java byte code and therefore runs on the JVM.  As such,  CIS-CAT Pro dashboard can be deployed in many RDBMS, and on any web container that can handle java WAR's.  The documentation below describes how to deploy CIS-CAT Pro Dashboard on the following stack, which is officially supported by CIS:  MySQL 5.6 running on Ubuntu 16.04, a separate Ubuntu 16.04 server for Tomcat 8 and Apache 2.  Prerequisite to using this documentation is having two separate Ubuntu 16.04 instances running in your environment.

**Reiteration:**  Although this document is written assuming a MySQL database, Ubuntu servers, Tomcat and Apache HTTP Server, they are not required. The Hibernate data model ensures a database-agnostic environment, so many relational database management system can be used (i.e. MySQL until version 5.7 (including MariaDB),MS SQL Server, and Oracle). Further, any operating system can host the application server, which can also utilize any software capable of hosting a java web application archive (.war file).

## System Recommendations ##
 There are no strict requirements associated with our Dashboard application. Any OS will be suitable so long as it can run Tomcat. Disk space will be minimal on the application server, but will require more space on your database server depending upon the size of your organization and the amount of endpoints you have.

Our test environment uses an AWS t2.large instance (designed for burst processing), which has:

 - 8GB RAM
 - 2 vCPU’s with 4 cores each
 - Ubuntu 16.04
 
The application is fairly lightweight on processor and memory use. However, when importing results, the usage will spike. Our recommendation would be to conduct importing off hours and then the functionality of the application will not be hindered during business hours.

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

run: `sudo apt-get install mysql-client`

This will give you access to the mysql command.
From the MySQL database instructions above, note the hostname/ip of the database server as `<hostname>`, and the configured `<username>` and `<password>`
run: `mysql -h <hostname> -u <username> -p`, you will then be prompted for the `<password>`.  If you see a `mysql>` prompt, then you have successfully connected to the database.
 
run: `create schema ccpd;`

This will create a DB schema named `ccpd` for CIS-CAT Dashboard Pro data.

**NOTE:** If your database server is not installed on the same machine as tomcat, make sure that your database user has privileges for remote access.
Here is a SQL statement to enable access for the remote user:
	
	GRANT ALL ON *.* TO <user>@'<tomcat_server_IP>' IDENTIFIED BY '<password>';
	
For more details, refer to [this article](https://support.rackspace.com/how-to/mysql-connect-to-your-database-remotely/).

### Java 8 ###
Because CIS-CAT Pro Dashboard is a java-based application, members will need to ensure that java is installed.

	# Install a OpenJDK JDK/JRE
	sudo apt-get install default-jdk
	
	# Ensure Java is installed at the proper version level 
	java -version

Ensure that the Java version displayed starts with 1.8.***

### Application Server ###
Install Apache Tomcat 8 by following [this article](https://www.digitalocean.com/community/tutorials/how-to-install-apache-tomcat-8-on-ubuntu-16-04), through the end of Step 6.

**NOTE:** The tomcat version in the article is no longer available. <br/>
Please access to [this url](http://apache.mirrors.ionfish.org/tomcat/tomcat-8/) in order to know the latest version of tomcat 8.5. Then replace the `VERSION_NUMBER` tags in the following link and use it to download tomcat in the curl command:
 
	http://apache.mirrors.ionfish.org/tomcat/tomcat-8/v8.5.<VERSION_NUMBER>/bin/apache-tomcat-8.5.<VERSION_NUMBER>.tar.gz

As an example, if your version of tomcat is `8.5.28`, your curl command would look like the below:
	
	curl -O http://apache.mirrors.ionfish.org/tomcat/tomcat-8/v8.5.28/bin/apache-tomcat-8.5.28.tar.gz


Open `/opt/tomcat/conf/server.xml` and find this line:

    <Connector port="8080" protocol="HTTP/1.1"
           connectionTimeout="20000"
           redirectPort="8443"/>

and add the maxPostSize attribute:

    <Connector port="8080" protocol="HTTP/1.1"
           connectionTimeout="20000"
           redirectPort="8443"
           maxPostSize="35728640"/>

This will increase the max allowable file size for upload.  Many CIS-CAT Pro Assessor ARF reports will be larger than the default size.

Open `/opt/tomcat/bin/catalina.sh` and add the following lines to the top of the file, which will add some environment variables that CIS-CAT Pro Dashboard requires to run.

    export CCPD_CONFIG_FILE="/opt/tomcat/ccpd-config.yml"
    export CCPD_LOG_DIR="/opt/tomcat/logs"

**Note:**  The values of these environment variables can be configured to any location the administrator wishes, and must be to file system locations to which the Tomcat application server can access on startup.

To set the JVM Heap Settings, add the following line in setenv.sh or catalina.sh:

	export CATALINA_OPTS="-Xms1024M -Xmx2048M "


#### Security Considerations ####
As a final step we want to remove the default applications available from the Tomcat install, including the examples and management applications.  These default sites can and will give away information about the environment and present an information security risk.

	# Navigate to the Tomcat applications directory
	cd /opt/tomcat/webapps
	
	# Remove the default applications
	sudo rm -R *  

This will remove all the default applications from the tomcat installation to help reduce the attack surface of the application server.

<a name="confAndDeploymentInstaller"></a>
##Configuration and Deployment - Installer##
<b>This section describes how to configure and deploy the Dashboard using the Installer.  For instructions on how to configure and deploy the Dashboard manually, see [Configuration and Deployment - Manual](#confAndDeploymentManual).</b>

The permissions on the configuration file (ccpd-config.yml) and the logs directory for CIS-CAT Pro Dashboard need to allow the tomcat user to read and/or write.

Locate the latest version of CIS-CAT Pro Dashboard, pinned at the top of the Downloads section, from [CIS WorkBench](https://workbench.cisecurity.org/). Download the CIS-CAT Pro Dashboard Unix bundle from [CIS WorkBench](https://workbench.cisecurity.org/). Place and extract the bundle on your tomcat instance.

After the CIS-CAT Pro Dashboard Unix bundle has been extracted, please confirm that its contents look similar to the following image:

![](https://i.imgur.com/h8sedNq.png)				

We recommend that the Tomcat application server has been stopped before continuing. Additionally, ensure that component installation including installation of Java8, Tomcat 8.5 and a Database (MySQL, SQL Server or Oracle) has been completed before continuing.

Execute the CIS-CAT Pro Dashboard Installer (`CIS-CAT_Pro_Dashboard_Installer.sh` in this example).

You must be logged in as root or as a user that has root privileges (use "sudo" or "su" to elevate your privileges) when you run the CIS-CAT Pro Dashboard Installer.

####Welcome (First-time user)####
First-time users of the CIS-CAT Pro Dashboard Installer tool will be presented with the below screen. On this screen you can also see the location of the temporary installation log.

![](https://i.imgur.com/kshuehK.png)

####Welcome (From previous installation)####
Users with previous successful use of CIS-CAT Pro Dashboard Installer tool will be presented with the below screen. The screens will navigate only to the information required to be collected for the installation actions selected.

![](https://i.imgur.com/PX7OErd.png)

####Preload configuration file####

The Installer might be able to detect an existing CIS-CAT Pro Dashboard configuration file (`ccpd-config.yml`). If one is found you will be given the choice of using that one or manually entering the location of one, in order to pre-populate some of the Installer screens. The screens will navigate only to the information required to be collected for the installation actions selected.

![](https://i.imgur.com/cBSRjH1.png)

![](https://i.imgur.com/APv8rx1.png)

####Installation Actions####
As part of Installation Actions, at least one action must be selected in order to navigate to the next step. As an option, the actions can also be completed together. Both actions must be completed successfully (together or separately) as part of the overall CIS-CAT Pro Dashboard Deployment Guide. The screens will navigate only to the information required to be collected for the installation actions selected.

![](https://i.imgur.com/V6MyIj1.png)

####Configuration File Location####
If this is a new installation and “Create/update CIS-CAT Pro Dashboard configuration file (ccpd.config.yml)” was previously selected, the Configuration File Location screen will be presented. This is the location where CIS-CAT Pro Dashboard configuration file (`ccpd-config.yml`) will be created.

![](https://i.imgur.com/wyxp8Ag.png)

####Application Server Location####
For users performing the Installation Action, “This action deploys the WAR file to the application server. The WAR file has to be present in the same location as this installer. Install/update the CIS-CAT Pro Dashboard application”, use the below screen to specify the application server home directory. The default value appearing in the field is the recommended location for the application server. However, each environment may vary. For example, if the Tomcat home directory is `C:\tomcat`, then the CCPD.war will be created under `C:\tomcat\webapps\CCPD.war`.

![](https://i.imgur.com/iBxuUXQ.png)

####Import Directory####
It is required to setup processing folders that the Dashboard will use while importing files.Example report folder structure is shown within the CIS-CAT Pro Dashboard Installer.

![](https://i.imgur.com/YtqoBnL.png)

####Environment Variables####
Specify the environment variables needed by the CIS-CAT Pro Dashboard. CCPD_CONFIG_FILE points to CIS-CAT Pro Dashboard runtime configuration file (`ccpd-config.yml`). CCPD_LOG_DIR is the logs directory for CIS-CAT Pro Dashboard.

![](https://i.imgur.com/7vH7u1j.jpg)

####Application Server URL
Specifies the application URL of the CIS-CAT Pro Dashboard application. Example formats are shown within the CIS-CAT Pro Dashboard Installer.

![](https://i.imgur.com/2bUnL1f.png)

####Email Configuration####
The email configuration information is optional and is intended for users that want to send email messages such as password reset requests. CIS-CAT Pro Dashboard must be able to connect to and utilize a valid SMTP server in order to send email messages. CIS-CAT Pro Dashboard utilizes the Grails mail plugin for email communication.
Along with the default sender email address, CIS-CAT Pro Dashboard's mailing configuration must also include connection to a valid SMTP server in order to correctly distribute the "forgot password" messages. Numerous SMTP services exist, such as Gmail, Hotmail, Amazon SES, or in-house SMTP services available through corporate emailing technologies, such as Exchange. CIS-CAT Pro Dashboard can support these SMTP servers, as long as the connection information entered below is correct. By default, the plugin assumes an unsecured mail server configured at `localhost` on `port 25`. However, this can be modified in the email configuration screen.

![](https://i.imgur.com/KFgNYhC.png)

![](https://i.imgur.com/PwLvG3Z.png)

####Database Configuration####
The primary purpose of this screen is to assist in establishing a connection to the database for the CIS-CAT Pro Dashboard. Three types of databases are currently supported: MySQL, SQL Server and Oracle. Optional function available in this screen:

 - **Test Database Connection:** Enter correct Hostname/IP, Port, Username, Password, and Schema name and select the “Test Database Connection” button. A message will indicate if the connection was successful.

![](https://i.imgur.com/Hvs91zN.png)

![](https://i.imgur.com/l2mYhy8.png)

####Summary####
The Summary screen is intended for a final review of all information provided in previous screens. If any information is incorrect, you can cancel and navigate to the appropriate screen and make a correction.

![](https://i.imgur.com/9k1UT1I.png)

![](https://i.imgur.com/ytHeFkf.png)

####Installation####
The system may ask for permission to create a backup of the current configuration file (`ccpd-config.yml`) and/or a backup of the current CCPD.war file. This is a recommended procedure.

The installer does not preserve or setup LDAP configuration. This is done manually using the backup file to merge any existing LDAP settings to the latest ccpd-config.yml

![](https://i.imgur.com/kOuO5kl.jpg)

####Complete####
If the installer process was successful, the Complete screen will be presented. On this screen you can also see the location of the final installation log, which is created with a unique date and timestamp added to its name. The tomcat application server can now be started if it was previously stopped.

![](https://i.imgur.com/PiIB6RR.jpg)

####Installer Logs
During the installation, the Installer will create logs. The logs will be created in a directory within the temporary directory of the operating system. Each finished installation will create an individual log with a timestamp. If you have trouble with the installation, please provide this log file to [support@cisecurity.org](mailto://support@cisecurity.org). 

The permissions on the configuration file (ccpd-config.yml) and the logs directory for CIS-CAT Pro Dashboard need to allow the tomcat user to read and/or write.

<a name="confAndDeploymentManual"></a>
##Configuration and Deployment - Manual##
<b>This section describes how to configure and deploy the Dashboard manually.  For instructions on how to configure and deploy the Dashboard using the Installer, see [Configuration and Deployment - Installer](#confAndDeploymentInstaller).</b>

At this point, it is prudent to deploy the CIS-CAT Pro Dashboard application (the "war") included in the downloadable bundle.  The initial deployment of the web application can then be tested via direct connection to the application server on port `8080`, without the web server.  This can help identify any application or application server misconfigurations prior to web server installation/configuration.


The permissions on these files need to allow the tomcat user to read/write.  The user who will be managing the Legacy imports will need write access to the Legacy folder.

Create the CIS-CAT Pro Dashboard runtime configuration file: `/opt/tomcat/ccpd-config.yml`, including the following lines:

	environments:
	    production:
	        grails:
	                serverURL: 'http://<url_of_apache_or_tomcat>:8080/CCPD' #if apache is not set up, only tomcat
	                #serverURL: 'http://<url_of_apache_or_tomcat>/CCPD' #if apache is set up use
	                #serverURL: 'https://<url_of_apache_or_tomcat>/CCPD' #if HTTPS is set up use
	        server:
	            'contextPath': '/CCPD'
	        dataSource:
	            dbCreate: update
	
	            #MySQL DB Settings
	
	            driverClassName: org.mariadb.jdbc.Driver
	            dialect: org.hibernate.dialect.MySQL5InnoDBDialect
	            url: jdbc:mysql://<path_to_mysql_database_server>:3306/<schema_name_of_mysql_database>
	            username: <db_user>
	            password: <db_password>
	
	            #SQL Server DB Setting
	
	            #driverClassName: com.microsoft.sqlserver.jdbc.SQLServerDriver
	            #dialect: org.hibernate.dialect.SQLServer2008Dialect
	            #url: jdbc:sqlserver://<path_to_mysql_database_server>:1433;databaseName=<schema_name_of_database>
	            #username: <db_user>
	            #password: <db_password>
	
	            #Oracle DB Settings
	
	            #driverClassName: oracle.jdbc.OracleDriver
	            #dialect: org.hibernate.dialect.Oracle10Dialect
	            #url: jdbc:oracle:thin:@<path_to_mysql_database_server>:1521:<schema_name_of_database>
	            #username: <db_user>
	            #password: <db_password>
	
	            properties:
	                  jmxEnabled: true
	                  initialSize: 5
	                  maxActive: 50
	                  minIdle: 5
	                  maxIdle: 25
	                  maxWait: 10000
	                  maxAge: 600000
	                  # validationQuery: SELECT 1 from DUAL #ORACLE
	                  validationQuery: SELECT 1 #Non-Oracle
	                  validationQueryTimeout: 3
	                  validationInterval: 15000
	                  defaultTransactionIsolation: 2 # TRANSACTION_READ_COMMITTED
	                  #defaultTransactionIsolation: 1 #SQL SERVER ONLY
	                  dbProperties:
	                        autoReconnect: true
	
	
	grails:
	    mail:
	        host: <smtp server host name>
	        port: 465
	        username: <username>
	        password: <password>
	        props:
	            mail.transport.protocol: "smtps"
	            mail.smtp.port: "465"
	            mail.smtp.auth: "true"
	            mail.smtp.starttls.enable: "true"
	            mail.smtp.starttls.required: "true"
	
	database: MySQL
	#database: SQLServer
	#database: Oracle


This file is also available in the CIS-CAT-Pro-Dashboard bundle, you will just need to replace the pertinent information, marked with <>, with the specifics of your environment.


**NOTE: MySQL Timezone setting** It is important for the timezone on the application server and the database server to be the same.  If this is not the case in your environment you can set the timezone of the database connection using a jdbc option:

	jdbc:mysql://<path_to_mysql_database_server>:3306/<schema_name_of_mysql_database>?useLegacyDatetimeCode=false&serverTimezone=<3-letter-timezone>

**MySQL 5.7 specific:** <br/>
If you use MySQL 5.7 version, you need to replace:
 
	database: MySQL

by:

	database: MySQL.5.7

### Database Configuration ###
By default the ccpd-config.yml is configured to utilize a MySQL database.  Starting with version v1.0.3 you will be able to use MS SQL Server and Oracle Databases as well.  In the ccpd-config.yml, there are several settings you need to make to utilize these other DBMS:


**SQL Server**

You will need to comment out the MySQL configuration and uncomment the SQL Server Section:

	#MySQL DB Settings
	
    #driverClassName: org.mariadb.jdbc.Driver
    #dialect: org.hibernate.dialect.MySQL5InnoDBDialect
    #url: jdbc:mysql://<path_to_mysql_database_server>:3306/<schema_name_of_mysql_database>
    #username: <db_user>
    #password: <db_password>

    #SQL Server DB Setting

    driverClassName: com.microsoft.sqlserver.jdbc.SQLServerDriver
    dialect: org.hibernate.dialect.SQLServer2008Dialect
    url: jdbc:sqlserver://<path_to_mysql_database_server>:1433;databaseName=<schema_name_of_database>
    username: <db_user>
    password: <db_password>

Then enter the connection information appropriate to your SQL Server database.

In the database properties section, comment out the following property for SQL Server:

	#defaultTransactionIsolation: 2 # ORACLE AND MYSQL
	defaultTransactionIsolation: 1 #SQL SERVER ONLY

At the very bottom of the file comment out the database entry for MySQL and uncomment the entry for SQL Server:

	#database: MySQL
	database: SQLServer
	#database: Oracle


**Oracle**

You will need to comment out the MySQL configuration and uncomment the SQL Server Section:

	#MySQL DB Settings
	
    #driverClassName: org.mariadb.jdbc.Driver
    #dialect: org.hibernate.dialect.MySQL5InnoDBDialect
    #url: jdbc:mysql://<path_to_mysql_database_server>:3306/<schema_name_of_mysql_database>
    #username: <db_user>
    #password: <db_password>

    #Oracle DB Settings
	
    driverClassName: oracle.jdbc.OracleDriver
    dialect: org.hibernate.dialect.Oracle10Dialect
    url: jdbc:oracle:thin:@<path_to_mysql_database_server>:1521:<schema_name_of_database>
    username: <db_user>
    password: <db_password>

Then enter the connection information appropriate to your Oracle database.


At the very bottom of the file comment out the database entry for MySQL and uncomment the entry for Oracle:

	#database: MySQL
	#database: SQLServer
	database: Oracle

You will also need to comment out the validationQuery in the dataSource properties, and replace it with the one appropriate to Oracle:

	validationQuery: SELECT 1 from DUAL #ORACLE
	#validationQuery: SELECT 1 #Non-Oracle

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
			    mail.smtp.auth: "true"
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

And the configuration for sending via a Outlook account:
#### *Mail Configuration - Outlook* ####
    ---
    grails:
	    mail:
		    host: "smtp-mail.outlook.com"
		    port: 587
		    username: "youracount@outlook.com"
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

**Security Considerations**

We want the ccpd-config.yml to be as secure as possible.  Only the tomcat user needs to be able to read the file.  We recommend limiting permissions to the file by running the following commands:

	> sudo chown tomcat ccpd-config.yml
	> sudo chgrp tomcat ccpd-config.yml
	> sudo chmod 400 ccpd-config.yml

### Deploy WAR ###
Connect to the application server and transfer the `CCPD.war` to the `/opt/tomcat/webapps` directory.  If Tomcat is running, it should automatically deploy the application.  If Tomcat is not running, starting it will deploy the application.

Once Tomcat is done with its deployment you should be able to access the application by entering `http://<public url of application server>:8080/CCPD/`, into a browser.

You may need to restart tomcat in order to complete the deployment,  you can do so using the following command

	> sudo service tomcat restart

CIS-CAT Pro Dashboard will bootstrap in a user with:

    username: admin
    password: @admin123

**Notes**:
Your network must allow traffic on 8080
(If deployed to AWS) Your security group in AWS must allow traffic on 8080

### Web Server ###
The final configuration is for a web server to proxy the Tomcat instance and to answer traffic on port `80/443`.

#### Apache HTTP Server for Ubuntu####
Install Apache 2.4 for Ubuntu by running: 
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
    sudo service apache2 restart

Execute the following commands to enable the reverse proxy to Tomcat:

    sudo a2ensite ccpd.conf
    sudo service apache2 reload

The Apache HTTP server should now be configured to serve requests to CIS-CAT Pro Dashboard.  The application should be available at the URL entered into your `ccpd-config.yml`: `http://<public url http server>/CCPD`

### Securing Web Traffic ###
The steps above will have the CIS-CAT Pro Dashboard application running over normal HTTP on port `80`.  This presents a risk as data, including user credentials, will be transmitted in clear text.  It is recommended that traffic be secured using HTTPS.  The following article explains how to create a self-signed certificate and apply it to a web server in Ubuntu 16.04: [https://www.digitalocean.com/community/tutorials/how-to-create-a-self-signed-ssl-certificate-for-apache-in-ubuntu-16-04](https://www.digitalocean.com/community/tutorials/how-to-create-a-self-signed-ssl-certificate-for-apache-in-ubuntu-16-04)

**NOTE:** 
After completing the steps in the article, you will have to change your connector in ccpd.conf to use port 443 instead of port 80.  Also, when you chose a common name in the certificate creation process, the name must match the DNS name portion of the serverURL in ccpd-config.yml.  i.e.  if your serverURL is https://www.example.com/CCPD, then the common name in the certificate must be www.example.com

### Ensuring Trust ###
In the browser's URL bar, navigate to the CIS-CAT Pro Dashboard application.  Click on the HTTPS certificate chain (next to URL address). In the Google Chrome browser, if the user sees a "Not Secure" label next to the URL, the certificate is not trusted.  Click on the "Not Secure" link to display the certificate information:

- Click the "Details" link to display the "Security Overview" information.
- Click on the "View Certificate" button to display the certificate details.

The certificate information needs to be exported, and added to the servers trust store.  On the "details" tab of the certificate information, click the "Copy to File..." button.  The "certificate export wizard" will be displayed.

- Click the "Next" button to navigate to the second screen, the export type selection:

Select the "DER encoded binary X.509 (.CER)" radio button.  This selection should be the default.  Click "Next" to continue.

Select or browse to a location to where the `.cer` file will be saved.  Clicking "Next" and finally "Finish" to complete the wizard and save the file to the selected location/filename.  This file will need to be transferred via SCP, S/FTP to the application server hosting the CIS-CAT Pro Dashboard application.  Note the location to which the file is transferred.  For the purposes of this User's Guide, assume the location of the `ccpd_cert.cer` file is `/home/ubuntu/ccpd_cert.cer`.

#### Importing the certificate into the java trust store ####
This step is required if you want to use the "POST Reports to URL" with https in order to upload an Asset Report Format (ARF) results from CIS-CAT Pro Assessor to CIS-CAT Pro Dashboard.
For more details about the integration between the Assessor and the Dashboard, please read the "Importing CIS-CAT Assessor Results" section of CIS-CAT Pro Dashboard User's Guide.

Once the file has been transferred to the application server, locate the `$JAVA_HOME` file location, i.e. the filesystem location of the java executable being used to launch Tomcat and host CIS-CAT Pro Dashboard.  For the purposes of this guide, assume the location of `$JAVA_HOME` can be found at `/usr/lib/jvm/java-8-openjdk-amd64`.  When using Java 8, the required `cacerts` file will be located at `$JAVA_HOME/lib/security/cacerts`.  The location of the `cacerts` file should be noted before importing the certificate using `keytool`.

Finally, using the java `keytool` application, import the certificate:

    # Navigate to the $JAVA_HOME folder's bin directory.
    cd /usr/lib/jvm/java-8-openjdk-amd64/bin
     
    # Execute keytool
    sudo keytool -import -alias ccpd -keystore ../lib/security/cacerts -file /home/ubuntu/ccpd_cert.cer
The user should be prompted to enter the keystore credentials.  By default, this credential is `changeit`.

    # The user will be asked for confirmation
    Trust this certificate? [no]:

Answering `yes` to this confirmation prompt will import the certificate into the trust store.

**NOTE**:  If the application server is currently running, it must be restarted in order to incorporate the trust store.

###Set up Import Folders###
In order to import files into the Dashboard, you need to set up the temp folders that it uses for processing.  Login to the system as a user with ROLE_ADMIN, and set the following three settings to directories on the server running the dashboard application:

	legacy.sourceDir = /opt/legacy/source
	legacy.processedDir = /opt/legacy/processed
	legacy.errorDir = /opt/legacy/error

Make sure the tomcat user has read/write permissions to the above directories, here is the command:

	sudo chown -R tomcat /opt/legacy
	
You can also set the processedRetention number, which determines how many processed xml files are saved in the process directory after being imported into the Dashboard:

	legacy.processedRetentionNumber = 1000


##Upgrading from a previous version##

To upgrade from an earlier version of CCPD:

1) stop your application server.  if you followed the deployment guide that would be tomcat, and you would stop with:  

	sudo service tomcat stop

2) copy the new CCPD.war into the tomcat/webapps directory.

3) Start the application server:  

	sudo service tomcat start

That will redeploy the new war over the old application.  CCPD has a bootstrap initialization script that will run on its first deployment that will make necessary data changes.

If there are any additional upgrading steps specific to a release version, information about those steps will be included in that versions README.txt


##LDAP/Active Directory Integration (Optional)##
With this integration, LDAP/Active Directory will be used to manage user authentication and permissions within CCPD. 

Users will use their LDAP/AD credentials to log into the application. LDAP/AD roles and user properties such firstname, lastname and email will be imported. If the user doesn't exist in CCPD, he will be created when he logs in, and granted with basic user roles (ROLE_BASIC\_USER and ROLE\_USER) by default, plus additional LDAP Roles.

####LDAP Configuration####
Here is an example of LDAP structure in OpenLDAP:
![](https://i.imgur.com/6nNTW0X.png)

Based on the above structure, add the following section in the `/opt/tomcat/ccpd-config.yml` file (noted above): 
		
	---
	grails:
		plugin:
			springsecurity:
				providerNames: ['ldapAuthProvider','rememberMeAuthenticationProvider','restAuthenticationProvider','anonymousAuthenticationProvider']
				ldap:
					active: true
					context:
						managerDn: 'cn=Manager,dc=cisecurity,dc=org'
						managerPassword: 'my_manager_password'
						server: 'ldap://127.0.0.1:389'
					authorities:
						retrieveDatabaseRoles: false                   
						retrieveGroupRoles: true
						groupSearchBase: 'ou=Groups,dc=cisecurity,dc=org'                  
						groupSearchFilter: 'uniquemember={0}'
						groupRoleAttribute: 'cn'        
						clean:
							prefix: 'CCPD_'
					search:
						base: 'dc=cisecurity,dc=org' 
						filter: '(uid={0})'
					authenticator:
						passwordAttributeName: 'userPassword'
					mapper:
						passwordAttributeName: 'userPassword'               
					useRememberMe: true  
				rememberMe:
					persistent: true

**Note**: If you have previously configured the application to connect to a SMTP server, you need to remove `grails:` from the above section. So the section will start with `plugin:` instead.
   
####Active Directory Configuration####
Here is an example of Active Directory structure in Windows server 2016:
![](https://i.imgur.com/BsKQRuq.png)

Based on the above structure, add the following section in the `/opt/tomcat/ccpd-config.yml` file (noted above): 
		
	---
	grails:
		plugin:
			springsecurity:
				providerNames: ['ldapAuthProvider','rememberMeAuthenticationProvider','restAuthenticationProvider','anonymousAuthenticationProvider']
				ldap:
					active: true
					context:
						managerDn: 'CN=Administrator,CN=Users,DC=corp,DC=cisecuritytest,DC=org'
						managerPassword: 'my_manager_password'
						server: 'ldap://127.0.0.1:389'
					authorities:
						ignorePartialResultException: true
						retrieveDatabaseRoles: false                   
						retrieveGroupRoles: true
						groupSearchBase: 'DC=corp,DC=cisecuritytest,DC=org'                  
						groupSearchFilter: 'member={0}'
						groupRoleAttribute: 'CN'     
						clean:
							prefix: 'CCPD_'                 
					search:
						base: 'DC=corp,DC=cisecuritytest,DC=org' 
						filter: 'sAMAccountName={0}'
					auth:
						hideUserNotFoundExceptions: false
					authenticator:
						passwordAttributeName: 'userPassword'
					mapper:
						passwordAttributeName: 'userPassword'               
					useRememberMe: true                                 
				rememberMe:
					persistent: true

**Note**: If you have previously configured the application to connect to a SMTP server, you need to remove `grails:` from the above section. So the section will start with `plugin:` instead.

####Configuration Options####
Here is a description of some configuration options used for LDAP/AD integration:

- **managerDn/managerPassword:** manager credential to access to LDAP server.

- **server:** URL of the LDAP server.

- **groupSearchBase:** the base directory to start the group search. Note: if  we don't want to retrieve groups (roles) from LDAP, set retrieveGroupRoles to false.

- **groupSearchFilter:** the pattern to be used for the user search. {0} is the user’s DN.

- **groupRoleAttribute:** the ID of the attribute which contains the role name for a group.

- **clean.prefix:** an optional string prefix to strip from the beginning of LDAP group names. For example, 'CCPD\_' will convert CCPD\_ADMIN to ROLE\_ADMIN

- **base:** the base directory to start the search.

- **filter:** the filter expression used in the user search. For LDAP, uid field is typically used as username for the filter. In Active Directory, sAMAccountName (also called "User logon name") is typically used. 

- **passwordAttributeName:** the name of the password attribute to use.

####LDAP/AD Requirements####
The email address is a required field, make sure that LDAP/AD user email field is set properly.

The entire group name needs to be in uppercase in LDAP/AD. 

If some users were previously created in CCPD before the LDAP integration, make sure the username matches with the one in LDAP (uid) or AD (sAMAccountName, also called "User logon name").  

The api user needs to be created in LDAP/AD in order to generate an authentication token to import Asset Report Format (ARF) results from CIS-CAT Assessor. 

Once LDAP/AD authentication is integrated to CCPD, the database authentication will be automatically disabled.




