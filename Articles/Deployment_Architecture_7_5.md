# Deployment Architecture in Captiva 7.5: Designer vs. Administrator

## Introduction

The shift from Captiva Administrator as the primary deployment tool to Captiva Designer marks a fundamental change in how capture processes move from development to production. Understanding this transition isn't just about learning new UI workflows-it's about grasping the architectural rationale behind why certain operations are now restricted, why others have been completely reimplemented, and how the platform's treatment of legacy processes creates a dual-path deployment model that must be navigated carefully.

This article examines the deployment behavior changes introduced in Captiva 7.5, focusing on the technical constraints that emerge when CodeBehind processes encounter the legacy deployment feature set.

---

## The Dual-Path Deployment Model

Captiva 7.5 introduces a critical distinction: processes are now classified as either **legacy** or **CodeBehind-enabled**. This classification determines which deployment operations are available and, more importantly, which ones will fail silently or with cryptic error messages.

### Why This Distinction Exists

In earlier versions, all processes were monolithic IPP files compiled as opaque binaries. Captiva Administrator could manipulate these binaries through well-defined administrator operations: copy, delete, rename, upgrade.

CaptureFlows change this equation. A CaptureFlow compiles to an IAP file (still binary), but the supporting artifacts-CodeBehind script DLLs and autogen DLLs-exist as separate files in the process's SystemFiles directory. This decoupling creates a problem that legacy processes don't face: if you delete the IAP but leave the DLLs behind, or if you rename the IAP without updating the DLL references, the system becomes inconsistent.

The solution: restrict administrator operations on CodeBehind processes and require developers to use Captiva Designer instead, where deployment logic understands the full artifact graph and can maintain consistency automatically.

---

## The Legacy Process Column

The foundation of this feature gating is a hidden column in the Captiva Administrator Processes view labeled **"Legacy"**. This boolean flag indicates whether a process uses CodeBehind:

- **Legacy flag checked**: Process uses traditional IPP architecture. All legacy deployment features are available.
- **Legacy flag unchecked**: Process is CodeBehind-enabled (a CaptureFlow). Legacy deployment features are disabled.

This column is hidden by default, but can be surfaced through the **Column Manager** dialog in the Processes view. The decision to hide it reflects that it's a diagnostic artifact-end users should not need to know this classification. However, developers and administrators must understand it to navigate feature availability.

![EMC Captiva Administrator Column Manager showing the Legacy column selection](images/fig_legacy_column.png)
*Figure 1: The Column Manager dialog in EMC Captiva Administrator displays available columns for the Processes view. The "Legacy" column (highlighted in red) indicates process type: checked for legacy processes, unchecked for CodeBehind-enabled processes.*

---

## Feature Availability Matrix

The operational impact of the legacy/CodeBehind distinction is precise and deterministic. Here is the complete feature availability table:

| Operation | Legacy Process | CodeBehind Process | Rationale |
|---|---|---|---|
| **Add Upgraded Process** | ✓ Available | ✗ N/A | Designer handles versioning natively |
| **Add Process** | ✓ Available | ✗ Error if attempted | Requires CodeBehind DLL deployment |
| **Delete Process** | ✓ Always available | ✓ Available only if no batches exist | DLL decoupling prevents safe deletion with active batches |
| **Rename Process** | ✓ Available | ✗ Read-only | CodeBehind execution depends on process name |
| **Copy Process** | ✓ Available (single to multiple servers) | ✗ Designer handles multi-server deployment | Multi-server deployment is automatic in Designer |
| **Copy Settings** | ✓ Available (copy, paste) | ✓ Single Setup Value only | Proactive sync replaces legacy copy/paste |
| **Paste Settings** | ✓ Available (single setup value) | ✓ Single Setup Value only | Proactive sync replaces legacy copy/paste |

This matrix is not intuitive on first reading. The reasoning becomes clear only when examining each operation in detail.

---

