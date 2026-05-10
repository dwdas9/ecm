## Background

In AFU Phase 1, the document capture pipeline was built and deployed. Scanning, image enhancement, blank page detection, indexing, and Documentum export were all operational across 18 processes.

The indexing stage had a problem that Phase 1 did not fully solve. When an operator keyed an account number — the GCIN — into the `RefNo` field, there was no way to confirm in real time whether that account number was valid. The system accepted whatever the operator typed and the batch moved forward. Errors were caught downstream, after export, when the Documentum routing job or the ALPS processor noticed something was wrong. By then the batch was already closed.

Phase 3 fixed this. A SOAP web service was built and deployed on the Captiva server. The Captiva IndexPlus scripting modules called this service during the `Validate` event of the `RefNo` field, before the document could advance. The service connected to the bank's Oracle backend and returned a real-time answer: valid account, invalid account, new account, or exception. The operator was told immediately, while the document was still on screen.

The same service also handled a second validation requirement: document expiry date checking for NR (Non-Resident) and other departments where passports, visas, and time-limited documents were being processed.

---

## The Service: What It Was and Where It Lived

The service was an **ASP.NET ASMX web service** (`Service1.asmx`) deployed as part of a web application called `IACustomWebServices`.

**Server:** `[SRV-MASKED]`
**Port:** `81`
**Endpoint:** `http://[SRV-MASKED]:81/IACustomWebServices/Service1.asmx`
**WSDL:** `http://[SRV-MASKED]:81/IACustomWebServices/Service1.asmx?WSDL`
**Protocol:** SOAP over HTTP, WS-I Basic Profile 1.1 conformant
**Namespace:** `http://tempuri.org/`

The service exposed **three web methods**:

| Method | Purpose |
|---|---|
| `ValidateSL210PQ` | Validates an account number — pass/fail only, no customer data returned |
| `SL210PQ` | Validates an account number and optionally returns customer name data |
| `ValidateValidTill` | Validates a document expiry date against business rules per document type and business type |

The Oracle connection string was read from `web.config` under the key `CustomerDatabase`. Credentials were not hardcoded in the service code. This was a deliberate change from the earlier `eICV3_0` approach where the Oracle DSN, user ID, and password were passed as part of the `strIn` parameter — visible in the IndexPlus setup screen.

---

## The Input Protocol: strIn and the Universal Delimiter

All three methods received their input as a single string parameter (`strIn`). Fields within the string were separated by the custom delimiter `|%|%` (named `UNIVERSAL_DELIMITER` in the service code).

This delimiter was chosen specifically to avoid collision with banking data. Branch names, product names, and transaction descriptions can contain commas, pipes, and percent signs individually. The three-character sequence `|%|%` was unlikely to appear in any real field value.

---

## Method 1: ValidateSL210PQ

### What It Did

`ValidateSL210PQ` validated an account number against the core banking Oracle stored procedure `CALLSL210PQ`. It returned a pass or fail code — no customer data was returned by this method. It was the lightweight validation path.

### Input Format

```
strIn = accountNo |%|% transactionCode |%|% branchCode |%|% productCode |%|% holderNo
```

After splitting on `|%|%`:

| Index | Field | Example |
|---|---|---|
| `[0]` | Account Number (GCIN) | `XXXXXXXXXX` |
| `[1]` | Transaction Code | `AO` |
| `[2]` | Branch Code | `XXXXXX` |
| `[3]` | Product Code | `XXX` |
| `[4]` | Holder Number | `1` |

### Processing Logic

i) Split `strIn` on `|%|%` to extract the five fields.

ii) Check `C:\CaptivaCustom\DLL\TCodes_DLL.ini`. If the transaction code is listed in this file, it is a new account opening transaction — the account does not yet exist in the core banking system. Return `1000` (PASS_NEWACCOUNT) immediately without calling Oracle.

iii) If not a new account code: open an ADODB connection using the `CustomerDatabase` connection string from `web.config`.

iv) Call Oracle stored procedure `CALLSL210PQ` with four input parameters: `branch_cd`, `product_cd`, `account_no`, `holder_no`.

v) Execute the stored procedure. Set command timeout to 60 seconds.

vi) Close the connection immediately after execution.

