# Qlik-Simple-Email-Example
Example of how to leverage Qlik Web Connectors to send email alerts from within your Qlik (Sense or View) load scripts.

## Overview
This example uses Qlik Web Connectors (Notification) to send notifications/alerts to users. Example use cases could be to alert end users of changes in their data or to send messages to DevOps, Administrators, etc... who need to be alerted of load script/data issues.

### Basic Flow Diagram
![Alt](/images/SimpleEmailBasicFlow.png "Qlik Email Notification Basic Flow")


## Installation & Setup
1. Install, Configure, and License Qlik Sense or QlikView [1 OR 2](#references).
1. For Qlik Sense, ensure that you disable Standards Mode for the Engine by following the steps in the Qlik Documentation [3](#references).
1. Install, Configure, and License Qlik Web Connectors (QWC)[4](#references).
1. Test sending an email from QWC
    1. From the Qlik Web Connectors home page, select the "Standards" Connectors tab
    1. Click on the "Notiication Connector"
    1. Select "SendEmail" and select "Parameters" button
    1. Complete the fields and select "Save Inputs & Run Table" to validate it works
    1. This is a required step in order to make sure the credentials are cached on the QWC server and can be used from within your script.
    1. Select the Qlik Sense or QlikView tab in QWC and copy the Script.
1. Open any of your existing Qlik Apps that you want to send an email from in Qlik Sense (or QlikView).
1. Edit the load script and paste the Script you copied above.
1. Reload the App and check your email.
1. After testing is completed, you can implement this in your production applications.


## Example Application
This [Example Simple QWC Email Example][/example/ExampleSimpleQWCEmailExample.qvf] has the load script below as an example of what this could look like in your final application. In this example, an email with the application name and id can be sent to your administrator on a successful loading of the application. This example application has the following four sections:
1. [Main](#section-main) - Standard Main section with specific variables defined.
1. [Data Load](#section-data-load) - Example data load, can be any existing load script you have.
1. [Script Utilities](#section-script-utilities) - Loading the URL Encoding table from w3schools to encode strings.
1. [Send Email](#section-send-email) - Actual email sending portion of example script.

### SECTION: Main
```
SET ThousandSep=',';
SET DecimalSep='.';
SET MoneyThousandSep=',';
SET MoneyDecimalSep='.';
SET MoneyFormat='$#,##0.00;($#,##0.00)';
SET TimeFormat='h:mm:ss TT';
SET DateFormat='M/D/YYYY';
SET TimestampFormat='M/D/YYYY h:mm:ss[.fff] TT';
SET FirstWeekDay=6;
SET BrokenWeeks=1;
SET ReferenceDay=0;
SET FirstMonthOfYear=1;
SET CollationLocale='en-US';
SET CreateSearchIndexOnReload=1;
SET MonthNames='Jan;Feb;Mar;Apr;May;Jun;Jul;Aug;Sep;Oct;Nov;Dec';
SET LongMonthNames='January;February;March;April;May;June;July;August;September;October;November;December';
SET DayNames='Mon;Tue;Wed;Thu;Fri;Sat;Sun';
SET LongDayNames='Monday;Tuesday;Wednesday;Thursday;Friday;Saturday;Sunday';
```

### SECTION: Data Load
```
//Sample data load for illustration purposes only. This section is really unnecessary and should be replaced with your code.

Characters:
Load Chr(RecNo()+Ord('A')-1) as Alpha, RecNo() as Num autogenerate 26;
 
ASCII:
Load 
 if(RecNo()>=65 and RecNo()<=90,RecNo()-64) as Num,
 Chr(RecNo()) as AsciiAlpha, 
 RecNo() as AsciiNum
autogenerate 255
 Where (RecNo()>=32 and RecNo()<=126) or RecNo()>=160 ;
 
Transactions:
Load
 TransLineID, 
 TransID,
 mod(TransID,26)+1 as Num,
 Pick(Ceil(3*Rand1),'A','B','C') as Dim1,
 Pick(Ceil(6*Rand1),'a','b','c','d','e','f') as Dim2,
 Pick(Ceil(3*Rand()),'X','Y','Z') as Dim3,
 Round(1000*Rand()*Rand()*Rand1) as Expression1,
 Round(  10*Rand()*Rand()*Rand1) as Expression2,
 Round(Rand()*Rand1,0.00001) as Expression3;
Load 
 Rand() as Rand1,
 IterNo() as TransLineID,
 RecNo() as TransID
Autogenerate 1000
 While Rand()<=0.5 or IterNo()=1;

 Comment Field Dim1 With "This is a field comment";
```

### SECTION: Script Utilities
```
//URL Encoding Mapping Load - Helps to encode the URL to avoid errors
URL_Encoding_Reference:
MAPPING LOAD
    Replace(Character,'space',' ')	 as Character,
	Text("From UTF-8")				         as URL_Encoding
FROM [https://www.w3schools.com/tags/ref_urlencode.asp]
(html, utf8, embedded labels, table is @1);
```

### SECTION: Send Email
Before loading this app, change the base url (STEP #1), SMTP Server Name (STEP #3), and message attributes (STEP #4)
```
// STEP 1: Change the QWC Server path
SET vQWCBaseURL = 'http://localhost:5555/data';

// STEP 2: Verify/Modify Notification connect string here (if necessary, typically is not)
SET vQWCNotification 	= '$(vQWCBaseURL)?connectorID=NotificationConnector&table=SendEmail';

// STEP 3: Change the SMTP server name, after testing on QWC server and caching the credetials
SET vQWCNotification = '$(vQWCNotification)&SMTPServer=smtp.gmail.com&useSSL=True';

// STEP 4: Change the following variables to affect the recipient, sender, subject, and message
LET vSubject 	= 'Application: ' & DocumentTitle() & ' successfully loaded';
LET vBody		= 'Application: ' & DocumentTitle() & ' (ID: ' & DocumentName() & ') successfully loaded';
LET vTo			= 'steve.newman@qlik.com';
LET vFrom 		= 'newmans99@gmail.com';


// Encode the fields so that they get to QWC correctly, this requires the URL_Encoding_Reference ApplyMap in Script Utilities
LET vSubject 	= MapSubString('URL_Encoding_Reference','$(vSubject)');
LET vBody 		= MapSubString('URL_Encoding_Reference','$(vBody)');
LET vTo 		= MapSubString('URL_Encoding_Reference','$(vTo)');
LET vFrom 		= MapSubString('URL_Encoding_Reference','$(vFrom)');

SET vQWCNotification = '$(vQWCNotification)&html=True&to=$(vTo)&subject=$(vSubject)&message=$(vBody)&from=$(vFrom)&delayInSeconds=15&appID=';

LET vMessage = 'QWC Connection: $(vQWCNotification)';
TRACE $(vMessage);

NotificationConnector_SendEmail:
LOAD status as SendEmail_status,result as SendEmail_result,filesattached as SendEmail_filesattached
FROM [$(vQWCNotification)] (qvx);

```



## References:
1. Qlik Sense Install Documentation: http://help.qlik.com/en-US/sense/Subsystems/Installation/Content/InstallationLicensing/Server-Deployment-Introduction.htm
1. QlikView Install Documentation: http://help.qlik.com/en-US/qlikview/Subsystems/Server/Content/QlikView%20Server/QVSRM_Install_Upgrade.htm
1. Disable Standards Mode: http://help.qlik.com/en-US/sense/Subsystems/Hub/Content/LoadData/disable-standard-mode.htm
1. QWC Install Documentation: http://help.qlik.com/en-US/connectors/Subsystems/Web_Connectors_help/Content/2.1/Installation/Installing-the-web-connectors.htm