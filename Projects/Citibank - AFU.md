# AFU Data Capture and Enterprise Content Management Platform – Architecture and Implementation

**Project Name:** AFU (Application Fulfilment Unit) – Citibank
**Stream:** Loan Processing / Data Capture Solution
**Platform:** EMC Captiva InputAccel + EMC Documentum + ALPS
**My Role:** Principal Architect — solution design, module configuration, custom Java development, and system integration

---

## 1. Executive Summary

This is a write-up of one of the more complex projects I've worked on - the Application Fulfilment Unit, or AFU. The goal was straightforward to state but hard to execute: take a bank's entirely paper-driven document lifecycle and automate it from end to end. That meant everything from the moment a customer's application form hit the branch counter, all the way through scanning, classification, indexing, repository storage, and on-demand retrieval by a downstream loan processing system.

The stack we built on was EMC Captiva InputAccel for the capture side and EMC Documentum as the content repository and BPM engine. Tying it all together was a custom Java job I wrote, and a SOAP web service integration with the bank's internal loan processing platform, ALPS. I owned the full solution — requirements through deployment.

The business case was clear. Reduce cost, improve processing speed, eliminate manual retrieval bottlenecks, and give the processing team real-time access to every document without ever leaving their primary system. We achieved all of it.

---

## 2. Business Overview

The AFU — Application Fulfilment Unit — is the operational heart of the bank's retail lending back office. Every credit card application and mortgage loan application that comes in has to be received, classified, scanned, stored, and made available to the processing team. At the volumes this bank was handling, that's a lot of paper moving through a lot of hands.

Documents aren't just attachments here. They drive decisions. A credit card application doesn't move forward without the supporting documents being verified. A mortgage doesn't get sanctioned without the income proof, identity proof, and property documents all being in order. Every processing stage — credit decisioning, verification, sanctioning, disbursement — depends on someone being able to see the right document at the right moment.

When I came onto this project, the bank was handling all of that manually. The scale of the problem, and the operational cost it was generating, is what made the AFU project worth building properly.

---

## 3. Business Document Model

### 3.1 Application Forms

Two product lines drive the AFU's document pipeline. Credit card applications and mortgage (home loan) applications. Both are multi-page documents — the application form itself, sometimes spanning four or more pages, treated as a single logical unit in the system. Many of these forms carry a printed barcode, which I used as the primary document separator mechanism. More on that later.

### 3.2 Supporting Documents

Beyond the application form, customers must submit a range of supporting documents. These vary by product and customer profile, but the common ones include PAN Card, Form 16, Driving Licence, salary slips, bank statements, affidavits, birth certificates, audit rolls, and account opening forms. Each one is distinct. Each one belongs to a parent application. And the link between them — the thread that holds the whole thing together — is the Application Number.

### 3.3 Unclassified Documents

Not every document that arrives at the branch can be immediately identified. Sometimes it's a handwritten instruction, sometimes it's a form type the branch staff haven't seen before. These go into an unclassified queue and get classified by an operator during the indexing stage. It's a safety valve for everything the automated classification can't handle.

### 3.4 The Parent-Child Relationship and Why the Application Number Matters So Much

The document model is hierarchical. Application form is the parent. Supporting documents are children. The Application Number is what makes the link. This sounds obvious, but it has a real operational implication: supporting documents don't always arrive with the application. A customer might send the application one day and the Form 16 three days later. When that Form 16 arrives, the bank needs to attach it to the right pending application. The way we handled that was simple — the bank tells the customer to write the Application Number on any supporting documents they send separately. When that document is scanned, the operator enters that number in the `RefNo` field during indexing, and the system links it back. No ambiguity, no manual matching.

---

## 4. Real-World Business Scenarios

Before I get into how we built this technically, it's worth grounding the design in the actual situations the system had to handle. These weren't hypothetical edge cases — they were daily realities at the branch.

**Scenario 1 — Everything arrives together.** Customer submits the application form and all supporting documents in one visit. We scan the full set in one batch, blank-page-separated, and everything gets linked from day one. Clean and simple.

**Scenario 2 — Supporting documents come later.** Customer submits the application, processing starts, but it gets parked because the Form 16 is missing. Bank follows up, customer sends the Form 16 later with the Application Number written on it. We scan it in a fresh batch. The operator keys in the Application Number at index time. Done — the supporting document joins its parent application.

**Scenario 3 — No barcode on the form.** The operator knows exactly what type of application it is, but the physical form doesn't have a barcode. This happens more than you'd think. The solution was to give operators separate Captiva processes to manually assign — one for Credit Card, one for Mortgage. The process assignment itself carries the product type into the system.

**Scenario 4 — Nobody knows what this document is.** Branch receives something that can't be classified on sight. It gets scanned through the Unclassified process, lands in a manual indexing queue, and an operator classifies it after reviewing the image. It's a slower path but an important one.

---

## 5. What Was Broken Before We Built This

### 5.1 ORBIFLOW Was an Island

The bank already had a scanning setup before AFU. Scanned images went into a system called ORBIFLOW. The problem was ORBIFLOW and ALPS — the loan processing platform — had absolutely no integration. They didn't talk to each other. A processor working inside ALPS who needed to view a scanned document had to stop what they were doing, open ORBIFLOW in a separate session, search for the document manually, view it there, and then come back to ALPS. Every. Single. Time.

