# CIS-CAT Pro Dashboard Deployment Guide #

----------
## Introduction ##
CIS-CAT Pro Dashboard is a web application built using the Grails Framework.  The grails framework uses a hibernate data model to remain DBMS agnostic.  Grails code compiles into java byte code and therefore runs on the JVM.  As such,  CIS-CAT Pro dashboard can be deployed in any RDBMS which supports JDBC, and on any web container that can handle java WAR's.  The documentation below describes how to deploy CIS-CAT Pro Dashboard on the following stack, which is officially supported by CIS:  MySQL 5.6 running on Ubuntu 16.04, a separate Ubuntu 16.04 server for Tomcat 8 and Apache 2.  Prerequisite to using this documentation is having two separate Ubuntu 16.04 instances running in your environment.

**Reiteration:**  Although this document is written assuming a MySQL database, Ubuntu servers, Tomcat and Apache HTTP Server, they are not required.  The Hibernate data model ensures a database-agnostic environment, so any relational database management system can be used (MySQL, Oracle, MS SQLServer, etc.).  Further, any operating system can host the application server, which can also utilize any software capable of hosting a java web application archive (.war file).

### Database ### 
Install MySQL Server 5.6 onto the database Ubuntu server using these instructions: https://www.digitalocean.com/community/tutorials/how-to-install-mysql-on-ubuntu-16-04
By the end you know you have a running MySQL server and you need to note the following information:
hostname/IP of database server
port for connecting, by default this is 3306
username/password for connecting to the database
Application Server 
On a separate server from the database server, install Ubuntu 16.04.  It is important that your database server be separated from your application server for security reasons.  Your application server needs to be accessible over the network so that you can access your application, but if the application database were on the same server, it would also be exposed.  By having a separate server, your database can be configured to only communicate over the network via the MySQL port, and therefore be more secure.  Secondly, this configuration is more scalable.  You can add application servers to the environment as your load increases and have the applications communicate with the same database server.
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

