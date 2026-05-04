
# Dwaipayan Das
### ECM Architect · OpenText Captiva & Documentum · Azure · Senior Manager

📍 Toronto, Canada · Canadian PR
🔗 [LinkedIn](https://www.linkedin.com/in/dwds/) · das.d@hotmail.com

---

## About Me

I am an ECM Architect and Senior Manager with 15+ years of hands-on experience delivering large-scale document capture, content management, and data platform solutions across banking, financial services, insurance, government, oil & gas, and global retail — across Singapore, Canada, the Netherlands, and India.

My core expertise is **OpenText Captiva / Intelligent Capture** (InputAccal 5.x through IC 22.x) and **OpenText Documentum** (5.x through 20.x) — covering the full lifecycle from CaptureFlow development, SOAP/REST API integration, and HA/DR infrastructure through to version upgrades, Azure cloud migrations, and L3 production support in regulated environments.

I have operated in some of the most demanding ECM environments — government tax authorities handling sensitive taxpayer PII, global payment networks, major APAC banks, and oil & gas multinationals — where data governance, audit trails, RBAC, and system availability are non-negotiable.

---

## What I Do

| Area | Details |
|---|---|
| **Document Capture** | CaptureFlow design and development, multi-channel ingestion (MFD, scanner, email, network folder, web), Dispatcher Classification & Recognition, OCR, image enhancement, barcode separation |
| **API & Custom Development** | REST/SOAP APIs (ASP.NET Core), .NET Code Modules, C# WinForms (InputAccal client), metadata validation DLLs, .NET Assembly Proxy, Oracle/SQL Server stored procedure integration, ADODB |
| **Scripting & Automation** | PowerShell (VM health checks, service sequencing, deployments), Python, PySpark |
| **Infrastructure & HA/DR** | MSCS Active/Active & Active/Passive clustering, SQL Server AlwaysOn, ScaleServer, Windows Server 2003–2022, IIS 6–10, Apache Tomcat, Docker |
| **Cloud & Azure** | App Service, ADLS, ADF, Key Vault, Entra ID, VNet, Private Endpoints, Managed Identities, ARM Templates, Azure DevOps |
| **Platform Modernisation** | Evaluating and replacing OpenText Captiva CaptureFlows with Microsoft AI Builder and Power Automate to reduce licensing costs |
| **SharePoint** | WSS 3.0 through SharePoint Online — multi-tier farm architecture, MSCS SQL clustering, database mirroring, migration, governance |

---

## Project Experience

---

### Government Agency· Confidential · 2025–2026
**Senior Manager & Solution Architect · OpenText Intelligent Capture 22.4 · Azure GCC**

Led paperless office automation and intelligent document capture initiatives on Azure Government Commercial Cloud (GCC) handling sensitive tax data. Integrated tax workflows with other government agencies as part of a broader whole-of-government digital transformation effort.

**Tech Stack:** OpenText IC 22.4 (Standard Import, Image Processor, Multi, .NET Code, ODBC Export, Documentum Export), Azure App Service, ADLS, Key Vault, Entra ID, VNet, Private Endpoints, MSSQL on Azure VM (IaaS), C#.NET, Python, PowerShell, REST APIs

- Integrated Individual Income Tax and Property Tax Division data via APIs into CaptureFlows for external validation
- Designed CaptureFlows for digitisation of paper GST supporting documents — MFD-based ingestion, Dispatcher classification, OCR extraction, and REST API validation against tax registries
- Built RESTful APIs in C#.NET hosted on Azure App Service for metadata validation between Captiva CaptureFlows and backend MSSQL tax systems
- Integrated XML output from Standard Export module into ADLS for downstream tax analytics
- Configured RBAC across Azure, Captiva Administrator Console, and SQL Server stored procedure-level access — maintaining minimum privilege for sensitive taxpayer PII
- Enabled audit logging for all capture and API workflows for government compliance
- Provided L3 support — weekend green zone patching, CAB processes, incident triage for Windows-based Captiva on Azure GCC VMs
- Developed PowerShell scripts for Captiva VM health checks, service state monitoring, and automated service start sequencing

---

### Global Retail · Singapore · 2024–2025
**Assistant Manager & Data Architect · OpenText Intelligent Capture 21.4 · Azure Databricks**

Led modernisation of legacy OpenText Captiva infrastructure — phasing out third-party scanning vendor dependency and reducing OpenText licensing costs — alongside Azure data engineering workstreams.

**Tech Stack:** OpenText IC 21.4 (XML Export, Standard Import, Dispatcher), Azure App Service, ADLS, ADF, DevOps, Key Vault, Entra ID, Logic Apps, C#.NET, Python, PowerShell, Microsoft AI Builder, Power Automate, Canvas Apps

- Integrated XML output from the Standard Export module into ADLS for downstream Databricks Lakehouse analytics
- Developed Python (FastAPI) REST APIs on Azure App Service for document metadata validation — integrated with Captiva via .NET Code Modules against ERP and Dynamics 365, secured with Azure AD OAuth 2.0
- Developed PowerShell scripts for Captiva environment health checks and automated deployment tasks
- Led technical evaluation and phased replacement of Captiva CaptureFlows with Microsoft AI Builder and Power Automate to reduce licensing costs

---

### Global Fintech / Payments · Singapore & USA · 2019–2023
**Analyst / Technical Lead · OpenText Intelligent Capture 6.5→16.4 · Azure · M365**

Part of a 30-member cross-functional team spanning Singapore and the United States — responsible for the full lifecycle of Documentum and Captiva platforms alongside Azure and M365 workstreams.

**Tech Stack:** OpenText Documentum 6.5/16.4, OpenText IC 6.5/16.4 (ScanPlus, IndexPlus, Image Enhancement, Image Processor, Image Converter, Multi, ODBC Export, Documentum Advanced Export, eInput, Dispatcher, .NET Code, ScaleServer), Windows Server 2016, SQL Server 2016 AlwaysOn, MSCS Active/Passive, IIS 10, Azure VM, Azure SQL, Blob Storage, SharePoint Online, Power Automate, AI Builder

**Platform Upgrade (v6.5 → v16.4):**
- Upgraded Captiva and Documentum from v6.5 to v16.x on Azure VMs, re-architected to HA/DR design
- Designed HA/DR infrastructure — MSCS Active/Passive WSFC, SQL Server AlwaysOn Availability Groups, and Captiva ScaleServer for distributed workload processing
- Coordinated with Infrastructure team on Azure VM sizing, OS provisioning, static IP allocation, service accounts, IAS folder permissions, and Entra ID integration
- Configured firewall rules, inbound/outbound port whitelisting, and extended intranet connectivity for third-party scanning vendor integration
- Redesigned legacy IPPS-based capture workflows to Captiva Designer-based processes
- Designed and developed SOAP-based validation APIs consumed by Captiva .NET Code Modules via proxy class — backend validation via SQL Server stored procedures
- Executed Documentum Content Server upgrade — repository install, docbroker configuration, database schema migration, filestore and keystore transfer, legacy decommission

**BAU L3 Support:**
- Accountable for L3 incident resolution, enhancement delivery, and change request management
- Coordinated Windows patch deployments; performed post-patch Captiva Input Agent and Documentum health checks
- Performed MSSQL backend maintenance — index rebuilds, query plan analysis, backup validation, AlwaysOn health monitoring
- Participated in DR failover drills — MSCS cluster failover, SQL Server AlwaysOn secondary promotion, Captiva ScaleServer failover, full application recovery within RTO/RPO
- Monitored batch processing queues via Captiva Admin Console; diagnosed and resolved failed batches, recognition errors, export processing issues

---

### Banking & Federal Government · Canada · 2014–2019
**Solution Architect / Project Manager · OpenText Captiva/Documentum 6.5→7.x · Azure**

Worked on-site for a Canadian federal government corporation and a US-based banking client. Travelled extensively across Canada for requirements gathering and on-site delivery.

**Tech Stack:** OpenText Documentum 6.5/7.2, OpenText Captiva 6.5/7.5 (ScanPlus, IndexPlus, Image Enhancement, Multi, ODBC Export, Documentum Advanced Export, eInput, Dispatcher, ScaleServer, ClickOnce), Windows Server 2012 R2/2016, SQL Server 2012–2016 (AlwaysOn, MSCS), SharePoint 2016/Online, Azure VM, Azure SQL, AzCopy, C#.NET, IIS 8.5

**6-Country APAC ECM Upgrade (Captiva & Documentum 6.5 → 7.x):**
- Led upgrade across 6 countries — 40+ Captiva servers, 20+ processes per country, MSCS clustering and SQL AlwaysOn
- Enhanced and upgraded existing Captiva processes replacing deprecated modules and adding new functionality
- Created .NET assembly proxy to consume SOAP web services for metadata validation; MSSQL stored procedures for data validation and reporting
- Managed OpenText vendor relationship, licensing repackaging, and third-party coordination

**Azure Migration:**
- Migrated Captiva to Azure VMs (separate tiers for unattended, attended, web components); Documentum in Docker containers on Azure VM; SQL to Azure SQL PaaS
- Redesigned CaptureFlows for multi-channel ingestion; SPOC for MFD firmware upgrades across 100+ field offices

**SharePoint 2016 Farm:**
- Architected 3-tier SharePoint 2016 farm with SQL Server database mirroring across primary and secondary data centres for HA/DR

---

### Oil & Gas · India · 2012–2014
**Application Developer · OpenText Captiva 6.5 · Documentum 6.x**

Centralised document management and archiving for a major oil & gas client across EMEA, Iberia, and ASPAC. Multi-channel Captiva ingestion, Documentum storage, SAP integration, multi-vendor environment.

**Tech Stack:** OpenText Documentum 6.x (Content Server, Composer, Webtop, DFC, BPM), OpenText Captiva 6.5 (InputAccal Server, ScanPlus, IndexPlus, Image Enhancement, Multi, ODBC Export, Documentum Export, eInput, Dispatcher, ScaleServer), Oracle 11g, SQL Server 2008, Apache Tomcat 6.0, ASP.NET, C#.NET, VBA

- Created multi-channel Captiva processes to ingest documents from network folders, emails, and scanners
- Built Dispatcher Classification and Recognition workflows with OCR-based template matching and barcode recognition
- Created SOAP-based APIs consumed via proxy classes in .NET Code Module for data validation
- Performed Captiva and Documentum upgrade from 5.3 to 6.5 with HA/DR using MSCS cluster and ScaleServer
- Architected multi-tier SharePoint farm with NLB and MSCS SQL clustering; designed HA/DR with database mirroring
- Provided L3 support; served as primary liaison between the client, OpenText/EMC, and third-party scanning vendors

---

### Product Company · Netherlands · 2011
**Technical Engineer · SharePoint & ECM**

Worked with enterprise clients on SharePoint farm infrastructure consolidation and WCM product support.

**Tech Stack:** MOSS 2007, SQL Server 2008 R2 (MSCS Active/Passive, Database Mirroring), Windows Server 2008, IIS 7, C#.NET, T-SQL, NetBackup

- Assessed three MOSS 2007 farms (24,000 users, 200+ sites) and designed consolidated farm in a European data centre
- Designed consolidated farm with MSCS Active/Passive SQL clustering and database mirroring — RTO 8 hours, RPO 120 minutes, 99.35% availability
- Developed T-SQL scripts for automated SharePoint content database switching to mirrored secondary data centre

---

### Banking · Singapore, Australia & India · 2007–2011
**Software Engineer → Lead Software Engineer · OpenText Captiva InputAccal 5.x/6.x · Documentum 5.x/6.x**

Operated out of a major bank's Singapore office as part of the ECM Centre of Excellence — the team that first introduced OpenText Documentum and Captiva to the bank. The Singapore hub managed implementations across Malaysia, Thailand, Vietnam, Indonesia, Hong Kong, Australia, and the Philippines, as well as development work for India.

**Tech Stack:** OpenText Documentum 5.3/6.x (Content Server, DA, Composer, Webtop/WDK, DFC, BPM, DQL, Method Server), OpenText Captiva InputAccal 5.x/6.x (InputAccal Server, ScanPlus, IndexPlus, Image Enhancement, Watch/MDW, Email Import, Multi, ODBC Export, Values to XML, Documentum Export, eInput, Dispatcher, ScaleServer, ClickOnce), Kofax Capture, KTM 10, MSCS Active-Active/Active-Passive, SQL Server 2005/2008, Oracle 9i/11g, UNIX, Windows Server 2003/2008, ASP.NET, C#.NET, C# WinForms, VB6, VBA, Java

**Projects:**

**Paperless Banking Automation** — Introduced a digital Workflow and Imaging Solution across bank branches to automate back-office banking operations, eliminate physical document handling, and enable same-day processing.

**HR Recruitment & Document Digitisation** — Automated the HR recruitment process and digitised all employee documents at a global banking services entity. The solution captured documents through scanners and integrated with the core CMS for processing, storage, and retrieval.

**Account Opening & Mainframe Integration** — Automated account opening processes across multiple product lines including credit cards, investment profiling, and unit trusts. First paperless account opening system at the bank's Singapore branches — integrated with the core mainframe.

**Regional ASPAC Workflow Solution** — Implemented an Imaging & Workflow Solution across all branches and offices in five countries to achieve near-paperless back-office automation. Covered document capture, retention policies, records management, role management, and email integration.

**Content Migration & Data Segregation** — Migrated legacy workflows to the main platform with strict data segregation protocols separating BAU data from a newly acquired business unit.

**Enterprise Content Management — 40+ Branches** — Implemented ECM and document capture across 40+ bank branches for banking, insurance, and mortgage lines of business. Integrated capture with backend CMS and extended to third-party scanning vendors.

**Responsibilities across all projects:**
- Worked with EMC, infrastructure, and network teams to install and deploy Captiva and Documentum across ASPAC branches (Dev/SIT/UAT/Prod, MSCS, ScaleServer)
- Managed environment setup and infrastructure (BOM, CPU/RAM sizing, node planning, network/security, service accounts, IAS permissions)
- Performed installation, patching, upgrades, and Windows validation for Captiva and Documentum
- Built 20+ Captiva InputAccal IPPs per ASPAC country (Watch, Scan, Index, Multi, Dispatcher Classification/Recognition/Extraction, Image Enhancement, eInput, Documentum Export)
- Led digitisation of card processing using high-speed third-party scanning integrated with Captiva
- Rolled out eInput (eScan, eIndex) across 40+ branches via Microsoft ClickOnce
- Built backend integrations — ODBC with Oracle, Java/VB.NET/C# components, SOAP services via .NET Code Module with Oracle stored procedure validation
- Developed C# DFC utility (DfSessionManager, DfDocument, DfProperty) for metadata extraction from the Documentum repository
- Executed DQL-based validation and repository admin — jobs, sessions, workflows, object validation, metadata fixes
- Enabled audit/logging and batch reporting; monitored MDW/watch folders, batch volumes, and processing performance
- Provided L3 production support — batch failures, recognition errors, export failures, workflow issues
- Performed end-to-end testing (Unit, Integration, Functional, System, UAT) across capture and ECM workflows

---

## Certifications

| Certification | Issuing Organisation |
|---|---|
| MCSA: SQL 2016 Database Administration | Microsoft |
| AWS Certified Solutions Architect – Associate | Amazon Web Services |
| Microsoft Certified: Azure Developer Associate (AZ-203) | Microsoft |
| SDL Tridion Advanced Infrastructure | SDL Plc (Now RWS) |
| SDL Tridion System Administrator | SDL Plc (Now RWS) |
| Docker and Kubernetes | VISA Inc. |
| Get Started with Databricks for Data Engineering | Databricks |
| Fundamentals of the Databricks Lakehouse Platform | Databricks |
| Verified International Academic Qualifications | World Education Services (WES) |

---

## Education

**Master of Computer Applications (MCA)** · West Bengal University of Technology (MAKAUT) · 2004–2007  
**Bachelor of Science (Hons), Physics & Mathematics** · University of Calcutta · 2001–2004