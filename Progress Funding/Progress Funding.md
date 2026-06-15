
# Progress Funding / Dealer Usage Cross-Database Audit

## Purpose
This document summarizes the current findings across the databases reviewed for the **Progress Funding** process, with emphasis on:

- whether dealers use the database(s)
- where the **Credit Application** process actually lives
- whether dealers can access **Progress Funding**
- which workflows run after Progress Funding uploads
- whether another database must be treated as the system of record

---

## TeamDesk Behavior Confirmed from Documentation

TeamDesk access is role-based. Record access, column access, form visibility, and view access can all vary by role. Users may also exist in multiple databases independently, with different roles in each database.      

TeamDesk supports:
- **Record Change Triggers** that run when physical column values change, including file attachment columns  
- **Call URL** workflow actions for outbound API/integration requests  
- **Periodic Triggers** for ongoing scheduled synchronization or checks  
- **Form behavior rules** that can show/hide fields or sections by role and criteria, making a section visible to some users and hidden from others  

---

# Database 1: `PWP Customer Service Portal`
Root URL: `https://waterdesk.teamdesk.net/secure/db/76449/setup/default.aspx`

## Inspected Setup Pages
- [Credit Application table properties](https://waterdesk.teamdesk.net/secure/db/76449/setup/table.aspx?table=925268)
- [Credit Application columns](https://waterdesk.teamdesk.net/secure/db/76449/setup/columns.aspx?table=925268)
- [Credit Application form layout](https://waterdesk.teamdesk.net/secure/db/76449/setup/formdefault.aspx?table=925268)
- [Credit Application form behavior](https://waterdesk.teamdesk.net/secure/db/76449/setup/formrules.aspx?table=925268)
- [Credit Application record access](https://waterdesk.teamdesk.net/secure/db/76449/setup/recordsaccess.aspx?table=925268)
- [Credit Application workflow triggers](https://waterdesk.teamdesk.net/secure/db/76449/setup/wftriggers.aspx?table=925268)
- [Promissory Note Uploaded trigger](https://waterdesk.teamdesk.net/secure/db/76449/setup/wftrigger.aspx?wftrigger=2956826)
- [Progress Funding UDF Send action](https://waterdesk.teamdesk.net/secure/db/76449/setup/wfaction.aspx?wfaction=5340101)
- [Promissory Note Email Alert action](https://waterdesk.teamdesk.net/secure/db/76449/setup/wfaction.aspx?wfaction=5340100)

---

## Findings

### 1. Dealers do use the Credit Application process in this database
This is confirmed by the [Credit Application record access](https://waterdesk.teamdesk.net/secure/db/76449/setup/recordsaccess.aspx?table=925268) rules.

Dealer-facing roles include:
- [Dealer Role](https://waterdesk.teamdesk.net/secure/db/76449/setup/role.aspx?role=259454)
- [Lead Manager](https://waterdesk.teamdesk.net/secure/db/76449/setup/role.aspx?role=286333)
- [Sales Manager Role](https://waterdesk.teamdesk.net/secure/db/76449/setup/role.aspx?role=286866)
- [Credit Application Only](https://waterdesk.teamdesk.net/secure/db/76449/setup/role.aspx?role=286183)

### 2. Dealer roles have Credit Application access, but not confirmed Progress Funding access
Dealer-oriented roles can view/add/modify Credit Applications based on dealer-number and user-property rules in [Credit Application record access](https://waterdesk.teamdesk.net/secure/db/76449/setup/recordsaccess.aspx?table=925268).

However, the **Progress Funding** form section is controlled separately by [form behavior](https://waterdesk.teamdesk.net/secure/db/76449/setup/formrules.aspx?table=925268), and the inspected rule makes it visible only for:
- [Default Role](https://waterdesk.teamdesk.net/secure/db/76449/setup/role.aspx?role=230530)
- [PWP](https://waterdesk.teamdesk.net/secure/db/76449/setup/role.aspx?role=231375)
- [PWP (Management)](https://waterdesk.teamdesk.net/secure/db/76449/setup/role.aspx?role=236757)
- specific named users

That means:

> **Confirmed:** dealers can access Credit Applications in this database  
> **Confirmed:** the Progress Funding section is not presently exposed to Dealer Role by the inspected form rule

### 3. Progress Funding is fully configured in this database
The [Credit Application form layout](https://waterdesk.teamdesk.net/secure/db/76449/setup/formdefault.aspx?table=925268) contains a **Progress Funding** section with:
- [Progress Funding Request Document](https://waterdesk.teamdesk.net/secure/db/76449/setup/column.aspx?column=75061732)
- [Progress Funding Promissory Note](https://waterdesk.teamdesk.net/secure/db/76449/setup/column.aspx?column=75181177)

### 4. Promissory Note upload triggers end-to-end automation
The [Promissory Note Uploaded trigger](https://waterdesk.teamdesk.net/secure/db/76449/setup/wftrigger.aspx?wftrigger=2956826) runs when:
- the record is **Added** or **Modified**
- [Progress Funding Promissory Note](https://waterdesk.teamdesk.net/secure/db/76449/setup/column.aspx?column=75181177) changes
- the field is not blank

It launches:
- [Promissory Note Email Alert](https://waterdesk.teamdesk.net/secure/db/76449/setup/wfaction.aspx?wfaction=5340100)
- [Progress Funding UDF Send](https://waterdesk.teamdesk.net/secure/db/76449/setup/wfaction.aspx?wfaction=5340101)

### 5. Backend Aspire update is confirmed
The [Progress Funding UDF Send action](https://waterdesk.teamdesk.net/secure/db/76449/setup/wfaction.aspx?wfaction=5340101) is a POST Call URL action to Aspire:


| Syntax | Description |
| ----------- | ----------- |
| Method | POST |
| Url | ```https://<Aspire Http Link>.leaseteam.net/LeaseTeam.Aspire.API/1/udfs/group``` |
| Headers | None |
| Body | see JSON below |

```json
{
  "Type": "Contract",
  "Fields": [
    {
      "Value": "True",
      "ValueType": "Text",
      "RecordId": {
        "Value": "CON_30",
        "Type": "Record"
      }
    }
  ],
  "RecordId": {
    "Value": "<Contract#>",
    "Type": "Record"
  }
}
```

### 6. Email notification is confirmed
The [Promissory Note Email Alert](https://waterdesk.teamdesk.net/secure/db/76449/setup/wfaction.aspx?wfaction=5340100) sends:
- From: ```Info@Waterdesk.net```
- To:
  - ```Jackie@purewaterpartners.com```
  - ```MBrowne@purewaterpartners.com```
- Cc:
  ```Credit@purewaterpartners.com```
It attaches:
- [Progress Funding Promissory Note](https://waterdesk.teamdesk.net/secure/db/76449/setup/column.aspx?column=75181177)


### 7. Duplicate submission risk is confirmed
Because [Promissory Note Uploaded](https://waterdesk.teamdesk.net/secure/db/76449/setup/wftrigger.aspx?wftrigger=2956826) runs on Modified and on value changes to the attachment column, re-uploading or replacing the file can cause:
- duplicate email alerts
- duplicate Aspire POSTs

This is a confirmed issue.


### 8. No Progress Funding-specific periodic trigger was confirmed
The [Credit Application workflow triggers](https://waterdesk.teamdesk.net/secure/db/76449/setup/wftriggers.aspx?table=925268) page contains many periodic triggers for broader Credit Application/Aspire status processing, but none were confirmed as Progress Funding-specific.


___________________________________________________________________________________


# Progress Funding – Documentation
## Purpose

The purpose of this document is to define the full Progress Funding integration between TeamDesk and Aspire.
It includes the user-driven document upload flow, all confirmed API interactions, related workflow actions, and trigger-based automation that continues after users upload Progress Funding documents.

TeamDesk documentation confirms that:

record access is role-based and may differ by role and database 1
form behavior can separately show or hide sections by role and criteria 2
record change triggers run on changes to physical columns, which includes file attachment columns 3
TeamDesk supports Call URL workflow actions and Periodic Triggers for ongoing synchronization 4 5

This document is based on inspection of these actual setup pages in PWP Customer Service Portal:

Credit Application table properties
Credit Application form behavior
Credit Application record access
Credit Application workflow triggers
Progress Funding Document Attached
Send To Aspire(Progress Funding)
Progress Funding Request Email
Promissory Note Uploaded
Progress Funding UDF Send
Promissory Note Email Alert
Update ContractStatus From Aspire
Term Change(Equipment Asset Only)
Create Billing Location

## Steps
Full Step-by-Step Walkthrough
### Progress Funding Request Flow
1. A user opens a Credit Application record.
2. TeamDesk exposes the Progress Funding section only for the roles/users allowed by Credit Application form behavior. The inspected rule shows this section only to Default Role, PWP, PWP (Management), and specific named users.
3. The user uploads Progress Funding Request Document.
4. **Step 1** runs Progress Funding Document Attached, a record change trigger.
5. TeamDesk sends a notification using Progress Funding Request Email.
6. TeamDesk sends the uploaded document to Aspire using Send To Aspire(Progress Funding) with ```POST /Documents/ProgressFunding/Contract/<Trans#>/transaction```.
7. Aspire stores the uploaded Progress Funding request document against the related contract transaction.

### Progress Funding Promissory Note Flow
A user opens the same Credit Application record and uploads Progress Funding Promissory Note.
Step 2 runs Promissory Note Uploaded, another record change trigger.
TeamDesk sends an email using Promissory Note Email Alert.
TeamDesk performs Progress Funding UDF Send using POST /udfs/group.
Aspire updates the contract UDF group to mark the Progress Funding condition as true.
The external contract remains tracked in Aspire as the system of record.


### Alternate Flow Name – Re-upload / Replacement Flow

This flow differs from the primary upload flow because the triggers are configured for **Added, Modified** and run when the attachment value changes.

That means:

- replacing the Progress Funding request document reruns the request-document flow
- replacing the promissory note reruns the promissory-note flow
- duplicate email alerts and duplicate outbound API calls are possible

This behavior is consistent with TeamDesk documentation for record change triggers on physical column changes 3

1. Progress Funding Document Attached

Description:
This trigger starts the Progress Funding request-document flow when a request file is uploaded to the Credit Application record.

Why this step exists:
Aspire and the internal credit team need the uploaded Progress Funding request document after the user submits it in TeamDesk.

What TeamDesk does:
TeamDesk watches the Progress Funding Request Document attachment column.
The trigger runs when:

the record is Added or Modified
Progress Funding Request Document changes
the field is not blank

It then runs:

Progress Funding Request Email
Send To Aspire(Progress Funding)

Result:
The request document is distributed internally and then posted to Aspire.

2. Progress Funding Request Email

Description:
This email alert notifies the credit team that a Progress Funding request document has been uploaded.

Why this step exists:
The internal team needs immediate notice that a dealer/PWP-side user submitted a Progress Funding request.

What TeamDesk does:
TeamDesk sends an HTML email with:

From: Info@Waterdesk.net
To: credit@purewaterpartners.com, MBrowne@purewaterpartners.com
Cc: Kyle Lawson, Marie Thompson, Me

Subject:

TEXT
Copy Code
[Dealer Name] - Progress Funding Request for Trans [Trans#] - [Company Full Legal Name]


It attaches:

Progress Funding Request Document

Result:
The credit team receives the uploaded request document and a direct TeamDesk record link.

3. Send To Aspire(Progress Funding)

Description:
This Call URL action sends the Progress Funding request document from TeamDesk to Aspire.

Why this step exists:
The uploaded request document must be associated with the correct Aspire contract transaction.

What TeamDesk does:
TeamDesk performs a POST request using the configured authorization account Used for the Credit Application.
The target URL uses:

Aspire Http Link
Trans#, converted to text and trimmed before the decimal

The request body is sent as:

TEXT
Copy Code
asdf=<%[Progress Funding Request Document]%>


The action notes indicate:

document type is currently coming through as "Unknown"

Result:
Aspire receives the Progress Funding request document for the related transaction. The inspected action log shows successful 200 responses and at least one prior 500 response, confirming both active usage and occasional integration failure handling.

API Details
Syntax	Description
Method	POST
Url	https://<%[Aspire Http Link]%>.leaseteam.net/LeaseTeam.Aspire.Api/1/Documents/ProgressFunding/Contract/<%Left(ToText([Trans#]),".")%>/transaction
Headers	Content-Type: application/json
Body	see payload below
JSON
Copy Code
{
  "rawBody": "asdf=<%[Progress Funding Request Document]%>"
}

4. Promissory Note Uploaded

Description:
This trigger starts the promissory-note side of the Progress Funding process after a promissory note file is uploaded.

Why this step exists:
The promissory note upload must notify internal users and update Aspire so the contract reflects the funding milestone.

What TeamDesk does:
TeamDesk watches the Progress Funding Promissory Note attachment column.
The trigger runs when:

the record is Added or Modified
Progress Funding Promissory Note changes
the field is not blank

It then runs:

Promissory Note Email Alert
Progress Funding UDF Send

Result:
The uploaded promissory note causes both an internal alert and an Aspire contract update.

5. Promissory Note Email Alert

Description:
This email alert notifies internal credit/funding recipients that a promissory note has been uploaded.

Why this step exists:
The funding team needs the promissory note immediately after it is submitted in TeamDesk.

What TeamDesk does:
TeamDesk sends an HTML email with:

From: Info@Waterdesk.net
To: Jackie@purewaterpartners.com, MBrowne@purewaterpartners.com
Cc: Kyle Lawson, Marie Thompson, Me, Credit@purewaterpartners.com

Subject:

TEXT
Copy Code
Progress Funding: Promissory Note submitted for [Trans#] - [Company Full Legal Name] - [Dealer Name]


It attaches:

Progress Funding Promissory Note

Result:
The funding/credit team receives the promissory note and a direct link to the TeamDesk record.

6. Progress Funding UDF Send

Description:
This Call URL action updates an Aspire contract UDF group when the promissory note is uploaded.

Why this step exists:
Aspire is the system of record, so TeamDesk needs to signal that the Progress Funding promissory note milestone has been reached.

What TeamDesk does:
TeamDesk performs a POST request to Aspire using the configured authorization account Used for the Credit Application.

The body:

identifies the object type as Contract
sets a UDF field value to True
targets UDF record CON_30
targets the contract using Contract#

Result:
Aspire updates the contract UDF group to mark the contract as having reached the Progress Funding milestone represented by CON_30.

API Details
Syntax	Description
Method	POST
Url	https://<%[Aspire Http Link]%>.leaseteam.net/LeaseTeam.Aspire.API/1/udfs/group
Headers	None
Body	see JSON below
JSON
Copy Code
{
  "Type": "Contract",
  "Fields": [
    {
      "Value": "True",
      "ValueType": "Text",
      "RecordId": {
        "Value": "CON_30",
        "Type": "Record"
      }
    }
  ],
  "RecordId": {
    "Value": "<%[Contract#]%>",
    "Type": "Record"
  }
}

Record Change & Periodic Triggers

The Progress Funding process does not end after submission. TeamDesk continues synchronization using triggers.

These triggers support:

status synchronization
payment or term updates
post-submission structural changes such as billing-location updates

The inspected database does not show a dedicated periodic trigger specifically for Progress Funding only.
However, the Credit Application record continues to participate in broader post-submission sync logic.

7. Update ContractStatus From Aspire

Purpose:
This periodic trigger keeps TeamDesk’s contract status aligned with Aspire after submission.

When it runs:

Type: Periodic
Schedule/Condition: periodic nightly sync; the trigger note says it runs when ContractStatus is not booked and uses recent modified data

Which records it checks:
From prior inspection and current trigger set:

ContractStatus is not Booked
recently modified / in recent period
records where Aspire may have newer status information

What fires the trigger:
This is a periodic trigger, so it is schedule-driven rather than field-change-driven.

What TeamDesk does:

Selects qualifying Credit Application records
Calls Aspire for updated contract status data
Writes refreshed status back into TeamDesk fields

Why this matters:
Progress Funding records still live on Credit Application records. Any later Aspire contract status changes must continue to flow back into TeamDesk.

Result:
TeamDesk remains synchronized to Aspire’s contract status state after the initial Progress Funding uploads.

8. Term Change(Equipment Asset Only)

Purpose:
This trigger recalculates and resends payment information after term or billing changes on submitted applications.

When it runs:

Type: Record Change
Schedule/Condition: when matching submitted equipment-only applications are modified

Which records it checks:
Confirmed from prior inspection:

submitted equipment-only credit applications
triggered by term/frequency-related changes

What fires the trigger:
Changes in:

Billing Freq(New)
billing frequency fields
Term(New)

What TeamDesk does:

Removes old rate sheet data
Re-queries rates
Rebuilds funding values
Re-sends payment information to Aspire
Continues related term-change logic

Why this matters:
If Progress Funding happens on a Credit Application whose financial terms later change, the broader contract data in Aspire remains synchronized.

Result:
Aspire contract payment information is refreshed after qualifying term/frequency changes.

9. Create Billing Location

Purpose:
This trigger creates and applies a separate billing location in Aspire after submission or approval.

When it runs:

Type: Record Change
Schedule/Condition: when qualifying billing-address fields change after submission/approval

Which records it checks:
Confirmed from prior inspection:

Bulk Load Units = No
Use Customer Address = false
ContractStatus is in approved/submitted states

What fires the trigger:
Changes in billing address fields such as:

B-Street 1
B-City
Use Customer Address

What TeamDesk does:

Creates a bill-to location in Aspire
Updates the Aspire contract to use it
Refreshes payment stream information
Updates related attached-unit bill-to values

Why this matters:
Progress Funding records still belong to the same contract lifecycle, so later billing structure changes must stay aligned in Aspire.

Result:
Aspire contract and billing-location structure remain accurate after post-submission billing changes.

Appendix
Example Scenario 1 – Request Document Upload
PWP user opens a Credit Application record.
User uploads Progress Funding Request Document.
Progress Funding Document Attached runs.
TeamDesk emails the credit team.
TeamDesk posts the file to Aspire’s Documents/ProgressFunding/Contract/<Trans#>/transaction endpoint.
Example Scenario 2 – Promissory Note Upload
User uploads Progress Funding Promissory Note.
Promissory Note Uploaded runs.
TeamDesk emails internal recipients with the file attached.
TeamDesk posts a UDF update to Aspire using Progress Funding UDF Send.
Edge Cases
Confirmed issue: re-uploading or replacing either attachment can rerun the trigger, because both triggers run on Added, Modified and on value changes to the file attachment column. This can create duplicate emails and duplicate API calls. This behavior matches TeamDesk trigger behavior on physical column changes 3
Confirmed issue: Send To Aspire(Progress Funding) notes that the uploaded document type is currently arriving in Aspire as "Unknown".
Confirmed issue: Progress Funding UDF Send logs show prior 405 and 422 responses before later successful 200 responses, indicating the integration has had format/state-related failures in production.
Notes
Dealers do use the Credit Application table, based on Credit Application record access.
Dealer roles are not currently confirmed to see the Progress Funding section, because Credit Application form behavior exposes it only to Default Role, PWP, PWP (Management), and named users.
Aspire should be treated as the system of record for contract state after these uploads.

Managing Access For Table Records

Managing Access To Columns

Record Change Triggers

Workflow Triggers

Call Url