## The Add Upgraded Process Operation

**Availability:** Legacy processes only

The "Add Upgraded Process" feature is a delete-and-replace mechanism. When you invoke it, you select a newer IAP or IPP file, and the administrator performs these steps:

1. Backs up the existing process (optional checkbox)
2. Backs up the process metadata (optional checkbox)
3. Removes the old deployed IAP and all associated files
4. Deploys the new file
5. Applies the upgrade to selected servers in a ScaleServer group

![Upgrade Process dialog showing process selection and server selection interface](images/fig_upgrade_process.png)
*Figure 2: The Upgrade Process dialog in Captiva Administrator. Users select a replacement IAP or IPP file, optionally back up the current process, and select target servers from the Available Servers and Selected Servers lists.*

For CodeBehind processes, this operation is **not available**. Why? Because Captiva Designer already implements this as the standard deployment model. When you deploy a CaptureFlow in Designer, it:

1. Connects to the IC Server
2. Detects the currently deployed version
3. Removes the old IAP, CodeBehind DLL, and autogen DLL
4. Deploys the newly compiled artifacts

The Designer implementation is superior because it understands the full dependency graph. Attempting to use Administrator's Add Upgraded Process on a CodeBehind process would bypass this logic and potentially leave orphaned DLLs or broken references. The feature gate prevents this mistake.

---

## The Add Process Operation

**Availability:** Legacy processes only

The "Add Process" operation (labeled "Install Process" in the UI) deploys a single IAP or IPP file to specified servers. The dialog collects:

- Process name
- Path to the IAP/IPP file
- Priority for new batches created from this process
- Target servers (multi-select)
- Optional description field

![Install Process dialog showing process file selection and multi-server deployment options](images/fig_install_process.png)
*Figure 3: The Install Process dialog in Captiva Administrator. It displays an error message indicating a CodeBehind process ("The process was compiled in CFD. Use CFD to deploy it"), demonstrates the file path selection field, server selection interface, and description area.*

For legacy processes, this is straightforward: copy the file, register it with the server.

For CodeBehind processes, **this operation fails with an explicit error message**: "The process was compiled in CFD. Use CFD to deploy it." (CFD is the internal designation for CaptureFlow Designer.)

The technical reason: the administrator has no way to deploy the CodeBehind DLL and autogen DLL that accompany the IAP. The file system layout on the server for a CodeBehind process looks like this:

```
IAS_root/
  processes/
    ProcessName/
      ProcessName.IAP          <- Compiled CaptureFlow graph
      SystemFiles/
        CodeBehind.dll         <- User's C# code module
        autogen.dll            <- Designer-generated wrappers
        *.xpp                  <- Embedded XML definitions
        [other artifacts]
```

The Administrator has no context about which DLLs are correct, where to find them, or how to validate their compatibility. Designer knows all of this from the compilation step. The feature gate enforces the invariant: CodeBehind processes are deployed only via Designer.

---

## The Delete Process Operation

**Availability:** Legacy processes (always) | CodeBehind processes (conditional)

Delete is the most complex operation in the matrix because its availability differs by process type *and* by runtime state.

### Legacy Processes

For legacy processes, Delete removes the deployed IAP and associated files. The operation is unrestricted-there is no batch state check. This reflects the monolithic nature of legacy processes: the IPP is self-contained, so deletion is safe regardless of active batches.

### CodeBehind Processes

For CodeBehind processes, the delete constraint is strict: **deletion is allowed only if no batches exist that were created from this process**.

Why this constraint? Because the CodeBehind DLL and autogen DLL are decoupled from the process definition. Consider this scenario:

1. Process "Invoice" version 1 is deployed with `Invoice_v1.dll`
2. Batch B001 is created from "Invoice" and runs several tasks
3. A developer deploys "Invoice" version 2, which removes v1 and deploys `Invoice_v2.dll`
4. Batch B001 is sitting in the Identification queue, waiting for operator review
5. The operator opens B001 to review a field