vii) Read `err_cd` and `err_txt` from the output parameters.

viii) If `err_cd = 0`: return `1001` (PASS_VALIDACTNUM).

ix) If `err_cd != 0`: return `1002|%|%errTxt` (PASS_INVALIDACTNUM with the Oracle error message).

x) On any exception: return `1003|%|%ex.Message`.

### Response Codes

| Code | Constant | Meaning |
|---|---|---|
| `1000` | PASS_NEWACCOUNT | New account — transaction code in TCodes_DLL.ini, no Oracle call made |
| `1001` | PASS_VALIDACTNUM | Valid account — Oracle returned err_cd = 0 |
| `1002\|%\|%<message>` | PASS_INVALIDACTNUM | Invalid account — Oracle returned err_cd != 0 |
| `1003\|%\|%<message>` | FAIL_EXCEPTION | Exception during processing |
| `1004` | FAIL_EOFUNCREACHED | End of function reached without returning — should never occur |

---

## Method 2: SL210PQ

### What It Did

`SL210PQ` was the full validation method. It did everything `ValidateSL210PQ` did, plus it returned customer name data from the Oracle stored procedure when the account was valid and `flag_populate` was set to `True`. This was the method called during interactive indexing — the one that gave the operator a visual confirmation of the customer name alongside the account number they had just entered.

### Input Format

Same as `ValidateSL210PQ`:

```
strIn = accountNo |%|% transactionCode |%|% branchCode |%|% productCode |%|% holderNo
```

Plus a boolean parameter:

```
flag_populate = True   (return customer data in response)
flag_populate = False  (validate only, don't return customer data)
```

### Processing Logic

The logic was identical to `ValidateSL210PQ` up to the point of reading the Oracle output. The difference was in what happened when `err_cd = 0`:

```
If err_cd = 0 AND flag_populate = True:
    Read output_txt1 from Oracle output parameter
    Read output_txt2 from Oracle output parameter
    Return "1001 |%|% output_txt1 |%|% output_txt2"

If err_cd = 0 AND flag_populate = False:
    Return "1001"

If err_cd != 0:
    Return "1002 |%|% errTxt"
```

`flag_populate` was always set to `True` in the IndexPlus calling code. The `False` path existed for scenarios where the caller only needed a validity check and did not want the overhead of customer data in the response.

### Response Format (when flag_populate = True)

```
1001 |%|% <customer name line 1> |%|% <customer name line 2>
```

The IndexPlus module split this response on `|%|%` and wrote:
- `output_txt1` → `CL0_eICVPrOTxt` (customer name, displayed in the indexing form and later written to Documentum)
- `output_txt2` → `CL0_eICVPrOTxt2`

### Why Two Separate Methods?

`ValidateSL210PQ` and `SL210PQ` served the same backend but exposed different response contracts. `ValidateSL210PQ` was designed for callers that only needed a pass/fail — for example, a pre-validation check or a batch-level validation step that didn't need to display customer names. `SL210PQ` was designed for interactive indexing where the customer name had to be shown to the operator.

In practice, `SL210PQ` with `flag_populate = True` was the method used in all production IndexPlus calls.

---

## Method 3: ValidateValidTill

### What It Did

`ValidateValidTill` validated a document expiry date. This was required for departments — particularly NR (Non-Resident) — where operators were indexing identity documents like passports and visas that carried an expiry date. The business rule was: some document types require a valid expiry date, some do not. For those that do, the date must be in the future. For those that don't, the date field can be left empty.

The service enforced both sides of this rule.

### Input Format

```
strIn = valid_till |%|% document_type |%|% business_type

Example: 31/12/2026 |%|% PASSPORT |%|% NR
Example:             |%|% bank_acc_app_form |%|% BAN
```

After splitting on `|%|%`:

| Index | Field | Example |
|---|---|---|
| `[0]` | Valid Till (date string, may be empty) | `31/12/2026` |
| `[1]` | Document Type | `PASSPORT` |
| `[2]` | Business Type | `NR` |

### The INI File System for Document Type Rules

Whether a date was mandatory or optional depended on the **document type** and the **business type** together. That rule was not hardcoded in the service. It was maintained in a set of INI files — one per business type:

