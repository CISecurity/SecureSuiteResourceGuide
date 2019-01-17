![](http://i.imgur.com/5yZfZi5.jpg)

# Change Log #

----------

## CIS-CAT Pro Dashboard v1.1.5 ##

FUNCTIONAL ENHANCEMENTS

 - CIS Controls V7.0 support: For Benchmarks mapped to CIS Controls, users can view how recommendations relate to CIS Controls. See our upcoming blog to learn more. 
 - The CIS-CAT Pro Dashboard's HTML report has been modified in style to match the Assessor's HTML report for consistency purposes. 
 - For Dashboard Installer users, existing database settings will be detected and used instead of changed to CIS-CAT recommended settings.


SYSTEM ENHANCEMENTS

 - Oracle users upgrading from Dashboard versions prior to 1.1.3 will benefit from better database performance due to established indexing. Oracle users who have upgraded to Dashboard versions 1.1.3 or later to 1.1.5, will obtain the necessary indexes on upgrade. 
 - The Installer has been whitelisted with Symantec anti-virus. 

BUGS

 - Resolved an issue for SQL Server users when drilling down on the Benchmark View chart. 


## CIS-CAT Pro Dashboard v1.1.4 ##

FUNCTIONAL ENHANCEMENTS

 - Installation and Upgrade Tool: A step-by-step embedded tool for the install/upgrade of dashboard that steps members through each process.
 - Graphs Viewable With or Without Internet Connectivity: CIS-CAT Pro Dashboard can now display assessment results in a graphical form whether your application server is on or off-line.

SYSTEM ENHANCEMENTS

 - MySQL database driver replaced by Maria DB driver (increase performance). **IMPORTANT: in ccpd-config.yml, "com.mysql.cj.jdbc.Driver" driverClassName needs to be replaced by "org.mariadb.jdbc.Driver". Can be done with the Installer or manually.**
 - Jobs run in succession (queue) to avoid simultaneous access to the database.

BUGS

 - Fixed reset password link and password expired redirection for users using a webserver. 
 - Fixed a bug when importing duplicate oval variables.
 - Fixed a bug when importing benchmark front-matter and rear-matter xml elements.

## CIS-CAT Pro Dashboard v1.1.3 ##

FUNCTIONAL ENHANCEMENTS

 - Implemented a Difference Report for comparing an assessment result with the previous assessment result.
 - Added ability to delete individual Configuration Assessments and Vulnerability Assessments.
 - Conversion abilities to convert existing data to the new data model.
 - Added ability to attach existing vulnerability assessments to NVD data updated after the import of the assessment.

SYSTEM ENHANCEMENTS

 - Delete target systems Performance improvements.
 - Added "Attach CVEs to Existing Definitions" button on the Vulnerabilities List.  This will allow NVD data to be imported after vulnerability assessments.
 - New Assessment Data Model which will offer performance improvements in: Import, Export, Delete functionality.
 - Support for WebLogic appliction server

BUGS

 - automated database changes required when updating from a version prior to v1.1.2.

## CIS-CAT Pro Dashboard v1.1.2 ##

SYSTEM ENHANCEMENTS

 - Import Performance improvements.
 - Implemented a new way to identify duplicate imports, to maintain a smaller and more accurate version of content, especially OVAL content.

BUGS

 - Fixed a bug with imports sometimes misidentifing duplicate target systems
 - Fixed a bug with importing of older CIS content, such as AIX 6 results.
 
## CIS-CAT Pro Dashboard v1.1.0 ##

FUNCTIONAL ENHANCEMENTS

 - CCPD can now except custom target system identifiers from CIS-CAT Pro Assessor.  A custom element can be configured in CIS-CAT Pro Assessor to be passed to CCPD via the import functionality.
 - Group Exceptions - you can now add exceptions to entire groups or sub-groups of recommendations
 - Vulnerability Reports - you can now upload vulnerability reports from CIS-CAT Pro Assessor.  These reports also have exception funcitonality like the configuration assessment reports
 - Vulnerability Dashboard - you can view your vulnerability data over time using the Vulnerability Dashboard.
 - NVD Data Import - you can now import CVE and CVSS information directly from the NVD.  This data is used to support vulnerability report scoring

 
SYSTEM ENHANCEMENTS

 - Target System UI - redesigned to better present information,  including the profile level of each asseessment result.  added tabs for configuration results and vulnerability results.
 - Improved performance on Complete Report and Remediation Report exports.

BUGS

 - fixed a bug from 1.0.5 where importing subsequent ARF files would create additional Target System records



## CIS-CAT Pro Dashboard v1.0.5 ##

FUNCTIONAL ENHANCEMENTS

 - Added Title information to the alert dialog
 - User Favorites - added a user favorites section where users can mark their favorite benchmarks or target systems.  The Benchmark Dashboard and Target System Dashboard now use user favorites to display the options available for graphing
 - Inbox UI Improvements
   - added an All/Unread toggle to the User Inbox to easily view only unread messages.  
   - added orange text to unread tasks in the User Inbox, to provide a visual distinction from other types of unread alerts.
   - added batch delete and mark read/unread to the User Inbox
 - Exception workflow - added alerting to recipient list when an exception is approved/rejected.  Previously the alert would just go to the requester, now it will go to everyone on the recipient list, which can be managed in the admin section
 - added security to exception end dating to only allow ROLE_ADMIN to end date exceptions
 - Exception End date alert - recipient list will be notified when an exception is end dated
 
SYSTEM ENHANCEMENTS

 - Target System Primary Identification customization - added ability to customize the primary identifier used for target systesms at an application level, and per target system.  By default, hostname will be the primary identifier of all target systems, but you can now change to use another identifier type, such as fqdn, mac-address, or a custom identifier.

BUGS

 - fixed bug where CCPD would not run using SQL Server 2014
 - fixed missing "Back" button on Target Systems Dashboarad when toggling between multi/single view
 - fixed bug preventing deletion of target systems
 
## CIS-CAT Pro Dashboard v1.0.4.1 ##

FUNCTIONAL ENHANCEMENTS

 - Device View Dashboard - you can now search by device, or multiple devices, instead of having them all listed.

SYSTEM ENHANCEMENTS


BUGS

 - fixed bug where viewing assessment results would make the end time on the report 12:00am
 - fixed an import bug where missing oval definitions would cause an import to fail.

## CIS-CAT Pro Dashboard v1.0.4 ##


FUNCTIONAL ENHANCEMENTS

 - Alerting functionality 
 - users will now receive configurable alerts for: Test Results imported,  Low Scoring Results imported, Errors importing Results
 - Exception Workflow  - when an exception is created it will go into a Pending status, and an task will be created to approve/reject the exception.  this defaults to users with ROLE_ADMIN, but is configurable
 - User Inbox - new inbox on the menu bar for alerts and workflow tasks
 - Exceptions View on Test Results - a new view was added to Test Results to show all exceptions that apply to that test result in a single list.
 - Exceptions by Target System - the target system screen now displays all exceptions that apply to that target
 - Exceptions Search - you can now search for exceptions by: Target System, Tag, Benchmark, Dates
 - User Tags - users can now be tagged, this allows for alerts to be sent to users by tag
 - Role Tags - Roles can now be tagged, no current functionality is effected by this.
 - Added hostname to the Target System Search, Remediation Report Search, Complete Results Search

SYSTEM ENHANCEMENTS

 - Added support for Oracle Databases
 - Added Support for SQL Server databases
 - Performance improvements for the import process
 - CCPD software version now appears on the title bar
 - Functional Area now applicable by action - the functional areas for security use to only be able to control access at the controller level,  administrators can now control access at the action level, whichi is much more granular

BUGS

 - fixed vulnerability where a basic user could become an administrator

## CIS-CAT Pro Dashboard v1.0.3 ##

FUNCTIONAL ENHANCEMENTS

 - Ability to Tag Users
 - Ability to Tag Roles
 - Ability to Search Users
 - Profile selected added to the Assessment Results List and Assessment Results Search pages.

SYSTEM ENHANCEMENTS

 - Performance improvements on Assessment Result Importing

BUGS

 - Fixed concurrency issues when uploading and importing Assessment Results simultaneously


## CIS-CAT Pro Dashboard v1.0.2 ##

FUNCTIONAL ENHANCEMENTS

 - Assessment Results Search page has been added to the Reports menu.  Users can now search for assessment results by: hostname, date range, benchmark, and tags.
 - Exceptions added in CCPD will now show in the HTML export document.
 - Added guidance on the dashboard screens to explain the data.
 - Months now appear by name instead of number on the Dashboard Graphs for improved readability.
 - Added a rule to prevent duplicate assessment results imports.
 - Added links to the CCPD User Guide, and The CIS WorkBench, in the main navigation menu at the top right of the application.

SYSTEM ENHANCEMENTS

 - Created an initialization service to process data fixes required for new releases, bypassing the need to manually run data scripts.
 - Made search criteria modular so that criteria and new searches can be added to the system easier.
 - Assessment Results List now uses a pre-calculated score, which improves performance.
 - Assessment Result import/export processes now use the weight from the XCCDF rule-result element of the TestResult, instead of the rule element of the Benchmark.
 - Made performance improvements to the Assessment Results View page.

BUGS

 - Users can now import assessment results from target systems with underscores in their hostname.
 - Users can no longer access user functionality when unauthenticated.
 - Dashboard scoring calculations to include recommendation weights and exceptions.
 - All users are now assigned ROLE_BASIC_USER which allows access to user functionality: profile, forgot password, password reset.
 - HTML export will now take the weight of a recommendation from the results rather than the benchmark, as the imported version could have different weights than the results.
 - Users with ROLE_API can no loger be added to other roles on the Show Role Screen.
 - Remediation Reports with no failed rules will now show a congratulatory message instead of a blank result.
 - Assessment Results can now be imported concurrently


## CIS-CAT Pro Dashboard v1.0.1 ##
 - Converted documentation to online documents available at: http://cis-cat-pro-dashboard.readthedocs.io
 - Fixed issues with CIS-CAT Pro Dashboard running on tomcat v8.5.11 including:
	 - user/role administration dialogs not working
	 - dashboards not showing
 - Fixed scoring inconsistency between the Assessment Results List, individual Assessment Results View screen, and the HTML export of the assessment results.  The inconsistancy  was the result of weighting of recommendations and exceptions added to recommendations
 
## CIS-CAT Pro Dashboard v1.0 ##
Initial Release
 