At this point, the system attempts to load the CodeBehind DLL for B001, but version 1 has been deleted. The batch enters an error state.

By preventing deletion when active batches exist, the platform enforces an important invariant: active batches always have their supporting DLLs available. Note that "active" in this context means "exists in the batch tree with any non-terminal state," not "currently being processed."

---

## The Rename Process Operation

**Availability:** Legacy processes only

The Rename operation changes the deployed process name on the server. For legacy processes, this is a file system rename operation-move the IAP file from one name to another and update the server's configuration.

For CodeBehind processes, the Name field in the Process Settings dialog is **read-only**. Attempting to edit it yields no effect.

![Process Settings dialog showing the read-only Name field for a CodeBehind process](images/fig_process_settings_readonly.png)
*Figure 4: The Process Settings dialog displays a CodeBehind process with the Name field marked as read-only. Additional fields show compile time, InputAccel process version, original capture flow name, and process compiler version. The name field cannot be edited for CodeBehind processes.*

The technical constraint is that CodeBehind execution has a hard dependency on the process name. When the IC Server routes a task to the CodeBehind module, it passes the process name. The module loads the corresponding DLL from the process directory and invokes it. If the process is renamed but the DLL is not moved, or if the DLL exists in the old directory, execution fails with an obscure error.

Designer manages this constraint automatically: if you rename a process in CaptureFlow Designer and redeploy, Designer updates all references and moves all files to the new directory. But the Administrator cannot do this atomically, so rename is simply not allowed.

---

## The Copy Process Operation

**Availability:** Legacy processes only

The Copy Process operation deploys a single process from one server to multiple servers within a ScaleServer group. The UI presents two lists-Available Servers and Selected Servers-with directional arrows to move servers between them.

![Copy Process dialog showing multi-server deployment interface](images/fig_copy_process.png)
*Figure 5: The Copy Process dialog in Captiva Administrator. It shows a process being copied (PERF(75)-2014-11-06 from KT-2012-IAS1), with Available Servers and Selected Servers lists and navigation buttons to move servers between them.*

This operation has no equivalent in CaptureFlow Designer because Designer's deployment model is inherently multi-server. When you deploy a CaptureFlow in Designer, you specify the IC Server and Designer automatically distributes the process to all configured scale group members. The deployment is coordinated, atomic, and consistent.

For legacy processes, the Administrator's Copy operation is the mechanism to propagate a single process across a scale group. For CodeBehind processes, the Developer uses Designer's built-in multi-server deployment, which is far superior.

---

## Copy and Paste Settings: The Dual Implementation

**Availability:** Legacy processes (full) | CodeBehind processes (Single Setup Value only)

This is where the migration story becomes most visible. Settings management has been completely reimplemented between legacy and CodeBehind processes.

### Legacy Copy and Paste Settings

For legacy processes, you can:

1. **Copy Process**: Select a process and copy all its settings. The UI offers options for which settings to include.
2. **Copy Settings**: Choose between copying all settings or just a single setup value.
3. **Paste Settings**: Paste either all copied settings or just a single setup value to another process.

This workflow is useful when you have multiple similar processes that share module configurations (e.g., five different OCR processes with nearly identical NuanceOCR settings). Copy settings from one, paste to the others, then make minor adjustments.

### CodeBehind: Proactive Sync

For CodeBehind processes, the legacy Copy and Paste workflow is replaced by a runtime mechanism called **proactive sync**. Here's how it works:

1. **In CaptureFlow Designer**, you select a module step and open Module Settings.
2. **Designer connects to the IC Server** and detects whether module settings in the deployed process differ from settings stored in the CaptureFlow XPP file (the source file).
3. **If there is a mismatch**, Designer presents a dialog with two choices:
   - **"Use server settings and overwrite local"**: Download the current module settings from the server and apply them to the local XPP.
   - **"Use local settings and overwrite server"**: Keep local settings and update the server's deployed process.

