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

## Users ##
To create a new User in CIS-CAT Pro Dashboard you need to log in as an administrator. 

Login as an administrator. (NOTE: By default the user: admin, with the password: @admin123 has ROLE_ADMINISTRATOR and ROLE_BASIC_USER)  Navigate to the Administration -> User Management -> User menu item.  This will navigate to the User List.

- Creating a New User - On the user list, click the New User button.  On the create user screen, you enter a first and last name for your user, required unique username, a password, as well as selecting user roles for the user (all users must have ROLE_BASIC_USER). The password must adhere to this format: at least one letter, number, and special character: !@#$%^&, and be between 8 and 64 characters in length.   
- Note that the password you enter is temporary. The user will be asked to change the password during their initial login. 
- View an existing User - To view an existing user simply click on the users row in the table, this will navigate to the User view page.  From there you can see the various details about the user, as well as edit or delete the user.
- Editing an existing user - Once you've clicked on an existing user and navigate to the view page, you can edit that user by selecting the edit button.  On the edit page you can change the users first and last name,  change their password, enable/disable their account, or change their user roles
- Deleting a user - Once you've navigated to the User view page you can delete a user by selecting the delete button.  A confirmation message will appear to allow you to insure you want to delete the user.  Once you click "yes", the user will be deleted

## Roles ##
Roles in CIS-CAT Pro Dashboard are assigned to users, and allow access to functionality.  The role section is used for creating new roles, but you will need to add them to users and to Functional Areas in order for them to control access meaningfully.  

**NOTE:** when creating a new role, you must prefix the name you chose with "ROLE_",  otherwise the role will be unrecognizable to spring security.

A specific Role which has significance in a CIS-CAT Pro Assessor/Dashboard hybrid environment is the "ROLE_API" role.  This role is built in to an out-of-the-box CIS-CAT Pro Dashboard deployment and must exist in order to use CIS-CAT to upload results to the Dashboard application.  Once a user is assigned the "ROLE_API" role, the User's information page will display a button labeled "Generate CIS-CAT Authentication Token".  

