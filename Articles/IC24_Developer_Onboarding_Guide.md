---
description: >-
  A dense, high-signal onboarding guide for engineers migrating from
  OpenText Captiva 7.x (InputAccel) to Intelligent Capture 24.x.
  Not a beginner guide. Not a product manual.
---

# OpenText Intelligent Capture 24.x — Developer Onboarding Guide

**Target audience:** Engineers with hands-on Captiva 7.x experience — IPPs, batch pipelines, Dispatcher, custom modules.\
**Based on:** OpenText Intelligent Capture CE 24.2 (ECPCORE240200)

---

## Who This Guide Is For

You already know:

* How IPPs sequence modules and wire IA values
* How Dispatcher classifies and routes documents
* How to write custom modules or VB/C# scripts against the InputAccel SDK
* What a ScaleServer group is and why it exists

What this guide does: **reset your mental model**. Most difficulty experienced by 7.x developers on IC 24 comes from mapping old patterns onto a system that has fundamentally changed its execution model.

---

## Table of Contents

1. [Mental Model Reset — Captiva Then vs. Now](ic24-developer-onboarding.md#1-mental-model-reset)
2. [Internal Architecture Deep Dive](ic24-developer-onboarding.md#2-internal-architecture-deep-dive)
3. [Pipeline Execution Model — Old vs. New](ic24-developer-onboarding.md#3-pipeline-execution-model)
4. [Development Model — Critical Section](ic24-developer-onboarding.md#4-development-model)
5. [Integration Patterns](ic24-developer-onboarding.md#5-integration-patterns)
6. [Recognition and AI Evolution](ic24-developer-onboarding.md#6-recognition-and-ai-evolution)
7. [Migration Mindset — Practical Guidance](ic24-developer-onboarding.md#7-migration-mindset)
8. [Critical Extension Points](ic24-developer-onboarding.md#8-critical-extension-points)
9. [Deprecation Quick Reference](ic24-developer-onboarding.md#9-deprecation-quick-reference)
10. [Getting Productive — First Steps](ic24-developer-onboarding.md#10-getting-productive)

---

## 1. Mental Model Reset

### 1.1 The Three Shifts

Before touching any configuration or code, discard three deeply ingrained 7.x assumptions.

#### Shift 1 — Monolithic IPP Pipelines → CaptureFlow-Orchestrated Execution

In 7.x, the **IPP** (Integrated ProcessFlow Project) was the backbone of every solution: a compiled script that sequenced every module in a hard-coded chain. Changing the sequence meant editing the IPP, recompiling, and redeploying.

In IC 24, the equivalent artefact is the **CaptureFlow** — a graphical, configurable process model authored in **CaptureFlow Designer**. It is a directed graph of module steps (attended and unattended) that defines how batches are created and routed. It compiles to an IAP file. More importantly: CaptureFlow steps are **dynamically parameterized at runtime** via IA values, and conditional routing requires no scripting.

{% hint style="info" %}
IPP files from Dispatcher 6.0 SP3, 6.5, and 6.5 SP1 can be imported into a Recognition design area in IC Designer if you hold an Advanced Recognition license. Field placements on existing templates are preserved. This is a one-time migration aid — maintain the result as a CaptureFlow going forward.
{% endhint %}

#### Shift 2 — Script-Heavy Customization → Configuration-First Development

In 7.x, the answer to almost every non-trivial requirement was a script — VBScript in the IPP, VBA in Dispatcher, custom COM modules, or .NET Quick Modules. Scripts were the primary lever for routing logic, validation, enrichment, and export control.

In IC 24, **the first question is always: can this be done in configuration?** The Designer's profiles, document type definitions, recognition projects, export profiles, and CaptureFlow connector conditions cover 80–90% of real-world requirements without a line of code. Code is reserved for logic that genuinely cannot be expressed through configuration.

{% hint style="danger" %}
Replicating 7.x script patterns directly in IC 24 is the single most common migration anti-pattern. It produces solutions that are harder to maintain, harder to upgrade, and miss the architectural intent of the platform. Read [Section 7](ic24-developer-onboarding.md#7-migration-mindset) before writing any code.
{% endhint %}

#### Shift 3 — Thick Client Tools → Web-Based and API-Driven Interfaces

In 7.x, development happened in thick client tools: Dispatcher Designer, InputAccel Administrator, recognition project editors. In IC 24:

* **IC Designer** is the centralized IDE for all design tasks (profiles, document types, recognition, export, CaptureFlows).
* Operators use the **IC Web Client** (browser-based) or desktop Completion/ScanPlus modules.
* The **IC REST Services** layer exposes the full capture and Module Server surface to external applications. New integrations default to REST.

---

### 1.2 Conceptual Equivalence Map

| Captiva 7.x | Intelligent Capture 24.x | Notes |
|---|---|---|
| IPP (Integrated ProcessFlow) | **CaptureFlow** (.IAP) | Graphical, compiled, deployed to IC Server |
| Dispatcher project (.DPP) | **Recognition project** in IC Designer | Import DPP from Dispatcher 6.5/6.5 SP1 via Advanced Recognition |
| InputAccel Server | **IC Server** | Identical role. Still called "InputAccel Server" internally in some APIs |
| Dispatcher module (client service) | **Classification module** | Advanced Recognition license required |
| NuanceOCR module | **NuanceOCR module** | Same module. AmpLib barcode engine removed in 24.2 |
| Custom VB/C# IPP script | **.NET Code module step** (C# DLL) | VB.NET scripting is deprecated — migrate to C# |
| Process Developer tool | **CaptureFlow Designer** | Process Developer no longer shipped in 24.x |
| ScaleServer group (TCP/IP) | **ScaleServer group** (SQL Server-backed) | Same concept, requires external SQL Server DB |
| MDF (Module Definition File) | **MDF** | Same format, unchanged |
| IA values | **IA values** | Same categories. Dynamic IA values now support Unicode |
| Thick client operator stations | **IC Web Client** or desktop Completion/ScanPlus/Identification | |
| SOAP Web Services I/O | **WS Input / WS Output** (still ships) | Prefer IC REST Services for new integrations |

---

## 2. Internal Architecture Deep Dive

IC 24 has two execution paths. Understanding which one applies to your scenario drives every design decision.

### 2.1 Component Overview

| Component | Role | 7.x Equivalent |
|---|---|---|
| **IC Server** (InputAccel Server) | Central broker: manages batch lifecycle, schedules tasks, pushes tasks to production modules via TCP/IP | InputAccel Server — identical role |
| **CaptureFlow Designer** | Centralized IDE: design, configure, compile, deploy CaptureFlows, profiles, document types, recognition projects, export profiles | Dispatcher Designer + IPP editor + Process Developer |
| **Production Modules** | Client-side Windows services or attended apps. Receive tasks from IC Server, process them, return results | InputAccel client modules |
| **Module Server** | Windows service managing a pool of service module instances (classification, OCR, image conversion) for the REST subsystem. Scales horizontally | No direct equivalent — new in the REST era |
| **IC REST Service** (IIS) | JSON REST web service on IIS. Accepts batch creation requests → routes to IC Server, or ad hoc page requests → Module Server | Web Services Hosting (partial analogy) |
| **IC Web Client** | Browser-based operator interface. Calls IC REST Service under the hood | Thick client attended modules |
| **SQL Server DB** (optional) | Configuration, batch metadata, licensing, audit logs, ScaleServer group state. Required for ScaleServer and reporting | Same optional external DB in 7.x ScaleServer |

---

### 2.2 The Two Processing Paths

{% tabs %}
{% tab title="Path A — Async Batch (IC Server + Modules)" %}
This is the traditional Captiva execution model, now expressed as a CaptureFlow.

**Flow:**
1. Batch is created (ScanPlus, Standard Import, WS Input, or IC REST Service)
2. Batch stored on IC Server
3. IC Server schedules tasks and pushes them to available production module instances
4. Modules process, return results, server advances to next CaptureFlow step

**Characteristics:**
- Asynchronous by design — the caller does not wait for completion
- Batch nodes are locked during processing
- Multiple module instances on different machines process tasks in parallel
- Correct for high-volume, multi-stage workflows with operator review (Completion, Identification)

{% hint style="info" %}
This path is what your entire 7.x experience maps to. The internal mechanics (IA value triggers, stage files, task push model) are unchanged.
{% endhint %}
{% endtab %}

{% tab title="Path B — Synchronous Real-Time (REST + Module Server)" %}
New in the IC REST era. Designed for mobile, web, and LOB integrations.

**Flow:**
1. External application calls IC REST Service via HTTPS
2. **Batch request**: REST Service creates an IC batch, triggers CaptureFlow on IC Server
3. **Ad hoc request**: REST Service forwards directly to Module Server → processes and returns results in the same HTTP call

**Characteristics:**
- Synchronous for ad hoc requests (classify/extract a single page, return result in the HTTP response)
- Module Server manages a pool of service instances; scales by launching additional instances
- Correct for real-time capture in mobile apps, inline document processing in LOB systems

{% hint style="info" %}
The two paths are not mutually exclusive. A REST-initiated batch runs through a CaptureFlow using production modules — Path B can feed Path A. The distinction is about how the batch is created and whether the caller needs a synchronous response.
{% endhint %}
{% endtab %}
{% endtabs %}

---

### 2.3 The Batch Model

The batch model is **unchanged** from 7.x. Understanding it precisely is required for all development work.

A batch is a hierarchical tree of nodes with **up to 8 levels**:

```
Level 7 — Batch (root)
Level 6 — (available for custom grouping)
Level 5 — (available for custom grouping)
Level 4 — (available for custom grouping)
Level 3 — (available for custom grouping)
Level 2 — Folder (in recognition)
Level 1 — Document
Level 0 — Page (leaf)
```

State is tracked via **IA values** attached to nodes. **Stage files** (images, OCR output, data files) are stored on the IC Server associated with nodes. The server uses **trigger IA values** to determine when a node is ready for the next step.

{% hint style="info" %}
The batch folder on disk is structured as `Batches\XXXX\YYY\ZZZ` based on batch ID split 4-3-3. Stage file extensions are sequential hex numbers per node (e.g., `23e.1`, `23e.2`). This is unchanged from 7.x and matters when debugging batch file issues directly.
{% endhint %}

---

### 2.4 IA Values — The Nervous System

IA values are the primary data transport mechanism. Every metadata value, routing decision, and file pointer flows through them. The categories are identical to 7.x:

| Category | Purpose | Example |
|---|---|---|
| **Input/Output file values** | Pointers to stage files. Connect module steps together | `InputImage`, `OutputImage` |
| **Trigger values** | Subset of input file values. When all triggers are non-zero, module can process the task | `InputImage` (most common trigger) |
| **Setup values** | Per-task configuration sent to the module. Can differ per task | OCR language, scanner settings, index field definitions |
| **Client processing results/statistics** | Output metadata from each module | Timestamps, operator name, OCR results, error codes |
| **Batch values** | Dynamic values scoped to batch nodes, created at runtime | Custom counters, flags |
| **System values** | Global, not batch-scoped | `$user`, `$module`, `$machine`, `$server` |
| **Non-MDF values** | Batch name, ID, priority, description | Appear in batch creation UI |

The reserved values `IATaskRouting` and `IADepartments` still control department-based routing — the IC 24 equivalent of Dispatcher routing rules. Set them in CaptureFlow steps or dynamically in code.

---

## 3. Pipeline Execution Model

### 3.1 7.x IPP Model — What It Was

{% tabs %}
{% tab title="IPP Characteristics" %}
- **Linear by default.** Branching required explicit IA value conditionals written in script.
- **Stage-based with fixed points.** Changing step order required editing and recompiling the IPP.
- **No graphical authoring.** Process Developer provided a view, but design was text/script-driven.
- **Dispatcher owned classification routing.** The DPP was a separate artefact with its own VBA scripting environment.
- **Script was the extension mechanism.** Every non-trivial customization lived in IPP scripts or Dispatcher VBA.
{% endtab %}
{% endtabs %}

### 3.2 IC 24 CaptureFlow Model — What Replaced It

{% tabs %}
{% tab title="CaptureFlow Characteristics" %}
- **Graphical-first.** Steps are connected visually in CaptureFlow Designer. Conditional routing is configured via IA value expressions on connectors — no scripts.
- **Configuration-driven parameterization.** Profiles, document types, and recognition projects are reusable service components uploaded separately. A step references them by name; they can be swapped without recompiling the CaptureFlow.
- **Dynamic routing via IA values.** Set `IATaskRouting` or `IADepartments` in a preceding step to route tasks to specific module instances or departments.
- **Process versioning.** The server maintains numbered version folders (`XPP`, `IAP`, `DLL`). Multiple versions coexist. Batches are pinned to the version that created them.
- **Attended and unattended steps.** Unattended steps run as Windows services. Attended steps (Completion, Identification, ScanPlus) require a human.
{% endtab %}
{% endtabs %}

---
### 3.3 Where Decisions Are Made — Priority Order

This is the most important table in this guide. Follow this order every time:

| Priority | Mechanism | When to Use |
|---|---|---|
| **1st** | **CaptureFlow connector conditions** | IA value expressions on step transitions. No code. Use this first. |
| **2nd** | **CaptureFlow step configuration** | Profiles control processing rules; document types enforce validation constraints |
| **3rd** | **Recognition project rules** | Classification thresholds, extraction zone definitions, document separation |
| **4th** | **Client-side scripts in Completion/ScanPlus** | UI-level validation and field population specific to a document type |
| **5th** | **.NET Code module step** | Logic that cannot be expressed by any of the above — external system calls, batch tree restructuring |
| **6th** | **REST API integration** | External system lookups, validation, enrichment from outside the capture pipeline |

{% hint style="warning" %}
Every time you reach for a `.NET Code` module step, ask: "Can this condition be expressed as a CaptureFlow connector expression?" The connector expression evaluator supports IA value comparisons, string operations, and boolean logic — no code required.
{% endhint %}

---

## 4. Development Model

### 4.1 The Configuration Layer — Do This First

The IC Designer provides design areas, each producing a **reusable service component** that can be uploaded to the server and referenced by multiple CaptureFlows:

| Design Area | What You Build | 7.x Equivalent |
|---|---|---|
| **Image Processing** | Profiles: image enhancement filters, blank page detection, deskew, barcodes, annotations | Image Processor module setup |
| **Image Conversion** | Profiles: format conversion (TIFF→PDF, etc.), splitting, merging, color conversion | Image Converter module setup |
| **Standard OCR** | Profiles: zonal/full-page OCR, OCR cache, PDF/text output | NuanceOCR zones and settings |
| **Recognition** | Recognition projects: templates, classification logic, extraction zones, document separation. Import DPP files from Dispatcher 6.5/6.5 SP1. | Dispatcher project (.DPP) — direct equivalent |
| **Document Types** | Index field definitions, validation rules, data entry form layout, field controls, indexing families | Completion field setup + Dispatcher index families |
| **Export** | Export profiles: CSV, XML, free text, email, CMIS, Content Server, AppEnhancer, Documentum, ODBC | Standard Export + Enterprise Export modules |
| **CaptureFlow Designer** | The process model: step graph, IA value wiring, conditional routing, module assignments | Process Developer + IPP editor |

---

### 4.2 Scripting in IC 24 — Four Distinct Contexts

Mixing these up causes bugs that are hard to diagnose. Know which context you are in.

{% tabs %}
{% tab title="Profile Scripting" %}
**Scope:** Document types and page-level image enhancements. Completely independent of any specific process, batch, task, or IA values.

**Rules:**
- ❌ Do NOT access IA values, task objects, or process state
- ✅ Correct use: field-level validation formulas, custom field controls, image annotation logic
- Language: **C#** (VB.NET deprecated)
{% endtab %}

{% tab title="Task Scripting" %}
**Scope:** Task and batch node manipulation. Has access to IA values and the batch tree. Can call profile script APIs to manipulate document objects.

**Rules:**
- ❌ Do NOT use profile scripting events or UI-related APIs
- ✅ Correct use: batch-level IA value manipulation, node restructuring, routing decisions based on extracted data
- Language: **C#** (VB.NET deprecated)
{% endtab %}

{% tab title="Client-Side Scripting" %}
**Scope:** Runs as part of a module step within a CaptureFlow, triggered by module-defined events. Does NOT require a separate CaptureFlow step.

**Rules:**
- ✅ Correct use: operator UI event handlers, dynamic field visibility, validation popups, external lookups triggered during Completion
- Language: **C#** (new CaptureFlows without existing VB.NET create C# only)
{% endtab %}

{% tab title="Recognition Scripting (VBA)" %}
**Scope:** Customizing Classification and Identification recognition projects. Uses a VBA-compatible scripting environment with VBA Script Editor and debugger.

**Rules:**
- ✅ Correct use: custom classification thresholds, extraction post-processing, template logic
- This is the context closest to the old Dispatcher VBA environment
{% endtab %}
{% endtabs %}

---

### 4.3 The .NET Code Module Step

The `.NET Code` module runs a custom .NET assembly (DLL) as an independent CaptureFlow step. It exposes a .NET interface to read and write batch data — IA values, batch nodes, stage files.

```csharp
// Minimal entry point — implement IModule from InputAccel35.dll
using InputAccel.QuickModule;
using InputAccel.IAValues;

public class MyModule : IModule
{
    public void ProcessTask(IModuleTask task)
    {
        // Read IA value from the batch node
        string custId = task.GetIAValue("CustomerID");

        // Call external system, populate result
        string custName = LookupCustomer(custId);
        task.SetIAValue("CustomerName", custName);

        // Pass through the stage file to the next step
        task.SetOutputImage(task.GetInputImage());
    }
}
```

**Critical facts for .NET Code development:**

| Requirement | Specification |
|---|---|
| Target framework | **.NET 4.8** or higher |
| SDK DLL | **`InputAccel.*35.dll`** — NOT `InputAccel.*.dll` (old SDK) or `InputAccel.QuickModule.dll` (SDK 5.x .NET 1.1) |
| IDE | Visual Studio 2013+ (Professional or Standard recommended; Express has limited debugging) |
| Deployment | Compile DLL → upload via IC Designer → CaptureFlow step references by name |
| Custom module licensing | Modules with a Module ID/Name using network licensing must implement OTDS licensing APIs (SDK guide ECPCOREPK-PGD, section 2.8.3) |

{% hint style="warning" %}
**SDK migration path for old custom modules:**

- **`IAClient32.dll` (COM, SDK 5.x)** → Cannot run on 24.x. Rewrite as `.NET Code` module step.
- **SDK 5.x `.NET 1.1` (`InputAccel.QuickModule.dll`)** → Replace .NET 1.1 dependency with .NET 4.8, rebuild against `InputAccel.*35.dll`.
- **IC 7.0–7.7 (`.NET 3.5` / `.NET 4.5.2`)** → May run as-is under .NET 4.8. Test with `QuickModuleHost.exe -modulename:<module>` before committing to a full rewrite.
{% endhint %}

---

### 4.4 Config vs. Code vs. REST — Decision Table

| Scenario | Right Approach | Wrong Approach |
|---|---|---|
| Route documents by document type | CaptureFlow connector condition on classification result IA value | Script to read IA value and set routing flags |
| Validate a field against a regex pattern | Document type field property: regular expression constraint | Client-side script with regex logic |
| Populate fields from external DB during indexing | Client-side script in Completion step OR .NET Code step | Custom module opening DB connection during task processing |
| Export to a CMIS repository | Standard Export with CMIS profile | Custom export module built from scratch |
| Complex batch tree restructuring (split/merge nodes) | Multi module (built-in) or .NET Code module | Client-side script (wrong scope) |
| Classify and extract a single page in real time | IC REST API: `ClassifyExtractPage` on Module Server | Full async batch submitted to CaptureFlow |
| Custom module with hardware integration | .NET Code module with `InputAccel35.dll` | COM module (`IAClient32.dll`) — not supported on 24.x |

---

## 5. Integration Patterns

### 5.1 REST-Based Batch Ingestion

External applications submit documents to IC 24 by calling the IC REST Service. This replaces WS Input for new integrations.

```http
POST /icws/rest/batch
Content-Type: multipart/form-data

Body: process name, document files, IA values to pre-populate
```

The caller receives a batch ID. The CaptureFlow processes asynchronously. The caller polls for batch status or uses event-based notification if configured.

{% hint style="info" %}
WS Input (SOAP) still ships and is appropriate when maintaining legacy integrations or when the upstream system cannot use REST.
{% endhint %}

---

### 5.2 Ad Hoc Page Processing (Module Server)

For synchronous classification and extraction **without creating a persistent batch**:

```http
POST /icws/rest/services/classify
POST /icws/rest/services/classifyextractpage
POST /icws/rest/services/classifyextractdocument
```

These endpoints accept an image or PDF, classify it against a deployed recognition project, optionally extract field data, and return structured results in the same HTTP response. Foundation for mobile capture and inline LOB processing.

{% hint style="info" %}
**IC 24.2 addition:** New `defaultDocumentType` and `AllowedDocumentType` parameters on Classify, ClassifyExtractPage, and ClassifyExtractDocument REST APIs. Use these to constrain the classification result set when you already know the document type family in context.
{% endhint %}

---

### 5.3 External Validation and Enrichment During Capture

The most common integration pattern during indexing. Choose the right variant:

{% tabs %}
{% tab title="Pattern A — Client-Side Script (Operator-Triggered)" %}
Write a C# client-side script attached to the `OnValidate` or button-click event in the Completion step. Script makes an HTTP call and populates field values.

**When to use:** Lookup is operator-triggered; result must be shown before commit; low latency acceptable (sync HTTP call on operator action).

**Avoid:** Long-running calls or calls that block the Completion UI for more than a few seconds.
{% endtab %}

{% tab title="Pattern B — .NET Code Step (Automatic Enrichment)" %}
Add a `.NET Code` module step before Completion in the CaptureFlow. Step reads IA values, calls the external system, writes results back as IA values for downstream steps.

**When to use:** Enrichment should happen automatically, without operator involvement, as part of unattended processing.

**Example:** Look up vendor in SAP by the vendor ID extracted during OCR; populate invoice metadata fields before Completion opens.
{% endtab %}

{% tab title="Pattern C — WS Output/Input (Explicit Orchestration)" %}
Use `WS Output` to call an external web service mid-pipeline and `WS Input` to receive the response.

**When to use:** The external interaction must be modeled as an explicit CaptureFlow step for audit/compliance reasons, or the interaction has its own retry and error handling requirements.
{% endtab %}
{% endtabs %}

---

### 5.4 ERP Lookup During Indexing — Concrete Example

**Scenario:** After OCR extraction, look up a customer number in an ERP system and populate additional fields before Completion.

1. In the CaptureFlow, insert a `.NET Code` step **between** the Extraction step and the Completion step.
2. In the `.NET Code` assembly: read `CustomerNumber` IA value, call the ERP REST API, parse the response, write `CustomerName`, `CustomerAddress`, `AccountType` IA values.
3. In the Completion document type: mark `CustomerName`, `CustomerAddress`, `AccountType` as **read-only display fields** — they will be pre-populated from IA values when the operator opens the batch.
4. On the CaptureFlow connector **after** the `.NET Code` step: add a condition — if `CustomerNumber` IA value is empty, route to an exception handling step instead of proceeding to Completion.

---

### 5.5 Custom Export

**Standard Export with a built-in profile** covers most targets: CMIS, Content Server, AppEnhancer, Documentum, SharePoint, ODBC. For targets not covered:

| Approach | When to Use |
|---|---|
| **WS Output** | Target system accepts SOAP/REST and you want the interaction modeled as a CaptureFlow step |
| **.NET Code step at batch/document level** | Full control over export logic — read stage files and IA values, construct payload, call target API |
| **Standard Export free-text/XML profile → .NET Code** | Target accepts file-based import; generate the import file via Standard Export, then call the import API from a `.NET Code` step |

{% hint style="danger" %}
**ApplicationXtender Export** (LiveLink Export to Content Server) is **deprecated** in 24.2. Migrate to Standard Export + AppEnhancer profile or Standard Export + Content Server profile respectively.
{% endhint %}

---

## 6. Recognition and AI Evolution

In 7.x, recognition was entirely **template-based**: train templates against sample images, define extraction zones, Dispatcher classifies by graphical/textual similarity. Improving accuracy meant adding templates or tuning zones manually.

IC 24 retains template-based recognition and adds **IEE (Information Extraction Engine)** — ML-assisted classification and extraction.

### 6.1 IEE vs. Template-Based Extraction

| | Template-Based (Graphical) | IEE (ML-Assisted) |
|---|---|---|
| **Classification** | Graphical similarity score against template images | Keyword analysis, image-based analysis, text string analysis, free-form recognition |
| **Extraction** | Fixed pixel zones defined per template | Learned positions and patterns; no zone definitions required |
| **Accuracy improvement** | Manual — add templates, tune zones | Automatic — continuous learning from operator corrections |
| **Best for** | Highly structured, fixed-format documents (tax forms, standard invoices with fixed layouts) | Variable-format documents (invoices from multiple vendors, correspondence) |
| **Developer effort** | High initial setup; low ongoing | Low initial setup; configure the learning pipeline once |

---

### 6.2 Configuring the IEE Learning Loop

The learning loop is the main operational difference from 7.x recognition. It requires three components in the CaptureFlow:

```
[Extraction/Classification step]
        ↓ (low-confidence result)
[Collector module step]  ← tags document as "collectable"
        ↓
[Completion step]  ← operator corrects extraction
        ↓
[Production Auto-Learning Supervisor service]  ← scheduled; creates new templates, updates field positions
```

**Your job as a developer:**
1. Configure the IEE profile in Designer (training set, document type mappings, thresholds).
2. Add the Collector module step in the CaptureFlow for low-confidence paths.
3. Configure the Production Auto-Learning Supervisor service to schedule learning runs.
4. Do NOT manually define extraction zones for IEE document types — zones are learned.

{% hint style="info" %}
IC 24.2 added the **ResetIEETool** to reset IEE learning model data — classification learning, extraction learning per document type, and document separation learning. Use this when migrating an existing IEE configuration or when the model has degraded from bad training data.
{% endhint %}

{% hint style="warning" %}
**Known issue (CAPC-13768):** IEE profile names are **case-sensitive**. Be consistent with naming in Designer and when referencing profiles in CaptureFlow steps.
{% endhint %}

---

## 7. Migration Mindset

### 7.1 Stop Doing These Things

#### ❌ Stop — Reaching for Scripts Before Configuration

Every module step has a profile. Every profile has configurable options. Every document type has field-level validation rules, regex constraints, lookup tables, and auto-population formulas. The reflexive 7.x response of "I'll script this" is **wrong most of the time in IC 24**. Check the profile options first.

#### ❌ Stop — Building Rigid, Linear CaptureFlows

IC 24 CaptureFlows can branch, loop, and conditionally skip steps. Do not replicate a linear 7.x pipeline just because it's familiar. Design for the actual conditional logic the business requires — separate paths for different document types, exception branches for low-confidence extractions, expedited paths for high-priority batches — from the beginning.

#### ❌ Stop — Using VB.NET for New Scripting

VB.NET CaptureFlow scripting is planned for deprecation. New CaptureFlows without existing VB.NET scripts create new scripts in **C# only**. Do not invest in new VB.NET logic. Convert existing VB.NET scripts to C# using Visual Studio or a third-party converter.

#### ❌ Stop — Replicating the Dispatcher Architecture

Dispatcher combined classification, extraction, routing, and scripting into one VBA environment. In IC 24, these concerns are separated by design. Do not try to consolidate everything back into recognition scripting as a Dispatcher analog.

---

### 7.2 Start Doing These Things

#### ✅ Start — Designing Flows, Not Pipelines

A CaptureFlow is a directed graph. Design from the business process outward: what are the possible document types? What happens when classification confidence is low? What happens when a required field is missing? Model these as branches from the beginning.

#### ✅ Start — Using REST as a First-Class Integration Point

Any external system that triggers capture, queries batch status, or receives processed results should use the IC REST API. The REST surface has been significantly expanded and is the strategic integration path. SOAP works but is legacy.

#### ✅ Start — Thinking in Service Components

Profiles, document types, and recognition projects are reusable service components deployed independently of CaptureFlows. Design them to be reused across multiple processes. A single Standard OCR profile referenced by five CaptureFlows means one place to update OCR settings.

---

### 7.3 Common Pitfalls

| Pitfall | Symptom | Correct Approach |
|---|---|---|
| Duplicating module setup per CaptureFlow step | Configuration change requires updating multiple steps | Create shared profiles in Designer. Reference the same profile from multiple steps. |
| Writing .NET Code for routing logic | Business logic is buried in DLL source; CaptureFlow is unreadable | Use CaptureFlow connector conditions with IA value expressions. Reserve .NET Code for side-effecting operations. |
| Ignoring the IEE feedback loop | IEE accuracy does not improve over time despite operator corrections | Configure the Collector module and Auto-Learning Supervisor. Corrections only feed the model if the collection pipeline is set up. |
| Using deprecated export modules | Solutions break when modules are removed in future releases | Migrate ApplicationXtender Export → Standard Export + AppEnhancer profile. Migrate ODBC Export module → Standard Export + ODBC profile. |
| Building custom modules with `IAClient32.dll` | Modules fail to load on 24.x clients | Migrate to .NET Code module step with `InputAccel35.dll` SDK. |
| Using the file-based internal database in production | Audit logging, ScaleServer, and Web Services do not work | Use SQL Server external database for any production deployment. File-based DB is for evaluation/demo only. |

---

## 8. Critical Extension Points

### 8.1 .NET Code Module Interface

```csharp
// Key references (from InputAccel35.dll)
using InputAccel.QuickModule;  // IModule, IModuleTask
using InputAccel.IAValues;     // IA value read/write
using InputAccel.Batch;        // Batch node navigation

// Reading/writing IA values
string value = task.GetIAValue("FieldName");
task.SetIAValue("FieldName", "NewValue");

// Reading input stage file, passing through to next step
string inputStagePath = task.GetInputImage();
task.SetOutputImage(inputStagePath);

// Iterating child nodes (if triggered at document level, level 1)
foreach (IBatchNode page in task.CurrentNode.ChildNodes)
{
    string pageStageFile = page.GetIAValue("OutputImage");
    // process page...
}
```

**Deployment steps:**
1. Compile DLL targeting .NET 4.8, referencing `InputAccel.*35.dll`
2. In IC Designer: CaptureFlow step → upload DLL
3. The CaptureFlow step triggers at the level you specify (level 7 for batch-level, level 1 for document-level, level 0 for page-level)

---

### 8.2 REST Endpoints You Will Actually Use

| Endpoint | Use Case |
|---|---|
| `POST /icws/rest/batch` | Create a batch and trigger a CaptureFlow (async) |
| `GET /icws/rest/batch/{batchId}` | Poll batch processing status |
| `POST /icws/rest/services/classify` | Synchronously classify a page |
| `POST /icws/rest/services/classifyextractpage` | Synchronously classify + extract fields from a page |
| `POST /icws/rest/services/classifyextractdocument` | Synchronously classify + extract a multi-page document |

Full REST API reference: *OpenText Intelligent Capture REST Services Development Guide v2.9* (available on My Support).

---

### 8.3 External Call Pattern from Client-Side Script (Completion)

```csharp
// C# client-side script attached to OnValidate event in Completion step
using System.Net.Http;
using System.Threading.Tasks;

// Call external validation service
private async Task<string> LookupVendor(string vendorId)
{
    using var client = new HttpClient();
    var response = await client.GetAsync(
        $"https://erp.internal/api/vendor/{vendorId}");
    return await response.Content.ReadAsStringAsync();
}

// In event handler:
// 1. Read field value from the Completion form
// 2. Call LookupVendor()
// 3. Parse JSON response
// 4. Populate additional fields or set error message
```

---

### 8.4 Export Hook via .NET Code Step

```csharp
// .NET Code step triggered at document level (level 1)
// Exports document to a custom target

public void ProcessTask(IModuleTask task)
{
    // Get the stage file path for this document's image
    string imagePath = task.GetInputImage();

    // Get index fields from IA values
    string invoiceNumber = task.GetIAValue("InvoiceNumber");
    string vendorId      = task.GetIAValue("VendorID");
    string totalAmount   = task.GetIAValue("TotalAmount");

    // Build payload and POST to target system
    PostToTargetSystem(imagePath, invoiceNumber, vendorId, totalAmount);

    // Signal the step is done — pass through the image
    task.SetOutputImage(imagePath);
}
```

---

## 9. Deprecation Quick Reference

### 9.1 Deprecated / Removed in 24.2

| Item | Status | Action Required |
|---|---|---|
| **Process Developer** | No longer shipped | Migrate all processes to CaptureFlow Designer |
| **InputAccel SDK 5.x (`IAClient32.dll`, COM)** | Not supported | Rewrite as .NET Code module using `InputAccel35.dll` |
| **InputAccel SDK 5.x (`.NET 1.1`)** | Must migrate | Replace with .NET 4.8, rebuild against `InputAccel.*35.dll` |
| **VB.NET CaptureFlow scripting** | Deprecated — planned removal | Convert to C#; new CaptureFlows use C# only |
| **ApplicationXtender Export module** | Deprecated | → Standard Export + AppEnhancer profile |
| **Export to Content Server (LiveLink)** | Deprecated | → Standard Export + Content Server profile |
| **Copy module** | Deprecated | → Batch Copy module |
| **East Euro / APAC OCR engine** | No longer shipped | → Advanced OCR/ICR engine or Standard OCR module |
| **Nuance AmpLib barcode recognition** | Removed in 24.2 | Use other supported barcode recognition |
| **Version 6.x licenses with IndexPlus/Validation** | Deprecated | Contact OpenText Support for no-charge license refresh |

### 9.2 To Be Deprecated in a Future Release

| Item | Replacement |
|---|---|
| **ODBC Export module** | Standard Export + ODBC profile |
| **Documentum Exporter module** | Standard Export + Documentum profile (requires REST-enabled Documentum) |

---

### 9.3 Platform Requirements Summary (IC 24.2)

| Component | Requirement |
|---|---|
| IC Server OS | Windows Server 2022, 2019, or 2016 (64-bit) |
| SQL Server (optional, required for ScaleServer/reporting) | SQL Server 2014 SP3 through SQL Server 2022 |
| Client modules OS | Windows 10 Enterprise (1803+), Windows 11 Enterprise, or Windows Server 2022/2019/2016 |
| .NET Framework | **4.8 or higher** on all server and client machines |
| IC Designer | Requires .NET 4.8 Developer Pack **exclusively** — no other .NET version on the Designer machine |
| Web Client browsers | Chrome 94+, Firefox 94+, Microsoft Edge Chromium |
| IIS (for REST Service) | IIS 7.5 through IIS 10 |
| ScaleServer | Up to 8 servers; requires external SQL Server |
| OTDS (for network licensing) | OTDS 24.2 for IC and Real Time OTDS-based network licenses |

---

## 10. Getting Productive

Follow this progression. Each step validates your understanding of the previous one.

**Step 1 — Install IC Designer and read the Quick Start**\
Work through *Designing a CaptureFlow* (Designer Guide, section 4) and the Quick Start: Creating a Simple Process (section 4.4). Half a day.

**Step 2 — Build and run the minimal CaptureFlow**\
`Standard Import → Standard OCR → Standard Export`. Deploy it, create a test batch, run it end to end. This validates your environment and gives you a working baseline.

**Step 3 — Import your existing Dispatcher project**\
Import a `.DPP` file (from Dispatcher 6.5/6.5 SP1) into a Recognition design area. Validate that templates and field placements were imported correctly before building on top of them.

**Step 4 — Replace one 7.x script with configuration**\
Pick one IPP script or custom module. Determine whether its logic can be replaced by a CaptureFlow connector condition, a profile setting, or a document type rule. Implement the replacement and compare output.

**Step 5 — Write a minimal .NET Code step**\
Read one IA value, call an external HTTP endpoint, write the result back to a different IA value. Build up from there. Start small — the runtime behavior of the step (trigger level, stage file passthrough) is more important than the business logic inside it.

**Step 6 — Call the REST API directly**\
Hit `ClassifyExtractPage` with a sample document and see the synchronous JSON response. This is the foundation for all real-time integrations. Understand the response structure before designing any LOB integration.

---

*Das*