```
C:\CaptivaCustom\P_V\DocTypes_INI_Files\
    BAN_Codes.ini
    HOME_Codes.ini
    CC_Codes.ini
    MORT_Codes.ini
    NR_Codes.ini
```

Each file contained a list of document type codes for which a `Valid Till` date was **mandatory**. If the document type appeared in the file for that business type, the date could not be empty. If it did not appear, the date was optional.

This design meant that when a new document type was added to the system, no code change and no service redeployment was needed. The operations team added the document type code to the relevant INI file and the rule was immediately active.

### Processing Logic

**Case 1: Valid Till is empty**

i) Select the INI file for the given `business_type` — BAN, HOME, CC, MORT, or NR. Anything else returns an exception.

ii) Check if `document_type` is listed in that INI file.

iii) If YES (document type requires a date): return `9999|%|%"You can't enter an empty Date for combination; Document Type: X, and Business Type: Y."`.

iv) If NO (date is optional): return `1000`.

**Case 2: Valid Till is provided**

i) TryParseExact against format `dd/MM/yyyy` using `en-GB` culture.

ii) If parse fails: return `9999|%|%"invalid date format"`.

iii) If date is not greater than today: return `9999|%|%"must be greater than current date"`.

iv) If valid and future: return `1000`.

**Case 3: Malformed input**

If splitting on `|%|%` does not produce exactly three elements: return `9999|%|%<description of malformed input>`.

### Response Codes

| Code | Meaning |
|---|---|
| `1000` | Valid — date acceptable or not required |
| `9999\|%\|%<message>` | Invalid — message shown directly to operator |

---

## The Oracle Backend: CALLSL210PQ

Both `ValidateSL210PQ` and `SL210PQ` called the same Oracle stored procedure:

```
Input parameters:
    branch_cd    VARCHAR2(200)
    product_cd   VARCHAR2(200)
    account_no   VARCHAR2(200)
    holder_no    VARCHAR2(200)

Output parameters:
    output_txt1  VARCHAR2(3000)   -- Customer name line 1
    output_txt2  VARCHAR2(3000)   -- Customer name line 2
    err_cd       INTEGER          -- 0 = success, non-zero = error
    err_txt      VARCHAR2(200)    -- Error text if err_cd != 0
```

Command timeout was set to 60 seconds. Both the connection and the command object were explicitly closed and set to `Nothing` immediately after execution — inside the `Try` block as well as in the `Catch` block — to prevent connection leaks under error conditions.

The connection string came from `web.config`:

```xml
<connectionStrings>
    <add name="CustomerDatabase" connectionString="[MASKED]" />
</connectionStrings>
```

This was the key architectural improvement over the earlier `eICV3_0` COM DLL approach. In `eICV3_0`, the Oracle DSN, user ID, and password were part of the `strIn` string — visible in plain text in the IndexPlus setup screen. In this service, they lived in `web.config` on the server. Not visible to any Captiva administrator or operator.

---

## TCodes_DLL.ini — The New Account Bypass

Both `ValidateSL210PQ` and `SL210PQ` checked the transaction code against `C:\CaptivaCustom\DLL\TCodes_DLL.ini` before making any Oracle call. This file listed the transaction codes for which the account was a new account — one being opened for the first time. For these codes, the GCIN did not yet exist in Oracle. Any call to `CALLSL210PQ` would return an error. The INI check bypassed Oracle entirely and returned `1000` (PASS_NEWACCOUNT).

The file was maintained by the operations team. No service redeployment was needed when new account opening transaction codes were added.

---

## The Proxy: How Captiva Consumed the Service

The Captiva IndexPlus modules (`AO_IndexPlus`, `TP_IndexPlus`) did not call the service via raw SOAP. A strongly-typed VB.NET proxy class was generated from the service WSDL using `wsdl.exe`:

```bat
"C:\Program Files\Microsoft SDKs\Windows\v6.0A\bin"\wsdl /language:vb http://[SRV-MASKED]:81/IACustomWebServices/Service1.asmx?WSDL
```

Successful output:

```
Microsoft (R) Web Services Description Language Utility
[Microsoft (R) .NET Framework, Version 2.0.50727.3038]
Copyright (C) Microsoft Corporation. All rights reserved.
Writing file 'C:\Service1.vb'.
```

