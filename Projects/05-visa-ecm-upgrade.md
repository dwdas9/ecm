# VISA Inc. - ECM Platform Upgrade
**Role:** Analyst / Technical Lead - ECM  
**Platform:** OpenText Documentum 6.5→16.4 · OpenText Intelligent Capture 6.5→16.6 · Azure · M365

---

## Overview

Part of a 30-member cross-functional team across Singapore, Foster City, and Denver. Responsible for the full lifecycle of Documentum and Captiva platforms alongside Azure and M365 workstreams. Managed major platform upgrade from v6.5 to v16.x with HA/DR re-architecture.

---

## Tech Stack

- **OpenText:** Documentum 6.5/16.4, IC 6.5/16.6 (ScanPlus, IndexPlus, Image Enhancement, Image Processor, Image Converter, Multi, ODBC Export, Documentum Advanced Export, eInput/eScan/eIndex, Dispatcher, .NET Code, ScaleServer)
- **Infrastructure:** Windows Server 2016, SQL Server 2016 AlwaysOn, MSCS Active/Passive Clustering, IIS 10
- **Azure:** VM, Azure SQL, Blob Storage, SharePoint Online, Power Automate, AI Builder, Azure DevOps

---

## Platform Upgrade (v6.5 → v16.x)

- **Architecture Redesign:** Upgraded OpenText Captiva and Documentum environment from v6.5 to v16.x on Azure VMs, re-architected to High Availability / Disaster Recovery (HA/DR) design
- **HA/DR Infrastructure:** Designed infrastructure leveraging MSCS Active/Passive Windows Server Failover Clustering (WSFC), SQL Server AlwaysOn Availability Groups, and Captiva ScaleServer for distributed workload processing
- **Cloud Coordination:** Coordinated with Infrastructure team to configure Azure VM sizing, OS provisioning, static IP allocation, service account creation, IAS folder permissions, and Azure Entra ID integration
- **Network Security:** Worked with Network and Security teams to configure firewall rules, inbound/outbound port whitelisting, and extended intranet connectivity for third-party scanning vendor integration
- **Workflow Modernization:** Redesigned 15+ legacy IPPs-based capture workflows to Captiva Designer-based processes
- **API Development:** Designed and developed SOAP-based validation APIs consumed by OpenText Captiva .NET Code Modules via proxy class integration; backend validation logic executed through SQL Server stored procedures
- **Cross-Team Coordination:** Coordinated across OpenText, Infrastructure, Network, and third-party scanning provider teams; led post-implementation support for stable production rollout

---

## BAU Support & Operations

- **L3 Support:** Functioned within the application owner team - accountable for L3 incident resolution, enhancement delivery, change request management, and stakeholder coordination
- **Patch Management:** Coordinated with Windows technical team for service pack and patch deployments; performed post-patch application validation
- **Database Maintenance:** Performed MSSQL backend maintenance - index rebuilds, statistics updates, query plan analysis, backup validation, and AlwaysOn health monitoring
- **Compliance & Auditing:** Enabled Documentum audit trail extensions and Captiva logging configuration for compliance, traceability, and operational monitoring
- **DR Drills:** Participated in DR failover drills - validated MSCS cluster failover, SQL Server AlwaysOn secondary promotion, Captiva ScaleServer failover, full application recovery within RTO/RPO
- **Enhancements:** Developed and modified CaptureFlow scripts, Captiva Designer workflows, and Documentum DFC customizations per change requests
- **Batch Monitoring:** Monitored batch processing queues and page counts via Captiva Admin Console; diagnosed and resolved failed batches, recognition errors, and export processing issues

---

## Impact

Successfully upgraded mission-critical payment network ECM infrastructure from legacy v6.5 to modern v16.x with enterprise-grade HA/DR, enabling 99.9%+ uptime for global payment processing across VISA's Singapore and US operations.
