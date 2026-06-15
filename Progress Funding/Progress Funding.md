
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
