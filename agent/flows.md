# Flows used as agent actions

Both are autolaunched Flows (no trigger). Agentforce can call a Flow as an action only when its variables are marked Available for input or Available for output, and the agent reads each variable's description to decide what to pass in, so every variable has one.

## Get_Order_Status

Looks up an order on the Retail Order object. It returns details only when the order number **and** the email match, so the agent can't reveal someone else's order.

| Variable | Type | Direction | Description |
|---|---|---|---|
| `orderNumber` | Text | Input | The order number, format MRG-###### |
| `customerEmail` | Text | Input | The email address used on the order |
| `orderFound` | Boolean | Output | True if an order matches both the number and the email |
| `status` | Text | Output | Order status |
| `carrier` | Text | Output | Shipping carrier |
| `trackingNumber` | Text | Output | Tracking number |
| `estimatedDelivery` | Date | Output | Estimated delivery date |
| `statusNote` | Text | Output | Reason for a delay or cancellation |
| `items` | Text | Output | Items in the order |

Steps:

1. **Get Records** on Retail Order where the order number (Name) equals `orderNumber` and `Customer_Email__c` equals `customerEmail`. Store only the first record.
2. **Decision:** was a record found?
3. **If found, Assignment:** `orderFound = true`, and copy status, carrier, tracking number, estimated delivery, status note and items into the outputs.
4. **If not found, Assignment:** `orderFound = false`. Nothing else is returned.

In production the Get Records step would be replaced by a call to the order management system (through MuleSoft or an HTTP callout with a Named Credential). The inputs and outputs stay the same, so the agent doesn't change.

`statusNote` was not marked Available for output in the first version, so the agent couldn't explain delays. That was found in test run 1 and fixed in a new Flow version (CR-003 in the [POC record](../docs/poc-record-example.pdf)).

## Create_Support_Case

Creates a Case for the care team and returns its number.

| Variable | Type | Direction | Description |
|---|---|---|---|
| `customerEmail` | Text | Input | The customer's email address |
| `caseSummary` | Text | Input | A short summary of the issue and what the customer wants |
| `caseReason` | Text | Input | Damaged item, Complaint, Privacy request or Other |
| `caseNumber` | Text | Output | The new case number to share with the customer |

Steps:

1. **Create Records:** a Case with Subject = "Maplecrest Assistant: " + `caseReason` (a formula resource), Description = `caseSummary`, Supplied Email = `customerEmail`, Origin = Web, Priority = Medium. Store the new record ID.
2. **Get Records:** the Case with that ID.
3. **Assignment:** `caseNumber` = the Case Number.
