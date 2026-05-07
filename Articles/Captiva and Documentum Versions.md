# Captiva and Documentum Versions

---

## Captiva

### EMC Captiva InputAccel 5.x

| Version                        | Service Packs   |
| ------------------------------ | --------------- |
| EMC Captiva InputAccel 5.0     | -               |
| EMC Captiva InputAccel 5.1     | -               |
| EMC Captiva InputAccel 5.2     | -               |
| EMC Captiva InputAccel 5.3     | SP1, SP2, SP3   |

---

### EMC Captiva InputAccel 6.x

In Captiva 6.0, MSSQL database support was introduced for the first time. Previously, files were stored only in the filesystem with no database concept.

| Version                        | Release Date | Sustaining Support Start |
| ------------------------------ | ------------ | ------------------------ |
| EMC Captiva InputAccel 6.0     | -            | -                        |
| EMC Captiva InputAccel 6.0 SP1 | -            | -                        |
| EMC Captiva InputAccel 6.0 SP2 | -            | -                        |
| EMC Captiva InputAccel 6.5     | 03/2011      | 03/2014                  |
| EMC Captiva InputAccel 6.5 SP1 | 11/2011      | 03/2014                  |
| EMC Captiva InputAccel 6.5 SP2 | 06/2013      | 01/2015                  |

---

### EMC Captiva Capture 7.x

| Version                    | Release Date | Sustaining Support Start |
| -------------------------- | ------------ | ------------------------ |
| EMC Captiva Capture 7.0    | 12/2012      | 01/2016                  |
| EMC Captiva Capture 7.1    | -            | -                        |
| EMC Captiva Capture 7.5    | 06/2015      | 07/2019                  |
| EMC Captiva Capture 7.6    | 11/2016      | 12/2020                  |

---
Here's your article rewritten as bullet points:

---

**7.0**
- IndexPlus, Validation, Image Enhancement, Multi-Directory Watch, and Email Import deprecated but still functional and backward compatible
- New modules introduced:
  - **Image Processor** - replaces Auto Annotate and Image Enhancement
  - **Standard Export** - replaces File System Export, Image Export, Index Export, PDF Export (file system saving only), and Values to XML
  - Email export functionality (previously only available through EMC Consulting)
- **Optional Database** - 7.0 now allows a file-based internal (embedded) database instead of requiring an external SQL Server; note that the following are not supported with the internal DB option:
  - Microsoft Failover / Clustering
  - ScaleServer
  - Audit Logging and Reporting
  - Web Services
  - Upgrading from external SQL to internal DB
- **CaptureFlow Designer** now integrated into Captiva Designer (previously a separate module); new features include:
  - Custom values on any CaptureFlow step (not just the CustomValues virtual step)
  - Direct process upload to server from within the designer
  - Step setup directly in the designer (previously required Administration Console)
  - Embedded script editor for any CaptureFlow step
  - In-production script debugging via the embedded editor
- **Captiva Desktop** - new unified operator client replacing IndexPlus, Image Quality Assurance, and Dispatcher Validation; key capabilities include:
  - KFI, rubberbanding, and table extraction
  - Extracted data validation
  - Document type changes and structure management (split/merge)
  - Table row editing (insert, delete, move, merge)
  - Field/page/document flagging
  - Image zoom/pan, print, rotate, annotate
  - Low-confidence character and validation error correction
  - New Operator Productivity Report
- **Extraction** - replaces Dispatcher Recognition; unattended OCR-based data extraction per page, combined into a single document output; new sample reports included
- **Captiva Administrator** - replaces the web-based Administration Console; no longer requires IIS; new features include:
  - Document Type Editor
  - Bulk Batch Retrigger command-line utility
  - New reports: Operator Productivity, Page Extraction, Field Extraction
- **Modules no longer shipped** (no longer supported):
  - Client Script Engine
  - Dispatcher License Manager
  - IBM CMIP-390 Index
  - IBM CMIP-390 Export
  - Image Divider
  - Web-based Administration Console *(note: cannot be used with 7.0 at all)*
  - createDB utility (superseded by Database Manager utility)

