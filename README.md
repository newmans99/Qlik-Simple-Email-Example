# Qlik-Simple-Email-Example
Example of how to leverage Qlik Web Connectors to send email alerts from within your Qlik (Sense or View) load scripts.

## Overview
This example uses Qlik Web Connectors (Notification) to send notifications/alerts to users. Example use cases could be to alert end users of changes in their data or to send messages to DevOps, Administrators, etc... who need to be alerted of load script/data issues.

### Basic Flow Diagram
![Alt](/newmans99/Qlik-Simple-Email-Example/blob/master/images/SimpleEmailBasicFlow.png "Qlik Email Notification Basic Flow")


## Installation & Setup
<ol>
<li> Install, Configure, and License Qlik Sense or QlikView
<ol>
<li> For Qlik Sense, ensure that you disable Standards Mode for the Engine by following the steps in the [Qlik Documentation][1].
</ol>
<li> Install, Configure, and License Qlik Web Connectors (QWC)
<li> Test sending an email from QWC
<ol>
<li> From the Qlik Web Connectors home page, go to the "Standards" Connectors tab then...
<li> Click on the "Notiication Connector"
<li> Select "SendEmail" and select "Parameters" button
<li> Complete the fields and select "Save Inputs & Run Table" to validate it works
<li> This is a required in order to make sure the credentials are cached and can be used.
<li> Select the Qlik Sense or QlikView tab in QWC and copy the Script.
</ol>
<li> Open any existing Qlik App that you want to send an email from in Qlik Sense (or QlikView).
<li> Edit the load script and paste the Script you copied above.
<li> Reload the App and check your email.
<li> After testing is completed, you can implement in your production applications.
</ol>

## Documentation




[1]: http://help.qlik.com/en-US/sense/Subsystems/Hub/Content/LoadData/disable-standard-mode.htm "Disable Standards Mode"