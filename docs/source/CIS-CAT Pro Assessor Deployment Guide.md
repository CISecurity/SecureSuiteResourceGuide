![](http://i.imgur.com/5yZfZi5.jpg)

# CIS-CAT Pro Assessor Deployment Guide #
*For use with CIS-CAT Pro Dashboard*

----------
## Introduction ##
The purpose of this document is to describe the different configurations of CIS-CAT Pro Assessor which enable the upload of assessment results to the CIS-CAT Pro Dashboard database. There are two methods in which CIS-CAT results reports can be automatically uploaded to the CIS-CAT Pro Dashboard application: POST'ing the results reports from a single CIS-CAT assessment (using either the graphical user interface or command-line user interface), and a modified version of the "cis-cat-centralized" scripts, as deployed by the "Assessing Multiple Windows Targets" and "Assessing Multiple Unix/Linux Targets" sections of the CIS-CAT User's Guide.

The following sections detail each deployment method.

## Standard CIS-CAT Assessment ##

A number of enhancements have been added to the CIS-CAT software to allow for report uploads to CIS-CAT Pro Dashboard.

- **POST Reports to URL:**  This functionality is actually pre-existing in CIS-CAT.  The -u command-line option has existed in CIS-CAT, allowing all generated reports to be transmitted to a web server equipped to process the POST'ed content.  Enhancements to the CIS-CAT graphical user interface have added this functionality to the Report Output Options screen.
- **Asset Report Format (ARF) generation:**  The ARF report was previously implemented in CIS-CAT in support of SCAP 1.2 validation efforts.  Initially, the ARF report could only be generated if a user assessed content created as a SCAP 1.2 data-stream collection.  Because CIS-CAT content is generally not packaged as SCAP 1.2 data-stream collection documents, the ARF functionality was quite limited.  CIS-CAT has been enhanced to allow for XCCDF 1.2-based data streams (in which most CIS benchmarks are packaged) to generate ARF results reports.
- **Authentication Token:**  In order to properly validate that CIS-CAT results are being imported to the appropriate CIS-CAT Pro Dashboard installation, user authentication/authorization must take place.  When a user installs the application and configures users to access it, they can then generate an Authentication Token which will perform that authentication.  CIS-CAT has been enhanced to allow for the token to be identified in the CIS-CAT properties file, or supplied via command-line options.

## Modified "Centralized" Deployment ##

New CIS-CAT Pro Dashboard-specific scripts have been developed to allow for reports to be generated in the correct format and POST'ed to the CIS-CAT Pro Dashboard database.

**Microsoft Windows Environments: cis-cat-centralized-ccpd.bat**

The Windows batch script has been developed to allow users to quickly enter the information necessary to execute the standard CIS-CAT "centralized" workflow, and automatically transmit the results to CIS-CAT Pro Dashboard. The CIS-CAT User's Guide contains setup instructions for Windows environments under the "Assessing Multiple Windows Targets" section.

**Create CIS Share on the CIS Hosting Server**

The instructions in the CIS-CAT User's Guide should be followed, except for Step 5. When configuring uploads to Shepherd, move the CIS\cis-cat-full\misc\cis-cat-centralized-ccpd.bat script to the root of the CIS folder.

**Security Considerations**

Ensure the "Execute" permission is granted to "Authenticated Users" on the CIS\cis-cat-centralized-ccpd.bat script.

**Update cis-cat-centralized-ccpd.bat**

The following information describes various configurable items contained inside the script, which must be set appropriately for CIS-CAT results to be uploaded to CIS-CAT Pro Dashboard.  If you want to use the CIS-CAT.bat/sh file you can set many of these properties in the misc/ciscat.properties file.
 
Line 27:

	SET DEBUG=0

This option enables (1) or disables (0) debugging during the execution of this script.
 
Line 36:

	SET AUTODETECT=1

This option enables (1) or disables (0) automatic detection of the user's operating system, as well as the appropriate selection of benchmark for assessment.
 
Line 45:

	SET SSLF=0

This option enables (1) or disables (0) the execution of the "higher security" profile of the selected benchmark. Typically, the disabled setting will select the "Level 1" profile for assessment, and the enabled setting will select the "Level 2" profile, if available for the selected benchmark.
 
Line 51:

	SET NetworkShare=\\\[NETWORK_SHARE]\CIS

The NetworkShare setting configures the UNC path to the folder containing the script to be executed. This value should be configured to the UNC path of the folder which contains the "cis-cat-full" CIS-CAT installation folder. Consider the following screen shot:

![](http://i.imgur.com/gpNQ5va.png)

When shared on a user's network, the folder to be shared is the "CIS-CAT-AUTODETECT" folder. Assume the folder is shared as \\CIS-CAT-SERVER\CIS-CAT-AUTODETECT. This would become the value of the NetworkShare variable in the cis-cat-centralized-ccpd.bat script:

	SET NetworkShare=\\CIS-CAT-SERVER\CIS-CAT-AUTODETECT
 
Line 65:

	SET JavaPath=Java\jre

This setting determines the path, relative to the path indicated by the NetworkShare, to a 32-bit Java Runtime Environment to be used to execute CIS-CAT.
 
Line 71:

	SET JavaPath64=Java64\jre

This setting determines the path, relative to the path indicated by the NetworkShare, to a 64-bit Java Runtime Environment to be used to execute CIS-CAT. Technically, this path can also be configured to reference a 32-bit JRE as well, and could be configured to the same value as line 65 above, if only a 32-bit JRE is used.
 
Line 77:

	SET JavaMaxMemoryMB=768

This setting indicates the maximum heap size, in megabytes, to be allocated by the JRE during CIS-CAT execution.
 
Line 83:

	SET CisCatPath=cis-cat-full

This setting indicates the path, relative to the path indicated by the NetworkShare, to the folder containing the CIS-CAT application (the CISCAT.jar file)
 
Line 92:

	SET CCPDUrl=http://\[YOUR-SERVER\]/Shepherd/api/reports/upload

This setting indicates the web application's resource URL, based on the member's installation of the CIS-CAT Pro Dashboard application. The value of [YOUR-SERVER] should be configured to the location of the web/application server (including any port values, if needed) on which CIS-CAT Pro Dashboard is installed. Because the report upload process is "resource-oriented", the /api/reports/upload path MUST be part of the CCPDUrl value, otherwise CIS-CAT will not invoke the appropriate CIS-CAT Pro Dashboardresource, for example:

	SET CCPDUrl=http://ccpd.example.org/CCPD/api/reports/upload
 
Line 102:

	SET CISCAT_OPTS=-arf -ui –n

This setting configures the reporting options for CIS-CAT's interactions with Shepherd. This value SHOULD remain fixed as –arf –ui –n. These options configure the following:

|Option|Description|
|---|---|
|-arf|This option indicates that CIS-CAT generates the Asset Reporting Format (ARF) report. This is the standardized reporting format which is accepted by CIS-CAT Pro Dashboard and imported into the application's database.|
|-ui|This (optional) command-line option indicates that CIS-CAT should ignore any SSL certificate warnings/errors when uploading the generated ARF report.|
|-n|This option explicitly disables the creation of the CIS-CAT HTML report. Only the ARF results can be automatically uploaded to CIS-CAT Pro Dashboard, so the HTML report must be disabled from being generated.|
 
Line 107:

	SET AUTHENTICATION_TOKEN=[Generate_An_Authentication_Token_In_CCPD]

The setting for Authentication Token is a key piece of the integrations between CIS-CAT "Assessor" and CIS-CAT Pro Dashboard. Once a user has been configured in the CIS-CAT Pro Dashboard application to be assigned to the API Role (ROLE_API in the application), that user can generate an authentication token to be used in CIS-CAT. This token verifies the user's privileges in CIS-CAT Pro Dashboard when attempting to upload CIS-CAT results into the application's database.

![](http://i.imgur.com/l2HSbC1.png)

Once the authentication token is generated in CIS-CAT Pro Dashboard, it MUST be copied into this setting in order for the upload functionality to work.
 
**NOTE:** By default there is a user named apiuser which has ROLE_API.  The default password for this user is @apiuser123.  In order to generate the token correctly, you must:

 1. Login once as the apiuser and reset the temporary default password.  
 2. Login as an administrator and navigate to the user management page for the apiuser.
 3. Generate the token using the new password you assigned to the apiuser.

Line 118:

	SET Benchmark=CIS_Microsoft_Windows_Server_2003_Benchmark_v3.1.0-xccdf.xml

This setting configures a specific benchmark for evaluation. This setting only applies if the AUTODETECT setting from line 36 is disabled (0).
 
Line 129:

	SET Profiles=xccdf_org.cisecurity.benchmarks_profile_Level_1_Domain_Controller xccdf_org.cisecurity.benchmarks_profile_Level_1_Member_Server

This setting configures the specific profiles for evaluation. This setting only applies if the AUTODETECT setting from line 36 is disabled (0).
 
**Validate the Install**

To test the setup, log into one of the target systems as a user capable of executing commands from an elevated command prompt, such as a domain admin. Execute the following command ***from an elevated command prompt:***

![](http://i.imgur.com/SrDD4Wc.png)

Note that the "CIS-CAT-SERVER" value should be substituted with the actual name or IP of the machine configured as the [NETWORK_SHARE] on line 51 of the script, above. If successful, the above command will detect the operating system, select the appropriate benchmark and profile, execute the CIS-CAT assessment, and transmit the generated reports to CIS-CAT Pro Dashboard:

![](http://i.imgur.com/lVKRtMA.png)

Because the import process to CIS-CAT Pro Dashboard must validate the uploaded report and process it into the application database, reports may not be visible through the CIS-CAT Pro Dashboard application for a few minutes.

**Configuring the Scheduled Task via Group Policy**

The scheduled task configuration information can be followed from the CIS-CAT User's Guide. The only difference arises in Step 7, when entering information into the Actions tab of the task configuration. Instead of executing the cis-cat-centralized.bat script, configure the task to execute the cis-cat-centralized-ccpd.bat script.

## Unix/Linux Environments: cis-cat-centralized-ccpd.sh ##

Implementing the CIS-CAT Pro Dashboard workflow in a "centralized" Unix/Linux environment only requires that the CIS-CAT Pro Dashboard application URL be provided in the bundled cis-cat-centralized-ccpd.sh script. The CIS-CAT Pro Dashboard-specific script looks very similar to the standard cis-cat-centralized.sh script, except for line 89, which configures the URL of the CIS-CAT Pro Dashboard application resource:
 
	#
	# The URL for the CIS-CAT Pro Dashboard API to which CIS-CAT reports are POST'ed
	# The resource for CIS-CAT Pro Dashboard upload is ALWAYS mapped to the /api/reports/upload location,
	# so the path to the application is all that should be modified here,
	# for example: http://applications.example.org/CCPD/api/reports/upload
	#
	CCPD_URL=http://[YOUR-SERVER]/CCPD/api/reports/upload

Similar to the configuration for Windows above, the CCPD_URL value must end with the /api/reports/upload pattern to correctly identify the resource handling CIS-CAT uploads:

	CCPD_URL=http://applications.example.org/CCPD/api/reports/upload

All other configuration items in this script should be pre-configured out of the box. Once this script is saved and closed, the installation can be tested.
 
**Validate the Install**

In order to test the proper installation of the cis-cat-centralized-shepherd.sh script, log into one of the target systems in the environment as either a root user or a user capable of executing commands using sudo. Execute the following command, where /path/to/cis represents a file system path to the CIS Host Server:

	> /path/to/cis/cis-cat-centralized-ccpd.sh

If configured correctly, the standard CIS-CAT assessment output will be displayed to the user, and a successful CIS-CAT Pro Dashboard upload message will be displayed at the conclusion of the assessment.

