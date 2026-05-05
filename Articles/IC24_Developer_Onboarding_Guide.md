# Migrating from Captiva 7.x to Intelligent Capture 24.x

## Table of Contents

1. [Mental Model Reset](#mental-model-reset)
2. [Internal Architecture](#internal-architecture)
3. [Pipeline Execution Model](#pipeline-execution-model)
4. [Development Model](#development-model)
5. [Integration Patterns](#integration-patterns)
6. [Recognition and AI Evolution](#recognition-and-ai-evolution)
7. [Migration Mindset](#migration-mindset)
8. [Critical Extension Points](#critical-extension-points)
9. [Deprecation Reference](#deprecation-reference)
10. [Getting Productive](#getting-productive)

---

## Mental Model Reset

The first week I spent on IC 24 was genuinely confusing. Not because the product is hard, but because I kept trying to map 7.x concepts onto it. Once I stopped doing that, everything clicked. Before anything else, there are three assumptions from 7.x that you need to consciously set aside.

### The Three Shifts

#### Shift 1: Monolithic IPP Pipelines to CaptureFlow Orchestration

In 7.x, the IPP was the backbone of everything. A compiled script that sequenced every module in a hard-coded chain. If you needed to change the order or add a branch, you edited the IPP, recompiled, and redeployed. Simple in theory and painful in practice the moment requirements changed.

In IC 24, the equivalent is the **CaptureFlow**: a directed graph of module steps authored graphically in CaptureFlow Designer. It compiles to an IAP file just like an IPP did, so the server-side mechanics are familiar. What is different is that CaptureFlow steps are dynamically parameterized at runtime via IA values, and conditional routing is just a connector expression with no scripting required.

{% hint style="info" %}
If you have Dispatcher 6.5 or 6.5 SP1 DPP files, you can import them directly into a Recognition design area in IC Designer (needs the Advanced Recognition license). Field placements on existing templates survive the import. I used this to bootstrap a migration and it saved a lot of time. From that point on it is all CaptureFlow though.
{% endhint %}

#### Shift 2: Heavy Scripting to Configuration First Development

In 7.x, the answer to almost every non-trivial requirement was a script. VBScript in the IPP, VBA in Dispatcher, .NET Quick Modules. Scripts were the primary lever for everything: routing logic, field validation, enrichment, and export. If you were a 7.x developer worth your salt, you had a library of reusable script snippets.

In IC 24, **the first question is always whether something can be done in configuration**. The answer is yes far more often than you would expect. The Designer's profiles, document type definitions, recognition projects, export profiles, and CaptureFlow connector conditions cover the vast majority of real-world requirements. Code is reserved for what genuinely cannot be expressed through configuration.

{% hint style="danger" %}
Replicating 7.x script patterns directly in IC 24 is the single most common mistake I have seen experienced 7.x developers make. You end up with solutions that work but are unnecessarily complex, hard to upgrade, and miss the architectural intent of the platform. Read the [Migration Mindset](#migration-mindset) section before writing any code.
{% endhint %}

#### Shift 3: Thick Client Tools to Web and API Driven Interfaces

In 7.x, everything happened in thick clients. Dispatcher Designer, InputAccel Administrator, recognition project editors. Operators ran thick client modules on dedicated workstations.

In IC 24, **IC Designer** is the centralized IDE for everything: profiles, document types, recognition, export, and CaptureFlows. Operators use the **IC Web Client** in a browser, or the desktop Completion and ScanPlus modules if you need the full operator toolset. Anything that used to talk to the system via custom integrations should now use the **REST Service** layer. That is not aspirational; it is how the platform is designed to be consumed.

---

### Conceptual Equivalence Map

This is the table I built on day one to stop second-guessing terminology.

| Captiva 7.x | IC 24.x | Notes |
|---|---|---|
| IPP (Integrated ProcessFlow) | **CaptureFlow** (.IAP) | Graphical, compiled, deployed to IC Server |
| Dispatcher project (.DPP) | **Recognition project** in IC Designer | Import DPP from Dispatcher 6.5/6.5 SP1 |
| InputAccel Server | **IC Server** | Identical role. Still called InputAccel Server internally in some parts of the codebase |
| Dispatcher module (client service) | **Classification module** | Requires Advanced Recognition license |
| NuanceOCR module | **NuanceOCR module** | Same module, still ships. AmpLib barcode engine was removed in 24.2 |
| Custom VB or C# IPP script | **.NET Code module step** (C# DLL) | VB.NET is on its way out. Write new code in C# |
| Process Developer tool | **CaptureFlow Designer** | Process Developer is gone in 24.x, not just renamed |
| ScaleServer group | **ScaleServer group** (SQL Server backed) | Same concept, requires external SQL Server now |
| MDF (Module Definition File) | **MDF** | Same format, unchanged |
| IA values | **IA values** | Same categories, same behavior. Dynamic IA values now support Unicode |
| Thick client operator stations | **IC Web Client** or desktop Completion/ScanPlus/Identification | |
| SOAP Web Services I/O | **WS Input / WS Output** (still ships) | Still works, but REST is the right choice for new integrations |

---

## Internal Architecture

### Component Overview

| Component | Role | 7.x Equivalent |
|---|---|---|
| **IC Server** | Central broker: manages batch lifecycle, schedules tasks, pushes tasks to production modules via TCP/IP | InputAccel Server, genuinely identical |
| **CaptureFlow Designer** | Centralized IDE: design, compile, deploy CaptureFlows, profiles, document types, recognition projects, export profiles | Dispatcher Designer plus IPP editor plus Process Developer combined |
| **Production Modules** | Client-side Windows services or attended apps. Receive tasks from IC Server, process them, return results | InputAccel client modules |
| **Module Server** | Windows service managing a pool of service module instances (classification, OCR, image conversion) for the REST subsystem. Scales by launching additional instances | No equivalent in 7.x |
| **IC REST Service** (on IIS) | JSON REST endpoint. Routes batch creation requests to the IC Server and ad hoc page requests to the Module Server | Web Services Hosting, but much more capable |
| **IC Web Client** | Browser-based operator interface. Calls IC REST Service under the hood | Desktop Completion and ScanPlus, but in a browser |
| **SQL Server DB** (optional) | Configuration, batch metadata, licensing, audit logs, ScaleServer state. Required for ScaleServer and reporting | Same optional external DB from the 7.x ScaleServer era |

---

### The Two Processing Paths

This took me a while to internalize because it has no clean analog in 7.x.

{% tabs %}
{% tab title="Path A: Async Batch via IC Server" %}
This is the traditional Captiva execution model expressed as a CaptureFlow. Everything you know from 7.x maps here.

**How it works:**
1. A batch is created by ScanPlus, Standard Import, WS Input, or a REST call
2. The batch sits on the IC Server
3. The IC Server schedules tasks and pushes them to available production module instances
4. Modules process, return results, and the server advances to the next CaptureFlow step

**When to use this path:**
- High volume, multi-stage document workflows
- Anything involving operator review such as Completion or Identification
- Any workflow needing the full batch tree structure and IA value tracking

The internal mechanics, meaning IA value triggers, stage files, and the task push model, are unchanged from 7.x. This path will feel familiar.
{% endtab %}

{% tab title="Path B: Synchronous Real-Time via REST" %}
This is the path with no 7.x equivalent. It exists to support real-time integrations from mobile apps, web apps, and line of business systems.

**How it works:**
1. External app calls the IC REST Service over HTTPS
2. **Batch request**: REST Service creates an IC batch and triggers a CaptureFlow, async, same as Path A but initiated via REST
3. **Ad hoc request**: REST Service forwards directly to the Module Server, which processes and returns results in the same HTTP call

**When to use this path:**
- Classifying and extracting a single uploaded page and returning results immediately
- Inline document processing inside a web or mobile application
- Any scenario where the caller needs a synchronous result

{% hint style="info" %}
These two paths are not mutually exclusive. A REST-initiated request can create a batch that then runs through a full CaptureFlow using Path A mechanics. The distinction is about how the batch is initiated and whether the caller needs to wait for a result.
{% endhint %}
{% endtab %}
{% endtabs %}

---

### The Batch Model

Unchanged from 7.x. A batch is a hierarchical tree of nodes with up to 8 levels.

```
Level 7  Batch (root)
Level 6  available for custom grouping
Level 5  available for custom grouping
Level 4  available for custom grouping
Level 3  available for custom grouping
Level 2  Folder (in recognition)
Level 1  Document
Level 0  Page (leaf)
```

State is tracked via **IA values** on nodes. **Stage files** (images, OCR output, data files) are stored on the IC Server associated with nodes. The server uses **trigger IA values** to determine when a node is ready for the next step, which is the same push model from 7.x.

{% hint style="info" %}
The batch folder on disk is still structured as `Batches\XXXX\YYY\ZZZ` based on batch ID split 4-3-3. Stage file extensions are still sequential hex numbers per node, for example `23e.1` and `23e.2`. If you are debugging a batch file issue directly on the server, this knowledge from 7.x still applies.
{% endhint %}

---

### IA Values

The same six categories with the same behavior. Listed here for completeness.

| Category | Purpose | Classic Example |
|---|---|---|
| **Input/Output file values** | Pointers to stage files; connect module steps | `InputImage`, `OutputImage` |
| **Trigger values** | Subset of input file values; when all are non-zero the module can process the task | `InputImage` (most common trigger) |
| **Setup values** | Per-task configuration pushed to the module; can differ per task | OCR language, scanner settings, index field definitions |
| **Processing results and statistics** | Output metadata from each module | Timestamps, operator name, OCR results, error codes |
| **Batch values** | Dynamic values scoped to batch nodes, created at runtime | Custom counters, flags set mid-pipeline |
| **System values** | Global, not batch scoped | `$user`, `$module`, `$machine`, `$server` |

The reserved values `IATaskRouting` and `IADepartments` still control department-based routing. This is the IC 24 equivalent of the Dispatcher routing rules you were setting in 7.x. You can set them statically in a CaptureFlow step or dynamically from code.

---

## Pipeline Execution Model

### What the IPP Model Was

{% tabs %}
{% tab title="7.x IPP Characteristics" %}
- **Linear by default.** Branching required explicit IA value conditionals written in script.
- **Stage-based with fixed points.** Changing step order meant editing and recompiling the IPP.
- **No true graphical authoring.** Process Developer was a view onto the IPP, not a design tool.
- **Dispatcher owned classification routing.** The DPP was a separate artefact with its own VBA scripting environment. Classification logic and routing logic were intertwined.
- **Script was the extension mechanism.** Every non-trivial customization lived in IPP scripts or Dispatcher VBA.
{% endtab %}
{% endtabs %}

### What CaptureFlow Replaced It With

{% tabs %}
{% tab title="CaptureFlow Characteristics" %}
- **Graphical first.** Steps are connected visually. Conditional routing is configured via IA value expressions on connectors with no script required for branching.
- **Configuration driven parameterization.** Profiles, document types, and recognition projects are reusable components uploaded separately. A step references them by name. Swap a profile without touching the CaptureFlow.
- **Dynamic routing via IA values.** Set `IATaskRouting` or `IADepartments` in a preceding step to route tasks to specific module instances or departments.
- **Process versioning.** The server maintains numbered version folders. Multiple versions of a process coexist. Batches are pinned to the version that created them.
- **Attended and unattended steps.** Unattended steps run as Windows services. Attended steps like Completion, Identification, and ScanPlus require a human operator.
{% endtab %}
{% endtabs %}

---

### Where Routing Logic Lives

This is the most practically important table in this guide. I follow this order on every project.

| Priority | Mechanism | When to Reach For It |
|---|---|---|
| **1st** | **CaptureFlow connector conditions** | IA value expressions on step transitions. No code, no compile. Use this first, always. |
| **2nd** | **CaptureFlow step configuration** | Profiles control processing rules; document types enforce validation and field constraints |
| **3rd** | **Recognition project rules** | Classification thresholds, extraction zones, document separation logic |
| **4th** | **Client-side scripts in Completion or ScanPlus** | UI-level validation and field population specific to a document type |
| **5th** | **.NET Code module step** | Logic that genuinely cannot be expressed by any of the above, such as external calls or batch tree restructuring |
| **6th** | **REST API integration** | External system lookups, validation, enrichment from outside the capture pipeline |

{% hint style="warning" %}
Every time you instinctively reach for a `.NET Code` step, stop and ask whether the condition can be expressed as a CaptureFlow connector expression. The connector expression evaluator supports IA value comparisons, string operations, and boolean logic. It handles more than you would expect.
{% endhint %}

---

## Development Model

This is the section I would have most wanted to read on day one.

### The Configuration Layer

The IC Designer has design areas, each producing a **reusable service component** that gets uploaded to the server independently of any CaptureFlow.

| Design Area | What You Build | 7.x Equivalent |
|---|---|---|
| **Image Processing** | Profiles: enhancement filters, blank page detection, deskew, barcodes, annotations | Image Processor module setup |
| **Image Conversion** | Profiles: format conversion such as TIFF to PDF, splitting, merging, color conversion | Image Converter module setup |
| **Standard OCR** | Profiles: zonal and full-page OCR, OCR cache, PDF and text output | NuanceOCR zones and settings |
| **Recognition** | Recognition projects: templates, classification logic, extraction zones, document separation. Import DPP from Dispatcher 6.5/6.5 SP1 | Dispatcher project (.DPP), direct equivalent |
| **Document Types** | Index field definitions, validation rules, data entry form layout, field controls | Completion field setup plus Dispatcher index families |
| **Export** | Export profiles: CSV, XML, free text, email, CMIS, Content Server, AppEnhancer, Documentum, ODBC | Standard Export plus Enterprise Export modules |
| **CaptureFlow Designer** | The process model itself: step graph, IA value wiring, conditional routing | Process Developer plus IPP editor |

The key insight is that these components are **reusable across CaptureFlows**. One Standard OCR profile shared by five CaptureFlows means one place to update OCR settings. Design them to be reused from the start.

---

### The Four Scripting Contexts

This is where I see the most confusion from 7.x developers. There are four distinct scripting contexts, each with different scope and available APIs. Mixing them up causes bugs that are genuinely hard to diagnose.

{% tabs %}
{% tab title="Profile Scripting" %}
**Scope:** Document types and page-level image enhancements. Completely decoupled from any specific process, batch, task, or IA values.

- Do NOT access IA values, task objects, or process state
- Use for field-level validation formulas, custom field controls, image annotation logic
- Language: **C#**

Think of profile scripts as reusable library functions. They have no idea what CaptureFlow is running them.
{% endtab %}

{% tab title="Task Scripting" %}
**Scope:** Task and batch node manipulation. Has access to IA values and the batch tree. Can call profile script APIs.

- Do NOT use profile scripting events or UI-related APIs
- Use for batch-level IA value manipulation, node restructuring, routing decisions based on extracted data
- Language: **C#**

This is the context that maps most closely to what you were doing in IPP scripts in 7.x.
{% endtab %}

{% tab title="Client-Side Scripting" %}
**Scope:** Runs as part of a module step inside a CaptureFlow, triggered by module-defined events. Does NOT need its own CaptureFlow step as it embeds into an existing one.

- Use for operator UI event handlers, dynamic field visibility, validation popups, external lookups triggered during Completion
- Language: **C#**. If your CaptureFlow does not already have VB.NET scripting, new scripts are C# only.

This is the closest equivalent to the Completion event scripts you were writing in 7.x.
{% endtab %}

{% tab title="Recognition Scripting" %}
**Scope:** Customizing Classification and Identification recognition projects. Uses a VBA-compatible environment with its own editor and debugger.

- Use for custom classification thresholds, extraction post-processing, template logic
- Language: **VBA**

This is the scripting context closest to the old Dispatcher VBA environment. If you are coming from Dispatcher, this one will feel familiar.
{% endtab %}
{% endtabs %}

---

### The .NET Code Module Step

The `.NET Code` module runs a custom C# DLL as an independent step in a CaptureFlow. It exposes a .NET interface to read and write batch data.

```csharp
// Minimal entry point, implement IModule from InputAccel35.dll
using InputAccel.QuickModule;
using InputAccel.IAValues;

public class MyModule : IModule
{
    public void ProcessTask(IModuleTask task)
    {
        // Read IA value from the batch node
        string custId = task.GetIAValue("CustomerID");

        // Call external system, transform data, etc.
        string custName = LookupCustomer(custId);
        task.SetIAValue("CustomerName", custName);

        // Pass the stage file through to the next step
        task.SetOutputImage(task.GetInputImage());
    }
}
```

| Requirement | What to Use |
|---|---|
| Target framework | **.NET 4.8** or higher |
| SDK DLL | **`InputAccel.*35.dll`** and not `InputAccel.*.dll` (old SDK) or `InputAccel.QuickModule.dll` (SDK 5.x .NET 1.1) |
| IDE | Visual Studio 2013 or later, Professional or Standard. Express works but debugging is limited. |
| Deployment | Compile DLL, upload in IC Designer, CaptureFlow step references it by name |

{% hint style="warning" %}
**SDK migration: the three cases you will encounter**

- **`IAClient32.dll` (COM, SDK 5.x)** cannot run on 24.x. Full rewrite as a `.NET Code` module step.
- **SDK 5.x .NET 1.1 (`InputAccel.QuickModule.dll`)**: replace the .NET 1.1 dependency with .NET 4.8 and rebuild against `InputAccel.*35.dll`.
- **IC 7.0 through 7.7 (.NET 3.5 or .NET 4.5.2)** may run as-is under .NET 4.8. Test with `QuickModuleHost.exe -modulename:<yourmodule>` before committing to a rewrite. I have had these work without changes more often than I expected.
{% endhint %}

---

### Config vs Code vs REST

| Scenario | Right Approach | Wrong Approach |
|---|---|---|
| Route documents by document type | CaptureFlow connector condition on classification result IA value | Script to read IA value and set routing flags |
| Validate a field against a regex pattern | Document type field property: regular expression constraint | Client-side script with regex logic |
| Populate fields from external DB during indexing | Client-side script in Completion step OR .NET Code step | Custom module that opens a DB connection during task processing |
| Export to a CMIS repository | Standard Export with CMIS profile | Building a custom export module from scratch |
| Complex batch tree restructuring | Multi module (built-in) or .NET Code module | Client-side script (wrong scope) |
| Classify and extract a single page in real time | REST API `ClassifyExtractPage` on Module Server | Full async batch submitted to a CaptureFlow |
| Custom module with hardware integration | .NET Code module with `InputAccel35.dll` | COM module (`IAClient32.dll`), not supported |

---

## Integration Patterns

### REST-Based Batch Ingestion

External systems submit documents to IC 24 by calling the IC REST Service.

```http
POST /icws/rest/batch
Content-Type: multipart/form-data

Body: process name, document files, IA values to pre-populate
```

The caller gets back a batch ID. The CaptureFlow runs asynchronously. The caller polls for status or relies on event-based notification if configured.

This replaces WS Input for new integrations. WS Input (SOAP) still ships and still works. Keep it if you have legacy systems that cannot talk REST, but do not build new integrations on it.

---

### Ad Hoc Page Processing via Module Server

For real-time, synchronous classification and extraction without creating a persistent batch:

```http
POST /icws/rest/services/classify
POST /icws/rest/services/classifyextractpage
POST /icws/rest/services/classifyextractdocument
```

Send an image or PDF page and get back structured results in the same HTTP response. This is what you use for mobile capture apps and inline document processing inside line of business systems.

{% hint style="info" %}
24.2 added `defaultDocumentType` and `AllowedDocumentType` parameters to the Classify and ClassifyExtract endpoints. Use these when you already know the document type family from context. It constrains the classification result and improves accuracy.
{% endhint %}

---

### External Validation and Enrichment During Capture

The most common integration question from teams migrating from 7.x is how to look up data from an external system during indexing. Here are the three patterns.

{% tabs %}
{% tab title="Pattern A: Client-Side Script" %}
A C# client-side script attached to the `OnValidate` event or a button in the Completion step. The script makes an HTTP call and populates field values or shows an error.

**Use when:** The lookup is triggered by an operator action and the result needs to be shown before the operator commits.

**Avoid:** Long-running calls. Anything that blocks the Completion UI for more than a couple of seconds will frustrate operators.
{% endtab %}

{% tab title="Pattern B: .NET Code Step" %}
A `.NET Code` module step in the CaptureFlow, positioned before Completion. It reads IA values, calls the external system, and writes results back as IA values for Completion to display.

**Use when:** Enrichment should happen automatically before the operator sees the document. No operator action should be required to trigger it.

My most common use case: extract a vendor ID via OCR, look up vendor details in the ERP before the document reaches Completion, so the operator sees a pre-populated form.
{% endtab %}

{% tab title="Pattern C: WS Output and Input" %}
Use `WS Output` to call an external web service mid-pipeline and `WS Input` to receive the response.

**Use when:** The external interaction needs to be a first-class step in the CaptureFlow for audit or compliance reasons, or when the interaction has its own retry requirements that you need to model explicitly.
{% endtab %}
{% endtabs %}

---

### ERP Lookup During Indexing

A concrete example I have implemented a few times. After OCR extraction, look up a customer number in an ERP system and populate additional fields before Completion.

1. In the CaptureFlow, insert a `.NET Code` step between the Extraction step and the Completion step.
2. In the `.NET Code` assembly: read the `CustomerNumber` IA value, call the ERP REST API, parse the response, then write `CustomerName`, `CustomerAddress`, and `AccountType` back as IA values.
3. In the Completion document type: mark `CustomerName`, `CustomerAddress`, and `AccountType` as read-only display fields. They pre-populate from IA values when the operator opens the batch.
4. On the CaptureFlow connector after the `.NET Code` step: add a condition so that if `CustomerNumber` is empty, the batch routes to an exception handling step instead of proceeding to Completion. Do not let bad data silently reach the operator.

---

### Custom Export

Standard Export with a built-in profile covers most targets: CMIS, Content Server, AppEnhancer, Documentum, SharePoint, and ODBC. When none of those fit:

| Approach | When to Use |
|---|---|
| **WS Output** | Target accepts SOAP or REST and you want the interaction visible as a CaptureFlow step |
| **.NET Code step at batch or document level** | Full control over export logic: read stage files and IA values, build the payload, call the target API |
| **Standard Export free-text or XML then .NET Code** | Target accepts file-based import; generate the import file via Standard Export then trigger the import from a `.NET Code` step |

{% hint style="danger" %}
**ApplicationXtender Export** and **LiveLink Export to Content Server** are both deprecated in 24.2. If you are inheriting a solution that uses either of these, migrate to Standard Export with the AppEnhancer profile or the Content Server profile respectively. Do not wait for a future release to break it.
{% endhint %}

---

## Recognition and AI Evolution

In 7.x, recognition was entirely template-based. Train templates against sample images, define extraction zones, and Dispatcher classifies by graphical and textual similarity scoring. Improving accuracy meant adding templates or manually tuning zone coordinates. I spent more time than I would like to admit on pixel-level zone adjustments.

IC 24 retains that template-based approach and adds **IEE (Information Extraction Engine)**: ML-assisted classification and extraction that learns from corrections.

### IEE vs Template-Based

| | Template-Based | IEE |
|---|---|---|
| **Classification** | Graphical similarity against template images | Keyword, image, and text analysis combined |
| **Extraction** | Fixed pixel zones per template | Learned positions with no zone definitions required |
| **Improving accuracy** | Manual: add templates, tune zone coordinates | Automatic: learns from operator corrections in Completion |
| **Best for** | Highly structured fixed-format documents such as tax forms or invoices with identical layouts | Variable-format documents such as invoices from many different vendors or mixed correspondence |
| **Initial setup effort** | High | Low |
| **Ongoing effort** | Periodic manual tuning | Set up the learning pipeline once, then mostly hands-off |

My rule of thumb: if every document of a type looks identical, use template-based. If documents vary in layout across sources, use IEE.

---

### The IEE Learning Loop

This is the part that catches people off guard. IEE does not automatically improve just because operators are correcting documents in Completion. You have to explicitly configure the learning pipeline.

```
[Extraction/Classification step]
         |
         | (low-confidence result)
         v
[Collector module step]        <-- tags document as collectable
         |
         v
[Completion step]              <-- operator corrects extraction
         |
         v
[Production Auto-Learning Supervisor]  <-- scheduled service; creates new templates, updates field positions
```

As a developer your job is to:

1. Configure the IEE profile in Designer: training set, document type mappings, confidence thresholds.
2. Add the Collector step in the CaptureFlow on the low-confidence path.
3. Configure the Production Auto-Learning Supervisor service to run on a schedule.
4. Not define extraction zones for IEE document types. Zones are learned, not configured.

{% hint style="info" %}
24.2 added the **ResetIEETool** for resetting IEE learning model data: classification learning, extraction learning per document type, and document separation learning. I have used this when migrating an existing IEE setup where old training data was making the model worse rather than better. A fresh start with clean corrections is usually the right call.
{% endhint %}

{% hint style="warning" %}
IEE profile names are case-sensitive. I have spent embarrassing amounts of time debugging what turned out to be a casing mismatch between the profile name in Designer and the reference in a CaptureFlow step. Be consistent from the start.
{% endhint %}

---

## Migration Mindset

This section is blunt. These are the patterns that trip up 7.x developers on IC 24, and I have made most of them myself.

### Stop Doing These Things

**Stop reaching for scripts before checking configuration**

Every module step has a profile. Every profile has configurable options. Every document type has field-level validation rules, regex constraints, lookup tables, and auto-population formulas. If you were a 7.x developer, your instinct is to script. Suppress that instinct until you have spent ten minutes in the relevant Designer area looking for a configuration option. Nine times out of ten you will find one.

**Stop building rigid, linear CaptureFlows**

IC 24 CaptureFlows are directed graphs, not sequences. Do not replicate a linear 7.x pipeline just because it is familiar. Every real workflow has edge cases: what if classification confidence is below threshold? What if a required field is empty? What if this document type needs a different export target? Model those as branches from the beginning. Adding them later is painful.

**Stop using VB.NET for new scripting**

VB.NET CaptureFlow scripting is on the deprecation path. New CaptureFlows without existing VB.NET scripts only create new scripts in C#. Do not invest in new VB.NET logic. If you have old VB.NET scripts, convert them to C#. Visual Studio can do most of this automatically.

**Stop trying to reconstruct the Dispatcher architecture**

Dispatcher combined classification, extraction, routing, and scripting into one VBA monolith. In IC 24, these concerns are intentionally separated: recognition projects own classification and extraction, CaptureFlow owns routing, and scripts own custom logic. Trying to consolidate everything back into recognition scripting leads to exactly the kinds of unmaintainable solutions that made Dispatcher projects painful to hand off.

---

### Start Doing These Things

**Design flows, not pipelines**

Design from the business process outward. What are the possible document types coming in? What happens when classification confidence is low? What happens when validation fails? What is the exception path? Map all of that as branches before you write a single line of code. The CaptureFlow diagram is the documentation for how your solution works.

**Use REST as a first-class integration point**

Any external system that needs to trigger capture, query batch status, or receive processed results should talk to the IC REST API. That is not just a preference. It is the strategic direction of the platform. Build new integrations on REST.

**Think in service components**

Profiles, document types, and recognition projects are reusable components deployed independently of CaptureFlows. Treat them like shared libraries. One well-designed Standard OCR profile used by five CaptureFlows is much easier to maintain than five slightly different copies. The Designer makes this easy.

---

### Common Pitfalls

| Pitfall | Symptom | Fix |
|---|---|---|
| Duplicating profile config per CaptureFlow step | A settings change requires updating five different steps | Create shared profiles in Designer and reference the same profile from multiple steps |
| Writing .NET Code for routing logic | Business logic is buried in DLL source and the CaptureFlow diagram is unreadable | Use CaptureFlow connector conditions. Reserve .NET Code for side-effecting operations like DB calls, file I/O, and external API calls |
| Not configuring the IEE learning pipeline | IEE accuracy flatlines despite operator corrections | Set up the Collector step and Auto-Learning Supervisor. Corrections only feed the model if the collection pipeline exists |
| Using deprecated export modules | Solution breaks silently in a future upgrade | Migrate ApplicationXtender to Standard Export plus AppEnhancer profile. Migrate ODBC Export module to Standard Export plus ODBC profile |
| Building modules against `IAClient32.dll` | Module fails to load on 24.x client machines | Rewrite as .NET Code module with `InputAccel35.dll` |
| Using the file-based internal database in production | ScaleServer, audit logging, and Web Services all fail | Use SQL Server external DB for production. File-based DB is for demos and local testing only |

---

## Critical Extension Points

### .NET Code Module Interface

```csharp
// Key namespaces, reference InputAccel35.dll
using InputAccel.QuickModule;  // IModule, IModuleTask
using InputAccel.IAValues;     // IA value read/write
using InputAccel.Batch;        // Batch node navigation

// Reading and writing IA values
string value = task.GetIAValue("FieldName");
task.SetIAValue("FieldName", "NewValue");

// Getting the input stage file and passing it through
string inputStagePath = task.GetInputImage();
task.SetOutputImage(inputStagePath);

// Iterating child nodes when triggered at document level (level 1)
foreach (IBatchNode page in task.CurrentNode.ChildNodes)
{
    string pageStageFile = page.GetIAValue("OutputImage");
    // process each page
}
```

**Deployment steps:**
1. Compile DLL targeting .NET 4.8 and referencing `InputAccel.*35.dll`
2. In IC Designer: upload DLL to the CaptureFlow step
3. The step triggers at the level you set: level 7 for one call per batch, level 1 for one call per document, level 0 for one call per page

---

### REST Endpoints You Will Actually Use

| Endpoint | What It Does |
|---|---|
| `POST /icws/rest/batch` | Create a batch and trigger a CaptureFlow (async) |
| `GET /icws/rest/batch/{batchId}` | Poll batch processing status |
| `POST /icws/rest/services/classify` | Synchronously classify a page against a recognition project |
| `POST /icws/rest/services/classifyextractpage` | Synchronously classify and extract fields from a single page |
| `POST /icws/rest/services/classifyextractdocument` | Synchronously classify and extract a multi-page document |

---

### External Call from a Client-Side Script

```csharp
// C# client-side script in a Completion step, attached to a field validation event
using System.Net.Http;
using System.Threading.Tasks;

private async Task<string> LookupVendor(string vendorId)
{
    using var client = new HttpClient();
    var response = await client.GetAsync(
        $"https://erp.internal/api/vendor/{vendorId}");
    return await response.Content.ReadAsStringAsync();
}

// In your event handler:
// 1. Read the field value from the Completion form
// 2. Call LookupVendor()
// 3. Parse the JSON response
// 4. Populate additional fields or flag a validation error
```

---

### Export Hook via .NET Code Step

```csharp
// .NET Code step triggered at document level (level 1)
// One execution per document in the batch

public void ProcessTask(IModuleTask task)
{
    string imagePath     = task.GetInputImage();
    string invoiceNumber = task.GetIAValue("InvoiceNumber");
    string vendorId      = task.GetIAValue("VendorID");
    string totalAmount   = task.GetIAValue("TotalAmount");

    PostToTargetSystem(imagePath, invoiceNumber, vendorId, totalAmount);

    task.SetOutputImage(imagePath);
}
```

---

## Deprecation Reference

Things to migrate now, not when the next release breaks you.

### Already Deprecated or Removed in 24.2

| Item | Status | What to Do |
|---|---|---|
| **Process Developer** | No longer shipped | Migrate processes to CaptureFlow Designer |
| **`IAClient32.dll` (COM, SDK 5.x)** | Not supported on 24.x | Rewrite as .NET Code module using `InputAccel35.dll` |
| **SDK 5.x .NET 1.1** | Must migrate | Replace with .NET 4.8 and rebuild |
| **VB.NET CaptureFlow scripting** | Deprecated, planned removal | Convert to C# |
| **ApplicationXtender Export module** | Deprecated | Use Standard Export plus AppEnhancer profile |
| **Export to Content Server (LiveLink)** | Deprecated | Use Standard Export plus Content Server profile |
| **Copy module** | Deprecated | Use Batch Copy module |
| **East Euro and APAC OCR engine** | No longer shipped | Use Advanced OCR/ICR or Standard OCR module |
| **AmpLib barcode recognition** | Removed in 24.2 | Use other supported barcode methods |

### Coming in a Future Release

| Item | Replacement |
|---|---|
| **ODBC Export module** | Standard Export plus ODBC profile |
| **Documentum Exporter module** | Standard Export plus Documentum profile (requires REST-enabled Documentum) |

---

### Platform Requirements for 24.2

| Component | Requirement |
|---|---|
| IC Server OS | Windows Server 2022, 2019, or 2016 (64-bit) |
| SQL Server (required for ScaleServer and reporting) | SQL Server 2014 SP3 through 2022 |
| Client modules OS | Windows 10 Enterprise (1803 or later), Windows 11 Enterprise, or Windows Server 2022/2019/2016 |
| .NET Framework | 4.8 or higher everywhere |
| IC Designer .NET requirement | .NET 4.8 Developer Pack only. No other .NET version on the Designer machine. |
| Web Client browsers | Chrome 94 or later, Firefox 94 or later, Microsoft Edge Chromium |
| IIS for REST Service | IIS 7.5 through IIS 10 |
| ScaleServer | Up to 8 servers, requires external SQL Server |

---

## Getting Productive

The order matters here. Each step validates the previous one.

**Step 1: Walk through the Designer Quick Start**

Install IC Designer and build the Quick Start simple process. Half a day. Do not skip this even if you are experienced. The tooling has changed significantly from Process Developer.

**Step 2: Run the minimal CaptureFlow end to end**

`Standard Import > Standard OCR > Standard Export`. Create a test batch, run it, check the output. This validates your environment before you build anything real on top of it.

**Step 3: Import your existing Dispatcher project**

If you have DPP files from Dispatcher 6.5 or 6.5 SP1, import them into a Recognition design area. Validate that templates and field placements survived the import before building anything that depends on them.

**Step 4: Replace one 7.x script with configuration**

Pick the simplest custom script or module from your existing solution. Determine whether its logic can be replaced by a CaptureFlow connector condition, a profile setting, or a document type rule. Usually it can. This exercise resets your instincts.

**Step 5: Write a minimal .NET Code step**

Read one IA value, call one external HTTP endpoint, write the result back to a different IA value. Start small. The runtime behavior of the step, meaning trigger level and stage file passthrough, is more important to understand than the business logic inside it.

**Step 6: Call the REST API directly**

Hit `ClassifyExtractPage` with a sample document and look at the JSON response. Do this before designing any real-time integration. Understanding the response structure early saves a lot of backtracking later.