# CaptureFlow Deployment: Captiva Designer vs Captiva Administrator

With Captiva 7.5, Captiva Designer became the primary tool for deploying CaptureFlow-based
processes. Several deployment operations that previously went through Captiva Administrator
are now either handled by Designer or are restricted to legacy (IPP-based) processes only.
Understanding what has changed prevents confusion when working with a mix of legacy and
modern processes.

## The Core Distinction: Legacy vs CaptureFlow Processes

In Captiva Administrator's Processes screen, a hidden "Legacy" column identifies whether
a process uses CodeBehind (CaptureFlow/XPP-based) or the older IPP format. When the
Legacy flag is unchecked, the process is a CaptureFlow process and several Captiva
Administrator deployment operations are either disabled or behave differently.

You can make the Legacy column visible through the Column Manager in Captiva Administrator.

## What Changed in 7.5

The following Captiva Administrator features are now either deprecated for CaptureFlow
processes or disabled entirely:

**Add Process:** Available only for legacy processes. Captiva Administrator will return an error
if you attempt to deploy a CodeBehind IAP file directly to the server this way. For CaptureFlow
processes, use Captiva Designer's deployment function instead, which handles deploying the
CodeBehind DLL and autogen DLL automatically.

**Add Upgraded Process:** Available only for legacy processes. For CaptureFlow processes,
use Captiva Designer. When you invoke deployment from Designer, it removes the old IAP,
CodeBehind DLL, and autogen DLL and replaces them with newly generated files.

**Copy Process:** Available only for legacy processes. For CaptureFlow processes, Captiva
Designer handles deployment to all servers in a ScaleServer group automatically. Using the
old Copy Process feature for a CaptureFlow process is unnecessary and not supported.

**Delete Process:** The legacy Delete Process only removed the IAP file. Captiva Administrator
has been enhanced for CaptureFlow processes to delete all process artifacts: the IAP, XPP,
CodeBehind DLL, and autogen DLL under the IAS root process\SystemFiles directory. Delete
is disabled for a CaptureFlow process if the server has batches currently created from that
process, because deleting it while batches reference it would break batch execution.

**Rename Process:** Disabled for CaptureFlow processes. Because CodeBehind execution
depends on the deployed process name, the name field is read-only for CaptureFlow processes
in Captiva Administrator.

**Copy and Paste Settings:** This legacy feature for copying module settings between processes
is disabled for CaptureFlow processes. It has been replaced by proactive sync in Captiva
Designer.

## Proactive Sync

When you open module settings in CaptureFlow Designer, Designer connects to the Captiva
server and compares the module settings in the deployed process against the settings stored
in the local XPP file. If there is a mismatch, Designer presents a dialog asking you to choose:
- Download module settings from the server and apply them to the local XPP, or
- Use the local settings and overwrite the module settings on the server.

This replaces the manual Copy and Paste Settings workflow.

## Summary

For any CaptureFlow process (Legacy flag unchecked), Captiva Designer is the authoritative
deployment tool. Use it for deploying, updating, and managing process versions. Use Captiva
Administrator for day-to-day operational tasks: monitoring batch traffic, managing roles and
permissions, licensing, and logging.