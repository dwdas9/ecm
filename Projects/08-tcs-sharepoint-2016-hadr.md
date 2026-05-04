# TCS Canada - SharePoint 2016 HA/DR Architecture
**Role:** Solution Architect / Project Manager  
**Organization:** Banking Client (CoE - Center of Excellence)  
**Platform:** SharePoint 2016 · SQL Server Database Mirroring

---

## Overview

Architected a 3-tier SharePoint 2016 farm with SQL Server database mirroring for High Availability and Disaster Recovery across primary and secondary data centres as part of the ECM modernization initiative.

---

## Tech Stack

- **SharePoint:** SharePoint 2016 (3-tier farm architecture)
- **Infrastructure:** SQL Server 2012-2016 (Database Mirroring), Windows Server 2012 R2/2016, IIS 8.5
- **Cloud:** Azure VM
- **Backup & Disaster Recovery:** NetBackup

---

## Architecture Design

### Three-Tier Farm Architecture

1. **Web Front-End (WFE) Tier** — User-facing SharePoint servers
2. **Application Tier** — Central admin, search, services
3. **Database Tier** — SQL Server with database mirroring

### High Availability & Disaster Recovery

- **Database Mirroring:** Principal/Mirror/Witness configuration
- **Geographic Distribution:** Primary and secondary data centres
- **Failover Strategy:** Automated failover with RPO and RTO optimization
- **Data Consistency:** Synchronous mirroring for critical databases

---

## Key Achievements

- **3-Tier Farm Design:** Architected modern 3-tier SharePoint 2016 farm following Microsoft best practices
- **Database Mirroring:** Configured SQL Server database mirroring (principal/mirror/witness) across data centres
- **High Availability:** Designed infrastructure for high availability with automated failover capability
- **Disaster Recovery:** Implemented geo-distributed HA/DR with primary and secondary data centre support
- **Data Centre Distribution:** Deployed infrastructure across primary and secondary data centres for geographic redundancy
- **Performance Optimization:** Optimized for performance and scalability across enterprise user base
- **Compliance & Backup:** Integrated with enterprise backup and disaster recovery processes

---

## Impact

Successfully architected enterprise-grade SharePoint 2016 infrastructure with geographic redundancy and database mirroring, enabling high availability and disaster recovery for mission-critical banking and ECM applications.
