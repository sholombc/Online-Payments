Configuration Instructions
=========================

Overview
--------
Groundwire Online Payments is a core package for integrating Salesforce.com with your payment gateway for the purpose of collecting one-time and recurring payments directly through the Salesforce.com user interface or a customizable Salesforce Sites page.  This package is the credit card processing engine that connects your Salesforce.com instance with a payment gateway like PayPal or Authorize.Net and hands off your transaction results to a customizable database processing class that exists outside of this managed package. The idea here is that each organization is unique, in terms of how you want your payments processed and recorded in the database, and this package makes no presumptions but allows you to write your own logic and connect it to this credit card engine.

This installation covers:
* Package installation from an install link or the AppExchange
* Overview of custom objects and configuration of apps and tabs included in this package.
* Configuration of one or more Payment Processor connections to PayPal or Authorize.Net, including sandbox environments for testing.
* Basic setup of Salesforce Sites to support a payment page
* Adding and testing of payment listeners, to handle asynchronous transactions and communications from PayPal's Instant Payment Notification service and Authorize.Net's Silent Post, including ongoing recurring payments.

Skills Required
---------------
* AppExchange package installation
* Setup and configuration of Salesforce Sites, including Public Access Settings, Field Visibility and Profile configuration for object and field permissions, tabs and application visibility permissions
* Basic Visualforce
* Configuration of page layouts

Accounts and Platform Licenses Required
---------------------------------------
* Salesforce.com Enterprise Edition or Unlimited Edition
* PayPal Website Payments Pro with Recurring Payments or Authorize.Net Payment Gateway with Automated Recurring Billing (ARB)

Installation Instructions
-------------------------
1. Login to the Groundwire Online Payments managed package instance
2. Navigate to Setup > App Setup > Create > Packages and click on the Versions tab.  Click on the latest Managed Released package and copy the install link to your clipboard.
3. Logout of all Salesforce instances and paste the install link into your browser.  Install for all users.  
4. If your install does not complete right away, you may need to check that the package has been deployed.  Go to Setup > App Setup > View Installed Packages and click on Groundwire Online Payments.  At the top of the package page will be a Deploy button.  If it is grayed out, you're good to go, otherwise you'll need to hit Deploy.

Custom Objects
--------------
Groundwire Online Payments includes three new custom objects:
* **Payment Processors** - This object holds your payment gateway settings to PayPal or Authorize.Net and the following custom fields:
** Payment Processor Name - Text - Your internal name of this payment gateway connection, for reference is search results, list views, etc.
** Payment Processor - Picklist - PayPal or Authorize.Net
** Default Connection - Checkbox - Check this box to use this payment processor connection by default when there are multiple connections available.
** Sandbox - Checkbox - Check this box for Authorize.Net or PayPal test accounts so they will log into the Sandbox URL for API access. If this option is not set properly, your API login will fail.
** API Username - Encrypted Text - Paypal only, required username field
** API Password - Encrypted Text - Paypal only, required password
** API Signature - Encrypted Text - Paypal only, required signature
** API Login Id - Encrypted Text - Authorize.Net only, required API Login Id
** Transaction Key - Encrypted Text - Authorize.Net only, required Transaction Key
* **Payment Notifications** - This objects holds every response that you get when your Salesforce instance communicates with Authorize.Net or PayPal for one-time, recurring payments, or refunds. It contains the following custom fields:
** Notification Title - Text - Auto-name field to indicate where this log originated
** Notification Type - Indicates whether this log came in through a webservice ping, and apex callout from Salesforce to the payment gateway, or an IPN/Silent Post
** Transaction Type - Text - The type of transaction the notification represents when provided by the processor. When we are calling the processor, this will be the name of the processor method we are calling.
** Payer Email - Email - email address of the payer of this transaction
** Payment Status - Text - The status from the Payment Processor: Completed, Authorized, Pending, Declined, Reversed, or Failed. It's not a picklist, because different processors might have different values.
** Payment Amount
** Item Number - Text - If this payment was the purchase of an item, this is the corresponding item number
** Item Name - Text - If this payment was the purchase of an item, this is the corresponding item name
** Test - Checkbox - Indicates whether or not this was a test transaction to a sandbox or test gateway account
** Transaction Id - Text - for single payments, the returned transaction Id of this transaction from your payment gateway
** Recurring Transaction Id - for recurring payments, the returned transaction Id of this transaction from your payment gateway
** Parent Transaction Id - for refunds, the returned transaction Id of this transaction from your payment gateway
** Payer Id - If this payer has an account in your payment gateways system, this is that payer's id in that system
** Processed - Checkbox - indicates whether or not this transaction was processed by Salesforce. This is an internal flag for our package to indicate that database processing has or has not yet occurred.
** Opportunity - Lookup - If the custom processing class that lives outside of this package creates opportunities, it can be set on a Payment notification record which Opportunity relates to this Payment Notification.
** Request or Parameters - Long Text - For caller's this is the full request xml. For listener's, this is a list of name value pairs of the parameters.
** Response - Long Text - The full response string from the processor, when our code has called the processor directly. This will be empty on notifications from the processor.
** Processing Result - Long Text - If the notification is processed, indicates the result of the processing, such as whether a new contact was created or an existing match found. If the notification can't be processed successfully, indicates the error encountered.
** errorLineNumber - Text - for debug purposes only, the line number of code that caused the exception error
** errorStackTrace - Text - for debug purposes only, the stack trace of code that caused the exception error
* **Payment Pages** - This package include a basic but configurable Visualforce page that allows users to create multiple versions of the same payment page with different content like prologue text, epilogue text, amount options, related campaign, or hardcoded item name. The payment page include the following custom fields:
** Payment Page Name - Your internal name of this payment page, for reference is search results, list views, etc.
** Page Title - Text - The page title defines a title in the browser toolbar, provides a title for the page when it is added to favorites and displays a title for the page in search-engine results
** Amount Options - Text - A radio button list of suggested amounts. This list should be separated by semicolons (e.g. 25;50;100;250;1000). A simple amount entry field will be displayed if amount options are not specified.
** Campaign - Lookup - Relate payments from this page to a particular campaign. NOTE: This is only possible if your processing class handles associating payments (like Opportunities) with a campaign, otherwise this field is ignored.
** Include Recurring Payment Option - Checkbox - If checked, the page will display a checkbox field on the payment page to make the amount a monthly recurring payment.
** Item Name - Text - The name of the item being purchased with this payment page. In the case of Groundwire Base, the Item Name is the Opportunity Record Type that is created
** Form Header - Text - The form header is the header text at the top of the form in the body of the page, displayed in H1 style just preceding the Form Prologue text
** Form Prologue - Rich Text - The prologue text to your payment form. This text will show above the payment form on the page.
** Form Epilogue - Rich Text - The epilogue text to your payment form. This text will show below the payment form on the page.
** Thank You Header - Text - The thank you header is the header text at the top of the form in the body of the page, displayed in H1 style upon successful submit of a payment
** Thank You Body Text - Rich Text - The thank you text is displayed in paragraph format in the body of the page upon successful submit of a payment
** Submit Button Text - Text - Text displayed on the submit button of the payment page

