# Full New Dealer Setup – Documentation

## Purpose

The purpose of this document is to explain the current TeamDesk process for **new dealer setup** across the databases and related integrations involved.

This process includes:

- creating the dealer in the **PWP Customer Service Portal**
- sending or updating the related Opportunity from the **Dealer WaterDesk Opportunity database**
- confirming that zip code, market, location, and office routing are already in place or updated as needed
- updating work-order routing when a new office is added
- confirming downstream TeamDesk integrations continue to work

This document is based on confirmed live setup findings where available and clearly notes anything that still needs to be confirmed before the process documentation is considered complete.

To make the process easier to follow, this guide separates what happens in:

- the **PWP Customer Service Portal**
- the **Dealer WaterDesk / Bottleless Nation database**
- the shared routing / integration setup that supports both

---

## Steps

### Full Step-by-Step Walkthrough

#### Primary Flow

### A. PWP Customer Service Portal

1. Dealer creation begins in the **PWP Customer Service Portal** using [Create New Dealer](https://waterdesk.teamdesk.net/secure/db/76449/setup/custbtn.aspx?custbtn=1186132).

2. TeamDesk copies the source Account’s `[Trans #]` into `[Dealer Id]`.

3. TeamDesk runs [Create New Dealer Record](https://waterdesk.teamdesk.net/secure/db/76449/setup/wfaction.aspx?wfaction=2523053) to create the Dealer record.

4. TeamDesk runs [Dealer Record Created](https://waterdesk.teamdesk.net/secure/db/76449/setup/wfaction.aspx?wfaction=2523059) to mark the source Account as already processed.

5. After the Dealer is created, the new Dealer record may need to be updated if it should be treated as a **branch office** instead of a **head office**.

### B. Dealer WaterDesk / Bottleless Nation database

6. In the **Dealer WaterDesk Opportunity database**, an Opportunity is sent or updated using one of the PWP submission buttons.

7. TeamDesk sends that Opportunity to the PWP Customer Service Portal using this endpoint:

   `POST https://waterdesk.teamdesk.net/secure/api/v2/76449/Credit Application/upsert.json`

8. On new-create paths, TeamDesk stores the returned IDs back on the Opportunity:
   - `Aspire CS Portal ID`
   - `Aspire CS Portal Record ID`

9. In the **Water Desk / Bottleless Nation database**, if the new dealer also involves a new office, work-order routing may need to be updated so equipment updates, swaps, pickups, and shop repair work route to the correct location.

### C. Shared downstream setup

10. After that, everything depends on the supporting setup being correct for:
   - Companies
   - Zip Codes
   - Locations
   - Offices
   - Work Orders
   - Omniflow integration

---

## 1. [Create New Dealer](https://waterdesk.teamdesk.net/secure/db/76449/setup/custbtn.aspx?custbtn=1186132)

**Database:**  
**PWP Customer Service Portal**

**Description:**  
This is the starting point for creating a dealer in the **PWP Customer Service Portal**.

**Why this step exists:**  
It allows an Account to be converted into a Dealer.

**What TeamDesk does:**  
This button only appears when:

- `[Role] is Dealer`
- `[Dealer Record Created] is not checked`

The current live setup also shows:

- it appears on the record preview page
- it is under `Miscellaneous Actions`
- it is currently available to `Default Role`
- it shows the confirmation message: `Dealer has been created.`

When clicked, it:

- copies `[Trans #] → [Dealer Id]`
- runs [Create New Dealer Record](https://waterdesk.teamdesk.net/secure/db/76449/setup/wfaction.aspx?wfaction=2523053)
- runs [Dealer Record Created](https://waterdesk.teamdesk.net/secure/db/76449/setup/wfaction.aspx?wfaction=2523059)

**Result:**  
A new Dealer record is created, and the original Account is marked so the same dealer is not created twice.

---

## 2. [Create New Dealer Record](https://waterdesk.teamdesk.net/secure/db/76449/setup/wfaction.aspx?wfaction=2523053)

**Database:**  
**PWP Customer Service Portal**

**Description:**  
This is the action that actually creates the Dealer record.

**Why this step exists:**  
It takes information from the Account and builds the new Dealer record.

**What TeamDesk does:**  
It creates a record in the [Dealer table](https://waterdesk.teamdesk.net/secure/db/76449/setup/table.aspx?table=889363).

Confirmed field assignments include:

- `[Name] → Dealer Name`
- `[Name] → Parent Company`
- `true → Head Office`
- `[Dealer Id] → Dealer No`
- `[AccountId] → Dealer Account ID`
- address, city, state, postal code
- phone and email
- `"Active" → Account Status`
- `Today() → Account Status Date`
- `"Business" → Business or Individual`
- `[Name] → Name`
- `[** Main Phone] → Main Number`

Current live setup also shows:

- **Execute Triggers:** `No`

**Result:**  
A new Dealer record is created in the PWP Customer Service Portal.

---

## 3. [Dealer Record Created](https://waterdesk.teamdesk.net/secure/db/76449/setup/wfaction.aspx?wfaction=2523059)

**Database:**  
**PWP Customer Service Portal**

**Description:**  
This is the final step in the immediate dealer-creation flow.

**Why this step exists:**  
It prevents the same dealer from being created more than once from the same Account.

**What TeamDesk does:**  
It updates the source Account:

- `Dealer Record Created = true`

Current live setup also shows:

- **Execute Triggers:** `No`

**Result:**  
The Account is marked as already processed.

---

## 4. Dealer cleanup after creation

**Database:**  
**PWP Customer Service Portal**

**Description:**  
After the Dealer is created, the record may need to be updated depending on whether it should stay as a head office or be treated as a branch.

**Why this step exists:**  
The current create action always creates the new Dealer as a head office first.

**What TeamDesk does:**  
Current live inspection shows:

- [Head Office](https://waterdesk.teamdesk.net/secure/db/76449/setup/column.aspx?column=32194939) is a checkbox
- [Parent Company](https://waterdesk.teamdesk.net/secure/db/76449/setup/column.aspx?column=28868523) is a text field
- [Create New Dealer Record](https://waterdesk.teamdesk.net/secure/db/76449/setup/wfaction.aspx?wfaction=2523053) sets:
  - `Head Office = true`
  - `Parent Company = [Name]`

This means the system does **not** currently appear to automatically convert a new dealer into a branch structure after creation.

**What may need to be done manually for a branch office:**  

- uncheck [Head Office](https://waterdesk.teamdesk.net/secure/db/76449/setup/column.aspx?column=32194939)
- update [Parent Company](https://waterdesk.teamdesk.net/secure/db/76449/setup/column.aspx?column=28868523) to the correct parent dealer name

**Result:**  
A new dealer may need a manual cleanup step if it is meant to be a branch office and not the main office.

**Important note:**  
A Bottleless Nation-specific automatic cleanup rule was **not** confirmed from the pages inspected so far.

---

## 5. Company / Zip / Market / Location Routing Structure

**Database Area:**  
**Shared operational setup, mainly used by Water Desk routing**

**Description:**  
This is the setup that decides where work and activity go after the dealer is created.

**Why this step exists:**  
A dealer setup is not really complete unless the dealer can route correctly by market, territory, office, and location.

**What TeamDesk does:**  
Previously confirmed live setup shows:

### Companies

- `Zip Code` links to **Zip Codes**
- `Account Market` is pulled from the zip’s `Market`

### Zip Codes

Zip Codes contain routing fields such as:

- `Market`
- `Service Market`
- `Service/Sales Market`
- `Dealer`
- `Dealer Number`
- `Sales Rep`
- `Primary Tech`
- `Zip Territory`
- `Territory By Zip`
- `Sub Territory By Zip`
- `Multi-Territory`

### Locations

- `Zip Code` links to **Zip Codes**
- market and territory values are pulled from the zip
- `Market Form` uses `Market Override` if it exists, otherwise `Market`

### Offices

- Offices link to **Locations**
- they inherit market/location information from the linked Location

**Result:**  
Routing depends heavily on zip code, location, and office setup.

**Important note:**  
Some markets or locations may already exist. For example, **Nashville** was previously used as an example because that routing was already in place, not because every new dealer setup requires Nashville-specific work.

---

## 6. `Send To PWP CS Portal`

**Database:**  
**Dealer WaterDesk Opportunity database**

**Description:**  
This is the normal button used to send a new Opportunity into the PWP Customer Service Portal.

**Why this step exists:**  
It creates a new Credit Application in the PWP Customer Service Portal from the Opportunity record.

**What TeamDesk does:**  
Confirmed live behavior:

- roles:
  - `Admin`
  - `Admin 2`
  - `Default Role`

- filter:
  - `Aspire CS Portal ID is blank`
  - `Term >= 24`

It runs:

- `Send New To Aspire CS Portal Redux`

**Result:**  
A new create/upsert submission is started.

---

## 7. `Send New To Aspire CS Portal Redux`

**Database:**  
**Dealer WaterDesk Opportunity database** sending into **PWP Customer Service Portal**

**Description:**  
This is the action that sends the Opportunity data to the PWP Customer Service Portal.

**Why this step exists:**  
It creates or updates the Credit Application record in the PWP system and stores the returned IDs back on the Opportunity.

**What TeamDesk does:**  
Previously confirmed setup:

- **Authorization:** `Admin Auth`
- **Method:** `POST`
- **URL:** `https://waterdesk.teamdesk.net/secure/api/v2/76449/Credit Application/upsert.json`
- **Execute Triggers:** `No`

Confirmed payload includes:

- company legal name
- shipping address fields
- term
- billing frequency
- total units
- total monthly payment
- source app/opportunity IDs
- `Administrator Id = 600`
- market-based `Case([Market], ...)` mappings for:
  - Dealer No
  - Select the Distributor
  - Dealer Name
- source record IDs
- company phone and email
- `DBKEY`

Confirmed response assignments:

- `$[0].key → Aspire CS Portal ID`
- `$[0].id → Aspire CS Portal Record ID`

**Result:**  
The Opportunity is pushed into the PWP Customer Service Portal and the returned IDs are saved back onto the Opportunity.

### API Details

| Syntax | Description |
|--------|------------|
| Method | POST |
| Url | `https://waterdesk.teamdesk.net/secure/api/v2/76449/Credit Application/upsert.json` |
| Headers | `Content-Type: text/json` |
| Body | see JSON below |

```json
{
  "Company Full Legal Name": "<%[Company Name]%>",
  "New Main Address": "<%[Shipping Address Line 1]%>",
  "New Main Address#2": "<%[Shipping Address Line 2]%>",
  "New City": "<%[Shipping City]%>",
  "New State": "<%Upper([Shipping State])%>",
  "New Zip/Postal Code": "<%[Shipping Zip]%>",
  "Term(New)": "<%Left(ToText([Term]),'.')%>",
  "Billing Freq(New)": "<%[Billing Freq]%>",
  "Total Number of Units": "<%ToText([Total Qty])%>",
  "Total_Monthly_Payment_Amount": "<%ToText([Total TMP])%>",
  "Waterdesk ID": "<%AppId()%>",
  "Waterdesk Opp Id": "<%[Id]%>",
  "Administrator Id": "600",
  "Dealer No": "<%Case([Market], ...)%>",
  "Select the Distributor": "<%Case([Market], ...)%>",
  "Dealer Name": "<%Case([Market], ...)%>",
  "Opportunity Record ID": "<%RecordId()%>",
  "Company Record ID": "<%[Company Record ID]%>",
  "Location Record ID": "<%[Location Record ID]%>",
  "AS1-Phone": "<%[Company Phone]%>",
  "New Email": "<%[Company Email]%>",
  "DBKEY#": "<%Var[DBKEY]%>"
}

## 8. MTU and Update Paths

**Database:**  
**Dealer WaterDesk Opportunity database**

**Description:**  
There are also separate paths for Midterm Upgrade and update requests.

**Why this step exists:**  
Not every Opportunity is a brand-new create. Some are MTU requests, and some are updates to records already sent before.

**What TeamDesk does:**  
Confirmed live behavior:

### `Send to PWP CS Portal(MTU)`
- roles:
  - `Admin`
  - `Admin 2`
  - `Default Role`
  - `Partnership Program`
  - `ReadOnly`
- filter:
  - `Aspire CS Portal ID is blank`
  - `Aspire CS Portal Record ID is blank`
  - `Contract Type = Midterm Upgrade`

### `Send Update to PWP CS Portal`
- roles:
  - `Admin`
  - `Admin 2`
  - `Default Role`
  - `Partnership Program`
  - `ReadOnly`
- filter:
  - `Aspire CS Portal ID is not blank`

Also previously confirmed to exist:

- `Send New to Credit App MTU`
- `Send Update To CA Redux`

Earlier inspection also showed:

- MTU uses a different payload
- update path includes `Id` from `Aspire CS Portal ID`
- both still send to the same PWP Customer Service Portal endpoint

**Result:**  
MTU and update paths are part of the current process.

---

## 9. Work Order Routing

**Database:**  
**Water Desk / Bottleless Nation database**

**Description:**  
Work Order routing is one of the backend pieces that must be updated when a new office is added.

**Why this step exists:**  
A dealer rollout is not fully complete if service, install, pickup, or swap work orders do not go to the correct place.

**What TeamDesk does:**  
The following live trigger maintenance points are confirmed:

### Triggers
- `Service Swap Update`
- `Sales Swap Update`
- `Pickup Eq Update`

Each includes the note:

- `Add Location ID to Case Statement when a new office is spun up`

The following create-record action maintenance points are also confirmed:

- `Pickup Update`
- `Generate Shop Repair WO`
- `Create EQ Update for swap`
- `Sales Swap Create EQ Update`

The actual maintenance point is the assignment to **Updated Address**, using either:

```text
Case([Market], ...)
```
or pickup logic

```
If([Pickup Type]="Temporary Storage for Customer",[Name of Location],Case([Market], ... ))
```

## 10. Nashville example / market already in place

**Database:**  
**Water Desk / Bottleless Nation database**

**Description:**  
Some locations may already be built into the routing logic before the new dealer setup begins.

**Why this matters:**  
This changes whether the process needs market-routing setup or just dealer and office updates.

**What TeamDesk does:**  
Confirmed live Water Desk routing already includes:

- `TN - Nashville → 20260528-1009`

**Result:**  
Nashville is an example of a market/location that was already configured. It should be treated as an example in this guide, not as a required step for every new dealer setup.

---

## 11. Shared DB / Omniflow Dependency

**Database Area:**  
**Shared downstream setup**

**Description:**  
There is also another TeamDesk integration that pushes certain service work into Omniflow-related records.

**Why this step exists:**  
Even if dealer creation and Opportunity submission are correct, operations can still fail if this downstream step is not aligned.

**What TeamDesk does:**  
Previously confirmed trigger:

- `New Omniflow Service Ticket`

Previously confirmed matching:

- `Service Type = Field Service or Shop Repair`
- `Equipment Model = M6, S1, or S3`

Previously confirmed linked action:

- `Send To Omniflow`

Previously confirmed action settings:

- uses `Admin Auth`
- `POST https://waterdesk.teamdesk.net/secure/api/v2/91748/Service Ticket/upsert.json?match=Service Call Id`

Previously confirmed payload includes:

- Work Order Id → Service Call Id
- Equipment Serial → Serial Number
- Sub Type
- Technician Notes
- Model

**Result:**  
There is a live TeamDesk-to-TeamDesk dependency to database `91748`.

### API Details

| Syntax | Description |
|--------|------------|
| Method | POST |
| Url | `https://waterdesk.teamdesk.net/secure/api/v2/91748/Service Ticket/upsert.json?match=Service Call Id` |
| Headers | `Authorization via Admin Auth` |
| Body | see JSON below |

```json
{
  "Service Call Id": "<%[Work Order Id]%>",
  "Serial Number": "<%[Equipment Serial]%>",
  "Sub Type": "<%[Sub Type]%>",
  "Technician Notes": "<%[Technician Notes]%>",
  "Model": "<%[Model]%>"
}
```

---

## 12. User / Access / Role Visibility

**Database:**  
**Water Desk / Bottleless Nation database** and related areas

**Description:**  
The correct users need to be able to see the correct Opportunities and work the right records.

**Why this step exists:**  
Dealer setup is not complete unless the correct people can access the records they need.

**What TeamDesk does:**  
Confirmed live Water Desk Opportunity visibility depends on role formulas and linked relationship fields such as:

- `Record Owner`
- `Created By`
- `Account Exec Proxy`
- `Company`
- `Office Id`
- `Technician`
- `BDR`
- `Account Executive`

This means Opportunity visibility depends on which users are attached to the record and which related company/office/user fields are filled in.

Earlier confirmed button restriction also shows:

- `Send To PWP CS Portal` is limited to:
  - `Admin`
  - `Admin 2`
  - `Default Role`

**Result:**  
User visibility in Water Desk Opportunities depends on correct user attachment and relationship-linked fields, not just on the record existing.

---

## 13. What actually needs to be manually updated when adding a new Bottleless Nation office

**Description:**  
This section summarizes the parts of the process that may require manual updates when a new Bottleless Nation office is added.

**Confirmed manual-update areas:**

### In the PWP Customer Service Portal
- review the newly created [Dealer record](https://waterdesk.teamdesk.net/secure/db/76449/setup/table.aspx?table=889363)
- if the new dealer is meant to be a branch office:
  - uncheck [Head Office](https://waterdesk.teamdesk.net/secure/db/76449/setup/column.aspx?column=32194939)
  - update [Parent Company](https://waterdesk.teamdesk.net/secure/db/76449/setup/column.aspx?column=28868523) to the correct parent dealer name

### In the Water Desk / Bottleless Nation database
- review the routing triggers:
  - `Service Swap Update`
  - `Sales Swap Update`
  - `Pickup Eq Update`
- review the create-record actions:
  - `Pickup Update`
  - `Generate Shop Repair WO`
  - `Create EQ Update for swap`
  - `Sales Swap Create EQ Update`
- update the **Updated Address** assignment where the market/location logic uses:
  - `Case([Market], ...)`
  - or `If([Pickup Type]="Temporary Storage for Customer",[Name of Location],Case([Market], ... ))`
- add the correct **Location ID** when a new office is being added to the routing logic

### Visibility / record access review
- confirm that the correct user-linked fields are populated on Opportunity records so the intended users can actually see and work those records

**Result:**  
When a new Bottleless Nation office is added, the dealer record itself may need cleanup in PWP, and the routing logic may need updates in Water Desk.

---

# Record Change & Periodic Triggers

The process includes backend trigger dependencies, especially in **Work Orders**.

## Work-order routing triggers

**Purpose:**  
Keep dispatch, swap, pickup, and equipment update routing working correctly.

**When it runs:**  
These routing steps happen through trigger logic and create-record actions in Water Desk.

**Which records it checks:**  
Work Orders and related office/location setup.

**What fires the trigger:**  
Different swap, pickup, and equipment update situations.

**What TeamDesk does:**  

Confirmed Water Desk maintenance points include:

### Trigger maintenance points
- `Service Swap Update`
- `Sales Swap Update`
- `Pickup Eq Update`

### Create-record action maintenance points
- `Pickup Update`
- `Generate Shop Repair WO`
- `Create EQ Update for swap`
- `Sales Swap Create EQ Update`

The maintenance area to update is the **Updated Address** assignment using market/location logic.

**Why this matters:**  
If a new office is added and these are not updated, work may route to the wrong place.

**Result:**  
Water Desk routing maintenance is a confirmed part of the new dealer / new office rollout process.

## New Omniflow Service Ticket

**Purpose:**  
Send qualifying Work Orders into another TeamDesk Service Ticket table.

**When it runs:**  
Record Change

**Schedule/Condition:**  
Confirmed when:

- `Service Type = Field Service or Shop Repair`
- `Equipment Model = M6, S1, or S3`

**Which records it checks:**  
Work Orders

**What fires the trigger:**  
Matching service records for those service types and models

**What TeamDesk does:**  
Runs `Send To Omniflow` and upserts into DB `91748`.

**Why this matters:**  
This is a real downstream dependency that can affect operations after rollout.

**Result:**  
Eligible Work Orders create or update external TeamDesk service tickets.

---
