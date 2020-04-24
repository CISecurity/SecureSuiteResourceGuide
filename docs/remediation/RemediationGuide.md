#### Build Kits ####

For Windows technologies, Build Kits take the form of Group Policy Objects (GPOs). The Build Kits are zip files that contain a GPO for each profile within the corresponding CIS Benchmark. These GPOs are intended to be imported into the organization’s group policy management console and pushed out to machines in order to meet compliance with the CIS Benchmark. For additional information, please reference the Read Me document contained within each Build Kit.  The Build Kits for UNIX and LINUX environments are basic shell scripts that can be run from the machine or through another organizationally-approved tool.

These templates or scripts should be modified to align with your organization’s defined policies.  Prior to applying a Build Kit to your test environment, you need to edit the GPO or shell script to only implement the recommendations that you decided to implement in your analysis of the Assessment Results.  Editing of the windows Group Policy Objects or the Linux/Unix shell scripts should be done by a system administrator that is familiar with those technologies.  The system administrator should work with the security policy creator to make sure that the templates match the recommendations the organization decided to implement.

#### Manual Remediation ####

Another option for increasing compliance is through manual remediation.  The recommendation sections of the Assessment Results report contains a remediation segment.  This information is all you need to manually alter the configuration of the target to comply with the CIS Recommendation.  This can be a registry setting or a group policy setting in windows.  

A common workflow for manual remediation is for security policy authors to work with system administrators.  Together they can review the remediations that the security policy author wants to address.  The system administrator can choose recommendations or batches of recommendations that can be implemented and tested.  Doing this in smaller batches minimizes risk and slowly brings the target systems into compliance with the CIS Benchmark.

A tool to assist in this process is the Remediation Report in the CIS-CAT Pro Dashboard.  This report is specifically designed to show just the failing recommendations for a specific target system and benchmark.  The report shows the name of the recommendation and the remediation steps from the full Assessment Result.  This report can be given to an operator to help them manually remediate the failures.

#### Exceptions ####

After a CIS-CAT report is produced and all applicable security recommendations have been implemented according to your organization’s requirements, it is recommended to include an exception report to document the justification as to why some recommendations were not applied. CIS-CAT Pro users may also customize these recommendations to meet organizational requirements by using the tailoring functionality available through CIS WorkBench or by manually altering content in the XCCDF file of a particular CIS Benchmark.

#### What if certain recommendations within a benchmark does not align with my company policy and procedure? ####
Now you've reviewed the recommendations in which your system is not complying with CIS recommendations,  you've remediated the recommendations that fit with your organizations security policy and needs, and you still have failing recommendations that you do not plan to implement.  What now?  There are two paths you can chose to increase compliance after you've decided not to remediate more recommendations:  creating exceptions or an exceptions document or tailoring the CIS content.

**Create an Exceptions Document**
One way to track differences between CIS recommendations and your organizations security policy is to create an exceptions document to track any recommendations do not align with your security policy. The exception should list what target systems the exception applies to, the recommendation number and title and the rationale as to why that recommendation cannot be applied in your environment.

Exceptions are a good way to track this information for organizations that are being audited for CIS compliance.  Exceptions recorded as described usually provide the documentation needed to show that an organization has at least considered a recommendation,  decided not to implement the configuration, and provided a reason for there decision.

**Customize CIS-CAT Content**

 Tailoring CIS content to match your organizations security policy is another valid route to increasing compliance scores and dealing with recommendations you do not intend to implement.  Customization of a benchmark could range from turning on or off a recommendation or tailoring a recommendation to properly align with your organization, such as password length.  These customized security policies provide a personalized organizational policy that you can expect 100% compliance with.

Alterations of CIS-CAT Pro content can be made through the tailoring functionality within CIS WorkBench.

Modifications to the content can also be completed manually in the XML content such as the XCCDF or OVAL files in the Benchmarks folder of the CIS-CAT Pro Assessor. Upon saving the file with the alterations, the assessment will then run against the new modifications and the CIS-CAT report will produce results in correspondence with the changes made.

[Disabling a Recommendation Video](https://www.youtube.com/watch?v=eIEtWFpEAZE)

[Modifying a Recommendation Video](https://www.youtube.com/watch?v=7iO877jX9IM)
