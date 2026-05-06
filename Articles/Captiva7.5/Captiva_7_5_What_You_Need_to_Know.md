# Captiva 7.5: What Actually Changed and Why You Should Care

## The Setup

Captiva 7.5 arrived in June 2015, and like most point releases, it promises "new features" and "modern capabilities." The reality is more nuanced. Some of these changes are genuinely architectural shifts. Others are branding exercises. As a developer, you need to separate the two and understand which decisions actually affect your work.

This article walks through the significant changes from the perspective of someone who has to build and deploy on this platform.

---

## Module Reorganization: It Is Not What It Sounds Like

The marketing announcement was that "Captiva Desktop" became two modules: Completion and Identification. This sounds like a split. It is not. It is a reorganization of what already existed, wrapped in a rebranding story.

### What Happened

In earlier versions, Captiva had a thick client application called "Captiva Desktop" that handled both manual document classification and data entry validation. Two separate workflows, one interface.

In 7.5, this thick client is being rebranded. The data entry portion becomes "Completion." The classification portion becomes "Identification." They share the same underlying Desktop Family architecture but are now distinct modules in the licensing and configuration model.

### Why This Matters

The functional implication is minimal for most deployments. Your CaptureFlows do not change. Your batches do not change. What changes is the licensing line item and how you configure these steps in the CaptureFlow designer.

But here is where it matters: the old names in documentation, scripts, and configuration files are now deprecated. If you have legacy processes or custom integrations that reference "Captiva Desktop" by name, you need to audit them. The thick client will continue to work, but new development and documentation will reference Completion and Identification.

If you are deploying 7.5 in a brownfield environment with existing Captiva installations, prepare for configuration drift if teams do not coordinate on which naming convention they are using.

---

## Standard Import: The Quiet Workhorse

Two older modules are being retired: Multi-Directory Watch (MDW) and Email Import. Their replacement is a single new module called Standard Import.

### Why This Consolidation Makes Sense

MDW and Email Import were separate code paths. Developers had to learn two different configuration approaches for essentially the same workflow (import batches from a file system or email). Standard Import unifies this.

From a maintenance perspective, consolidating two modules into one is the right move. Less code to maintain, fewer integration points, simpler configuration surface.

### What Changed Practically

Standard Import is not a one to one feature replacement. Some feature combinations that were possible with the old modules may not map cleanly to the new one. OpenText explicitly states this: "The new modules are intended to provide functional equivalence but may not necessarily provide a one to one mapping of features."

Translation: you may need to adjust how you are importing batches. It is probably simpler and more maintainable in the end, but expect to spend time validating your specific import scenarios.

If you are currently using MDW or Email Import, you should test Standard Import against your actual batch import workflows before upgrading production systems. Do not assume it is a drop in replacement.

---

## The Identification Module Is Not Classification

This is a subtle but important distinction. The new Identification module replaces the old "Classification Edit" workflow inside the thick client. It is for manual document type assignment when automatic classification failed or was not configured.

It is not a replacement for the Advanced Recognition classification module. Do not confuse the two.

The Advanced Recognition classification engine (the one that actually recognizes document types automatically) is unchanged in 7.5. Identification is the UI and workflow for handling the exceptions when automatic classification does not work.

---

## The Web Client: A Real Shift in Architecture

The new Captiva Web Client is positioned as a replacement for the thick client operator tools. But it is not a one to one swap, and the licensing reflects that.

### What It Does

The Web Client is a browser based interface for:
- Scanning documents
- Indexing batches (entering metadata)
- Validating extracted data (if Advanced Recognition is available)

It requires no browser plugins. It supports Internet Explorer 10+, Chrome 32+, and Firefox 26+. These are modern standards for 2015.

### What It Requires

Here is the critical part: the Web Client is not just software. It is an architectural component that requires the REST Services layer on the server side.

If you are running Captiva Enterprise Server, REST Services for batch submission are included at no cost. If you are on Captiva Standard Server, you must upgrade to Enterprise level first.

This licensing boundary is not arbitrary. The Web Client internally uses REST to submit batches to the server. Standard Server does not have this capability. You cannot bolt the Web Client onto a Standard Server without upgrading the server license.

