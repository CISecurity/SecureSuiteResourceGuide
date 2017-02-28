![](http://i.imgur.com/5yZfZi5.jpg)

# CIS-CAT Pro Dashboard User's Guide #

## Introduction ##
CIS-CAT Pro Dashboard is a new companion application for CIS-CAT.  The application features a database back end for storing individual assessment results from target systems, which allows for reports and dashboard features beyond the capabilities of the previous CIS-CAT versions.  Dashboards now have drill down functionality. Users can navigate from a high level graphical overview of environmental compliance with CIS Benchmarks to the individual assessment results that produce that compliance score.  When viewing these individual assessment results,  users can now create exceptions for certain rules along with a reason for the exception.  This will remove the rule from compliance scoring as long as the exception is active, but provide supporting evidence for auditor as to why the rule was excepted.  The application also offers a Remediation Report, for an operator only concerned with "failure" results of a given assessment and a Complete Results Report, to provide auditors with the complete assessment results of a given endpoint, or group of end points.  Users can also view CIS-CAT assessment results through the lens of the CIS Critical Controls with the new Controls View of assessment Results.  CIS-CAT Pro Dashboard provides users with the capability to Tag target systems (endpoints) in order to group them together for aggregation onto these new dashboards and reports.  This guide is intended to assist CIS Members with deployment, configuration, and use of the Pro Dashboard application and all of these new features.

## Deployment ##
CIS-CAT Pro Dashboard is a companion application to CIS-CAT.  CIS-CAT collects and evaluates system characteristics as described by the CIS Benchmark content.  CIS-CAT traditionally provided assessment results in various report formats, including HTML, XML, CSV, and plain text.  CIS-CAT can now upload its assessment results to the web-based Pro Dashboard application using a REST API.  CIS-CAT Pro Dashboard will import these XML document-based results into its application database.  This section describes how to configure the web application in your environment, as well as how to configure CIS-CAT to send assessment results to the Pro Dashboard application.

### CIS-CAT Pro Dashboard Deployment ###
See here:  [CIS-CAT Pro Dashboard Deployment](./ciscat-pro-dashboard-deployment.html)
### CIS-CAT Pro Assessor Deployment ###
See here: [CIS-CAT Pro Assessor Deployment Guide](./ciscat-pro-assessor-deployment.html)

## User Administration ##
CIS-CAT Pro Dashboard leverages spring security to manage authentication and access rights for application users.  Within the application an administrator can create new users, create new user roles, assign multiple user roles to each user, and assign access rights to various functionality to those roles.  This section describes how to administer CIS-CAT Pro Dashboard users and security.

