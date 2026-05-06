# What's New in Captiva 7.5

Captiva 7.5 is built around the concept of "capture at the edge," which is really about breaking
the batch processing model. The goal is to move capture out of back-office mailrooms and into
real-time, front-line applications where users expect immediate results. This means mobile apps,
line-of-business workflows, and customer-facing processes that cannot wait for batch queues.

The platform achieves this through two major additions: Real-Time Capture Services (REST-based)
and Captiva Web Client (browser-based). But these are not just UI improvements. They represent
a fundamental shift in how the platform handles document processing.

---

## What Actually Changed and Why It Matters

The announcements are straightforward, but the implementation details matter more. Here is what you need to understand as a developer deploying this release.

---

## Real-Time Capture Services: Not Just REST

The traditional batch model worked fine for mailrooms but breaks down for real-time scenarios. You scan or upload a document, submit it to Captiva, and get results immediately. This is what Real-Time Capture Services enables.

REST Services 2.0 is the key infrastructure piece. It introduces a Module Server Windows service that runs the actual processing operations. Applications call individual REST endpoints for specific tasks:

- **DocType Enumeration**: List available document types
- **Image Conversion**: Format conversion and page manipulation
- **Image Processing**: Enhancement, deskew, blank page detection
- **Barcode Recognition**: Extract barcode data
- **OCR and PDF Creation**: Recognize text, generate searchable PDFs
- **Document Classification**: Automatic document type identification
- **Data Extraction**: Pull structured data from documents
- **Validation**: Verify extracted data against rules

The services are designed for horizontal scaling. You run as many Module Server instances as your infrastructure supports, and they all share the same configuration. Licensing is per environment (production, test, dev, DR), not per instance. This is a solid model for high-volume use cases.

One important caveat: Real-Time Services is separately licensed from batch Captiva. The page counters are completely separate. You cannot transfer pages between batch and real-time systems. This is intentional because real-time processing is higher value (immediate responsiveness) and priced accordingly.

---

## Web Client: Architecture Matters

Captiva Web Client is a browser-based operator interface built on HTML5. No plugins, no Active-X, no Java. It supports modern browsers: Internet Explorer 10+, Chrome 32+, Firefox 26+.

But here is what you need to know: the Web Client is not just a UI. It is architecturally dependent on REST Services. Internally, it calls REST endpoints to submit batches. This creates a hard dependency: if you want Web Client, you need REST Services on the server.

If you are running Captiva Enterprise Server, REST Services are included. If you are on Standard Server, you must upgrade to Enterprise first. This is not a licensing loophole, it is an architectural requirement.

For operators coming from eInput (the older web interface), the Web Client is better. But your eInput license does not cover Web Client. You need to negotiate with your account manager for a transition model.

One more thing: the Web Client is a front end for scanning, indexing, and data validation. It is not an administrative interface. Process deployment, configuration management, and profile editing still happen in the thick client or designer tools. Web Client is for operators, not administrators.

---

## Module Reorganization: Less Dramatic Than It Sounds

"Captiva Desktop" is being reorganized into two modules: Completion and Identification. They share the same Desktop Family architecture, but they are now distinct for licensing and configuration purposes.

**Completion** handles data entry and validation. **Identification** handles manual document type classification when automated classification fails or was not configured. Do not confuse Identification with the Advanced Recognition classification engine. Advanced Recognition automatically classifies documents. Identification is the manual fallback UI.

For most deployments, this reorganization is invisible. Your CaptureFlows do not change. Your batches do not change. What changes is the licensing line item and configuration naming. If you have custom integrations or scripts that reference "Captiva Desktop" by name, audit them for compatibility.

---

## Standard Import: Consolidation

Two older modules, Multi-Directory Watch (MDW) and Email Import, are being consolidated into a single Standard Import module. This makes architectural sense: they both handle batch import from external sources (file system, email), so consolidating them reduces code maintenance and simplifies configuration.

The catch: Standard Import is not a one-to-one feature replacement. Some feature combinations from the old modules may not map cleanly to the new one. If you are currently using MDW or Email Import, test Standard Import against your actual import workflows before upgrading production systems.