Application Permissions
-----------------------
For each custom profile, you will need to set object and field-level permissions for the objects and custom fields contained within this package. We recommend allowing access only to system administrators and user profiles for people who have top-level access to payment information within the organization. System Administrators have all object permissions enabled by default but to enable additional profiles to manage online payments:
* Go to Setup > Administration Setup > Manage Users > Profiles and edit the profiles you wish to enable. 
* In Custom App Settings, make sure the Online Payments app is visible
* In Tab Settings, make sure the following tabs are Default On - About Online Payments, Payment Notifications, Payment Pages, Payment Processors, and Payment Terminal
* In Custom Object Permissions, allow Read, Create, Edit, and Delete permissions on the following objects - Payment Notifications, Payment Pages, and Payment Processors

Configuring Payment Processors
------------------------------
You will need credentials to add at least one payment processor (payment gateway) on either Authorize.Net or PayPal.  This can be a sandbox or developer account if you are just testing the integration.  To add a payment processor:
1. Select Online Payments from the application drop-down menu on the top right of the screen.
2. Click on the Payment Processors tab and then the New button.
3. Enter in your credentials to either PayPal or Authorize.Net.  NOTE: The fields API Login Id and Transaction Key are only for Authorize.Net and the fields API Username, API Password, and API Signature are only for PayPal. You may not enter in credentials from more than one processor into the same record, you must create a new Payment Processor record for each of your Payment Processor connections.
4. If you only have one Payment Processor, or if the processor connection you are entering is your primary payment processor that you want used for all payment pages, be sure to check of the Default Connection checkbox. With some custom coding, you can hardcode a processor connection into your payment pages to indicate that that particular page should not use the default connection but this can only be setup by a consultant/developer.
5. For Sandbox and Developer accounts, you must indicate that the processor connection is a Sandbox by checking off that box on the Payment Processor record.
6. NOTE: Once a Payment Processor record is saved, the credentials are encrypted and can only be viewed by users with the profile permission to View Encrypted Data.  We recommend having no users with this permission if possible, since these credentials could allow someone to credit money directly from your bank account and present a significant financial security risk.  Choose users of this profile wisely.

Configuring Online Payment for Salesforce Sites
-----------------------------------------------
1. First setup your Salesforce.com Site according the the Sites documentation - <https://na12.salesforce.com/help/doc/en/salesforce_platform_portal_implementation_guide.pdf>
2. Once your site is setup, you'll need to enable the following Visualforce Pages - gwop.PaymentListenerAuthnet, gwop.PaymentListenerPaypal, gwop.PaymentSiteTemplate, gwop.payment.  To enable these Visualforce page, go to each site record in Salesforce by clicking on Setup > App Setup > Develop > Sites and click on the Site Label.  On the Site page, in the section called Site Visualforce Pages, click on Edit and locate the pages mentioned above and move them over to the Enabled Visualforce Pages.
3. Still on the Site record, click the Edit button and change the Active Site Home Page to gwop.payment and the Site Template to gwop.PaymentSiteTemplate and click Save
4. Optionally, you can edit the assignments for all of the Site error pages to customized Visualforce pages of your choosing. NOTE: there are no custom error pages included in the package, you would need to create these yourself.
5. Make sure your site is configured with Login = Not Allowed and that the site is Active (a checkbox).
6. Still from the Site record, click on Public Access Settings.  In the section called Field Level Security, under Payment Pages, click the View button and then Edit.  Make sure all fields are marked as Visible. Do the same for Payment Notification. DO NOT do this for Payment Processors. Click the Edit button in the Public Access Settings, and scroll down to Standard Object Permissions.  For Accounts, Contacts and Opportunities, allow Read and Create.  In Custom Object Permissions enable Read, Create, Edit, Delete on Payment Notifications and Payment Pages. Still in the Public Access Settings, click the edit button on Enabled Apex Class Access and enable all classes marked with the prefix gwop.
7. To test to see if your site works, go to the Site record, copy the URL of the Secure Web Address, paste it into a browser and add payment to the URL. For example - https://gwop-developer-edition.na12.force.com/payment - this should bring up the basic payment page.