### eInput Customers: No Automatic Upgrade Path

If you are coming from Captiva with eInput (the older web interface), the Web Client is better. But it is not a replacement that your existing eInput license covers.

eInput licenses do not automatically convert to Web Client licenses. You need to discuss with your account manager. There may be a discount model available, but it involves a renegotiation.

This is a business decision, not a technical one, but it affects your upgrade costs. Plan for it.

### One More Thing: Operator Scope

The Web Client is a front end for batch submission and data validation. It is not an administrative interface. You cannot manage processes, deploy CaptureFlows, or configure profiles from the Web Client. These remain administrator functions in the thick client or designer tools.

---

## REST Services: The Real Architectural Change

If you skipped the previous sections, pay attention here. REST Services is where the platform is actually moving.

### What Changed

In earlier Captiva versions, all programmatic integration happened through SOAP Web Services or thick client COM objects. Both approaches have friction. REST is the modern alternative.

Captiva 7.5 introduces a REST Services layer that allows you to:
- Submit batches asynchronously (same as SOAP)
- Submit ad hoc document requests synchronously and get results back in the same HTTP call

The synchronous path is new. It enables real time document processing for mobile apps, web frontends, and line of business integrations.

### How REST Services Relates to Web Client

This is worth clarifying because the documentation muddies it. REST Services and the Web Client are separate components, but they work together.

The Web Client internally calls REST Services to submit batches. So if you want the Web Client, you implicitly want REST Services. But you can have REST Services without the Web Client. REST Services is valuable even if all your operators are using thick clients.

### Licensing Model

REST Services for batch submission is now included with Captiva Enterprise Server at no cost. If you are on Standard Server, you must upgrade to Enterprise to get REST Services.

This is the same boundary I mentioned for the Web Client. Enterprise Server = REST Services included. Standard Server = must upgrade.

If you want to add REST to an existing Captiva Enterprise deployment, you order SKU 456-110-150 (Captiva License Refresh).

### Real Time Processing

The REST Services layer also enables a different execution model called "Captiva Real-Time Services." This is for synchronous document processing outside of the batch pipeline.

Example: a mobile app scans a document and needs classification and data extraction results immediately. Real-Time Services can process the page and return results in the same HTTP request. This is fundamentally different from the batch pipeline where work sits in queues.

Real-Time Services is separately licensed. The pricing is higher because the value proposition is different. Real-Time is about immediate responsiveness. Batch is about throughput and operator review.

They use overlapping configuration (profiles, document types, recognition projects) but separate page counters and licensing. You cannot transfer pages from one system to the other.

### Scaling REST Services

Here is something practical: you can run as many module service instances as you want. If one license supports one instance, you are still restricted to one instance per license.

However, the licensing is per environment, not per instance. Buy one Real-Time Services license for production, one for test, one for development, and one for disaster recovery. Each gets a separate license. Within each environment, you can spin up as many module instances as your infrastructure supports.

For most deployments, you need one production system, one test system, one development system, and one DR system. Four licenses minimum. If you are deploying multiple production systems geographically (to reduce latency), each site needs its own license.

The scaling is horizontal: add more servers, add more module instances, they all point to the same configuration file share. It is simple and works well.

---

## Migration Path: The Two Route Story

If you are running Captiva 6.x and considering 7.5, there is a specific recommendation: do not upgrade directly from 6.x to 7.5. Go through 7.0/7.1 first.

### Why the Intermediate Step

Captiva 7.0 introduced the CaptureFlow Designer and major architectural changes. Processes written for 6.x may need modification for 7.0. More importantly, the deprecated modules in 7.0 (IndexPlus, Validation, Image Enhancement) need to be addressed before moving to 7.5.

OpenText deprecated these modules in late 2012 as part of the 7.0 announcement. They are not supported in 7.5. They are still available in 7.0/7.1 for backward compatibility, but you need to migrate to their replacements.

If you jump directly from 6.x to 7.5 and you are using any of these deprecated modules, you will have broken processes. Not a fun discovery after upgrade.

### What To Expect

