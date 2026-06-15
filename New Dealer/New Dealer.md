# Full New Dealer Setup – Documentation

## Purpose

The purpose of this document is to define the current TeamDesk process for **new dealer setup** across the relevant databases and integrations.

This process includes:

- dealer creation in the **PWP Customer Service Portal**
- Opportunity submission and update from the **Dealer WaterDesk Opportunity database**
- zip-, market-, location-, and office-based routing dependencies
- work-order routing dependencies
- downstream TeamDesk integration dependencies

This document includes confirmed live setup findings and clearly identifies items that remain pending confirmation before the SOP can be considered fully complete.

---

## Steps

### Full Step-by-Step Walkthrough

#### Primary Flow

1. A user initiates dealer creation in the **PWP Customer Service Portal** using [Create New Dealer](https://waterdesk.teamdesk.net/secure/db/76449/setup/custbtn.aspx?custbtn=1186132).

2. TeamDesk assigns the source Account’s `[Trans #]` into `[Dealer Id]`.

3. TeamDesk runs [Create New Dealer Record](https://waterdesk.teamdesk.net/secure/db/76449/setup/wfaction.aspx?wfaction=2523053) to create the Dealer record.

4. TeamDesk runs [Dealer Record Created](https://waterdesk.teamdesk.net/secure/db/76449/setup/wfaction.aspx?wfaction=2523059) to mark the source Account as already processed.

5. In the **Dealer WaterDesk Opportunity database**, a user submits or updates an Opportunity using one of the PWP submission buttons.

6. TeamDesk executes a **Call URL** action to the PWP Customer Service Portal REST endpoint:

   `POST https://waterdesk.teamdesk.net/secure/api/v2/76449/Credit Application/upsert.json`

7. On create paths, TeamDesk stores returned identifiers back onto the Opportunity:
   - `Aspire CS Portal ID`
   - `Aspire CS Portal Record ID`

8. Operational routing then depends on the confirmed live structure around:
   - Companies
   - Zip Codes
   - Locations
   - Offices
   - Work Orders
   - downstream Omniflow integration

#### Alternate Flow Name

There are three submission variants in the current setup:

- **Standard create path**
- **Midterm Upgrade create path**
- **Update path**

There are also backend continuations involving routing and integrations that depend on:

- zip-driven routing
- office/location routing
- work-order routing
- Omniflow service-ticket creation
- possible downstream Credit Application automations

---

## 1. [Create New Dealer](https://waterdesk.teamdesk.net/secure/db/76449/setup/custbtn.aspx?custbtn=1186132)

**Description:**  
This is the dealer-creation entry point in the **PWP Customer Service Portal**.

**Why this step exists:**  
It starts dealer creation from the source Account.

**What TeamDesk does:**  
This custom button is available when:

- `[Role] is Dealer`
- `[Dealer Record Created] is not checked`

It:

- assigns `[Trans #] → [Dealer Id]`
- runs [Create New Dealer Record](https://waterdesk.teamdesk.net/secure/db/76449/setup/wfaction.aspx?wfaction=2523053)
- runs [Dealer Record Created](https://waterdesk.teamdesk.net/secure/db/76449/setup/wfaction.aspx?wfaction=2523059)

**Result:**  
A Dealer record is created and the source Account is marked as processed.

---

## 2. [Create New Dealer Record](https://waterdesk.teamdesk.net/secure/db/76449/setup/wfaction.aspx?wfaction=2523053)

**Description:**  
This is the **Create Record** workflow action used during dealer creation in the **PWP Customer Service Portal**.

**Why this step exists:**  
It creates the Dealer record from Account data.

**What TeamDesk does:**  
It creates a record in the [Dealer table](https://waterdesk.teamdesk.net/secure/db/76449/setup/table.aspx?table=889363) with confirmed assignments including:

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

**Result:**  
A new Dealer record exists in the PWP Customer Service Portal.

---

## 3. [Dealer Record Created](https://waterdesk.teamdesk.net/secure/db/76449/setup/wfaction.aspx?wfaction=2523059)

**Description:**  
This is the **Update Record** workflow action that completes the immediate dealer-creation sequence.

**Why this step exists:**  
It prevents duplicate dealer creation from the same Account.

**What TeamDesk does:**  
It updates the source Account:

- `Dealer Record Created = true`

**Result:**  
The source Account is marked as already processed.

---

## 4. Company / Zip / Market / Location Routing Structure

**Description:**  
The live routing structure for dealer operations is built around **Companies**, **Zip Codes**, **Locations**, and **Offices**.

**Why this step exists:**  
Dealer rollout depends on routing by market, service market, territory, office, and location.

**What TeamDesk does:**  
Confirmed live setup shows:

### Companies

- `Zip Code` references **Zip Codes**
- `Account Market` is looked up from zip `Market`

### Zip Codes

Contains routing fields including:

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

- `Zip Code` references **Zip Codes**
- market and territory values are looked up from zip
- `Market Form` uses `Market Override` if present, otherwise `Market`

### Offices

- reference **Locations**
- inherit market and location-derived data from the linked location

**Result:**  
Routing is zip- and location-driven.

---

## 5. Dealer Hierarchy / Parent Company / Head Office Model

**Description:**  
The live structure is company-centered and location-centered, but a complete parent/head-office/branch model is not fully established from the inspected setup.

**Why this step exists:**  
A complete new dealer SOP needs to know whether a dealer must be linked to a parent company or branch structure.

**What TeamDesk does:**  
Confirmed setup shows:

- the live structure is centered on **Companies** and **Locations**
- a company has many locations
- `Referral Company` is a self-reference on Companies
- `Billing Location` and `Primary Billing` exist on Locations

**Result:**  
Company-to-location structure is confirmed. A true parent/head-office/branch model remains pending confirmation.

---

## 6. `Send To PWP CS Portal`

**Description:**  
This is the standard Opportunity create-path button in the **Dealer WaterDesk Opportunity database**.

**Why this step exists:**  
It submits a new Credit Application to the PWP Customer Service Portal.

**What TeamDesk does:**  
Confirmed visibility:

- `Admin`
- `Admin 2`
- `Default Role`

Confirmed filter:

- `Aspire CS Portal ID is blank`
- `Term >= 24`

It runs:

- `Send New To Aspire CS Portal Redux`

**Result:**  
A standard create/upsert submission is initiated.

---

## 7. `Send New To Aspire CS Portal Redux`

**Description:**  
This is the standard **Call URL** action for Opportunity submission.

**Why this step exists:**  
It sends Opportunity data into the PWP Customer Service Portal Credit Application endpoint and stores returned identifiers.

**What TeamDesk does:**  
Confirmed setup:

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
A Credit Application is created/upserted in the PWP Customer Service Portal and response IDs are stored on the Opportunity.

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
```

---

## 8. MTU and Update Paths

**Description:**  
The process includes both Midterm Upgrade and update variants in addition to the standard create path.

**Why this step exists:**  
The process supports both MTU submissions and updates to existing Credit Applications.

**What TeamDesk does:**  
Confirmed to exist:

- `Send to PWP CS Portal(MTU)`
- `Send New to Credit App MTU`
- `Send Update to PWP CS Portal`
- `Send Update To CA Redux`

Earlier confirmed inspection established:

- MTU uses a different payload
- update path includes `Id` from `Aspire CS Portal ID`
- both call the same PWP Customer Service Portal endpoint

**Result:**  
MTU and update paths are part of the process. Detailed reconfirmation of their latest button filters, payloads, and visibility remains pending.

---

## 9. Work Order Routing

**Description:**  
Work Order routing is a backend operational dependency for new dealer rollout.

**Why this step exists:**  
Dealer rollout is incomplete if service, install, swap, and pickup work orders do not route correctly.

**What TeamDesk does:**  
Confirmed active routing-related triggers include:

- `Notify Tech`
- `Tech Not Assigned`
- `Service Swap Update`
- `Sales Swap Update`
- `Pickup Eq Update`
- `New Sales Rep Install Request`
- multiple hourly `Tech Grab` triggers

Confirmed note on several triggers:

- when a new office is spun up, a **Location ID must be added to a Case statement**

Confirmed routing structure:

- Locations relate to Work Orders
- Offices relate to Work Orders by market and office relation

**Result:**  
Work-order routing depends on market/office/location structure and contains hardcoded maintenance points that may require updates for a new office/dealer rollout.

---
## 10. Shared DB / Omniflow Dependency

**Description:**  
A downstream TeamDesk-to-TeamDesk integration is active for Omniflow-related service work.

**Why this step exists:**  
Dealer rollout may fail operationally even when local setup is correct if downstream service-ticket integration is not aligned.

**What TeamDesk does:**  
Confirmed trigger:

- `New Omniflow Service Ticket`

Confirmed matching:

- `Service Type = Field Service or Shop Repair`
- `Equipment Model = M6, S1, or S3`

Confirmed linked action:

- `Send To Omniflow`

Confirmed action settings:

- uses `Admin Auth`
- `POST https://waterdesk.teamdesk.net/secure/api/v2/91748/Service Ticket/upsert.json?match=Service Call Id`

Confirmed payload includes:

- Work Order Id → Service Call Id
- Equipment Serial → Serial Number
- Sub Type
- Technician Notes
- Model

**Result:**  
A live dependency to TeamDesk database `91748` is confirmed.

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

## 11. User / Access / Role Visibility

**Description:**  
Role and record-access restrictions are part of successful dealer rollout.

**Why this step exists:**  
Users must be able to see required records and use required buttons for the process to work outside admin testing.

**What TeamDesk does:**  
Confirmed live roles include:

- Admin
- Admin 2
- Owner
- Account Executive
- BDR
- Sales Manager
- Service Manager
- Service Tech
- additional read-only / hybrid / special-purpose roles

Confirmed record access pages were inspected for:

- **Companies**
- **Opportunities**

Confirmed schema-level user associations include:

- company sales/major account rep links
- zip sales rep / primary tech links
- office regional manager links

Confirmed button restriction:

- `Send To PWP CS Portal` is limited to `Admin`, `Admin 2`, and `Default Role`

**Result:**  
Access is a confirmed dependency. Full per-view, per-dashboard, and UI-level visibility remains pending confirmation.

---

# Record Change & Periodic Triggers

The process includes backend trigger dependencies, especially in **Work Orders**.

## Work-order routing triggers

**Purpose:**  
Maintain dispatch, assignment, swap, pickup, and service routing.

**When it runs:**  
Record-change and periodic triggers are both present in the Work Orders area.

**Schedule/Condition:**  
Multiple active hourly/periodic triggers exist, along with several active record-change routing triggers.

**Which records it checks:**  
Work Orders and related office/location/work-order relationships.

**What fires the trigger:**  
Various service/install/swap/pickup conditions. Not every individual routing trigger page was inspected in full detail.

**What TeamDesk does:**  

- routes work-order activity
- performs tech assignment and notification logic
- relies in part on office/location case-statement maintenance

**Why this matters:**  
New dealer rollout is not operationally complete if these routes are not aligned.

**Result:**  
Routing dependencies are confirmed. Full trigger-by-trigger logic remains pending confirmation.

## New Omniflow Service Ticket

**Purpose:**  
Push eligible Work Orders into another TeamDesk Service Ticket table.

**When it runs:**  
Type: Record Change

**Schedule/Condition:**  
Confirmed when:

- `Service Type = Field Service or Shop Repair`
- `Equipment Model = M6, S1, or S3`

**Which records it checks:**  
Work Orders

**What fires the trigger:**  
Matching Work Orders for eligible service types and models

**What TeamDesk does:**  
Runs `Send To Omniflow` and upserts into DB `91748`.

**Why this matters:**  
This is a live downstream operational dependency.

**Result:**  
Eligible Work Orders generate or update external TeamDesk service tickets.

---