The generated `Service1.vb` was added to both Visual Studio 2008 projects. It exposed synchronous, Begin/End async, and event-based async variants for all three methods. Only the synchronous variants were used.

The proxy hardcoded the endpoint in its constructor:

```vb
Public Sub New()
    MyBase.New
    Me.Url = "http://[SRV-MASKED]:81/IACustomWebServices/Service1.asmx"
End Sub
```

Calling the service from the IndexPlus `Validate()` event was then a single line:

```vb
Dim wsProxyObject = New Service1
wsResult_Original = wsProxyObject.SL210PQ(strIn, True)
```

The proxy handled all SOAP envelope construction, HTTP transport, and XML deserialization. The IndexPlus code only saw a string in and a string out.

> **Note:** If the service WSDL changed — new method, endpoint change, parameter change — `Service1.vb` was deleted from the project, regenerated using `wsdl.exe`, re-added, and the project was rebuilt.

---

## The Integration: How SL210PQ Was Called from Captiva IndexPlus

This is the core of the integration. The class `SL210PQAndCheckMandatoryNumeric10To16` implemented `IFieldEvents` from the Captiva .NET IndexPlus scripting API (`Emc.InputAccel.IndexPlus.Scripting`). It was attached to the `RefNo` field in the IndexPlus module setup for each department. The `Validate()` method fired on three events: `LeavingField`, `LeavingNode`, and `AcceptingTask`.

### Step 1 — Local Format Check (No Service Call)

Before any service call, the field value was checked locally:

- Must be 10 to 16 characters.
- Must be numeric.
- Must not be all zeros.

If any of these failed, the operator saw an error immediately with no network call made.

### Step 2 — Cache Check (nodeFlagDictionary)

```vb
If nodeFlagDictionary.ContainsKey(info.CurrentNode.Id) Then
    If nodeFlagDictionary(info.CurrentNode.Id) = True Then
        Return pass immediately
    End If
End If
```

`nodeFlagDictionary` was a `Private Shared Dictionary(Of Integer, Boolean)` maintained per DLL lifetime. Key = node ID. Value = `True` (validated this session) or `False` (not yet validated or changed since last validation).

Without this, every tab out of the `RefNo` field would trigger a SOAP call. With it, once an account number had been validated for a given node, no further calls were made unless the operator changed the value. The `Changed()` event reset the flag to `False` whenever the field value was modified.

### Step 3 — Collect Sibling Fields

```vb
referenceNo     = currentNodeFields.GetFieldById("ReferenceNumber").Value
transactionCode = currentNodeFields.GetFieldById("TransactionType").Value.Split("-")(1).Trim()
branchCode      = currentNodeFields.GetFieldById("CustomerBranch").Value.Split("|")(1).Trim()
productCode     = currentNodeFields.GetFieldById("Product").Value.Split("|")(1).Trim()
holderNo        = currentNodeFields.GetFieldById("Holder").Value
```

The fields `TransactionType`, `CustomerBranch`, and `Product` were stored in IndexPlus as display-value-plus-code pairs:

- `TransactionType`: `ACCOUNT OPENING-AO` → code after `-` → `AO`
- `CustomerBranch`: `Chennai Central|XXXXXX` → code after `|` → `XXXXXX`
- `Product`: `Savings Account|XX` → code after `|` → `XX`

If any mandatory sibling field was empty, a validation failure was returned before the service was called.

### Step 4 — Build strIn

```vb
strIn = referenceNo + UNIV_DELMTR + transactionCode + UNIV_DELMTR + branchCode + UNIV_DELMTR + productCode + UNIV_DELMTR + holderNo
```

Example:
```
XXXXXXXXXX |%|% AO |%|% XXXXXX |%|% XX |%|% 1
```

### Step 5 — Instantiate Proxy and Call the Service

```vb
Dim wsProxyObject = New Service1
wsResult_Original = wsProxyObject.SL210PQ(strIn, True)
```

One line. The proxy handled the SOAP envelope, the HTTP POST, and the XML deserialization. The IndexPlus code saw only the string it sent and the string it got back.

### Step 6 — Parse the Response