---

**7.5**
- IndexPlus, Validation, Image Enhancement, Multi-Directory Watch, and Email Import **removed entirely - no backward compatibility**
  - Use Completion instead of IndexPlus/Validation
  - Use Image Processor instead of Image Enhancement
  - Use Standard Import (with email connector) instead of Multi-Directory Watch and Email Import
- Process Developer still supported, but Microsoft removed support for VBA
- **New modules introduced:**
  - **Standard Import** - replaces Multi-Directory Watch and Email Import
  - **Identification** - new operator client for manual classification and pre-indexing; replaces Classification Edit; part of the Desktop Family
  - **Captiva Capture Web Client** - brand new browser-based client
  - **Captiva REST Services** - new real-time REST API services
  - **Desktop Family of Modules** - unified suite including Identification and the renamed Completion module (formerly Captiva Desktop)
- **Legacy modules** (still shipped, no enhancements, bug fixes only, labelled "(Legacy)" in Start menu):
  - Multi-Directory Watch → replaced by Standard Import
  - Email Import → replaced by Standard Import
  - Classification Edit → replaced by Identification
- **Modules no longer shipped** (not in installer; no support):
  - Administration Console → Captiva Administrator
  - Auto Annotate → Image Processor + Image Processing profiles
  - Automatic Quality Assurance
  - Client Script Engine → .NET Code module
  - Connector for eCopy ShareScan
  - Dispatcher License Manager
  - Dispatcher Recognition → Extraction
  - Dispatcher Statistics
  - Dispatcher Validation → Captiva Completion
  - ECM Web Services Importer Configuration
  - Excel Graphing
  - File System Export → Standard Export
  - IBM CMIP-390 Export
  - IBM CMIP-390 Index
  - Image Divider → Image Converter
  - Image (module) → Image Processor
  - Image Enhancement → Image Processor
  - Image Export → Standard Export
  - Image Quality Assurance → Captiva Completion
  - iManage WorkSite Server Export
  - Index Export → Standard Export
  - IndexPlus
  - InputAccel Remoting *(prior versions can still interoperate with 7.5 servers)*
  - PDF Export → Image Converter (for most functionality)
  - PrimeOCR Plus
  - Spawn *(no replacement)*
  - Values to XML → Standard Export

---

**7.7**
- Product manuals now titled **"OpenText Captiva Capture"** - "EMC" no longer appears in product documentation (previously "EMC Captiva" through 7.6)
- Internal folder names and paths (e.g., IAValues) still retain the word **"InputAccel"**

---

### OpenText Captiva Capture 7.7–16.x

| Version                            | Release Date | Sustaining Support Start |
| ---------------------------------- | ------------ | ------------------------ |
| OpenText Captiva Capture 7.7       | 01/2018      | 01/2023                  |
| OpenText Captiva Capture 16.5      | 12/2018      | 12/2023                  |
| OpenText Captiva Capture 16.6      | 05/2019      | 05/2024                  |

---

### OpenText Intelligent Capture

| Version                            | Release Date | Maintenance Ends |
| ---------------------------------- | ------------ | ---------------- |
| OpenText™ Intelligent Capture 20.2 | 01 Jun 2020  | 01 Jun 2025      |
| OpenText™ Intelligent Capture 21.4 | 01 Nov 2021  | 01 Nov 2026      |
| OpenText™ Intelligent Capture 22.1 | 01 Feb 2022  | 01 Feb 2027      |
| OpenText™ Intelligent Capture 22.2 | 01 Apr 2022  | 01 Apr 2027      |
| OpenText™ Intelligent Capture 22.3 | 01 Aug 2022  | 01 Aug 2027      |
| OpenText™ Intelligent Capture 22.4 | 01 Oct 2022  | 01 Oct 2027      |
| OpenText™ Intelligent Capture 23.1 | 01 Feb 2023  | 01 Feb 2028      |
| OpenText™ Intelligent Capture 23.2 | 01 May 2023  | 01 May 2028      |
| OpenText™ Intelligent Capture 23.3 | 01 Aug 2023  | 01 Aug 2028      |
| OpenText™ Intelligent Capture 23.4 | 01 Nov 2023  | 01 Nov 2028      |
| OpenText™ Intelligent Capture 24.1 | 01 Jan 2024  | 01 Jan 2029      |
| OpenText™ Intelligent Capture 24.2 | 01 May 2024  | 01 May 2029      |