The 6.x to 7.0/7.1 migration requires some work because of the architectural changes. But once you are on 7.0/7.1, the jump to 7.5 is minimal. Most of the heavy lifting is done in the first upgrade.

### Licensing for Older Versions

If you are upgrading from Captiva 6.x or 7.x, you do not need a new license. Your existing license continues to work, and the server automatically enables the new modules (Standard Import, Identification).

If you are on Captiva 5.3 or earlier, you need to request an updated license and submit a migration request to the vendor. This is a business discussion, not a technical one, but it is worth knowing upfront.

---

## Deprecated Modules and EOL Tracking

Captiva 7.5 removes support for:
- IndexPlus
- Validation
- Image Enhancement
- Multi-Directory Watch
- Email Import

These modules are genuinely gone, not just deprecated. If your processes depend on them, they will fail. There is no backward compatibility.

These modules were deprecated in 7.0. By 7.5, they are removed entirely. If you are still using them in production, you are technically not on a supported version anyway, but 7.5 will make this impossible to ignore.

### What To Migrate To

Each deprecated module has a replacement, though the feature mapping is not always one to one:
- IndexPlus and Validation: use standard CaptureFlow validation and the Completion module
- Image Enhancement: use Image Processing profiles in CaptureFlow
- MDW and Email Import: use Standard Import

---

## Process Developer vs. CaptureFlow Designer

Process Developer is still supported in 7.5. It is not deprecated. But here is the catch: new processes created in Process Developer are VBA-based, and Microsoft has officially ended support for the VBA runtime.

This is not an OpenText decision. This is Microsoft saying "we are done with VBA." OpenText is strongly encouraging developers to migrate new processes to CaptureFlow Designer (which uses C#) as soon as possible.

Process Developer will continue to work for existing processes. But new development should be in CaptureFlow Designer. This is not a hard requirement in 7.5, but it is a clear signal about the platform direction.

---

## Advanced Recognition and OCR Engines

One nice improvement in 7.5: the OCR engine is now decoupled from the extraction logic.

This means you can swap out OCR engines without rebuilding your recognition projects. For customers with high volume recognition workloads, this is valuable. You can test different OCR engines against your document set and choose the best one without reconfiguring everything.

The Advanced Recognition OEM API is not included in 7.5. It was last updated in 7.0 and is on an end of life path. If you need OEM style integration, REST Services is the replacement. It is more capable and well supported.

---

## Reporting: Crystal Reports Is Gone

Reporting changes in 7.5 because OpenText is removing Crystal Reports support across all products by end of 2015. This is a corporate mandate from above, not a Captiva specific decision.

What remains: reporting rules and data storage in SQL Server are unchanged. The standard reports that shipped with 7.1 are included in 7.5. You can download the free Crystal Reports runtime from SAP if you want to run those reports.

Alternatively, you can export the data from SQL and build reports using any tool of your choice. Many customers are moving to Power BI or other modern BI platforms anyway. This is a good time to evaluate what you actually need from reporting.

---

## 64-Bit Improvements

Captiva Server is now 64-bit, which provides performance improvements for high-volume workloads. All client modules remain 32-bit because they have 32-bit dependencies. This is a reasonable tradeoff: the bottleneck is usually on the server side anyway.

---

## The Practical Takeaway

Captiva 7.5 is a solid incremental release. The major architectural shift was 7.0. By 7.5, the platform is consolidating and stabilizing:
- Deprecated modules are being removed (less legacy code to maintain)
- REST Services are becoming the standard for integration (better than SOAP)
- CaptureFlow Designer is the future for process development (better than Process Developer)
- Web Client enables browser-based operations (no plugins required)
- Real-Time Services enable synchronous processing (new capability)

If you are planning an upgrade, the path is clear: 6.x users go through 7.0/7.1 first. 7.x users jump to 7.5. Prepare for licensing discussions if you are on Standard Server (you will need Enterprise). Plan your REST Services and Real-Time Services strategy upfront.

The biggest gotcha is assuming the Web Client and REST Services are automatic. They are not. You may need server upgrades and licensing changes to enable them.

Beyond that, 7.5 is straightforward. Spend the time on the upgrade path, not on the release itself.