### 5.2 Everything Was Manual

Grouping documents, indexing them, managing the supporting document types — all of it was human-driven and error-prone. There was no structured metadata, no validated classification, and no reliable way to know whether everything belonging to an application had actually been captured and stored.

### 5.3 SLAs Were Getting Hit

The bank had outsourced vendors accessing ALPS over dedicated leased lines as part of the processing chain. When one stage got delayed — because someone couldn't find a document, or because a classification was wrong — it cascaded. Downstream stages waited. SLAs broke. It was a systemic problem, not a people problem, and it needed a systemic fix.

---

## 6. What We Set Out to Build

Three things. First, a controlled, automated document capture pipeline that replaced paper handling entirely. Second, an intelligent indexing and storage layer that structured every document in a repository with validated metadata. Third, a live integration with ALPS so that any processor, at any stage, could pull the document they needed without leaving the system they were working in.

---

## 7. The Business Flow in Plain Terms

I'll keep the jargon out of this section. The technical detail comes later.

A customer walks into a branch and submits their application and supporting documents. The branch operator looks at what's in front of them — what product, is there a barcode, are the supporting docs complete or incomplete — and makes an initial classification judgment. Then the documents are arranged into a scan batch, with blank pages slipped between each one to tell the scanner where one document ends and the next begins.

From there, the operator scans everything — either using a scanner plugged into the bank's intranet, or through a web browser using the eInput thin client (which meant scanning could happen from anywhere with an internet connection, not just branch desktops).

The scanned images pass through automatic cleanup — crooked pages get straightened, scanner border artefacts get removed, noise disappears, and any page that's fed in upside-down gets corrected automatically. Then documents that can be automatically classified (via barcode) move straight through. Documents that need a human touch — no barcode, supporting docs, anything unclassified — go into an operator queue where someone reviews the image and enters the metadata.

Once indexed, every page of a document gets assembled into a single TIFF file and pushed into the Documentum repository. A background job I wrote sorts each document into the correct folder and kicks off the workflow. From that moment, any ALPS processor who needs that document can retrieve it instantly, inside ALPS, with two parameters: the Application Number and the Document Type. No ORBIFLOW. No manual hunting. Just the document.

At 2 AM every morning, the cleanup runs. Completed batches are deleted from the Captiva server automatically.

---

## 8. System Overview

### 8.1 The Three Platforms and What Each One Does

The solution runs on three platforms that each own a distinct responsibility.

**EMC Captiva InputAccel** is where the capture happens. Think of it as an intelligent document processing pipeline. You configure a series of modules — Scan, Image Enhancement, Quality Check, Index, Export — and documents flow through them sequentially. Captiva runs as a server process, manages the batch queue, handles the operator workstation experience, and connects directly to Documentum for export. It's the engine room.

**EMC Documentum** is the permanent home for everything that gets captured. It's not just a file store — it's a typed, hierarchical content repository with built-in workflow (BPM) capabilities. Documents in Documentum are objects with defined types, metadata attributes, and folder locations. Once a document lands in Documentum, the BPM engine picks it up and routes it through the verification, approval, and processing workflow. Documentum also exposes the web service that ALPS calls to retrieve documents.

**ALPS** is the bank's internal loan processing system. It's where credit officers, processors, and vendor teams spend their working day. Before AFU, it had no visibility into the scanned document world. After AFU, it could retrieve any document from Documentum on demand, mid-workflow, by calling a web service.

### 8.2 Infrastructure

