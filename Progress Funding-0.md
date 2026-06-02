#Progress Funding Feature – Documentation
##Overview
The Progress Funding feature enables dealers to submit a signed Progress Funding Promissory Note and triggers downstream processes including email notifications and API integration.
This functionality builds on the existing credit application workflow and introduces a new file upload field plus automation upon submission.

###Feature Flow
1. Dealer Action

The Dealer Admin initiates Progress Funding.
After signing the Progress Funding Promissory Note, they:

Open the Credit Application
Upload the signed document
Save the record




###UI / Data Model Changes
New Field

Field Name: Progress Funding Promissory Note
Type: File Attachment
Visibility:

Hidden by default
Displayed via Form Behavior when:

Progress Funding has been initiated





##Notes

The field accepts PDF files
No column-level restrictions are required
Visibility is controlled entirely through form logic


System Behavior (On Save)
When the file is uploaded and the record is saved, a change trigger executes the following:
1. Email Notification

An email is sent to designated recipients
Email Title: (predefined in requirements)
Purpose: Notify stakeholders that the signed promissory note is available


2. API Integration

A request is sent to an external API endpoint
Uses a predefined call structure

Environment Handling

Initial deployment targets a test endpoint (test2)
This allows validation before production rollout

Release Strategy

Push to test endpoint
Validate with stakeholders (e.g., Marie)
Confirm successful processing
Promote to live environment


Implementation Steps
Step 1 – Create Field

Add a new column:

Name: Progress Funding Promissory Note
Type: File Attachment
Set to hidden by default




Step 2 – Configure Form Behavior

Locate the new field in form configuration
Add logic to:

Show the field only when Progress Funding is active




Step 3 – Add Change Trigger


Trigger condition:

Field is populated AND record is saved



Actions:

Send email notification
Call API endpoint




Step 4 – Test in Sandbox

Use designated test application (e.g., “Test Company 555”)
Validate:

File upload functionality
Trigger execution
Email delivery
API request success




Step 5 – Validate in Test Environment

Push integration to test2 endpoint
Confirm:

API receives payload correctly
Downstream system processes request




Step 6 – Production Rollout

After validation:

Switch endpoint to production
Monitor initial runs




##Additional Notes / Learnings

Always hide new fields by default, then expose via form behavior
Use sandbox environments for safe testing (e.g., Project Bedrock sandbox)
Avoid manual input errors (e.g., copy-paste long names like “promissory”)
Validate data formatting issues (e.g., leading spaces in fields)
Consider using pattern-based logic (e.g., prefix checks like 700...) when filtering data


##Dependencies

Email service configuration
API endpoint availability
Form behavior logic tied to Progress Funding trigger


##Future Considerations

Add validation for file type (PDF-only enforcement)
Add audit tracking for file uploads
Improve UI clarity around when the field becomes available
Expand automation tied to contract IDs or patterns if needed