```vb
wsResult_Code = wsResult_Original.Split(univDelmtrArr, StringSplitOptions.None)(0)
```

### Step 7 — Act on the Response Code

```
Case "1000" (New account):
    Clear CL0_eICVPrOTxt and CL0_eICVPrOTxt2
    Set nodeFlagDictionary[nodeId] = True
    Return ValidationPassed = True

Case "1001" (Valid account):
    OUTPUT_TXT1 = wsResult_Original.Split(...)(1)
    OUTPUT_TXT2 = wsResult_Original.Split(...)(2)
    Write OUTPUT_TXT1 → CL0_eICVPrOTxt
    Write OUTPUT_TXT2 → CL0_eICVPrOTxt2
    Set nodeFlagDictionary[nodeId] = True
    Return ValidationPassed = True

Case "1002" (Invalid):
    wsResult_Message = wsResult_Original.Split(...)(1)
    Return ValidationPassed = False, Message = wsResult_Message

Case "1003" (Exception):
    wsResult_Message = wsResult_Original.Split(...)(1)
    Return ValidationPassed = False, Message = wsResult_Message
```

The operator saw a green tick or a red error message with the exact text returned from Oracle. The document could not advance until the field passed validation.

`CL0_eICVPrOTxt` and `CL0_eICVPrOTxt2` — the customer name fields populated from the Oracle response — were read by the Documentum Export step and written as the `customer_name` attribute on the `xxxxbank_captiva_doc` object in the repository.

---

## End-to-End Flow

```
1. Operator types GCIN into RefNo field in IndexPlus.

2. Operator tabs out of the field (LeavingField event fires).

3. TP_IndexPlus / AO_IndexPlus Validate() method executes:
   a. Local format check: 10-16 numeric digits, non-zero. Fail immediately if wrong.
   b. Check nodeFlagDictionary — already validated this session? Return pass.
   c. Read sibling fields: TransactionType, CustomerBranch, Product, Holder.
   d. Extract codes from each field value.
   e. Build strIn = accountNo |%|% txCode |%|% branchCode |%|% productCode |%|% holderNo
   f. Dim wsProxyObject = New Service1
   g. wsProxyObject.SL210PQ(strIn, True)

4. SOAP request travels to [SRV-MASKED]:81.

5. Service1.asmx receives request:
   a. Parse strIn on |%|%.
   b. Check TCodes_DLL.ini — new account? Return 1000.
   c. Open ADODB connection using web.config CustomerDatabase.
   d. Call CALLSL210PQ stored procedure on Oracle.
   e. Read err_cd, err_txt, output_txt1, output_txt2.
   f. Return response string.

6. SOAP response returns to IndexPlus module.

7. Module splits response on |%|%:
   Code 1000: New account. Clear output fields. Set flag True. Pass.
   Code 1001: Valid.  Write output_txt1 → CL0_eICVPrOTxt.
                      Write output_txt2 → CL0_eICVPrOTxt2. Set flag True. Pass.
   Code 1002: Invalid. Show error message to operator. Fail.
   Code 1003: Exception. Show error message to operator. Fail.

8. IndexPlus shows green tick (pass) or red error message (fail) to operator.

9. If fail: operator cannot advance. Must correct the GCIN and re-validate.
```

---

## Summary: What Was Achieved

Before this integration, Captiva had no awareness of the bank's core banking system. An operator could type any account number and the system would accept it. Errors were discovered downstream.

After this integration, every account number entered during indexing was validated in real time against Oracle before the document could advance. Every document expiry date was checked against business rules before the batch could be submitted. Invalid data could not enter the pipeline.

The SOAP web service was the bridge that made this possible. It sat between the Captiva client machines and the Oracle backend — centralising the database connection, removing Oracle client dependencies from operator workstations, and exposing the validation logic as a clean, typed, consumable interface that the IndexPlus .NET scripting modules could call with a single line of code.

The result: a Captiva indexing pipeline that validated metadata against a live legacy banking system in real time, with no Oracle client on any operator machine, no credentials visible in any Captiva configuration screen, and no dependency on any manual post-indexing correction step.

---

*Dwaipayan Das, Application Fullfilment Unit - ECM Architct*