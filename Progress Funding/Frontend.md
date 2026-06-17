# Progress Funding – Frontend User Guide
## Overview
The **Progress Funding** process in TeamDesk lets approved users upload two documents on a [Credit Application](https://waterdesk.teamdesk.net/secure/db/76449/setup/table.aspx?table=925268):
  1. **Progress Funding Request Document**
  2. **Progress Funding Promissory Note**


After each upload, TeamDesk automatically notifies the internal team and sends the related update to Aspire.

## Who can see the Progress Funding section
The **Progress Funding** section is hidden by default and is shown only by [Credit Application form behavior](https://waterdesk.teamdesk.net/secure/db/76449/setup/formrules.aspx?table=925268) for:
  - [Default Role](https://waterdesk.teamdesk.net/secure/db/76449/setup/role.aspx?role=230530)
  - [PWP](https://waterdesk.teamdesk.net/secure/db/76449/setup/role.aspx?role=231375)
  - [PWP (Management)](https://waterdesk.teamdesk.net/secure/db/76449/setup/role.aspx?role=236757)
  - specific approved users


If you do not see the section, your role/user access is likely the reason.

## Step 1 — Upload the Progress Funding Request Document
  1. Open the correct **Credit Application** record.
  2. Scroll to the **Progress Funding** section.
  3. Upload a file into [Progress Funding Request Document](https://waterdesk.teamdesk.net/secure/db/76449/setup/column.aspx?column=75061732).
  4. Save the record if needed.


### What happens automatically
When the file is uploaded, [Progress Funding Document Attached](https://waterdesk.teamdesk.net/secure/db/76449/setup/wftrigger.aspx?wftrigger=2951778) runs automatically.

TeamDesk then:
  - sends [Progress Funding Request Email](https://waterdesk.teamdesk.net/secure/db/76449/setup/wfaction.aspx?wfaction=5330711) to the credit team
  - sends the uploaded file to Aspire using [Send To Aspire(Progress Funding)](https://waterdesk.teamdesk.net/secure/db/76449/setup/wfaction.aspx?wfaction=5330712)

### Internal result
  - Credit team receives the uploaded document by email
  - Aspire receives the request document for the related transaction


## Step 2 — Upload the Progress Funding Promissory Note
  1. Open the same **Credit Application** record.
  2. In the **Progress Funding** section, upload a file into [Progress Funding Promissory Note](https://waterdesk.teamdesk.net/secure/db/76449/setup/column.aspx?column=75181177).
  3. Save the record if needed.


### What happens automatically
When the file is uploaded, [Promissory Note Uploaded](https://waterdesk.teamdesk.net/secure/db/76449/setup/wftrigger.aspx?wftrigger=2956826) runs automatically.

TeamDesk then:
  - sends [Promissory Note Email Alert](https://waterdesk.teamdesk.net/secure/db/76449/setup/wfaction.aspx?wfaction=5340100)
  - updates Aspire using [Progress Funding UDF Send](https://waterdesk.teamdesk.net/secure/db/76449/setup/wfaction.aspx?wfaction=5340101)

### Internal result
  - Funding/credit recipients receive the promissory note by email
  - Aspire marks the related contract milestone as complete through UDF CON_30

## Important note about re-uploads
Both upload triggers are configured for **Added, Modified** and run when the attachment field value changes in TeamDesk workflow trigger logic.

That means if a user replaces either file:
  - the related trigger runs again
  - email notifications can be sent again
  - the Aspire call can be sent again

So users should avoid replacing files unless they intentionally want to resubmit.