---

### OpenText Capture

| Version                | Release Date | Maintenance Ends |
| ---------------------- | ------------ | ---------------- |
| OpenText™ Capture 24.4 | 17 Dec 2024  | 31 Dec 2029      |
| OpenText™ Capture 25.2 | 23 Jun 2025  | 01 Jul 2030      |
| OpenText™ Capture 25.4 | 21 Nov 2025  | 30 Nov 2030      |

---

---

**5.3 SP3 / SP4 (InputAccel)**

The following modules were already part of the product at this stage - these are not new introductions but the baseline carried into the 6.x era:
- Scan, Rescan, Index, Watch
- ScanSoft OCR, PrimeOCR
- Documentum Server Export, IBM Content Manager Export
- Excel Graphing, Automatic Quality Assurance, iManage WorkSite Server Export
- Administrator and Supervisor modules

The following were formerly EMC Consulting-only modules that became standard in this era:
- Email Import
- Multi-Directory Watch (replaced Watch)
- Image Divider

---

**6.0 (InputAccel 6.0) - Released November 2008**

Version 6.0 was a major architectural overhaul. Key infrastructure changes included a multi-threaded server, a centralized SQL database, centralized licensing and logging, SOA/Web Services support, and tested support for VMware and Citrix virtualization.

- **New attended modules** (web-deployable via ClickOnce):
  - **ScanPlus** - replaces Scan; adds client-side scripting, shared scanner configurations, and batch manipulation
  - **RescanPlus** - replaces Rescan; includes all ScanPlus functionality except new batch creation
  - **IndexPlus** - replaces Index; adds index families, rubberbanding, client-side scripting, and document assembly tools
- **New unattended modules:**
  - **Script Engine** - runs client-side scripts independently between module steps in a process
  - **Image Divider** - formerly EMC Consulting only; now standard; splits multi-page image files into single pages
  - **Web Services Input** - new; SOA web services provider
  - **Web Services Output** - new; SOA web services consumer
  - **Documentum Advanced Export** - replaces Documentum Server Export
  - **PrimeOCR Plus** - replaces PrimeOCR for InputAccel
  - **NuanceOCR** - replaces ScanSoft OCR
- **Former EMC Consulting modules now standard:**
  - **Email Import**
  - **Multi-Directory Watch** - replaces Watch
- **New administration:**
  - **Administration Console** - replaces both the Administrator and Supervisor modules; web-based, requires IIS
