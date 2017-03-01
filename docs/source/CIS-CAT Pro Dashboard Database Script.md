![](http://i.imgur.com/5yZfZi5.jpg)

# CIS-CAT Pro Dashboard Database Script #
*For use with CIS-CAT Pro Dashboard*

----------
To insure that CIS-CAT Pro Dashboard has the latest CIS Benchmark Content, it is recommended that you use the bundled SQL script to import the most recent versions.
To run the script, first follow the CIS-CAT Pro Dashboard Deployment Guide.  Prior to importing any assessment results run the following command:

	mysql -h <hostname of DB server> -u <username of DB user> -p <schema name> < CIS-CATProDBStarter.sql

you'll need to replace:

<hostname of DB server> with either the IP or the hostname of the server where your CIS-CAT Pro Dashboard MySQL server is installed

<username of DB user> with a username that can access that database

<schema name> with the name you chose for your CIS-CAT Pro Dashboard database Schema.
 
The command will prompt you for a password due to the '-p' option.

This command will take several minutes to run, but once complete you will have a pristine set of CIS content for your Dashboarding and Reporting.  You can then begin to import your assessment results after following the CIS-CAT Pro Dashboard Deployment Guide - CIS-CAT
 