| Role | OS | Key Software |
|---|---|---|
| **Content Server** | AIX / Sun Solaris | Documentum Content Server 5.3 SP4, DFC, Connection Broker |
| **Application Server** | AIX / Sun Solaris | WebSphere 6.x / WebLogic 9.2, DFC 5.3 SP4, Documentum Web Services |
| **Oracle Repository** | AIX / Sun Solaris | Oracle 10g (Documentum's backend database) |
| **Captiva Server** | Windows 2003 Server | Captiva InputAccel Server 5.3, InputAccel Client 5.3 |

---

## 9. How Business Documents Map to the System

### 9.1 The Documentum Object Model

I created a custom Documentum object type for this project: `citibank_captiva_doc`. Every scanned document — regardless of whether it's a credit card application form, a mortgage application, or a PAN Card scan — lands in Documentum as an instance of this type. The type lives in a `dm_cabinet` → `dm_folder` hierarchy within a Docbase called `citi_pre_prod`.

Three object sub-types capture the three document categories:

```
credit_card_app_form    — Credit Card application forms
mortgage_app_form       — Mortgage/Loan application forms
supporting_doc          — All supporting documents
```

The routing logic that decides which sub-type and which folder a document lands in is handled by the CaptivaJob — described in detail in Section 15.

### 9.2 The Intermediate Folder Pattern

Captiva exports all documents to a single shared intermediate folder in Documentum, regardless of document type. This is by design. The Captiva export module doesn't need to know about business routing — that's the CaptivaJob's job. The CaptivaJob reads the intermediate folder, inspects each document, determines its correct final location, moves it, and then kicks off the workflow. Clean separation of concerns.

### 9.3 File Naming Convention

Naming consistency was non-negotiable. The bank already had a naming standard from AFU Phase I, and the new system had to follow it exactly — backward compatibility with the existing repository depended on it. The format looks like this:

```
B_S_BAN-TR-CC_App_03-02-01.tiff        (Credit Card Application Form)
B_S_BAN-TR-CC_F16_P_01.tiff            (Form 16 Supporting Document)
B_S_BAN-TR-ML_App_03-02-01.tiff        (Mortgage Application Form)
```

The `getNewObjectName()` method in the CaptivaJob derived the correct filename from the document's metadata, ensuring nothing ever landed in the repository with an ambiguous or non-compliant name.

### 9.4 Physical Scanning Standards

A few conventions had to be enforced operationally for the system to work reliably:

- Every document (all its pages) is scanned as a single file, named after the Application Number.
- All pages must be the same physical size — mixed-size pages break the image assembly.
- The first page of each logical document group must carry a type sticker at a fixed, predefined coordinate. Subsequent pages carry no sticker.
- That sticker coordinate must be consistent across every first page, because the OCR zone extraction is position-dependent. If the sticker drifts, the classification fails.

---

## 10. The Capture Pipeline — How It Works at a High Level

Before going into the module-by-module detail, here's the conceptual picture.

A batch comes in from the scanner. Blank pages mark the document boundaries. The image enhancement filters clean up each page. An automated quality check catches any mis-oriented pages and corrects them before anyone sees them. Documents that need human classification land in an operator queue; everything else flows straight through. An operator at the index station sees the image, enters or confirms the metadata fields, and the document is cleared for export.

Export assembles all pages of a document into a single multi-page TIFF, pushes it to Documentum, and holds the batch until every single document in it is confirmed as successfully exported. Then the CaptivaJob takes over, moves everything to its final location, starts the workflow, and cleans up. Repeat at 2 AM for deletions.

That's the whole thing at 10,000 feet. The next sections go into exactly how each piece was configured.

---

## 11. Detailed Captiva Process Design

### 11.1 What Was Running on the Server

When you opened the InputAccel Administrator and looked at the process list on server, this is what you saw — 18 processes in total, each representing a distinct combination of product type, document category, and capture mode:

![](images/20260505000050.png)

| Process Name | Purpose |
|---|---|
| `BANKING` | The primary Banking process (main subject of this write-up) |
| `DEV_BANKING` | Development/sandbox version of the Banking process |
| `eBANKING` | eInput-based version of the Banking process |
| `CREDITCARD` | Credit card application processing |
| `Einput - CC` | eInput variant for Credit Card |
| `CITIBANK_CLASSIFIED_WITHOUTBARCODE_CC` | Classified CC docs without barcode |
| `CITIBANK_SUPPORTING_CC` | Supporting docs for CC |
| `CITIBANK_UNCLASSIFIED` | Unclassified documents |
| `MORTGAGE` | Mortgage/loan application processing |
| `eMORTGAGE` / `eMORTGAGE_ADV_IDX` / `eMortgage2` / `eMORTGAGE3` | Various eInput mortgage process variants |
| `_eInput1 - eScan + eIndex + Values to XML + eStatus` | eInput pilot/test process |
| `Errorlogs` | Dedicated error logging process |
| `ExportCheck` | Export verification process |
| `log_banking` | Banking process logging |
| `NR` | Non-Resident banking process |

That's the full picture of what production looked like — every combination of product type, document type, and capture mode accounted for.

### 11.2 The BANKING Process Module Chain

The BANKING process is the core thick-client flow. Here's the module sequence:

![](images/20260505004117.png)

```
eScan / InputAccel Scan
        ↓
IE (Image Enhancement) — with Blank Page Detection
        ↓
Multi (Document Level Creation + Blank Page Deletion)
        ↓
AQA (Automatic Quality Assurance) — Orientation check
        ↓
Rotate (Auto-rotate based on AQA result)
        ↓
Hold (Queue buffer before Index)
        ↓
eIndex / Index (Validation-based Indexing)
        ↓
DctmExp / Export (EMC Documentum Server Export)
        ↓
Timer (Scheduled Batch Deletion at 02:00)
```

Each module runs as a separate task on the InputAccel server. Let me walk through each one.

### 11.3 Scan Module Configuration

The batch tree hierarchy I configured in the Scan Setup (Levels tab):


The most important setting here is in the Event Actions tab. I configured **Blank Page** events to trigger a **New Document** action. This is how the physical blank-page separator becomes a logical document boundary in the batch tree. New Stack also triggers New Document. Everything else flows from this.

Other settings I locked in:

- Binary colour format, **CCITT Group 4** compression. Standard for B&W document imaging — excellent compression ratio, lossless.
- Thumbnail size: Standard.
- Page side rotation: 0 degrees front and back.
- Primary and Secondary Process schema: both `BANKING`.
- Automatically delete empty batches: enabled.
- Batch page limit: 8000 pages for 16-bit modules.

![](images/20260505002143.png)

![](images/20260505002150.png)

![](images/20260505002158.png)



### 11.4 Image Enhancement (IE) Module

Six filters, applied in sequence to every page:

| Filter # | Name | Purpose |
|---|---|---|
| 1 | Deskew | Straightens pages scanned at a slight angle |
| 2 | Border Removal | Removes scanner bed artefacts at page edges |
| 3 | Smooth | Smooths jagged character edges |
| 4 | Hole Removal | Removes punch-hole artefacts |
| 5 | Noise Removal | Eliminates random pixel noise |
| 6 | Blank Page Detection | Identifies blank separator pages for downstream deletion |

The IE module has a real-time preview in the setup UI — you can load a sample document image and see exactly what each filter does before/after. I used this extensively during configuration to tune the filter chain against the actual document stock the bank was scanning.

![](images/20260505002209.png)



### 11.5 Multi Module — Document Levelling and Blank Page Deletion

The Multi module does two things in this flow. First, it physically removes the blank separator pages from the batch — those pages have done their job (triggering the New Document event) and don't need to go any further. Second, it ensures the correct document level hierarchy is maintained in the batch tree as pages flow downstream.

The sequence is worth understanding: IE detects the blank page → Scan's event action creates the document boundary node → Multi deletes the physical blank page. Three modules, one clean result. I specifically chose this three-stage approach because it keeps each module's responsibility clear and avoids any timing ambiguity in the pipeline.

### 11.6 AQA Module — Automatic Quality Assurance

I enabled only the Orientation check here. Noise and Skew checks were deliberately left off.

The reason is practical. The main quality risk at this bank's scanning stations wasn't skew — operators were reasonably careful about feeding documents straight. The real risk was documents being placed in the scanner tray upside-down or at 90 degrees. The Orientation test catches exactly that: pages that are upside-down or sideways get flagged before they ever reach the indexing queue.

### 11.7 Rotate Module

A second AQA-type module, configured specifically for auto-rotation correction. Whatever the Orientation check in AQA flagged, this module physically corrected. By the time a page exits the Rotate module, it's guaranteed to be upright. The operator sees a clean image.

### 11.8 Hold Module

Nothing fancy — it's a queue buffer between the automated processing side and the operator-driven indexing side. It holds documents until an index operator picks them up. The controlled handoff between automated and manual phases needed an explicit staging point, and Hold is it.

### 11.9 Export Module — EMC Documentum Server Export

This is the module that deposits documents into Documentum. Here's the exact configuration from the live setup:

**Docbase tab:**
- Export mode: **Export to this Docbase** (pinned, not first logged-in)
- Target Docbase: **`citi_pre_prod`**

**Objects tab:**
- Create/Search by object name
- Object hierarchy: `dm_cabinet` → `dm_folder` → `citibank_captiva_doc`

**Content tab:**
- Export mode: **Export image files**
- File Type: **TIFF (*.TIF)**
- Colour Format: **Binary**
- Compression: **CCITT Group 4**
- Multi-Page: **Enabled** — all pages assembled into one TIFF per document
- Merge Annotations: Disabled
- Content Type: `tiff`

**Errors tab:**
- If document already exported: **Replace — export document again**
- On error: **Automatically retry 1 time**
- If retry fails: **Prompt user** (Abort / Continue / Stop)

The "Replace on re-export" setting was a deliberate idempotency decision. If a batch fails mid-export and gets reprocessed, documents that already made it through are cleanly overwritten rather than duplicated. No orphaned objects in the repository.

The rule I enforced absolutely: **a batch is never, under any circumstances, marked ready for deletion until every single document in it has confirmed a successful export.** One failed image means the entire batch stays put.

![](images/20260505002353.png)

![](images/20260505002359.png)
### 11.10 Timer Module — Scheduled Batch Deletion

Three rules, running from 02:00 each morning:

| Rule ID | Time | Condition | Operation |
|---|---|---|---|
| 0200 | 02:00 | `$batch/day=0` | `$batch/day=0` |
| 0201 | 02:01 | `$batch/day=0` | `$batch/day=0` |
| 0202 | 02:02 | `$batch/day=0` | `$batch/priority=50` |

`$batch/day=0` matches same-day batches that are fully processed and ready for cleanup. Rule 0202 sets priority to 50 on deletion tasks — a standard trick to make sure deletion runs don't compete with live scanning and export that might still be in progress during that early-morning window.

Before I put this in place, batch cleanup was manual. That's just a disaster waiting to happen — the server fills up, processing slows, someone eventually notices. The timer eliminated that dependency entirely.
![](images/20260505002408.png)

---

## 12. Indexing Design

### 12.1 The Six Fields

I configured exactly six indexing fields for the BANKING process. These came directly from discussions with the business team about what metadata they actually needed to retrieve and route documents. Here's the live Index Setup configuration:

| Field # | Name | Type | Level | Validation / Picklist |
|---|---|---|---|---|
| 01 | **RefNo** | Edit (free text) | 1 – Document | Length: 10–16 characters; Auto-Validate |
| 02 | **BusinessType** | Picklist | 1 – Document | Default: `BAN`; List: `BAN`; Pre-validates on field load; Only list selections allowed |
| 03 | **DocumentType** | Picklist | 0 – Page | Extensive list; Pre-validates on field load; Only list selections allowed |
| 04 | **TransactionType** | Picklist | 0 – Page | Extensive list; Pre-validates on field load; Only list selections allowed |
| 05 | **Priority** | Picklist | 0 – Page | Picklist of priority levels |
| 06 | **Holder** | Picklist | 0 – Page | Default: `1`; List: 1, 2, 3, 4, 5, 6, 7, 8, 9, 99; Pre-validates on field load |

![](images/20260505002319.png)

### 12.2 Picklist Values

**DocumentType picklist** (partial, from live system):
```
bank_acc_app_form, atm_charge_slip, affidavit, late_st_annual_return,
aof, approvals, audit_rolls, bankers_attestn, birth_certificate,
business_continuity_doc, ...
```

**TransactionType picklist** (complete, from live system):
```
AO, LIAPP, SRK, CSP, PAP, PLI, MLI, HPP, HLTF, PA, HC, TR, SC, IOC,
HS, SWP, DMACCLS, OP, STP, SUVPR, RCSF, RCMIFT, ALOP, BLOP, CI, CCL
```

### 12.3 Three Levels of Metadata

The index schema captures metadata at three distinct granularities, each appropriate to a different scope:

**Batch level** — Branch information, captured as a combo box entry. One value per batch, shared across every document in it.

**Document level (Level 1)** — `RefNo` and `BusinessType`. One application number, one business type, per logical document. This is critical: putting `RefNo` at the document level is what enables the system to link supporting documents to parent applications. If it were at the page level, that link would be harder to enforce reliably.

**Page level (Level 0)** — `DocumentType`, `TransactionType`, `Priority`, and `Holder`. These can vary per page when needed, giving fine-grained control over multi-document-type submissions.

### 12.4 Validation — Why I Was Strict About It

`Pre-Validate when fields are loaded` on all picklist fields means the system checks against the valid list immediately on opening the index screen, not when the operator tries to submit. Stale or invalid selections get caught instantly, before the operator spends time on the rest of the fields.

`Only allow selections from list` is a hard constraint. No freeform text on picklist fields. This was important for a reason that isn't immediately obvious: the TransactionType and DocumentType values feed directly into the routing logic in the CaptivaJob and into the Documentum folder structure. A freeform entry that doesn't match a known value means the document can't be routed. Better to catch it here than discover it later.

And the Index module is the only module in the entire pipeline that **holds** a document on error rather than passing it forward. Every other module logs and moves on. Index doesn't, because an indexing error means incomplete or wrong metadata — and pushing a document to Documentum with wrong metadata corrupts the repository's data integrity. There's no recovery from that downstream; it has to be fixed here.

---

## 13. The eInput Flow

### 13.1 Why We Needed a Separate Path

The thick-client InputAccel Scan module requires the client software to be installed on the scanning machine and a direct intranet connection to the Captiva server. That works fine for branches with dedicated scanning workstations. But the bank also needed scanning capability from locations that didn't have that infrastructure — remote offices, field teams, satellite branches. That's where eInput came in.

eInput is Captiva's web-based thin client. eScan runs in Internet Explorer, connects to the eInput web server over HTTPS, and forwards batches to the InputAccel server. No client installation. No VPN requirement. Just a browser.

The eInput module chain I designed looks like this:

```
eScan (with Quality Check, blank page detection, and image processing)
        ↓
IE (Image Enhancement — with blank page detection)
        ↓
Multi (Blank page deletion + Document level creation + Batch deletion)
        ↓
eIndex (Validation-based indexing)
        ↓
DctmExp (Documentum Export — dedicated separate instance)
        ↓
Timer (Timer-based batch deletion)
```

Other capabilities the eInput approach gave us: offline scanning and indexing (batches sync when connectivity returns — essential for field scanning), private batch mode (the operator who scans is the same one who indexes — enforced by the system, not just policy), image streaming for large file preview, and annotations.

### 13.2 Two Key Architectural Calls

**Dedicated export instance.** The eInput flow uses a completely separate export module instance from the thick-client BANKING process. I made that call deliberately. The two capture paths have different folder routing logic, different metadata attributes, and different timing characteristics. Sharing one export instance would have caused configuration conflicts and, worse, potential misrouting of documents. When something like that goes wrong in production, it's painful to diagnose. I'd rather have the clear separation from the start.

**No AQA in the eInput chain.** In the thick-client flow, AQA sits downstream of IE and handles orientation detection and auto-rotation. In the eInput flow, I moved that quality checking into eScan itself. The reason is a known compatibility issue: AQA was designed to work with the traditional InputAccel Scan module. When you put it downstream of eScan, you run into execution problems. Rather than work around that mismatch, I just did the quality checking where it belongs in the eInput context — inside eScan.

---

## 14. The Documentum Integration — How ALPS Gets Its Documents

This is the piece that delivers the most visible business value, and it's worth explaining carefully.

Before AFU, an ALPS processor who needed a document had to stop, exit ALPS, log into ORBIFLOW, search manually, view the document there, and come back. Every stage of the loan processing workflow where a document was needed had this friction baked in. Multiply that by every processor, every stage, every application processed every day, and the productivity cost is significant.

The AFU integration eliminates it. Documents are available inside ALPS, on demand, at any stage, via a web service.

### 14.A The Web Service Design

Documentum exposes two service operations via SOAP/XML. SOAP is an XML-based messaging protocol — well-suited to this kind of enterprise banking integration because the service contract is strongly typed, auditable, and platform-independent. The ALPS team could consume it regardless of their underlying stack.

The service contract defines:
- **Service endpoint:** A URL hosted on the Application Server (WebSphere/WebLogic) where ALPS posts its requests.
- **Request format:** An XML-structured SOAP message with two mandatory parameters — the Application Number and the Document Type.
- **Response format:** A SOAP response carrying the document as a byte stream, or a status acknowledgment for attribute updates.
- **Message transport:** XML over HTTPS.

The contract is WSDL-style — a machine-readable service definition that the ALPS development team used to generate client stubs and bind to the service programmatically.

### 14.B The Request-Response Flow

Here's what happens when an ALPS operator requests a document:

```
Step 1: ALPS identifies that a document is required at a processing stage.
        It prepares a SOAP request containing:
            - Application Number (the unique reference linking the document)
            - Document Type (specifies which document to retrieve)

Step 2: ALPS dispatches the SOAP/XML request over HTTPS to the
        Documentum web service endpoint on the Application Server.

Step 3: The Application Server receives the request. The Documentum
        Foundation Classes (DFC) layer authenticates using
        server-to-server credentials and opens a session with
        the Documentum Content Server.

Step 4: The Content Server searches the Documentum repository
        (Docbase: citi_pre_prod) for the document matching the
        Application Number and Document Type parameters.

Step 5: The matching TIFF document object is located in the
        dm_cabinet/dm_folder/citibank_captiva_doc hierarchy.

Step 6: The document content (the TIFF file) is retrieved and
        returned to the Application Server.

Step 7: The Application Server encodes the document as a byte
        stream within the SOAP response envelope and returns it
        to the calling ALPS system.

Step 8: ALPS receives the byte stream and renders the document
        image for the operator within the ALPS interface.
```

### 14.C Handling Large Files

Banking documents aren't small. A four-page mortgage application with six supporting documents can produce a substantial TIFF. Pushing that through a SOAP envelope naively causes transport-layer size violations and timeout issues. To handle this, the document content is split into chunks at the Documentum end using standard splitting APIs before being included in the response. The chunks are transmitted sequentially. At the ALPS end, a corresponding reassembly mechanism reconstructs the complete TIFF before rendering. The split/reassemble uses the same standard APIs on both sides, so reconstruction is deterministic and lossless.

### 14.D Authentication

All communication between ALPS and Documentum uses server-to-server authentication. ALPS doesn't pass individual user credentials to Documentum. The ALPS application server authenticates itself to the Documentum Application Server using shared system-level credentials. Authentication happens at the service call level, not the user session level. This means access is controlled and auditable at the system boundary, regardless of which individual user is logged into ALPS. Security controls beyond this were enforced per the bank's enterprise security policy.

### 14.E The Two Operations

**Retrieve document:** An ALPS operator at any stage — credit decisioning, verification, disbursement — needs to view a customer's Form 16 for a mortgage application. They invoke the document view function in ALPS. ALPS sends the SOAP request with the Application Number and Document Type `supporting_doc`. Documentum locates and returns the TIFF. The operator sees the image inside ALPS in under 2 seconds above baseline. No context switch. No separate login.

**Update document attributes:** An operator marks a document as "Verified" or adds a processing remark after reviewing it. ALPS invokes the attribute update operation with the Application Number, Document Type, and updated attribute values. Documentum updates the metadata on the `citibank_captiva_doc` object. Every subsequent stage that touches that document sees the updated state.

### 14.F Why This Integration Changed Everything

ORBIFLOW and ALPS were two separate islands. Every document retrieval was a manual crossing between them. The Documentum-ALPS web service integration collapsed that gap entirely. Documents are in-context, real-time, and bidirectional — ALPS can retrieve and update, not just read. Because the integration is SOAP/XML, it's platform-independent. The ALPS system invokes the service regardless of its internal stack. And because authentication is server-to-server, the security model is clean and doesn't leak individual credentials across system boundaries.

### 14.G Deployment

The Documentum web service was deployed as an EAR module on the Application Server (WebSphere/WebLogic). DFC had to be installed on that same machine before deployment — it's the library the web service uses to talk to the Content Server. The Application Server and Content Server lived on separate UNIX boxes.

---

## 15. The CaptivaJob — Custom Java Processing

### 15.1 Why I Wrote This

Captiva's DCTM Export module does one thing: it takes indexed documents and drops them into a specified location in Documentum. In this case, that location was a single shared intermediate folder. Every document — credit card forms, mortgage forms, supporting docs — landed there first.

The business needed each document in its specific final folder, classified by product type and document type, with the BPM workflow triggered immediately after placement. Captiva's out-of-the-box export module can't do that routing. So I built the CaptivaJob — a custom scheduled Java background job that runs after export, reads the intermediate folder, and handles everything from there.

### 15.2 Class Hierarchy

```
com.citibank.afu.methods.captiva
    └── CaptivaJob  (extends CtbMethodBaseCaptiva)
            └── CtbMethodBase  (abstract base)
```

### 15.3 Methods

| Method | Description |
|---|---|
| `void execute(String[] arg)` | Entry point; orchestrates the full document routing workflow |
| `boolean isTypeValid(String objType)` | Validates the Documentum object type against known types |
| `void createDocument(IDfSession, IDfSysObject, boolean, String)` | Creates/moves document to correct folder with all attributes |
| `void createWorkFlowInstance(String, String, String)` | Instantiates the Documentum BPM workflow for the placed document |
| `void cleanMigratedDocuments()` | Removes successfully processed documents from the intermediate folder |
| `void doDelete(IDfSession, IDfSysObject)` | Deletes a document object from the session |
| `String getNewObjectName(IDfSession, IDfSysObject)` | Derives the final document name per naming conventions |
| `String getNewObjectLocation(IDfSession, IDfSysObject, String)` | Determines the target Documentum folder path by document/application type |
| `String doExport(IDfSysObject, String, String, String, IDfSession, int)` | Executes the export from intermediate to final location |
| `void executeOperation(IDfOperation)` | Runs a DFC operation with error handling |
| `void deleteTIFFiles(String)` | Cleans up local TIFF files after successful migration |

### 15.4 Supporting Classes

| Class | Role |
|---|---|
| `CtbMethodBase.java` | Abstract base: session lifecycle, date/time helpers, `execute()` contract |
| `SaveOutput.java` | Redirects and captures job output streams for logging |
| `Constants.java` | Central static configuration constants |
| `PropertyLoader.java` | Singleton for loading the configuration property file |

### 15.5 Object Types Used

```
credit_card_app_form    — Credit Card application forms
mortgage_app_form       — Mortgage/Loan application forms
supporting_doc          — All supporting documents
```

---

## 16. Exception Handling and Reporting

### 16.1 The Error Handling Framework

One thing I was adamant about on this project: every module in the pipeline had to have a defined, explicit error behavior. Banking is an audited environment. You can't have documents silently stalling in a queue, or worse, disappearing. Here's how I specified it:

| Module | Client-Side Error Action | Server-Side Error Action |
|---|---|---|
| **Scanning** | Display error to user | Generate error report |
| **Image Enhancement** | — | Log error; move document to next module |
| **DelBlank** (Blank page deletion) | — | Log error; move document to next module |
| **Empty Nodes** (IE to AQA transition) | — | Log error; move document to next module |
| **AQA & Rotate** | — | Log error; move document to next module |
| **Hold** | — | Log error; move document to next module |
| **Index** | Display validation errors to user for correction | Log error; **do NOT move to next module** (operator must resolve) |
| **Export** | — | Log error; reschedule for re-export on network/recoverable errors; **never mark batch for deletion until export confirmed** |
| **DelBatch** | — | Log error |
| **Timer** | — | Log error |

The Index module is the only place in the chain where I deliberately chose to hold rather than pass forward on an error. Every other module logs and moves on because a processing error in enhancement or quality checking doesn't compromise the document's metadata — you can retry it. But an indexing error means the metadata is wrong or missing. Forwarding that document to Documentum would corrupt the repository. You fix it here, or it doesn't go anywhere.

### 16.2 Report Formats

**Scan Report:**
```
User Id      :
Batch No     Document Name     Pages     Date & Time
```

**Export Report:**
```
User Id      :
Batch No     Document Name     Pages     Date & Time
```

**Debug/Error Log:**
```
[Date & Time] - [User Id] - [Process Name] - [Module Name] - [Batch Id/Name] - [Document Name] - [Node Id] - [Error/Debug Message]
```

These formats were designed for reconciliation — the bank's audit team needed to be able to match every document scanned against every document successfully exported. The User ID for scanning and indexing doesn't have to be the same person, which matters because in practice the person at the scanner and the person at the index station were often different operators. Reports were generated at every process level, not just at the end of the pipeline.

---

## 17. End-to-End Technical Flow

Putting it all together — the full step-by-step from document receipt to cleanup:

```
 1. Customer submits application + supporting documents.

 2. Bank operator performs manual pre-classification:
    - Product type: Credit Card or Mortgage?
    - Does the form have a barcode?
    - Are supporting documents present and complete?

 3. Documents are arranged in batches.
    - Blank separator pages inserted between individual documents.
    - All pages within one document are contiguous.

 4. Operator opens InputAccel Scan (thick client) or eScan (eInput thin client).
    - Selects the correct process (BANKING, MORTGAGE, etc.)
    - Creates a new batch and scans all documents.
    - On detecting a blank page → New Document event fires → document boundary recorded.

 5. IE processes each page:
    - Deskew → Border Removal → Smooth → Hole Removal → Noise Removal
    - Blank Page Detection marks blank separator pages.

 6. Multi module:
    - Deletes all detected blank pages.
    - Creates document level nodes in the batch tree.

 7. AQA module:
    - Runs Orientation check on every page.
    - Flags mis-oriented pages for correction.

 8. Rotate module:
    - Auto-rotates any pages flagged by AQA.

 9. Hold module:
    - Queues documents for the Index operator.

10. Index module:
    - Operator views image and enters/confirms:
        RefNo (application number, 10–16 digits)
        BusinessType (from picklist — default BAN)
        DocumentType (from picklist — e.g. bank_acc_app_form, affidavit, etc.)
        TransactionType (from picklist — e.g. TR, CC, MLI, etc.)
        Priority (from picklist)
        Holder (from picklist — 1 to 99)
    - Validation enforced on every field before document can advance.
    - Index errors hold the document — do not advance to export.

11. Export module (DCTM Export):
    - Connects to Docbase: citi_pre_prod.
    - Assembles multi-page TIFF (CCITT Group 4, Binary, Multi-Page enabled).
    - Creates/searches citibank_captiva_doc object in dm_cabinet/dm_folder hierarchy.
    - Uploads TIFF to intermediate folder in Documentum.
    - On error: automatically retries once; on retry failure: prompts operator.
    - Batch NEVER marked for deletion until all documents confirm successful export.

12. CaptivaJob (scheduled Java background job):
    - Reads documents from intermediate Documentum folder.
    - Validates document type (isTypeValid).
    - Determines final folder path (getNewObjectLocation).
    - Creates/moves object to correct folder with all attributes (createDocument).
    - Instantiates Documentum BPM workflow (createWorkFlowInstance).
    - Cleans up intermediate folder entries (cleanMigratedDocuments).
    - Deletes local TIFF temp files (deleteTIFFiles).

13. Documentum BPM workflow:
    - Routes document through downstream verification, approval, and processing stages.

14. ALPS system (loan processing):
    - Invokes Documentum web service (SOAP/XML) with Application No. + Document Type.
    - Receives document as byte stream (large files are split and reassembled).
    - Updates document attributes via secondary web service as processing progresses.

15. Timer module (02:00 daily):
    - Fires rules 0200, 0201, 0202 in sequence.
    - Deletes all batches meeting the day=0 condition.
    - Sets batch priority to 50 for deletion tasks.

16. Scan and Export reports generated at every stage for reconciliation and audit.
```

---

## 18. Non-Functional Requirements

| Requirement | Specification |
|---|---|
| Web service response time | ≤ 2 seconds above baseline ALPS response time |
| System availability | All working days including Saturdays |
| Downtime | Scheduled and ad-hoc maintenance only |
| Image format | TIFF, Binary, CCITT Group 4 compression |
| Export retry on network error | 1 automatic retry before operator prompt |
| Batch deletion safety | No deletion until all documents in batch confirmed exported |
| Disaster recovery | Physical documents available as fallback during extended outages |
| Security | Enterprise security policy; server-to-server auth on web service |
| Scan reports | Generated at all process levels for full reconciliation |

---

## 19. Licensing

| License Code | Module | Purpose |
|---|---|---|
| IASCAN | Scan | Document scanning |
| IAIPI | Image Enhancement | Image processing + barcode detection |
| IA Index | Index | Operator-driven and automatic indexing |
| IAEXIMG | Image Export | Multi-page TIFF creation |
| IAEXDM | DCTM Export | Documentum export (one license per instance — important for the dedicated eInput export) |
| IAMULTI | Decision Making (Multi) | Document routing and blank page handling |

---

## 20. Authentication and Access Control

User authentication runs through Windows Domain User ID — standard for this kind of Captiva deployment. Integration with SSO or ARMOR was explicitly out of scope; the Windows domain was the agreed boundary.

Role-based access was enforced at the process level, not just the module level. Different user groups had access to different Captiva modules based on their role. Index access was segmented by product — operators in the Credit Card stream couldn't see the Mortgage queue and vice versa. Separate processes for each product type is what made this clean to enforce.

---

## 21. What This Project Actually Achieved

Looking back, a few things stand out as genuinely worth noting.

Eighteen processes in production — covering every combination of product type, document type, capture mode, and environment — reflects a deployment that was thought through to the edges. It's not a proof-of-concept; it's a platform.

The blank-page-as-separator model is simple operationally but required careful pipeline design to make it work cleanly. Blank page detection in IE, document boundary creation in Scan's event actions, physical deletion in Multi — three modules, each doing one thing, achieving a result that required zero changes to how operators physically prepared their scan batches.

The indexing design — six fields with pre-validation, picklist enforcement, and three-level granularity — means no document ever reaches Documentum without complete, validated metadata. Data quality is enforced at the point of capture, not discovered as a problem after the fact.

The "never delete until fully exported" rule is the kind of thing that feels obvious in hindsight but has to be explicitly designed and enforced. Without it, a network blip during export could result in documents being deleted from Captiva before they've successfully landed in Documentum. That's unrecoverable. One rule, stated clearly, prevents an entire class of data loss.

And the ALPS integration is what transforms AFU from an archive into a live operational system. Documentum isn't just a storage destination — it's an active participant in the loan processing workflow, returning documents on demand and accepting metadata updates as processing progresses. That's the difference between building a filing cabinet and building an integrated platform.

---