- All new modules use .NET 2.0 client-side scripting (VB.NET or C#) and are ScaleServer compatible
- **Modules no longer installed** (backward compatible but not shipped):
  - Administrator, Supervisor
  - Scan, Rescan, Index, Watch
  - Documentum e-Content Server Compatible Export
  - IBM Content Manager Compatible Export
  - iManage WorkSite Server Compatible Export
- **Features no longer supported:**
  - FAT32 disk partition for the InputAccel Server
  - Audit Extensions - replaced by centralized database logging; could be upgraded to work with 6.0 but would not be supported in future releases
  - Alerts management in Administration Console

---

**6.5 (InputAccel 6.5)**

- **New modules introduced:**
  - **Image Converter** - replaces Image Divider; note: unlike Image Divider, has no client-side scripting capability
  - **.NET Code** - replaces Script Engine / Client Script Engine
  - **IBM CM Advanced Export** - replaces IBM Content Manager Export
  - **East Euro / APAC OCR** - new OCR engine for Eastern European and Asia-Pacific character sets
  - **ClickOnce Deployment Utility** - web-based or network deployment of attended modules
  - **InputAccel Remoting** - supports remote operators connecting over the Internet; requires IIS
  - **CaptureFlow Designer** - new visual process design tool; eliminates the need to write VBA code in Process Developer
- ScanPlus, RescanPlus, IndexPlus, and NuanceOCR all updated to 6.5 with multi-language and multi-code-page processing support
- **Modules not included in 6.5 installer** (can still run if already installed from 6.0):
  - Automatic Quality Assurance
  - Script Engine / Client Script Engine → replaced by .NET Code
  - Documentum Server Export → replaced by Documentum Advanced Export
  - ECM Web Services Importer Configuration
  - Excel Graphing
  - IBM Content Manager Export → replaced by IBM CM Advanced Export
  - Image Divider → replaced by Image Converter
  - iManage WorkSite Server Export
  - PrimeOCR Plus
- **SDK breaking change:** IDateTime / IADateTime (deprecated in 6.0 SP2) completely removed in 6.5; custom modules must migrate to .NET DateTime
- Dispatcher for InputAccel versions prior to 6.0 are not supported; upgrading to 6.5 requires simultaneous upgrade of all Dispatcher client machines

---

**7.0 (EMC Captiva Capture)**

The product was renamed from "InputAccel" to **"EMC Captiva Capture"** at this version.

- IndexPlus, Validation, Image Enhancement, Multi-Directory Watch, and Email Import deprecated but still functional and backward compatible
- **New modules introduced:**
  - **Image Processor** - replaces Auto Annotate and Image Enhancement
  - **Standard Export** - replaces File System Export, Image Export, Index Export, PDF Export (file system saving only), and Values to XML
  - **Captiva Desktop** - new unified operator client replacing IndexPlus, Image Quality Assurance, and Dispatcher Validation; key capabilities:
    - KFI, rubberbanding, and table extraction
    - Extracted data validation
    - Document type changes and structure management (split/merge)
    - Table row editing (insert, delete, move, merge)
    - Field/page/document flagging
    - Image zoom/pan, print, rotate, annotate
    - Low-confidence character and validation error correction
    - New Operator Productivity Report
  - **Extraction** - replaces Dispatcher Recognition; unattended OCR-based data extraction per page, combined into a single document output; new sample reports included
  - **Captiva Administrator** - replaces the web-based Administration Console; no longer requires IIS; new features:
    - Document Type Editor
    - Bulk Batch Retrigger command-line utility
    - New reports: Operator Productivity, Page Extraction, Field Extraction
  - Email export functionality (previously only available through EMC Consulting)
- **Optional Database** - 7.0 introduced the option of a file-based internal (embedded) database instead of requiring an external SQL Server; the following are not supported when using the internal DB:
  - Microsoft Failover / Clustering
  - ScaleServer
  - Audit Logging and Reporting
  - Web Services
  - Upgrading from external SQL to internal DB
- **CaptureFlow Designer** now integrated into Captiva Designer (previously a separate module); new features:
  - Custom values on any CaptureFlow step (not just the CustomValues virtual step)
  - Direct process upload to server from within the designer
  - Step setup directly in the designer (previously required Administration Console)
  - Embedded script editor for any CaptureFlow step
  - In-production script debugging via the embedded editor
- **Modules no longer shipped:**
  - Client Script Engine → .NET Code
  - Dispatcher License Manager
  - IBM CMIP-390 Index
  - IBM CMIP-390 Export
  - Image Divider → Image Converter
  - Web-based Administration Console *(cannot connect to 7.0 database at all)*
  - createDB utility → superseded by Database Manager utility

---

**7.5 (EMC Captiva Capture)**

- IndexPlus, Validation, Image Enhancement, Multi-Directory Watch, and Email Import **removed entirely - no backward compatibility**
  - Use Completion instead of IndexPlus / Validation
  - Use Image Processor instead of Image Enhancement
  - Use Standard Import (with email connector) instead of Multi-Directory Watch and Email Import
- Process Developer still supported, but Microsoft removed VBA support
- **New modules introduced:**
  - **Standard Import** - replaces Multi-Directory Watch and Email Import
  - **Identification** - new operator client for manual classification and pre-indexing; replaces Classification Edit; part of the Desktop Family
  - **Captiva Capture Web Client** - brand new HTML5 browser-based client; no ActiveX or Java plug-ins required; supports IE10/11, Chrome v32, Firefox v26
  - **Captiva REST Services / Real-Time Capture Services** - new real-time REST API services including DocType Enumeration, Image Conversion, Image Processing, Barcode Recognition, OCR, Document Classification, Data Extraction, and Validation
  - **Desktop Family of Modules** - unified suite including Identification and Completion (renamed from Captiva Desktop)
- **Legacy modules** (still shipped, bug fixes only, no enhancements, labelled "(Legacy)" in Start menu):
  - Multi-Directory Watch → Standard Import
  - Email Import → Standard Import
  - Classification Edit → Identification
- **Modules no longer shipped:**
  - Administration Console → Captiva Administrator
  - Auto Annotate → Image Processor
  - Automatic Quality Assurance
  - Client Script Engine → .NET Code
  - Connector for eCopy ShareScan
  - Dispatcher License Manager
  - Dispatcher Recognition → Extraction
  - Dispatcher Statistics
  - Dispatcher Validation → Captiva Completion
  - ECM Web Services Importer Configuration
  - Excel Graphing
  - File System Export → Standard Export
  - IBM CMIP-390 Export
  - IBM CMIP-390 Index
  - Image Divider → Image Converter
  - Image (module) → Image Processor
  - Image Enhancement → Image Processor
  - Image Export → Standard Export
  - Image Quality Assurance → Captiva Completion
  - iManage WorkSite Server Export
  - Index Export → Standard Export
  - IndexPlus
  - InputAccel Remoting *(prior versions can still interoperate with 7.5 servers)*
  - PDF Export → Image Converter
  - PrimeOCR Plus
  - Spawn *(no replacement)*
  - Values to XML → Standard Export
- **Flagged for future deprecation:**
  - Click-Once Deployment Utility
  - Page Registration Module

---

**7.6 (EMC Captiva Capture)**

An intermediate release between 7.5 and 7.7. The East Euro / APAC OCR module was still present but began to be disabled by default in Captiva Designer. Users who wanted to continue using this module after upgrading to 7.7 had to pass through 7.6 first - a direct upgrade from 7.5 or earlier to 7.7 would lose access to it.

---

**7.7 (OpenText Captiva Capture) - Released January 2018**

- Product manuals now titled **"OpenText Captiva Capture"** - "EMC" removed from all documentation from this version onward
- Internal folder names and paths (e.g., IAValues, InputAccel directories) still retain the word "InputAccel"
- **New modules / changes:**
  - **Standard OCR** - new module; performs data extraction from electronic documents and images by running an appropriate OCR engine per content type; produces an OCR data cache as output
  - **Recognition Designer** - new
  - **ScanPlus and RescanPlus** upgraded to **64-bit** for improved performance
  - **Process Developer** formally flagged for deprecation in a future release
  - CaptureFlow Designer: "Save As" dialog improved to carry process instance settings; CaptureFlow trigger variables now automatically reset on backwards jumps
  - Standard Import: Email improvements - EWS protocol support, NTLM authentication, inline attachment exclusion, and automatic decryption for server-encrypted emails
- **Modules no longer shipped:**
  - Email Import → Standard Import *(was legacy in 7.5, now fully removed)*
  - Multi-Directory Watch → Standard Import *(was legacy in 7.5, now fully removed)*
  - Page Registration → no replacement *(use Extraction module for zonal recognition instead)*
  - East Euro / APAC OCR → no replacement *(corresponding Advanced Recognition engine also removed; users on 7.5 or earlier must upgrade to 7.6 first)*
- **CAS (Central Authentication Service)** server no longer supported for REST Services and Web Client
- Custom modules built with InputAccel SDK 5.x (IAClient32.dll) must migrate to .NET Code module

---

**16.6 (OpenText Captiva Capture)**

Version numbering jumps from 7.x to 16.x following OpenText's product versioning strategy. The core module lineup at this stage:

- **Attended:** ScanPlus, RescanPlus, Completion, Identification
- **Import:** Standard Import, Web Services Input, Web Services Coordinator, Web Services Hosting
- **Export:** Standard Export, ODBC Export, Archive Export, Documentum Advanced Export, ApplicationXtender Export, FileNet Content Manager Export, FileNet Panagon IS/CS Export, Global 360 Export, IBM CSSAP Export, IBM CM Advanced Export, MS SharePoint Export, OpenText Livelink Advanced Export
- **Recognition:** NuanceOCR, Standard OCR, Extraction, East Euro/APAC OCR *(still present but disabled by default in Designer)*
- **Image Handling:** Image Converter, Image Enhancement, Page Registration
- **Advanced Recognition:** Classification, Collector, Auto-Learning Supervisor
- **Utilities:** .NET Code, Copy, Timer, Multi
- **Administration:** Captiva Administrator, Captiva Designer (with CaptureFlow Designer), Process Developer

---

**Intelligent Capture CE 21.4 - Released November 2021**

- Product rebranded to **"OpenText Intelligent Capture CE 21.4"** following OpenText's cloud strategy
- **New features:**
  - Full deployment to **Amazon Web Services (AWS)**
  - **Docker / Windows Container** support for Web Client and Real Time Services
  - **OpenText Directory Services (OTDS)** for authentication and authorization
  - **Vietnamese language** added to Full-page OCR REST API for Standard OCR
  - New **Information Extraction (IE) Database** installation option
  - New **Information Extraction Engine (IEE)** installer wizard
  - **Advanced Cloud OCR** - new OCR engine option using Google Cloud Vision API
  - Ability to convert between Advanced Recognition and Information Extraction document types in Recognition Designer
  - Documentation now delivered via **OpenText Global Help Server (GHS)** for live online access
- **Modules no longer shipped:**
  - East Euro / APAC OCR → no replacement *(the corresponding Advanced Recognition engine is also removed)*
- **Flagged for future deprecation:**
  - ODBC Export Module → migrate to ODBC Export profile
  - Process Developer → migrate to Intelligent Capture Designer
  - VB.NET scripting in CaptureFlow → migrate to C#
  - 6.x licenses containing IndexPlus and/or Validation entries → free license refresh available on request

---

**Intelligent Capture CE 22.2 - 2022**

- Fully settled under the **"OpenText™ Intelligent Capture"** identity
- Architecture now has two parallel tracks: traditional server-based processing and **Real Time Services** (REST-based, cloud/mobile oriented)
- Module lineup at this version:
  - **Attended:** ScanPlus, RescanPlus, Completion, Identification
  - **Import:** Standard Import, Web Services Input/Output/Coordinator/Hosting
  - **Export:** Standard Export, ODBC Export, Archive Export, OpenText ApplicationXtender Export, OpenText Documentum Advanced Export, FileNet Content Manager Export, FileNet Panagon IS/CS Export, Global 360 Export, Export for IBM Content Manager, Export for SAP Archive and AP Connect
  - **Recognition:** NuanceOCR, Standard OCR, Extraction
  - **Image Handling:** Image Converter
  - **Advanced Recognition:** Classification, Collector
  - **Utilities:** .NET Code, Copy, Timer, Multi
  - Note: East Euro/APAC OCR, Page Registration, Email Import, Multi-Directory Watch, Image Enhancement, and PrimeOCR Plus are all absent by this version

## Documentum

<!-- Documentum version tables to be added -->
