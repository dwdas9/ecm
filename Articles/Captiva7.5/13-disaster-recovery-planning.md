# Disaster Recovery Planning for Captiva Capture

Disaster recovery for Captiva Capture ranges from simple offsite backups to fully redundant
multi-site clustering. Where your plan lands on that spectrum depends on how critical document
capture is to your business and what your recovery time objectives are.

## Creating a Written Plan

Before configuring anything technical, write a plan that answers these questions:

- Who are the key personnel responsible for rebuilding the system and restoring production?
- Who acts as backup for each key person if they are unavailable?
- Where is the documentation kept, and who can access it?
- How will you train replacement or temporary workers?
- How long will it take to restore full production throughput?
- What happens if the production facility itself cannot be used and you need to relocate?

A plan that exists only in people's heads is not a disaster recovery plan.

## What to Back Up

Certain files are irreplaceable and must be backed up before any upgrade, and periodically
during normal operations. Key items include:

- **Activation files** (`C:\ias\activation\*.*`) - CAF files for each server. Label each archive
  by the server it came from.
- **InputAccel Database** - Back up the full SQL Server database.
- **Batch and stage files** (`C:\ias\batches\*.*`) - All in-process images and batch data. Each
  server has a unique set; label accordingly.
- **Process files** (`C:\ias\process\*.iap`) - Compiled process files used in daily production.
- **Captiva Designer working directory** - CaptureFlows (XPPs), profiles, document types, and
  scripting files. Default location: `C:\Users\<username>\My Documents\Captiva <version>\Default`
- **Module Definition Files** - Custom MDF files created by your developers or EMC Consulting.
- **IPP source files** - For Process Developer-based processes.
- **Settings.ini** (`C:\ProgramData\EMC\InputAccel\Settings.ini`) - Per-machine tuning settings
  for client modules. Label each archive by the client machine it came from.
- **Registry parameters** - `HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\InputAccel\Parameters`
- **Module configuration files** (`C:\ias\modules\*.*`) - Templates, reference images, and
  other shared configuration files stored on the server.
- **Scanner drivers** and **license files**
- **Custom client and server software** from your developers

For all server-based files, check whether each InputAccel Server in a ScaleServer group has
unique data that needs separate archiving, or whether one archive covers the group.

## EMC Disaster Recovery Licensing

EMC offers disaster recovery pricing that covers licensing and activation for periodic testing of
your DR environment and one-time use of a disaster continuation system. Check whether your
current license level includes this. Testing your DR environment on regular production licenses
without this coverage may trigger license violations.

## Technology Options

The right combination of technologies depends on your recovery time and recovery point
objectives:

**ScaleServer groups** provide continued processing if one server in the group goes down. Other
servers keep running. Note that batches on the failed server cannot be processed until it recovers.

**Microsoft Failover Clustering (Active/Passive)** provides automated failover for the
InputAccel Server if a node fails. Requires clustering license.

**SQL Server high availability** (mirroring or clustering) protects the InputAccel Database from
being a single point of failure.

**Offsite backups** with rotating media covering recently scanned paper protects against
facility-level disasters.

**Multiple geographically separate clusters** with replicated Storage Area Networks provides
the highest level of protection for mission-critical operations.

## Involving EMC Consulting

Implementing a robust disaster continuation system is complex and environment-specific. EMC
Consulting Services can assist with planning, implementing, and testing disaster recovery
environments. For anything beyond simple backup/restore, involving them early is worth the
investment.