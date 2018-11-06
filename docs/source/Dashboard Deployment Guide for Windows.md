![](http://i.imgur.com/5yZfZi5.jpg)

# CIS-CAT Pro Dashboard Deployment Guide for Windows #

----------
## Introduction ##
CIS-CAT Pro Dashboard is a web application built using the Grails Framework.  The grails framework uses a hibernate data model to remain DBMS agnostic.  Grails code compiles into java byte code and therefore runs on the JVM.  As such,  CIS-CAT Pro dashboard can be deployed in many RDBMS, and on any web container that can handle java WAR's.  The documentation below describes how to deploy CIS-CAT Pro Dashboard on the following stack, which is officially supported by CIS:  SQL Server 2017 running on Windows Server 2016, a separate Windows 2016 server for Tomcat 8.5 and IIS 10.0.  Prerequisite to using this documentation is having two separate Windows Server 2016 instances running in your environment.

**Reiteration:**  Although this document is written assuming a SQL Server database, Windows 2016 servers, Tomcat and IIS web server, they are not required.  The Hibernate data model ensures a database-agnostic environment, so many relational database management system can be used (i.e. MySQL until version 5.7 (including MariaDB),MS SQL Server, and Oracle).  Further, any operating system can host the application server, which can also utilize any software capable of hosting a java web application archive (.war file).

## System Recommendations ##
 There are no strict requirements associated with our Dashboard application. Any OS will be suitable so long as it can run Tomcat. Disk space will be minimal on the application server, but will require more space on your database server depending upon the size of your organization and the amount of endpoints you have.

Our test environment uses an AWS t2.large instance (designed for burst processing), which has:

 - 8GB RAM
 - 2 vCPU’s with 4 cores each
 - Microsoft Windows Server 2016 Base
 
The application is fairly lightweight on processor and memory use. However, when importing results, the usage will spike. Our recommendation would be to conduct importing off hours and then the functionality of the application will not be hindered during business hours.

\*\*Please be advised that it is a necessity for port 1433 and 8080 to be included as inbound rules\*\*

## Component Deployment ##
The following sections describe the installation and configuration of all components necessary to deploy CIS-CAT Pro Dashboard.

###Database###
#### SQL Server 2017 ###
 
Downloaded SQL Server 2017 from [the official website.](https://www.microsoft.com/en-us/sql-server/sql-server-editions-express)

Install the database following the important steps described below, for more details regarding the installation, please follow [the official guide](https://docs.microsoft.com/en-us/sql/database-engine/install-windows/installation-for-sql-server) or [this unofficial website](https://www.brentozar.com/archive/2017/10/sql-server-2017-installation-screenshot-tour-windows/).

Chose "Custom" type installation and a target location to download.

Once "SQL Server Installation Center" starts, click on "New SQL Server installation or add features to an existing installation"

During the SQL Server installation wizard, make sure that you follow those important steps:

- In "Install Rules" step, you might have a warning regarding Windows firewall ports which needs to be opened for TCP/IP, as following. Please ignore the warning, the firewall configuration will be set up during the next section.

![](https://i.imgur.com/GCYkrqN.png)

- In "Instance Configuration" step, make sure that you select **default instance** instead of "named instance". The default instance will listen on a static port (1433). The "named instance" will listen on a dynamic port which changes every time the server restarts.
  
![](https://i.imgur.com/hMxsvbn.png)
 
- In "Database Engine Configuration" step, select Mixed Mode for the authentication.
This allows the username and password to be added later to CIS-CAT Pro Dashboard configuration file (ccpd-config.yml). You also need to set the password of the "sa" user.

![](https://i.imgur.com/vyz7xas.png)

Please note the following information in order to configure later SQL Server client as well as CIS-CAT Pro Dashboard application:<br/>
- hostname/IP of database server (machine which hosts the database)<br/>
- port for connecting, by default this is 1433<br/>
- username/password for connecting to the database<br/>

#### Windows Firewall Configuration for SQL Server####
To access an instance of the SQL Server through a firewall, Windows firewall ports needs to be opened for TCP/IP port 1433.

Open Windows Firewall with Advanced Security application. Then in the left pane, right-clicked Inbound Rules, and click New Rule. In the Rule Type dialog box, select Port, then click Next. In the Protocol and Ports dialog box, select TCP. In Specific local ports, put 1433 (default instance SQL Server port), see screenshot. Then in the Action dialog box, select Allow the connection, then click Next.<br/>
For more information, please follow [the official website](https://docs.microsoft.com/en-us/sql/sql-server/install/configure-the-windows-firewall-to-allow-sql-server-access)

![](https://i.imgur.com/BIvxqBs.png)

**Notes:** Your network must also allow traffic on TCP/IP 1433, (If deployed to AWS) your security group in AWS must allow traffic on TCP/IP 1433.

#### Enable TCP/IP in SQL Server Configuration Manager####
Open "SQL Server Configuration Manager" tool and enable TCP/IP as following in "SQL Server Network Configuration/Protocols for <Name_of_the_instance>":

![](https://i.imgur.com/4cvP5t9.png)

Make sure that you restart the database service after, as following:

![](https://i.imgur.com/NJbvdRT.png)

At this point, you can verify that "sqlservr.exe" process is listening to TCP 1433 port by executing the following command in a Command prompt:

	netstat -a -b


#### SQL Server Management Studio 17 (client) ####
A SQL Server client should be installed in the distant server (the second server which aims to deploy the application) in order to test connectivity with the database and create the schema for CIS-CAT Pro Dashboard.

Download the client from [the official website.](https://docs.microsoft.com/en-us/sql/ssms/download-sql-server-management-studio-ssms)

Connected to the distant SQL server using as following:

![](https://i.imgur.com/8T3IzGt.png)
<br/>For "Server name", use the hostname/IP of the database server (machine which hosts the database)<br/>


Then click right on "Databases" folder on the left panel (I.E. Object Explorer) then click on "New Database". Set the Database Name to `ccpd`. 


###Java 8###
Because CIS-CAT Pro Dashboard is a java-based application, members will need to ensure that java is installed. On a Command Prompt, type the following command:

	#Ensure Java is installed at the proper version level (or higher)
	java -version

Ensure that the Java version is displayed as 1.8.0_111 (or higher).
If it's not the case, please install the jdk 1.8 from [the official website](http://www.oracle.com/technetwork/java/javase/downloads/index.html).

### Application Server###

Download tomcat 8.5 32-bit/64-bit Windows Service Installer from [the official website](https://tomcat.apache.org/download-80.cgi).

Install the application server as following, for more details, please follow [the official guide.](https://tomcat.apache.org/tomcat-8.5-doc/)<br/>

Check "Service startup" during the installation, so Tomcat service will start automatically when the computer starts. <br/> I also suggest to keep "Start Menu Items" checked.

![](https://i.imgur.com/JeJCORn.png)

The default tomcat location can be changed during the installation, for the purposes of this User's Guide, assume the tomcat location is set to "C:\tomcat" directory.    

To configure Tomcat Options, click right on system tray tomcat icon then click Configure, see screenshot. If the system tray tomcat icon is not present, go to "Start Menu/Apache Tomcat" and click on "Monitor Tomcat"<br/>
For example, I added `-XX:MaxPermSize=2048m` in java option and set up initial `Initial memory pool` to `1024` and `Maximum memory pool` to `2048`. 

![](https://i.imgur.com/LnauYAR.png)


Tomcat service can be started/stopped from the system tray tomcat monitoring icon or from the windows services screen. 


Open `C:\tomcat\conf\server.xml` and find this line:

    <Connector port="8080" protocol="HTTP/1.1"
           connectionTimeout="20000"
           redirectPort="8443"/>

and add the maxPostSize attribute:

    <Connector port="8080" protocol="HTTP/1.1"
           connectionTimeout="20000"
           redirectPort="8443"
           maxPostSize="25728640"/>

This will increase the max allowable file size for upload.  Many CIS-CAT Pro Assessor ARF reports will be larger than the default size.


**Security Considerations:** <br/>
As a final step we want to remove the default applications available from the Tomcat install, including the examples and management applications. These default sites can and will give away information about the environment and present an information security risk.<br/>
Please delete every directory inside `C:\tomcat\webapps\*` to help reduce the attack surface of the application server.


##CIS-CAT Pro Dashboard Configuration and Deployment - Using the Installer##
### CIS-CAT Pro Dashboard Runtime Configuration File###
Locate the latest version of CIS-CAT Pro Dashboard, pinned at the top of the Downloads section, from [CIS WorkBench](https://workbench.cisecurity.org/). Download the CIS-CAT Pro Dashboard bundle that corresponds to your Java installation (32-bit or 64-bit) from [CIS WorkBench](https://workbench.cisecurity.org/). Place and extract the bundle on your tomcat instance. 
 
After the CIS-CAT Pro Dashboard bundle has been extracted, please confirm that its contents look similar to the following image (64-bit Java version example shown):

![](https://i.imgur.com/xthTnYn.png)

We recommend that tomcat application server has been stopped before continuing. Additionally, ensure that component installation such as installation of Java8, Tomcat 8.5 and a Database (MySQL, SQL Server or Oracle) before continuing.

Execute the CIS-CAT Pro Dashboard Installer (`CIS-CAT_Pro_Dashboard_Installer-x64.exe` in this example).

![](https://i.imgur.com/w2QjvOK.png)

####Welcome (First-time user)####
First-time users of the CIS-CAT Pro Dashboard Installer tool will be presented with the below screen.

![](https://i.imgur.com/6m71Uwp.png)

####Welcome (From previous installation)####
Users with previous successful use of CIS-CAT Pro Dashboard Installer tool will be presented with the below screen.

![](https://i.imgur.com/9hEFkHE.png)

####Installation Actions####
As part of Installation Actions, at least one action must be selected in order to navigate to the next step. As an option, the actions can also be completed together. Both actions must be completed successfully (together or separately) as part of the overall CIS-CAT Pro Dashboard Deployment Guide. The screens will navigate only to the information required to be collected for the installation actions selected.

![](https://i.imgur.com/KyDgIxi.png)

####Configuration File Location####
If this is a new installation <u>and</u> “Create/update CIS-CAT Pro Dashboard configuration File” was previously selected, the Configuration File Location screen will be presented. This is the location where CIS-CAT Pro Dashboard configuration file (`ccpd-config.yml`) will be created.

![](https://i.imgur.com/zQfJDks.png)

####Application Server Location####
For users performing the Installation Action, “Install/update CIS-CAT Pro Dashboard application”, use the below screen to specify the application server home directory. The default value appearing in the field is the recommended location for the application server. However, each environment may vary. For example, if the Tomcat home directory is `C:\tomcat`, then the CCPD.war will be created under `C:\tomcat\webapps\CCPD.war`.

![](https://i.imgur.com/p5YGEBv.png)

####Import Directory####
It is required to setup processing folders that the Dashboard will use while importing files.

![](https://i.imgur.com/K1qmMyc.png)

####Environment Variables####
Specify the Windows environment variables needed by the CIS-CAT Pro Dashboard. `CCPD_CONFIG_FILE` points to CIS-CAT Pro Dashboard runtime configuration file (`ccpd-config.yml`). `CCPD_LOG_DIR` is the logs directory for CIS-CAT Pro Dashboard.

![](https://i.imgur.com/57N5KbG.png)

####Application Server URL 
Specifies the application URL of the CIS-CAT Pro Dashboard application. Example formats are shown within the CIS-CAT Pro Dashboard Installer.

![](https://i.imgur.com/vrbfX7f.png)

####Email Configuration####
The email configuration information is optional and is intended for users that want to send email messages such as password reset requests. CIS-CAT Pro Dashboard must be able to connect to and utilize a valid SMTP server in order to send email messages. CIS-CAT Pro Dashboard utilizes the Grails mail plugin for email communication.

Along with the default sender email address, CIS-CAT Pro Dashboard's mailing configuration must also include connection to a valid SMTP server in order to correctly distribute the "forgot password" messages. Numerous SMTP services exist, such as Gmail, Hotmail, Amazon SES, or in-house SMTP services available through corporate emailing technologies, such as Exchange. CIS-CAT Pro Dashboard can support these SMTP servers, as long as the connection information entered below is correct.
By default, the plugin assumes an unsecured mail server configured at `localhost` on `port 25`. However, this can be modified in the email configuration screen.

![](https://i.imgur.com/nctuocK.png)

####Database Configuration####
The primary purpose of this screen is to assist in establishing a connection to the database for the CIS-CAT Pro Dashboard. Three types of databases are currently supported: MySQL, SQL Server and Oracle. Optional functions are available in this screen:

 - **Create a Schema:** Enter correct Hostname/IP, Port, Username, Password and select the “Create Schema” button. This process currently applies only to MySQL and SQL Server databases. 
 - **Test Database Connection:** Enter correct Hostname/IP, Port, Username, Password, and Schema name and select the “Test Database Connection” button. A message will indicate if the connection was successful.

![](https://i.imgur.com/uGITRPm.png)

####Summary####
The Summary screen is intended for a final review of all information provided in previous screens. If any information is incorrect, the back button can be used to navigate to the appropriate screen and make a correction.

![](https://i.imgur.com/PHRhlma.png)

####Installation####
The system may ask for permission to create a backup of the current configuration file (`ccpd-config.yml`) and/or a backup of the current CCPD.war file. This is a recommended procedure.

![](https://i.imgur.com/7cPUDBf.png)

####Complete####
If the installer process was successful, the Complete screen will be presented. Select Finish to complete the selected Installer actions. The tomcat application server can now be started if it was previously stopped.

![](https://i.imgur.com/KYAkuGj.png)

##CIS-CAT Pro Dashboard Configuration and Deployment - Manual Installation##

### CIS-CAT Pro Dashboard Runtime Configuration File###
Download the CIS-CAT Pro Dashboard bundle from [CIS WorkBench](https://workbench.cisecurity.org/) and place the bundle on the tomcat instance. The latest version of CIS-CAT Pro Dashboard will be pinned at the top of the Downloads section.

Create the CIS-CAT Pro Dashboard runtime configuration file (configured by default for MySQL database): `C:\tomcat\ccpd-config.yml`, and add to the file the following lines:

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
	                  #validationQuery: SELECT 1 from DUAL #ORACLE
                  	  validationQuery: SELECT 1 #Non-Oracle
	                  validationQueryTimeout: 3
	                  validationInterval: 15000
	                  defaultTransactionIsolation: 2 # ORACLE AND MYSQL
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

Replace the pertinent information, marked with <>, with the specifics of your environment.

This file is also available in the CIS-CAT-Pro-Dashboard bundle.


**NOTE:** This file is configured for MySQL database by default. If you want to use SQL Server or Oracle, please follow the instructions in Database Configuration section. 


**MySQL Timezone setting:** It is important for the timezone on the application server and the database server to be the same.  If this is not the case in your environment you can set the timezone of the database connection using a jdbc option:

	jdbc:mysql://<path_to_mysql_database_server>:3306/<schema_name_of_mysql_database>?useLegacyDatetimeCode=false&serverTimezone=<3-letter-timezone>

**MySQL 5.7 specific:**
If you use MySQL 5.7, you need to replace:
 
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

Configuration of the mail plugin will be member-specific, and those configuration items will also be added to the `C:\tomcat\ccpd-config.yml` file, noted above.

#### Default Sender Email Address ####
CIS-CAT Pro Dashboard can be configured with a default "sender" email address, indicating that the application has sent a "forgot password" message, allowing users to enter new logon credentials.  In the `C:\tomcat\ccpd-config.yml` file (noted above), add the following section:

    ---
    grails:
	    plugin:
	    	springsecurity:
	    		ui:
	    			forgotPassword:
	    				emailFrom: "your-email@member-organization.com"

#### SMTP Configuration ####
Coupled with the default sender email address, CIS-CAT Pro Dashboard's mailing configuration must also include connection to a valid SMTP server, in order to correctly distribute the "forgot password" messages.  Numerous SMTP services exist, such as Gmail, Hotmail, Amazon SES, or in-house SMTP services available through corporate emailing technologies, such as Exchange.  CIS-CAT Pro Dashboard can support these SMTP servers, as long as the connection information is correct in the configuration file (the `C:\tomcat\ccpd-config.yml` noted above).

By default the plugin assumes an unsecured mail server configured at `localhost` on port `25`. However you can change this via the application configuration file (the `C:\tomcat\ccpd-config.yml` noted above). 

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


### Environment Variables ###
Create the following Windows Environment variables:
 
![](https://i.imgur.com/ONciwoJ.png)

`CCPD_CONFIG_FILE` points to CIS-CAT Pro Dashboard runtime configuration file (ccpd-config.yml). <br/>
`CCPD_LOG_DIR` is the logs directory for CIS-CAT Pro Dashboard.
   

**Note:**  The values of these environment variables can be configured to any location the administrator wishes, and must be to file system locations to which the Tomcat application server can access on startup.

###Initial War Deployment###
At this point, it is prudent to deploy the CIS-CAT Pro Dashboard application (the "war") included in the downloadable bundle.  The initial deployment of the web application can then be tested via direct connection to the application server on port `8080`, without the web server (IIS).  This can help identify any application or application server misconfigurations prior to web server (IIS) installation/configuration.

Connect to the application server and transfer the `CCPD.war` to the `c:\tomcat\webapps` directory.  If Tomcat is running, it should automatically deploy the application.  If Tomcat is not running, starting it will deploy the application.

Once Tomcat is done with its deployment you should be able to access the application by entering `http://<public url of application server>:8080/CCPD/`, into a browser.

You may need to restart tomcat in order to complete the deployment.

CIS-CAT Pro Dashboard will bootstrap in a user with:

    username: admin 
    password: @admin123

**Notes**:
Your network must allow traffic on 8080. Add an inbound rule in Windows firewall on TCP 8080 (if needed).
If deployed to AWS, your security group in AWS must allow traffic on 8080.

**Notes 2**: If you see this message in the logs: "LegacyImport - Source folder does NOT exist",  please ignore it, Import folder will be set up in the next section (Set up Import Folders).


###Set up Import Folders###
In order to import files into the Dashboard, you need to set up the temp folders that it uses for processing. <br/>

Create the three following directories:

	c:\tomcat\legacy\source
	c:\tomcat\legacy\processed
	c:\tomcat\legacy\error

Then login to the system as a user with ROLE_ADMIN, go to the settings menu (gear icon) and click on "System Settings". Set the three following settings:

	legacy.sourceDir = c:\tomcat\legacy\source
	legacy.processedDir = c:\tomcat\legacy\processed
	legacy.errorDir = c:\tomcat\legacy\error
	
You can also set legacy.processedRetentionNumber setting, which determines how many processed xml files are saved in the process directory after being imported into the Dashboard.  


## Web Server and Securing Web Traffic ##
The final configuration is for a web server to proxy the Tomcat instance and to answer traffic on port `80/443`.

### IIS Web Server Installation###
For the purposes of this User's Guide, IIS 10.0 will be installed in Windows Server 2016.

Install IIS 10.0 using Server Manager application as following:

Open Server Manager from the start menu and click on “Add roles and features”:
![](https://i.imgur.com/J8zLahq.png)

On "Installation type", select "Role-Based" as following:
![](https://i.imgur.com/hKiiz0D.png)


In "Server Selection", select your local server that you want to install IIS.

In "Server Roles". check "Web Server (IIS)" as following:
![](https://i.imgur.com/mAWg2AA.png)

Click "Next" until "Confirmation" then click "Install".<br/>
Once installed, IIS server will appear in Server Manager Dashboard.
  
As a final test, open a web browser and type `http://localhost/`, you should see the default IIS page.

For more details, please follow "Install IIS Through GUI" from [this website](https://www.rootusers.com/how-to-install-iis-in-windows-server-2016/). 

###Initial Site Configuration###
Access Internet Information Service (IIS) Manager tool as following:
![](https://i.imgur.com/2zUjdLS.png)

On the left panel expend the Sites directory and rename "Default Web Site" to "CCPD" as following. 
![](https://i.imgur.com/MXtA9Fz.png)

The advantage of using the Default Web Site is that it will work "out-of-the-box" with all definitions and permissions already taken care of.

**Note:** As a security concern, delete default documents inside `C:\inetpub\wwwroot` directory, it should contain only `iisstart.html` and `iisstart.png` files.

###Configure IIS Reverse Proxy to connect to Tomcat###
In order to configure IIS Reverse Proxy, two IIS modules need to be installed first:<br/>
Download and install "URL Rewrite" module from [this website.](https://www.iis.net/downloads/microsoft/url-rewrite)<br/>
Download and install Application Request Routing module from [this website.](https://www.iis.net/downloads/microsoft/application-request-routing)

In Internet Information Service (IIS) Manager tool, click on "CCPD" site on the left panel. Then double click on "URL Rewrite" icon on the right panel:
![](https://i.imgur.com/PfIvH22.png)

On URL Rewrite screen, click right and select "Add Rule(s)".

On the rule template, select "Reverse Proxy":
![](https://i.imgur.com/bvG7fsN.png)


Create the following Reverse Proxy rule:<br/>
![](https://i.imgur.com/KYEa7tx.png)

**Note:** Public url should use the default http port (80), so no need specify a port in the url.<br/>If your serverURL is http://www.example.com/CCPD, then set www.example.com to "To" field. 

For more details regarding URL Rewrite, please follow [this website.](https://blogs.msdn.microsoft.com/friis/2016/08/25/setup-iis-with-url-rewrite-as-a-reverse-proxy-for-real-world-apps/)

The IIS web server should now be configured to serve requests to CIS-CAT Pro Dashboard. 

Modify your `ccpd-config.yml` in order to use your public url `http://<public url of web server>/CCPD` instead of `http://<url_of_apache_or_tomcat>:8080/CCPD`, then restart tomcat.<br/>
Open a web browser and type `http://<public url of web server>/CCPD`, you should be redirected to CIS-CAT Pro Dashboard login page.

**Note 2**: Your network should now block traffic on 8080. Disable the inbound rule in Windows firewall on TCP 8080 (if needed).

### Securing Web Traffic ###
The steps above will have the CIS-CAT Pro Dashboard application running over normal HTTP on port `80`.  This presents a risk as data, including user credentials, will be transmitted in clear text. It is recommended that traffic be secured using HTTPS.  

#### Creating a Self-Signed Certificate using Windows Powershell ####
The following steps describe how to create a Self Signed Certificate. To request and use a certificate signed by a trusted third-party, please follow [this article.](https://www.digicert.com/csr-creation-ssl-installation-iis-10.htm)

**Note :** The Self-Signed certificate can be created from IIS but it doesn't allow to set the domain (common name), so a warning message will show up on the web browser when you access the website using https (i.e. "NET::ERR\_CERT\_COMMON\_NAME\_INVALID" error in Chrome).

Open Windows Powershell program (start menu/Windows Powershell) then execute the following commands one at a time:
	
	#Create Root Certificate
	New-SelfSignedCertificate -certstorelocation cert:\localmachine\my -dnsname "My Lab Root Certificate Authority" -KeyUsage DigitalSignature,CertSign

	#Replacing <> by the thumbprint returned by the previous command
	#This command will export the root certificate to c:\safeDirectory
	Export-Certificate -Cert cert:\localMachine\my\<Thumprint of Root Certificate> -FilePath "c:\rootCertificate.crt"
	
	#Load the root authority certificate	
	$rootcert = ( Get-ChildItem -Path cert:\LocalMachine\My\<Thumprint of Root Certificate>)
	
	#Create Self-Signed certificate ccpd
	#If your serverURL is https://www.example.com/CCPD, then the DnsName (common name) must be www.example.com  
	New-SelfSignedCertificate -DnsName "<Common name of the web server>"  -CertStoreLocation "cert:\LocalMachine\My" -FriendlyName "ccpd cert" -Signer $rootcert -KeyUsage KeyEncipherment,DigitalSignature 

For more details regarding `New-SelfSignedCertificate` Powershell commands, here is [the official website.](https://docs.microsoft.com/en-us/powershell/module/pkiclient/new-selfsignedcertificate?view=win10-ps)

#### Binding website certificate to IIS site####
Open IIS manager, click on CCPD site on the left panel then click on "Bindings..." on the right panel.

Click "Add" and add the following Site Binding, using "ccpd cert" certification created previously:
![](https://i.imgur.com/xTyBuwa.png)

Once created, remove the other binding for port 80, so the application will be accessible only in https.

In ccpd.conf, modify the serverURL so the url should start by https instead of http.

Restart the application server and the web server.

Now if you open a web browser and access the application using https, a message will show up regarding a problem with the website’s security certificate. This issue will be solved in the next section. (i.e. Importing Root Certificate into Trusted Root Certification Authorities)

#### Importing Root Certificate into Trusted Root Certification Authorities####
After completing "Creating a Self-Signed Certificate using Windows Powershell" section, the Root Certificate should be located in `c:\rootCertificate.crt`.

Open Microsoft Management Console (type "mmc" on Windows search). If "Certificates" folder doesn't show up on the left panel, click on File/"Add Remove Snap-in...":<br/>
![](https://i.imgur.com/wRcsFlX.png)

Double click on "Certificates" then click "Ok".

Now Click right on Certificates/Trusted Root Certification Authorities folder then select "All Tasks/Import":

![](https://i.imgur.com/up9swlU.png)

Click "Next", Browse `c:\rootCertificate.crt` file, Click "Next" then "Next" then "Finish". Verify that the certificate got imported inside "Trusted Root Certification Authorities" folder:

![](https://i.imgur.com/DkDNMdO.png)

Before opening a web browser, close every opened web browser first. If you use chrome click on 3 points/Exit or Ctr+Shift+Q instead of the cross icon. <br/>
Now if you access the application using https, the web browser will indicate that the connection is secure and the certificate is valid. 

Please repeat every step of this section for each client who wants to access the application. 

#### Importing the certificate into the java trust store ####
This step is required if you want to use the "POST Reports to URL" with https in order to upload an Asset Report Format (ARF) results from CIS-CAT Pro Assessor to CIS-CAT Pro Dashboard.
For more details about the integration between the Assessor and the Dashboard, please read the "Importing CIS-CAT Assessor Results" section from CIS-CAT Pro Dashboard User's Guide.

Find the location of the java executable being used to launch Tomcat and host CIS-CAT Pro Dashboard.  For the purposes of this guide, assume the location of java can be found at `C:\Program Files\Java\jdk1.8.0_144`.  When using Java 8, the required `cacerts` file will be located at `C:\Program Files\Java\jdk1.8.0_144\jre\lib\security\cacerts`.  The location of the `cacerts` file should be noted before importing the certificate using `keytool`.

Finally, using the java `keytool` application, import the certificate:

    # Navigate to the $JAVA_HOME folder's bin directory.
    cd C:\Program Files\Java\jdk1.8.0_144\jre\bin
     
    # Execute keytool
    keytool -import -alias ccpd -keystore ../lib/security/cacerts -file c:\rootCertificate.crt
The user should be prompted to enter the keystore credentials.  By default, this credential is `changeit`.

    # The user will be asked for confirmation
    Trust this certificate? [no]:

Answering `yes` to this confirmation prompt will import the certificate into the trust store.

**NOTE**:  If the application server is currently running, it must be restarted in order to incorporate the trust store.

##Upgrading from a previous version##

To upgrade from an earlier version of CCPD:

There are two ways to upgrade your current CCPD version.

You can follow the instructions given above and do so by using the CIS-CAT Pro Dashboard Installer or you can follow the manual instructions provided below:

1) Stop your application server. If you followed the deployment guide that would be tomcat.<br/>Tomcat service can be stopped from the system tray tomcat monitoring icon or from the windows services screen:  
![](https://i.imgur.com/S3FJsMG.png)

2) Copy the new CCPD.war into the c:/tomcat/webapps directory.

3) Start the application server, the service can be started from the system tray tomcat monitoring icon or from the windows services screen:   

![](https://i.imgur.com/po7YGqR.png)

That will redeploy the new war over the old application.  CCPD has a bootstrap initialization script that will run on its first deployment that will make necessary data changes.

If there are any additional upgrading steps specific to a release version, information about those steps will be included in that versions README.txt


##LDAP/Active Directory Integration (Optional)##
With this integration, LDAP/Active Directory will be used to manage user authentication and permissions within CCPD. 

Users will use their LDAP/AD credentials to log into the application. LDAP/AD roles and user properties such firstname, lastname and email will be imported. If the user doesn't exist in CCPD, he will be created when he logs in, and granted with basic user roles (ROLE_BASIC\_USER and ROLE\_USER) by default, plus additional LDAP Roles.

###LDAP Configuration###
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
   
###Active Directory Configuration###
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

###Configuration Options###
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

###LDAP/AD Requirements###
The email address is a required field, make sure that LDAP/AD user email field is set properly.

The entire group name needs to be in uppercase in LDAP/AD. 

If some users were previously created in CCPD before the LDAP integration, make sure the username matches with the one in LDAP (uid) or AD (sAMAccountName, also called "User logon name").  

The api user needs to be created in LDAP/AD in order to generate an authentication token to import Asset Report Format (ARF) results from CIS-CAT Assessor. 

Once LDAP/AD authentication is integrated to CCPD, the database authentication will be automatically disabled.