---

## Licensing and Architecture Changes

**Page Count Sharing**: Advanced Recognition licenses can now share page counts across servers in a ScaleServer group. This aligns with how server-level licensing already works and simplifies capacity planning.

**Runtime Changes**: Processes compiled in CaptureFlow Designer now require only the .NET runtime. The VBA runtime dependency is gone. This improves maintainability because Microsoft has officially ended VBA support. New code should be C# only. Existing VB.NET code continues to work, but it is on borrowed time.

**Hardware Security Keys**: Deprecated. All licensing now uses software CAF files. If you are using hardware keys, plan to migrate to software licensing.

**Administration Console**: No longer supported. All administrative work goes through Captiva Administrator. If you have automation scripts targeting the old Administration Console, they need to be rewritten.

**Framework Architecture**: The 7.5 framework has been rearchitected for improved scalability and performance. If you are doing custom development or integration, this mostly affects you at the integration points (REST Services, configuration file formats). The core APIs remain stable.

---

## Migration: Plan Your Path

If you are on Captiva 6.x, do not jump directly to 7.5. Go through 7.0/7.1 first. The reason: Captiva 7.0 introduced CaptureFlow Designer and deprecated several modules (IndexPlus, Validation, Image Enhancement). These were removed in 7.5. If you jump directly to 7.5 with 6.x processes using these modules, your processes will break.

The 6.x to 7.0/7.1 migration requires work because of the architectural changes. But 7.0/7.1 to 7.5 is straightforward.

If you are on Captiva 7.x or newer, jumping to 7.5 is simple. Your existing license continues to work. The server automatically enables the new modules (Standard Import, Identification) during installation.

If you are on Captiva 5.3 or earlier, you need an updated license. Contact the vendor for a migration request form.

---

## Process Developer vs. CaptureFlow Designer

Process Developer is still supported in 7.5. It is not deprecated. But understand the constraints: new processes created in Process Developer are VBA-based, and Microsoft has officially ended VBA support.

This is not an OpenText decision. This is Microsoft saying "we are done with VBA." OpenText is strongly encouraging developers to use CaptureFlow Designer (C# based) for new development. Process Developer will work for maintaining existing processes, but do not start new projects with it.

---

## OCR Engine Flexibility and Advanced Recognition OEM API

One improvement in 7.5: the OCR engine is now decoupled from extraction logic. You can swap OCR engines without rebuilding recognition projects. For high-volume recognition workloads, this is valuable. Test different OCR engines against your document set and choose the best performer.

The Advanced Recognition OEM API is not included in 7.5. It was last updated in 7.0 and is end-of-life. If you need OEM-style integration, REST Services is the modern replacement.

---

## Reporting and 64-Bit Improvements

**Reporting**: Crystal Reports support is being removed across all OpenText products by end of 2015. This is a corporate mandate, not Captiva-specific. Reporting rules and SQL Server data storage remain unchanged. Standard 7.1 reports ship with 7.5. You can download the free Crystal Reports runtime from SAP if needed, or export data and use any BI tool.

**64-Bit Server**: Captiva Server is now 64-bit, providing performance improvements for high-volume workloads. All client modules remain 32-bit due to legacy component dependencies. This is a reasonable tradeoff because server-side processing is usually the bottleneck.

---

## The Summary

Captiva 7.5 consolidates the platform after the major architectural shift of 7.0. You get:

- Real-Time Capture Services for synchronous processing
- Browser-based Web Client with no plugins
- Simplified module lineup (deprecated modules removed)
- REST as the standard for integration
- .NET only for new code (VBA support ending)
- Better scalability through improved framework architecture

The biggest planning requirement is understanding the licensing boundaries. If you want Web Client or REST Services and you are on Standard Server, you must upgrade to Enterprise. This is not arbitrary; it is an architectural requirement.

For most existing deployments, the upgrade path is straightforward: through 7.0/7.1 if you are on 6.x, then to 7.5. Spend the time planning your licensing and architecture upfront, and the actual upgrade is smooth.
