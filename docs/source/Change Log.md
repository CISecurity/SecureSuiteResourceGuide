![](http://i.imgur.com/5yZfZi5.jpg)

# Change Log #

----------

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
 