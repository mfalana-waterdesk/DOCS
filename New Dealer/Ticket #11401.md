Subject: Re: WD New Dealer Onboarding – Summit H2O LLC (PWP7.144.001)

The expected setup flow for **Summit H2O LLC (PWP7.144.001)** is as follows:

1. **Dealer creation in the PWP Customer Service Portal**  
   Yes — the dealer should be created in the **PWP Customer Service Portal** from the source Account using [Create New Dealer](https://waterdesk.teamdesk.net/secure/db/76449/setup/custbtn.aspx?custbtn=1186132).  
   That button is designed to run linked actions that create the Dealer record and mark the source Account as already processed, which matches how TeamDesk custom buttons and workflow actions are intended to work. [^1] [^2]

2. **Head Office vs. branch setup**  
   Based on the currently confirmed PWP setup, a new dealer created through [Create New Dealer Record](https://waterdesk.teamdesk.net/secure/db/76449/setup/wfaction.aspx?wfaction=2523053) is created with:
   - `Head Office = true`
   - `Parent Company = [Name]`

   That means the dealer will initially be created as a **Head Office**.  
   No Bottleless Nation-specific automation has been confirmed that would automatically convert it into a branch after creation.

   Unless there is a separate business decision that Summit H2O LLC should be a branch under another parent dealer, the safest interpretation of the current setup is:

   - create the dealer first
   - leave it as **Head Office** unless there is a confirmed instruction to convert it into a branch

   If Summit H2O LLC is intended to be a branch, the manual follow-up would be:
   - uncheck [Head Office](https://waterdesk.teamdesk.net/secure/db/76449/setup/column.aspx?column=32194939)
   - update [Parent Company](https://waterdesk.teamdesk.net/secure/db/76449/setup/column.aspx?column=28868523) to the correct parent dealer name

3. **User setup**
   The current confirmed findings do support that user setup is part of the process, and the PWP Dealer structure does connect to user-related setup through:
   - [Dealer User Properties](https://waterdesk.teamdesk.net/secure/db/76449/setup/table.aspx?table=889365)
   - [User Properties](https://waterdesk.teamdesk.net/secure/db/76449/setup/table.aspx?table=826175)

   However, the exact live role-mapping steps for:
   - **Don Tucciarone → Owner**
   - **Dilarya Tucciarone → Dealer Admin**

   were not re-opened and confirmed on the actual user setup pages in the current inspection.

   So the best current answer is:

   - those users should be added in the portal area tied to dealer/user setup
   - the final role assignment should follow the requested access form and the existing dealer/user property structure
   - the exact page-level role mapping remains **pending direct confirmation** from the user setup pages

4. **Opportunity submission**
   The Water Desk Opportunity process is already confirmed to use these buttons:
   - `Send To PWP CS Portal`
   - `Send to PWP CS Portal(MTU)`
   - `Send Update to PWP CS Portal`

   So if there is already an Opportunity for Summit H2O LLC, it should be sent using the appropriate button based on whether it is:
   - a new create
   - an MTU
   - an update

   If no Opportunity exists yet, then nothing should be sent to the PWP CS Portal until one is created.

   In short:

   - **if an Opportunity already exists, send it using the correct button**
   - **if no Opportunity exists yet, wait until one is created**

   TeamDesk documentation supports this pattern, where a custom button can trigger a Call URL action that sends data to another endpoint and stores returned values back on the record. [^2] [^3]

5. **Routing updates**
   No automatic assumption should be made that routing changes are required for every dealer.

   The current confirmed Water Desk findings show that routing updates are needed **when a new office is being added** and the new office/location is not already built into the routing logic.

   Confirmed Water Desk routing maintenance points are:
   - `Service Swap Update`
   - `Sales Swap Update`
   - `Pickup Eq Update`

   Confirmed related create-record actions are:
   - `Pickup Update`
   - `Generate Shop Repair WO`
   - `Create EQ Update for swap`
   - `Sales Swap Create EQ Update`

   These use location/market logic in the **Updated Address** assignment.

   So the practical answer is:

   - if Summit H2O LLC is using an already-configured office/location structure, no routing update may be needed
   - if Summit H2O LLC involves a brand-new office/location, routing logic should be reviewed and updated in Water Desk

### Recommended workflow for Summit H2O LLC

Based on the currently confirmed setup, the expected order is:

1. create the dealer in the **PWP Customer Service Portal** using [Create New Dealer](https://waterdesk.teamdesk.net/secure/db/76449/setup/custbtn.aspx?custbtn=1186132)
2. review the newly created Dealer record to confirm whether it should remain a **Head Office** or be manually changed into a branch
3. add the requested users into the appropriate portal/user setup area and assign roles according to the approved access request
4. send the Opportunity only if one already exists
5. review Water Desk routing only if Summit H2O LLC requires a new office/location that is not already configured

### Short answer version

- **Yes**, the dealer should be created from the Account using **Create New Dealer**
- **Head Office should remain checked by default** unless there is a confirmed business reason to set Summit up as a branch
- **User setup should be added in the portal/user setup area**, but the exact role-mapping page still needs direct confirmation if a page-by-page instruction is needed
- **Opportunity submission should wait unless an Opportunity already exists**
- **Routing updates are only needed if this onboarding introduces a new office/location that is not already built into Water Desk**

If needed, this can be converted into a cleaner send-ready email reply next.
