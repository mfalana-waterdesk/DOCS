# Progress Funding – Documentation

The purpose of this document is to define the full Progress Funding integration between TeamDesk and Aspire.
It includes the user-driven document upload flow, all confirmed API interactions, related workflow actions, and trigger-based automation that continues after users upload Progress Funding documents.

## Full Step-by-Step Walkthrough
### Progress Funding Request Flow
1. A user opens a [Credit Application](https://waterdesk.teamdesk.net/secure/db/76449/setup/table.aspx?table=925268) record.
2. TeamDesk exposes the **Progress Funding** section only for the roles/users allowed by [Credit Application form behavior](https://waterdesk.teamdesk.net/secure/db/76449/setup/formrules.aspx?table=925268). The inspected rule shows this section only to [Default Role](https://waterdesk.teamdesk.net/secure/db/76449/setup/role.aspx?role=230530), [PWP](https://waterdesk.teamdesk.net/secure/db/76449/setup/role.aspx?role=231375), [PWP (Management)](https://waterdesk.teamdesk.net/secure/db/76449/setup/role.aspx?role=236757), and specific named users.
3. The user uploads [Progress Funding Request Document](https://waterdesk.teamdesk.net/secure/db/76449/setup/column.aspx?column=75061732).
4. **Step 1** runs [Progress Funding Document Attached](https://waterdesk.teamdesk.net/secure/db/76449/setup/wftrigger.aspx?wftrigger=2951778), a record change trigger.
5. TeamDesk sends a notification using [Progress Funding Request Email](https://waterdesk.teamdesk.net/secure/db/76449/setup/wfaction.aspx?wfaction=5330711).
6. TeamDesk sends the uploaded document to Aspire using [Send To Aspire(Progress Funding)](https://waterdesk.teamdesk.net/secure/db/76449/setup/wfaction.aspx?wfaction=5330712) with ```POST /Documents/ProgressFunding/Contract/<Trans#>/transaction```.
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



### [1. Progress Funding Document Attached](https://waterdesk.teamdesk.net/secure/db/76449/setup/wftrigger.aspx?wftrigger=2951778)

**Description:** This trigger starts the Progress Funding request-document flow when a request file is uploaded to the Credit Application record.

**Why this step exists:** Aspire and the internal credit team need the uploaded Progress Funding request document after the user submits it in TeamDesk.

**What TeamDesk does:** TeamDesk watches the [Progress Funding Request Document](https://waterdesk.teamdesk.net/secure/db/76449/setup/column.aspx?column=75061732) attachment column.
The trigger runs when:
  - the record is **Added** or **Modified**
  - [Progress Funding Request Document](https://waterdesk.teamdesk.net/secure/db/76449/setup/column.aspx?column=75061732) changes
  - the field is not blank

It then runs:
  1. [Progress Funding Request Email](https://waterdesk.teamdesk.net/secure/db/76449/setup/wfaction.aspx?wfaction=5330711)
  2. [Send To Aspire(Progress Funding)](https://waterdesk.teamdesk.net/secure/db/76449/setup/wfaction.aspx?wfaction=5330712)

**Result:** The request document is distributed internally and then posted to Aspire.



### [2. Progress Funding Request Email](https://waterdesk.teamdesk.net/secure/db/76449/setup/wfaction.aspx?wfaction=5330711)

**Description:** This email alert notifies the credit team that a Progress Funding request document has been uploaded.

**Why this step exists:** The internal team needs immediate notice that a dealer/PWP-side user submitted a Progress Funding request.

**What TeamDesk does:** TeamDesk sends an HTML email with:
  - **From:** Info@Waterdesk.net
  - **To:** credit@purewaterpartners.com, MBrowne@purewaterpartners.com

Subject: ```[Dealer Name] - Progress Funding Request for Trans [Trans#] - [Company Full Legal Name]```

It attaches:
  - [Progress Funding Request Document](https://waterdesk.teamdesk.net/secure/db/76449/setup/column.aspx?column=75061732)

**Result:** The credit team receives the uploaded request document and a direct TeamDesk record link.




### [3. Send To Aspire(Progress Funding)](https://waterdesk.teamdesk.net/secure/db/76449/setup/wfaction.aspx?wfaction=5330712)

**Description:** This Call URL action sends the Progress Funding request document from TeamDesk to Aspire.

**Why this step exists:** The uploaded request document must be associated with the correct Aspire contract transaction.

**What TeamDesk does:** TeamDesk performs a ```POST``` request using the configured authorization account [Used for the Credit Application](https://waterdesk.teamdesk.net/secure/db/76449/setup/appaccount.aspx?account=1580).
The target URL uses:

  - [Aspire Http Link](https://waterdesk.teamdesk.net/secure/db/76449/setup/column.aspx?column=32697989)
  - [Trans#](https://waterdesk.teamdesk.net/secure/db/76449/setup/column.aspx?column=30847778), converted to text and trimmed before the decimal

The request body is sent as: ```asdf=<%[Progress Funding Request Document]%>```

The action notes indicate:

  | document type is currently coming through as "Unknown"

**Result:** Aspire receives the Progress Funding request document for the related transaction. The inspected action log shows successful ```200``` responses and at least one prior ```500``` response, confirming both active usage and occasional integration failure handling.

| Syntax | Description |
| ----------- | ----------- |
| Method | POST |
| Url | ```https://<%[Aspire Http Link]%>.leaseteam.net/LeaseTeam.Aspire.Api/1/Documents/ProgressFunding/Contract/<%Left(ToText([Trans#]),".")%>/transaction``` |
| Headers | ```Content-Type: application/json``` |
| Body | see JSON below |
 
```json
{
  "rawBody": "asdf=<%[Progress Funding Request Document]%>"
}
```




### [4. Promissory Note Uploaded](https://waterdesk.teamdesk.net/secure/db/76449/setup/wftrigger.aspx?wftrigger=2956826)

**Description:** This trigger starts the promissory-note side of the Progress Funding process after a promissory note file is uploaded.

**Why this step exists:** The promissory note upload must notify internal users and update Aspire so the contract reflects the funding milestone.

**What TeamDesk does:** TeamDesk watches the [Progress Funding Promissory Note](https://waterdesk.teamdesk.net/secure/db/76449/setup/column.aspx?column=75181177) attachment column.
The trigger runs when:
  - the record is **Added** or **Modified**
  - [Progress Funding Promissory Note](https://waterdesk.teamdesk.net/secure/db/76449/setup/column.aspx?column=75181177) changes
  - the field is not blank

It then runs:
1. [Promissory Note Email Alert](https://waterdesk.teamdesk.net/secure/db/76449/setup/wfaction.aspx?wfaction=5340100)
2. [Progress Funding UDF Send](https://waterdesk.teamdesk.net/secure/db/76449/setup/wfaction.aspx?wfaction=5340101)

**Result:** The uploaded promissory note causes both an internal alert and an Aspire contract update.





### [5. Promissory Note Email Alert](https://waterdesk.teamdesk.net/secure/db/76449/setup/wfaction.aspx?wfaction=5340100)

**Description:** This email alert notifies internal credit/funding recipients that a promissory note has been uploaded.

**Why this step exists:** The funding team needs the promissory note immediately after it is submitted in TeamDesk.

**What TeamDesk does:** TeamDesk sends an HTML email with:
  - **From:** Info@Waterdesk.net
  - **To:** Jackie@purewaterpartners.com, MBrowne@purewaterpartners.com
  - **Cc:** Kyle Lawson, Marie Thompson, Me, Credit@purewaterpartners.com

Subject:

```TEXT
Progress Funding: Promissory Note submitted for [Trans#] - [Company Full Legal Name] - [Dealer Name]
```


It attaches:
  - [Progress Funding Promissory Note](https://waterdesk.teamdesk.net/secure/db/76449/setup/column.aspx?column=75181177)

**Result:** The funding/credit team receives the promissory note and a direct link to the TeamDesk record.





### [6. Progress Funding UDF Send](https://waterdesk.teamdesk.net/secure/db/76449/setup/wfaction.aspx?wfaction=5340101)

**Description:** This Call URL action updates an Aspire contract UDF group when the promissory note is uploaded.

**Why this step exists:** Aspire is the system of record, so TeamDesk needs to signal that the Progress Funding promissory note milestone has been reached.

**What TeamDesk does:** TeamDesk performs a POST request to Aspire using the configured authorization account [Used for the Credit Application](https://waterdesk.teamdesk.net/secure/db/76449/setup/appaccount.aspx?account=1580).

The body:
  - identifies the object type as Contract
  - sets a UDF field value to True
  - targets UDF record CON_30
  - targets the contract using [Contract#](https://waterdesk.teamdesk.net/secure/db/76449/setup/column.aspx?column=30842436)

**Result:** Aspire updates the contract UDF group to mark the contract as having reached the Progress Funding milestone represented by CON_30.

| Syntax | Description |
| ----------- | ----------- |
| Method | POST |
| Url | ```https://<%[Aspire Http Link]%>.leaseteam.net/LeaseTeam.Aspire.API/1/udfs/group``` |
| Headers | ```None``` |
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
    "Value": "<%[Contract#]%>",
    "Type": "Record"
  }
}
```

### Record Change & Periodic Triggers

The Progress Funding process does not end after submission. TeamDesk continues synchronization using triggers.

These triggers support:

  - status synchronization
  - payment or term updates
  - post-submission structural changes such as billing-location updates

The inspected database does **not** show a dedicated periodic trigger specifically for Progress Funding only.
However, the Credit Application record continues to participate in broader post-submission sync logic.




### 7. [Update ContractStatus From Aspire](https://waterdesk.teamdesk.net/secure/db/76449/setup/wftrigger.aspx?wftrigger=2827983)

**Purpose:** This periodic trigger keeps TeamDesk’s contract status aligned with Aspire after submission.

**When it runs:**
- **Type:** Periodic
- **Schedule/Condition:** periodic nightly sync; the trigger note says it runs when ContractStatus is not booked and uses recent modified data

**Which records it checks:**
From prior inspection and current trigger set:
  - [ContractStatus](https://waterdesk.teamdesk.net/secure/db/76449/setup/column.aspx?column=30830792) is not Booked
  - recently modified / in recent period
  - records where Aspire may have newer status information

**What fires the trigger:** This is a periodic trigger, so it is schedule-driven rather than field-change-driven.

**What TeamDesk does:**
1. Selects qualifying Credit Application records
2. Calls Aspire for updated contract status data
3. Writes refreshed status back into TeamDesk fields

**Why this matters:** Progress Funding records still live on Credit Application records. Any later Aspire contract status changes must continue to flow back into TeamDesk.

**Result:** TeamDesk remains synchronized to Aspire’s contract status state after the initial Progress Funding uploads.




### 8. [Term Change(Equipment Asset Only)](https://waterdesk.teamdesk.net/secure/db/76449/setup/wftrigger.aspx?wftrigger=1216644)

**Purpose:** This trigger recalculates and resends payment information after term or billing changes on submitted applications.

**When it runs:**
  - **Type:** Record Change
  - **Schedule/Condition:** when matching submitted equipment-only applications are modified

**Which records it checks:**
  - submitted equipment-only credit applications
  - triggered by term/frequency-related changes

**What fires the trigger:**
Changes in:
  - [Billing Freq(New)](https://waterdesk.teamdesk.net/secure/db/76449/setup/column.aspx?column=30830795)
  - billing frequency fields
  - [Term(New)](https://waterdesk.teamdesk.net/secure/db/76449/setup/column.aspx?column=30830834)

**What TeamDesk does:**
  1. Removes old rate sheet data
  2. Re-queries rates
  3. Rebuilds funding values
  4. Re-sends payment information to Aspire
  5. Continues related term-change logic

**Why this matters:** If Progress Funding happens on a Credit Application whose financial terms later change, the broader contract data in Aspire remains synchronized.

**Result:** Aspire contract payment information is refreshed after qualifying term/frequency changes.




### [9. Create Billing Location](https://waterdesk.teamdesk.net/secure/db/76449/setup/wftrigger.aspx?wftrigger=1710404)

**Purpose:** This trigger creates and applies a separate billing location in Aspire after submission or approval.

**When it runs:**
  - **Type:** Record Change
  - **Schedule/Condition:** when qualifying billing-address fields change after submission/approval

**Which records it checks:**
  - [Bulk Load Units](https://waterdesk.teamdesk.net/secure/db/76449/setup/column.aspx?column=43648599) = No
  - [Use Customer Address](https://waterdesk.teamdesk.net/secure/db/76449/setup/column.aspx?column=30847779) = false
  - [ContractStatus](https://waterdesk.teamdesk.net/secure/db/76449/setup/column.aspx?column=30830792) is in approved/submitted states

**What fires the trigger:**
Changes in billing address fields such as:
  - [B-Street 1](https://waterdesk.teamdesk.net/secure/db/76449/setup/column.aspx?column=30830844)
  - [B-City](https://waterdesk.teamdesk.net/secure/db/76449/setup/column.aspx?column=30830846)
  - [Use Customer Address](https://waterdesk.teamdesk.net/secure/db/76449/setup/column.aspx?column=30847779)

**What TeamDesk does:**
  1. Creates a bill-to location in Aspire
  2. Updates the Aspire contract to use it
  3. Refreshes payment stream information
  4. Updates related attached-unit bill-to values

**Why this matters:** Progress Funding records still belong to the same contract lifecycle, so later billing structure changes must stay aligned in Aspire.

**Result:** Aspire contract and billing-location structure remain accurate after post-submission billing changes.



## Appendix
### Example Scenario 1 – Request Document Upload
1. PWP user opens a Credit Application record.
2. User uploads [Progress Funding Request Document](https://waterdesk.teamdesk.net/secure/db/76449/setup/column.aspx?column=75061732).
3. [Progress Funding Document Attached](https://waterdesk.teamdesk.net/secure/db/76449/setup/wftrigger.aspx?wftrigger=2951778) runs.
4. TeamDesk emails the credit team.
5. TeamDesk posts the file to Aspire’s ```Documents/ProgressFunding/Contract/<Trans#>/transaction``` endpoint.


### Example Scenario 2 – Promissory Note Upload
1. User uploads [Progress Funding Promissory Note](https://waterdesk.teamdesk.net/secure/db/76449/setup/column.aspx?column=75181177).
2. [Promissory Note Uploaded](https://waterdesk.teamdesk.net/secure/db/76449/setup/wftrigger.aspx?wftrigger=2956826) runs.
3. TeamDesk emails internal recipients with the file attached.
4. TeamDesk posts a UDF update to Aspire using [Progress Funding UDF Send](https://waterdesk.teamdesk.net/secure/db/76449/setup/wfaction.aspx?wfaction=5340101).