This mechanism is superior to legacy copy/paste because:

- **It detects drift automatically**. You don't have to remember to synchronize; Designer detects inconsistencies the moment you open Module Settings.
- **It is bidirectional**. You can pull settings from the server (useful if an operator or administrator modified settings post-deployment) or push local settings to the server.
- **It is granular**. The sync is per-module, not per-process, so you can update one module's settings without touching others.

### Single Setup Value Constraint

For CodeBehind processes, both Copy and Paste are restricted to **Single Setup Value** operations. This means:

- You cannot copy all settings at once from a CodeBehind process via the Administrator UI.
- You must copy individual setup values one at a time.
- This restriction encourages developers to use proactive sync in Designer instead.

The rationale is that bulk copy/paste in Administrator, while possible, undermines the proactive sync workflow. If you can bulk-paste settings in Administrator, you lose the automatic drift detection that proactive sync provides. By restricting to single values, the platform nudges developers toward the Designer-based workflow.

---

## Deployment at Install Time

When installing a new CodeBehind process or updating an existing one via Designer, two additional options are available that have no direct Administrator UI equivalent:

### Option 1: Copy Settings from Existing Process

When deploying a new version or a related process, Designer offers to copy module settings from another already-deployed process and apply them to the current process. This is useful when:

- You have a "master" process with carefully tuned module settings
- You are deploying a variant that should start with the same settings
- You want to avoid manual reconfiguration

### Option 2: Download Settings from Current Server Deployment

If the process is already deployed and you are updating it, Designer can download the current module settings from the server and apply them to the new build. This preserves operator-made or configuration-made adjustments when you update the process code.

These two options, available at deployment time in Designer, replace the post-deployment copy/paste workflow that legacy processes required.

---

## Practical Migration Implications

Understanding this deployment model is critical for projects migrating from Captiva 7.x to IC 24 (formerly known as Captiva 8+) because:

1. **Legacy processes will continue to work** through IC 24 via backward compatibility, but they cannot be deployed or managed via the modern Designer-based tools.

2. **Immediate decision**: When you begin a migration, you must decide which processes to port to CaptureFlow and which to keep as legacy. There is no mixed mode-once you convert a process to CaptureFlow, it leaves the legacy ecosystem entirely.

3. **Deployment becomes part of development**: In Captiva 7.x, deployment was often a separate step handled by administrators. In IC 24, deployment is integrated into the Designer workflow. Developers must understand deployment constraints because they affect design decisions.

4. **Settings management changes fundamentally**: The legacy copy/paste workflow is gone. Developers must adopt proactive sync and multi-process templates to manage shared configurations. This is actually an improvement, but the transition requires a mental shift.

5. **Batch lifecycle constraints are now enforced**: In Captiva 7.x, administrators could delete or rename processes even with active batches. In IC 24, this is prevented. Deployment scripts and CI/CD pipelines that relied on process deletion for cleanup must be rewritten.

---

## Summary

The deployment behavior changes in Captiva 7.5 reflect a broader architectural shift from administrator-centric to developer-centric process management. The legacy/CodeBehind distinction is not merely a UI feature gate-it is an expression of two fundamentally different process architectures and their respective deployment models.

For developers coming from Captiva 7.x, the key insight is this: **do not use Captiva Administrator to manage CodeBehind processes**. The restrictions are not arbitrary limitations; they are guardrails that preserve system consistency and prevent silent failures. Use Captiva Designer for deployment, and use Designer's proactive sync for settings management.

For administrators, the message is equally clear: **legacy processes are maintained for backward compatibility, but all new development should target CaptureFlow**. The restrictions on CodeBehind processes in Administrator are intended to be terminal-they are signs that you should be using Designer instead.

Understanding this distinction transforms what appears to be a confusing feature gate into a logical, defensible architectural boundary.