![](http://i.imgur.com/MPBh3Dy.png)

Clicking the button will open a dialog box where the user is required to enter the "ROLE_API" user's credentials.  Once that user has been re-authenticated, the token is generated and displayed on the page.  That token can now be incorporated into CIS-CAT, as per the instructions below, under "Import CIS-CAT Results - CIS-CAT Import"

## Security Functional Areas ##
Functional Areas are groups of functionality that are assigned to Roles to give them meaning.  Each functional area covers a specific group of functionality within CIS-CAT Pro Dashboard.  By default there is: Reports, Dashboards, Target Systems, System Administration, API, and Developer.  These roles individually encompass the functionality within the menu options at the top of the CIS-CAT Pro Dashboard Application.

Functional Areas can be administered through the Roles interface.  When you select a Role from the Role List,  you are taken to the show Role page.  Here you can add/delete users access to this role, and you can add/delete functional areas that this role allows access to.

![](http://i.imgur.com/cPSn0zC.png)

## Logging In/User Profile ##
When a user first navigates to CIS-CAT Pro Dashboard, they are asked to log into the system.  If a user account has been created for that user, they will initially be asked to reset their password following a successful login.

![](http://i.imgur.com/vnZJve1.png)

Once the user configures their password, they are asked to re-login using those new credentials.  Once logged in with updated credentials, the user is taken to the "Overview" Dashboard view.  In the top right-hand corner of the application now resides the display of the logged-in user's username:

![](http://i.imgur.com/bcyWVtc.png)

Clicking on the username, a menu will appear, showing the user options for controlling their user account, such as editing their user profile, or logging out of the application:

![](http://i.imgur.com/oGURHUy.png)

Click on the Profile link to navigate to the user's profile:

![](http://i.imgur.com/GptCUED.png)

The User's profile screen shows account and role information, as assigned by an administrator.  From this screen, an individual user may change their password, or edit those profile fields to which they have access to change.  Clicking the Change Password button opens a dialog box allowing the user to enter and confirm new credentials to be used when logging in to CIS-CAT Pro Dashboard:

![](http://i.imgur.com/iUreF3M.png)

Validations exist when users change their credentials.  The passwords entered must match, the new password must be between 8 and 64 characters in length, and must include at least one uppercase letter, one lowercase letter, a numeric character, and at least one special character, such as "!@#$%^&"
Clicking the Edit Profile button from the user's profile opens a dialog allowing the user to make changes to their information.  This information is limited because many user changes should only be performed by an administrator, such as editing user roles or activating/deactivating a user's account.

![](http://i.imgur.com/uxSQfmO.png)

Making changes to this dialog and clicking Save will update the user's information.

The user profile screen also features a list of Alerts that the user is currently subscribed to, as a well as why they are subscribed to that alert type:

![](http://i.imgur.com/Jku7pgm.png)

From this list users can choose to opt out of any of the alert types that they are receiving.

##User Inbox##

The Use Inbox contains all of the alerts/tasks assigned to the user.  Simply click on the Inbox item on the menu bar to navigate to the inbox:

![](http://i.imgur.com/qMJo1jH.png)

The bubble next to the Inbox will indicate how many unread messages you have.

![](http://i.imgur.com/cRWtZeH.png)

The inbox features serveral views, which can be navigated to using the tabs on the left hand side:

 - **Inbox** - has all new alerts and tasks that have not been deleted or completed.  Unread messages will appear in white with bold text
 - **Unread** - contains only unread messages
 - **Trash** - contains deleted messages
 - **Completed Tasks** - contains tasks that have already been worked
 - **All** - contains all messages
 
Clicking on the messages in any of the lists will pop up a dialog displaying the message.  Clicking on the delete button in the list will move the message to the trash folder.

**Sending Manual Alerts**

You can send a custom alert to any user or group of users in the system by clicking the Send button:

![](http://i.imgur.com/SNb9450.png)

This will open the manual alert dialog,  from here you can select the recipients you want and add a title and message to the alert:

![](http://i.imgur.com/o0lm3XK.png)

When complete click "Send" and your message will go to the inbox's of the selected recipients.

**Alert Types** - There are serveral different types of messages that you can receive in your inbox:

 - **Task** - A task has an action that you need to perform in order to close it.  When you open a Task, there will always be one or more Actions you can take to close the task
 - **Alert** - An alert informs you of a system event directly related to you,  such as the completion of an upload you initiated. 
 - **Event** - An event informs you of an occurrence in the system that you need to be informed of.
 - **Manual** - A manual message was sent directly to you by another user.
 
##Alert Management##

In the Administration menu there is an option for Alerts.

![](http://i.imgur.com/E7sGpvV.png)

This will navigate to the alert list, where you can select an alert to bring up the show Alert page where alerts can be managed.

![](http://i.imgur.com/7zqyySg.png)

The important feature of this page is the configurable recipient list.  This shows all the users that are configured to receive the alert, why they are, either directly, by a role they have assigned, or by a tag they have assigned.  Users can then use the Receiving Users, Receiving Roles, and Receiving Tags list to manage who will receive an alert.

The Recipient list also shows which users have opted out of the alert type.

**NOTE:** A user will only receive one instance of an alert,  even if they are included in the recipient list by multiple criteria.  i.e.  if they have a tag and a role that include them in the recipient list, they will still only recieve one alert.

## Importing CIS-CAT Assessor Results ##
**In-Application Import**

Importing CIS-CAT Assessor results using the CIS-CAT Pro Dashboard user interface assumes that a user has executed a CIS-CAT assessment and produced the Asset Reporting Format (ARF) results.  Once an ARF has been generated in CIS-CAT and saved to the designated reports location, open a web browser and log into the Dashboard application.  From the main navigation bar, select Reports -> Assessment Results List.  

![](http://i.imgur.com/m6lhmEm.png)


The "Assessment Results List" page will be displayed.  Click the "Import Assessment Results" button.  A file selection dialog will open, allowing the user to browse to the saved reports location and select the CIS-CAT-generated ARF or XML report.  

![](http://i.imgur.com/4nM6Ti0.png)

Click "Upload" to start the import process.  Note that the import processes asynchronously, so the user will see a message indicating that the report upload has begun.  This process can take up to a few minutes to complete.

**CIS-CAT Import**

Importing Asset Report Format (ARF) results from CIS-CAT assumes that the CIS-CAT Deployment instructions have been completed.  The end result of that configuration is that a user has been created in CIS-CAT Pro Dashboard, been assigned to the "ROLE_API" role, and an authentication token has been generated.  

![](http://i.imgur.com/roQk64u.png)

Once generated, that authentication token must be added to the CIS-CAT properties file in order for automated upload to function.

ciscat.properties:

	# Allow for an authentication token to be generated in "CIS-CAT Pro Dashboard", allowing upload of
	# generated ARF reports to the new database application.
	ciscat.post.parameter.ccpd.token=m9i0o2lrqno60dlq49qlln6gqrj2l7kt

Save the "ciscat.properties" file and execute CIS-CAT.


**Graphical User Interface (GUI)**

When executing the CIS-CAT GUI, users will select a benchmark and profile, and subsequently be navigated to the "Report Output Options" screen.  To upload reports to CIS-CAT Pro Dashboard, a user may select EITHER the XML results or the Asset Report Format to be generated.  At the bottom of the "Report Output Options" screen, click the button to "POST Reports to URL".  When clicked, a dialog box will open allowing users to select the URL to which the generated report is uploaded.  

![](http://i.imgur.com/aVN5Y6m.png)

**NOTE: The CIS-CAT Pro Dashboard API is "resource-based" and, as such, only a specific URL pattern can be entered.** This pattern will always end with "/api/reports/upload".  For example, if the context URL for a member's CIS-CAT Pro Dashboard deployment is http://myapp.example.com/CCPD, the URL for reports upload will always be http://myapp.example.com/CCPD/api/reports/upload.  

![](http://i.imgur.com/21WzjyZ.png)

**Command Line User Interface (CLI)**

To enable the CIS-CAT Command Line to import results directly into CIS-CAT Pro Dashboard the following options are used:

	-arf : This option indicates that CIS-CAT will generate the Asset Reporting Format (ARF) results
	-n   : This option indicates that CIS-CAT should NOT generate the HTML report
	-u   : This option allows users to specify the URL to which ARF reports will be uploaded.  This is the CIS-CAT Pro Dashboard URL
	-ui  : This optional argument allows users to ignore any certificate warnings/errors when connecting to the CIS-CAT Pro Dashboard URL
	 
	For example, assessing and uploading the Windows 7 Benchmark would look like:
	> CIS-CAT.bat -b benchmarks\CIS_Microsoft_Windows_7_Benchmark_v3.0.0-xccdf.xml -arf -n -u http://myapp.example.org/CCPD/api/report/upload -ui
 

**Legacy Data Import**

To help facilitate organizations migrating from static CIS-CAT dashboards and reporting to storing assessment results in CIS-CAT Pro Dashboard, a legacy data import process has been developed.  This process is configured as a recurring job running in the background of a CIS-CAT Pro Dashboard installation.  Users are required to configure 3 folder locations:

1. **The "Legacy" Folder:**  This folder will be the location for all CIS-CAT XML results to be placed, as the staging area for results waiting to be imported into CIS-CAT Pro Dashboard
2. **The "Legacy Processed" Folder:**  This folder will be the location for all CIS-CAT XML results which have been successfully imported into CIS-CAT Pro Dashboard from the staging area.
3. **The "Legacy Error" Folder:** This folder will be the location for any CIS-CAT XML results which were not successfully imported into CIS-CAT Pro Dashboard.

These folder locations are set in the application's configuration file ("the YAML").  Instructions for this configuration are contained in the CIS-CAT Pro Dashboard Deployment guide.

## Target Systems ##
**Creation**
Target Systems represent endpoints in your environment that have assesment data within CIS-CAT Pro Dashboard.  There are several ways to create target systems within the application:


1. **Import** - The section above describes the import of CIS-CAT data processes.  The assessment results produced by CIS-CAT relate to a specific Target System.  On import, CIS-CAT Pro Dashboard will check the existing target systems to see if the relevant Target already exists within in the system.  If not, CIS-CAT Pro Dashboard will create a new Target System and associate the imported assessment result with that new Target.  If the Target System already existed, based on the Hostname identifier,  the application will associate the imported result with the existing Target.
2. Online Entry - From the main application menu bar, users can navigate to the Target Systems List:

![](http://i.imgur.com/hEYEd1y.png)


Once there, you can select the "New Target System" button, which will open the creation dialog:

![](http://i.imgur.com/lVsQH9B.png)

Simply enter the Hostname and click Add Target,  this will create a new target system.

**Tagging**

Tags can be used to organize various system entities into groups, such as Target Systems, Users, Roles, or Exceptions.  A n entity can have any number of Tags which can signify a couple of things within CIS-CAT Pro Dashboard.  Tags can be used to search for entities that have a specific tag or group of tags, or one tag but not another.  Dashboards can be viewed by tag, so that user's can see how a specifically tagged group of endpoints is complying with CIS benchmarks.  Alerts and Workflow Tasks can be sent to Users by tag, so that a group of tagged users will all receive the alert or task.

**Creation**

Once a taggable system entity is created you can navigate to it's view page by clicking on the entity's row on the List Page, lets look at Target Systems as an example:


![](http://i.imgur.com/KjlNHvP.png)

Once navigated to the view page, users can manage the Targets tags by simply typing in the Tag box, or deleting from the tag box:

![](http://i.imgur.com/ygcH1Gn.png)

Tagging works the same for other taggable entities, such as Users and Exceptions.

**Searching**

Once tagged, you can use individual tags, or logical combinations of tags to search for a specific set of end points.  The search screen has a list of tags to include, either using an "AND" or "OR" operator and a list of tags to exclude from your search group.  You can also search directly by hostname.

![](http://i.imgur.com/EaITGHP.png)

- **Include Tags** - type into the include tags list the tags you would like to see in the search results.  i.e.  if you would like to see target systems with the "PCI" tag, simply type it in the box and click search.

  * **"and" operator** by default the "and" operator is selected.  This means that if you type multiple tags into the Include Tags box,  the resulting systems would need to contain ALL of the tags in the Include Tags box.  i.e.  If you typed in "PCI" and "Workstation"  all systems with BOTH of those tags would be returned.  If a system only contained the "PCI" tag, it would not be returned.

  * **"or" operator** -  The "or" operator can be selected using the available radio button.  When selected, if you type multiple tags into the Include Tags box, the resulting systems would need to contain ANY of the tags listed in the Include Tags box.  i.e.  if you typed in "PCI" and "Workstation"  all systems with EITHER of those tags would be returned.  If a system only contained the "PCI" tag, it would be in the result set.

- **Exclude Tags** - type into the Exclude tags list the tags that you do not want in your search results.  This is useful if there were particular tags you would like excluded from your search.  i.e.  Say you wanted to see all of your Servers that did not deal with PCI.  You could type the "Server" tag into the Include Tags box and "PCI" into the Exclude Tags box.

## Reports ##
CIS-CAT Pro Dashboard reports provide a variety of views of CIS-CAT Assessment Results.  An individual Test Results Report provides the same view as the legacy HTML report from CIS-CAT, with some enhanced features, including a controls based view, and the ability to create exceptions for specific rules.  The remediation report provides a list of only the latest failed results for a target or group of targets.  The intent is for a remediator to print this report and use it to manually remediate misconfigurations on the target.  The complete Results Report will give an abbreviated version of the complete results for a system.  This is intended for an auditor to get a full picture of CIS compliance for a specific target or set of targets.

**Assessment Results**

The individual test results report provides a complete picture of a given Target System's compliance with a CIS benchmark at a single point in time.  This report was designed to mimic the functionality of the HTML version of the CIS-CAT Security Configuration Assessment Results report.


 **Navigation** - there are several ways to navigate to the Test Results Report.  Under the reports menu, you can click the Assessment Results List menu item.  From the list, you can select the individual assessment result that you would like to view.  You can also navigate to an individual target system, and listed in the Results box are all of the benchmarks for which the current target has results stored in the database.  Clicking one of the benchmarks, will open the list of all the results for that target and that benchmark,  from there you can select an individual result.  Finally you can Navigate to the individual benchmarks, there is a results section which contains all the results in that system for that particular benchmark.


![](http://i.imgur.com/lcbqQud.png)

1. **Results View** - the results view shows the test result in the same structure as the original benchmark.  The results for each recommendation are organzied into the groups the same way as the benchmark.  Each group and subgroup is scored individually as a tally of all the rules contained within.  This is a dynamic version of the old CIS-CAT HTML report.  Users can also manage Exceptions to rules from this view (see below).
  
2. **Controls View** - when viewing a test results report, the default view is the traditional benchmarks view.  In this view the rules and results are organized into the structure and groups that they are in the CIS Benchmarks, as determined by the individual consensus communities.  This view mirrors the traditional CIS-CAT HTML report, with each group having rule totals and scoring information, as well as the actual evidence from the assessment.  The controls view takes the same set of results, and using mapping metadata from the rules in the benchmark, reorganizes the rules into CIS Critical Security Controls View.  In this view each of the 20 Critical Controls as well as the subcontrols contained within each control are listed.  You can see on this view if a particular control or subcontrol has any benchmark rules associated with it.  If so, you can open the control/sub-control and see all of rule results that provide evidence of implementation of that control/sub-control in your environment.  If there are no rules mapped to that  control,  clicking on it will simply provide more information about that particular control/subcontrol.

3. **Exceptions View** - the exceptions view lists all exceptions that apply to the recommendations in this benchmark.  An exception can be associated with a single test result either by applying directly to that target system,  applying to a tag that the target system has, or by being a global exception.  This view provides a complete list of exceptions applying to the test result.
 
**Remediation Report**

The remediation report is designed to allow an operator to have a list of failure results, as well as the remediation steps to fix the failure.  An operator can take this report, follow the remediation steps, and bring a target system or target systems into compliance.  To generate this report navigate to the Remediation report from the CIS-CAT Pro Dashboard Reports menu.  The first step is to chose the target systems you want included in the report.  First, use the search criteria to get a list of target systems, and the latest result for each benchmark.

![](http://i.imgur.com/41lCoBa.png)

You can then use the "Selected" checkboxes to choose which Assessment Results you would like to appear on the report.  Once you have the correct results you select the "Remediation Report" button and the report will be generated.

![](http://i.imgur.com/21ZzQIg.png)

The report lists the target system, the benchmark, the rule number and title, and the remediation steps for each failed result.  Users can then utilize the buttons in the upper right side of the screen to export the report in a variety of formats.

**Complete Results Report**

The complete results report will give you a full view of a target system or group of target systems compliance accross multiple CIS benchmarks.  Similar to the Remediation Report, you search for target systems,  select the ones you would like to see complete results for, then generate the report. 

![](http://i.imgur.com/KHBEHdb.png)

The complete report lists the Target System, Benchmark,  Rule Number and Title, as well as the overall pass fail result of each individual rule.

![](http://i.imgur.com/Q9CmiC0.png)

## Exceptions ##
The recommendations in CIS Benchmarks are just that,  recommendations.  Every recommendation does not necessarily apply to every organization or every target system within an organization.  CIS-CAT Pro Dashboard provides functionality to create "exceptions" to specific rules on a per machine,  global, or by tag basis.  This allows CIS-CAT to continue to assess the target system against the rules, but when viewing the Test Results Report within CIS-CAT Pro Dashboard, the rule will not negatively impact the targets compliance scoring.  When creating the exception, you can also provide a rationale for why the rule is being excepted.  This provides information to an auditor as to why the rule is not being scored.

  * **Creation** -  To create an exception simply navigate to the rule you would like to except in the Test Results Report.  Within the rule is the Exceptions section.  If there are no existing exceptions, you will simply see an "Add Exception" button.  If exceptions already exist for the rule, they will be displayed in a table, along with the "Add Exception" button.  
  ![](http://i.imgur.com/WqLpJzm.png)
  Click this button, and the exception creation dialog will be displayed:
  ![](http://i.imgur.com/W1TPu5g.png)

	By default the start date is set to the end time of the assessment report that you are currently viewing,  this would make an exception you create apply to this specific report, as well as any assessments that post date this report.  You can modify the date to apply to any time period you like.  Also required on this dialog is a rationale,  you must enter the reason you are creating an exception for this rule.  By default, any exception created will apply only to the target system that you are currently reviewing results for.  You can also check the global checkbox to make the exception apply to all target systems in your environment.  You can also enter any number of tags into the tag checkbox,  the exception will then apply to any target system that has any of the entered tags.  These are different ways to scope the exception.

  *	**Approval** - Once an exception is created it must be approved by a user with ROLE_ADMIN.  On creation, an exception will enter pending status, and, by default, a task will be created for all users with ROLE_ADMIN and sent to their user Inbox.  This task will allow the administrators to review the exception request and accept or reject the exception.
  ![](http://i.imgur.com/AOyj8Mw.png)
	If the exception is approved, it will take effect for the time period and targets specified.  If it is rejected, it will be ignore.  Either way,  the user who requested the exception will be notified of the result via an alert sent to their user inbox.

  * **End Date** - Exceptions are not meant to be permanant.  As such, CIS-CAT Pro Dashboard provides the ability to end date an exception when it is no longer needed.  To end date an exception, you simply need to click on it in the exception table for the rule.  this will bring up the end date dialog.  you can enter any end date you would like, then click save.  The Exception will now be end dated and on
  * **Viewing Exceptions** -  there are several ways to view exceptions in the application:
	  * **Exception List on Test Results** - described above, there is a tab on each test result showing all the exceptions that apply to that system
	  * **Target System Exceptions List** - on each target systems view page there is a list of exceptions that apply to that target.
	  * **Exception Search** - in the report menu there is an Exception Search option which allows users to search for exceptions by: hostname, benchmark, date range, or tag.  Searching by hostname will return all exceptions associated with that target system, even if they are associated by tag or by being global.
	  ![](http://i.imgur.com/jmRrgDQ.png)


## Dashboard ##
The CIS-CAT Pro Dashboard application's dashboard views provide a high level overview of organizational compliance with CIS Benchmarks.  There are several views, which comprise different aggregation levels which produce a graph that represents compliance over time.  The default views show all of the compliance results for the aggregation group selected, i.e.  "Overview" is all of your target systems for all benchmarks,  The "Benchmark View" is by benchmark,  the "Tag View" is all systems with a specific tag or set of tags.  Each point on the graph is an average score for the month.  Each of the points can be clicked to "drill-down" into the Monthly view.  This view has a point for each day in the selected month that has results.  Each of these points can be clicked on to drill down to that specific day,  which will display points for each time you have an assessment result.  The points on the daily view will take you straight to the individual assessment result that produced the score.  This way you can navigate from a very high level view of your compliance data, all the way to the details,  the individual assessment reports that comprise the high level graphical information.

**Overview** 

The overview contains a fully aggregated view of all endpoints across all benchmarks:

![](http://i.imgur.com/tB4uO1d.png)

**Benchmark View**

The benchmark view has results aggregated by benchmark.  You can select any number of benchmarks that you would like to see results for.  Each benchmark selected will be represented by a separate line on the graph.  This allows you to compare compliance against various CIS Benchmarks.

![](http://i.imgur.com/FCUQxHx.png)

**Device View**

The device view has the results aggregated by individual device.  The default device view is the Multi-device view, which allows you to select many target systems and compare their aggregated results.

![](http://i.imgur.com/wJVLA4t.png)


**Device by Benchmark View**

If you only select a single device, you can switch to the single device view.  This will allow you to select the benchmarks that have assessment results for the selected target system and compare the benchmark compliance for just a single target.  This allows you to see potentially which benchmarks are reducing the compliance score for a single system

![](http://i.imgur.com/r92Iy5U.png)

**Tag View**

The tag view allows you to aggregate compliance results for a group of target systems with the same tag, or with multiple tags.  Each tag entered will be represented by a single line, so that you could compare results accross multiple tags.

![](http://i.imgur.com/dm0LQK0.png)
 
## "Reference" Data Administration ##
Users assigned (currently) the ROLE_CIS user role are granted access to application views which allow for the import and viewing of assessment resources, such as Data Stream Collections, XCCDF Benchmarks, OVAL Definitions, and CPE Dictionaries.

In general, CIS content will be delivered using one of four structures:

1. **XCCDF 1.1 (Legacy):** A single-file format consisting of a benchmark XCCDF, containing CIS' proprietary embedded check language (ECL).  When importing legacy XCCDF 1.1 content, simply navigate to the "Benchmarks" list page (shown below) and complete the "Import" workflow.
2. **XCCDF 1.2 "data-stream":** A 2-or-4-file format consisting of a benchmark XCCDF, an OVAL Definitions file, and optionally, a CPE Dictionary and CPE OVAL Definitions file.  When importing XCCDF 1.2 information as part of a 4-file bundle of assessment content, first proceed to the CPE Dictionary import, followed by the CPE OVAL Definitions import, the OVAL Definitions import, and conclude with the XCCDF Benchmark import workflow.
3. **SCAP 1.2 "data-stream collection":** A single file format which acts as a sort-of "envelope" containing components representing the XCCDF, OVAL, CPE Dictionary, and CPE OVAL files.  The SCAP 1.2 "data-stream collection" import workflow simply involves navigating to the "Data Stream Collections" list, and subsequently importing the appropriate file.
4. **OVAL Definitions & OVAL Variables:**  A 2-file format in which the OVAL Variables file contains variable values to be used by the evaluation of the OVAL Definitions.  Importing an OVAL Definitions/Variables combination must begin with the import of the OVAL Variables file, followed by the OVAL Definitions file import.   Also, OVAL Results content may need to be imported into CIS-CAT Pro Dashboard.  OVAL Results information can include Vulnerability Assessment results produced from the CIS-CAT desktop application.

**CPE Dictionaries**

To import a CPE Dictionary XML file, simply navigate to Collections --> OVAL --> CPE Dictionary:

![](http://i.imgur.com/gEHG8YF.png)

Once navigated to the CPE Dictionary List screen, click on the "Import CPE Dictionary" button to display the file upload dialog.  Find and select the appropriate "-cpe-dict.xml" file and click the "Upload" button.  The CPE Dictionary will be imported into the CIS-CAT Pro Dashboard database and displayed for the user.

**OVAL Variables**

To import an OVAL Variables XML file, simply navigate to Collections --> OVAL --> OVAL Variable:

![](http://i.imgur.com/ZBN7pkS.png)

Once navigated to the OVAL Variables List screen, click on the "Import OVAL Variables" button to display the file upload dialog.  Find and select the appropriate "-variables.xml" file and click the "Upload" button.  The OVAL Variables will be imported into the CIS-CAT Pro Dashboard database and displayed for the user.

**OVAL Definitions**

To import an OVAL Definitions XML file, simply navigate to Collections --> OVAL --> OVAL Definition:

![](http://i.imgur.com/HVhhb2j.png)

Once navigated to the OVAL Definitions List screen, click on the "Import OVAL Definintions" button to display the file upload dialog.  Find and select the appropriate "-oval.xml" or "-cpe-oval.xml" file and click the "Upload" button.  The OVAL Definitions will be imported into the CIS-CAT Pro Dashboard database and displayed for the user.

**OVAL Results**

To import an OVAL Results XML file, simply navigate to Collections --> OVAL --> OVAL Result:

![](http://i.imgur.com/Tfjyx8s.png)

Once navigated to the OVAL Results List screen, click on the "Import OVAL Results" button to display the file upload dialog.  Find and select the appropriate OVAL Results file (such as CIS-CAT produced Vulnerability Assessment results XML) and click the "Upload" button.  The OVAL Results will be imported into the CIS-CAT Pro Dashboard database and displayed for the user.  Note that, depending on the size of the OVAL Results file, this import could take several minutes to complete, albeit asynchronously.

**Benchmarks**

To import a Benchmark XCCDF file, simply navigate to Collections --> Benchmarks:

![](http://i.imgur.com/2HioSTk.png)

Once navigated to the Benchmarks List screen, click on the "Import Benchmark" button to display the file upload dialog.  Find and select the appropriate "-xccdf.xml" file and click the "Upload" button.  The Benchmark will be imported into the CIS-CAT Pro Dashboard database and displayed for the user.

**View**

Once navigated to the Benchmarks List screen, the list of previously-imported benchmarks will be displayed in a table format:

![](http://i.imgur.com/5dHbm0k.png)

Each entry in the table represents a unique benchmark XCCDF document.  Benchmarks are uniquely identified by their internal ID and version number.  Therefore, there may be multiple instances of the CIS Microsoft Windows 7 benchmark, but with different version numbers, such as 1.0.0, 2.0.0, or 3.0.0.  Assessment results imported into CIS-CAT Pro Dashboard are associated with a specific version of a benchmark.

Once a user selects a benchmark to view, he/she is taken to the benchmark home page:

![](http://i.imgur.com/N7TGgpw.png)

The benchmark home page details the title and version number of the selected benchmark, and continues to display the descriptive information relevant to the benchmark.  Following an expandable statement of the CIS terms of use, a list of assessment results based on that benchmark is displayed.  Each of the displayed results is a hyperlink allowing the user to navigate to previously imported assessment results.  Expandable "accordions" displaying profile information follow.  After the profile listing, assessed/recommended values are displayed, followed by the high-level groups.  Groups may contain other groups, or may contain rules, which outline the specific benchmark recommendations.

![](http://i.imgur.com/Wq45DYv.png)

Each group is expandable to display any sub-groups or recommendations contained within:

![](http://i.imgur.com/mQX7WJq.png)

Note the description, rationale, remediation, impact statements, any references, and any mapped CIS Critical Controls are also displayed to the user.  Also note that the benchmark content remains in a read-only state.  CIS-CAT Pro Dashboard is merely a repository for already assessed information.  Benchmark tailoring is beyond the scope of CIS-CAT Pro Dashboard.

**Data Stream Collections**

To import a SCAP 1.2 Data Stream Collection XML file, simply navigate to Collections --> Data Stream Collection:

![](http://i.imgur.com/UiVppTm.png)

 
## Critical Controls ##
CIS Critical Controls information will be supplied with the initial version of the CIS-CAT Professional application/database.  This initial data load will contain information regarding the 20 Critical Controls and each respective sub-control.  Users may access controls information by selecting the "Critical Security Controls" menu item, in the main menu bar of the application:

![](http://i.imgur.com/PeJMrya.png)

Once navigated to the main controls "view" page, users have the ability to view information for each individual control:

![](http://i.imgur.com/c9f7aK2.png)

**View**

Once a user navigates to the list of the 20 Critical Controls, he/she may click on any individual control to display specific information about that control, including the control description, objective, and list of applicable sub-controls:

![](http://i.imgur.com/k4cWifY.png)

For each control, any number of sub-controls may be listed.  Users can click on each individual sub-control to display a dialog box containing information about the sub-control:

![](http://i.imgur.com/QDnBCm0.png)
