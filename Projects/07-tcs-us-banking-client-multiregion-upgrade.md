# TCS Canada - US Banking Client Multi-Region ECM Upgrade
**Role:** Solution Architect / Project Manager  
**Organization:** US-Based Banking Client  
**Platform:** OpenText Documentum 6.5→7.2 · OpenText Captiva 6.5→7.5 · Azure  
**Scope:** 6 APAC Countries, 40+ Captiva Servers

---

## Overview

On-site engagement for a US-based banking client. Led a major ECM upgrade spanning 6 APAC countries with 40+ Captiva servers and 20+ processes per country. Included full Azure cloud migration of the platform.

---

## Tech Stack

- **OpenText:** Documentum 6.5→7.2, Captiva 6.5→7.5 (ScanPlus, IndexPlus, Image Enhancement, Image Converter, Multi, ODBC Export, Documentum Advanced Export, eInput, Dispatcher, ScaleServer, ClickOnce)
- **Infrastructure:** Windows Server 2012 R2/2016, SQL Server 2012-2016 (AlwaysOn, MSCS)
- **Cloud:** Azure VM, Azure SQL, AzCopy, Docker
- **Languages:** C#.NET, IIS 8.5

---

## ECM Upgrade - Documentum & Captiva 6.5 → 7.x Across 6 APAC Countries

**Countries Covered:**
- Singapore
- Malaysia
- Thailand
- Indonesia
- Vietnam
- Australia

### Scope & Scale
- 40+ Captiva servers across 6 countries
- 20+ processes per country
- MSCS clustering and SQL AlwaysOn infrastructure

### Key Achievements

- **Multi-Country Coordination:** Led upgrade across 6 APAC countries with complex logistics and regional teams
- **Process Modernization:** Enhanced and upgraded existing Captiva processes, replacing deprecated modules and adding new functionalities
- **API Development:** Created .NET assembly proxy to consume SOAP web services for metadata validation across regions
- **Data Validation Framework:** Built MSSQL stored procedures for centralized data validation and cross-regional reporting
- **Vendor Management:** Managed OpenText vendor relationships, licensing repackaging, multi-country coordination, and third-party vendors
- **Quality Assurance:** Ensured consistent deployment across all 6 countries with standardized processes

---

## Azure Cloud Migration - ECM On-Prem to Azure

### Platform Transition
- **Captiva Migration:** Migrated Captiva to Azure VMs with separate tiers for unattended, attended, and web components
- **Documentum Containerization:** Deployed Documentum in Docker containers on Azure VM for improved scalability
- **Database Migration:** Migrated SQL to Azure SQL PaaS for managed database services
- **Multi-Tier Architecture:** Separated unattended agents (processing), attended clients (data entry), and web services

### Workflow & Process Redesign
- **Multi-Channel Ingestion:** Redesigned 10+ CaptureFlows for modern multi-channel document ingestion
- **Field Operations:** Served as SPOC for MFD firmware upgrades across 100+ field offices
- **Standardization:** Implemented standardized processes across all regions for consistency

---

## Impact

Successfully upgraded mission-critical banking ECM infrastructure across 6 APAC countries from legacy v6.5 to modern v7.x, migrated entire platform to Azure cloud, and enabled modern cloud-first architecture supporting 100+ field offices with multi-channel document capture and processing.
