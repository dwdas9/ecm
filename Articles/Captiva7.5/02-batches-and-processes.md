# Batches and Processes in Captiva Capture

Batches and processes are the two fundamental concepts that govern how work flows through
Captiva Capture. If you work with this system regularly, understanding the relationship between
them saves a lot of troubleshooting time.

## What Is a Process

A process defines how a batch moves through the system. It is built from a CaptureFlow, which
is a sequence of module steps. Each step specifies which module runs, at which level of the
batch tree it triggers, and what IA values it reads and writes. You design processes using either
Captiva Designer (CaptureFlow Designer) or the older Process Developer tool.

When a batch is created, it is associated with a specific process. That process determines every
step the batch will go through from creation to export.

## What Is a Batch

A batch is a collection of pages organized into a tree structure with up to eight levels. Pages
(individual scanned sides) are at level 0, and the batch root is at level 7. Between those levels,
you can define documents, folders, and other groupings as needed.

Batches are created by task creation modules such as ScanPlus, Standard Import, or Web Services
Input. Once created, the server manages the batch through every step of the process.

Each batch has a numeric ID that is unique within a ScaleServer group. Batch data is stored in a
folder hierarchy on the server, named after the batch ID split into three digit groups. Inside that
folder are the batch file (`.iab`), a recovery file, and stage files that hold the image and data
output of each processing step.

## How Batches Are Processed

The server queues all batches by priority, from 0 (lowest) to 99 (highest). The default priority
is 50. A batch set to priority 0 is excluded from processing entirely. Batches with the same
priority are processed in order of creation time.

When a module is ready, the server pushes the next available task to it. If multiple machines
are running the same module, the task goes to whichever module responds first. The batch node
being processed is locked for the duration, so no two modules can work on the same node
concurrently.

After a module completes a task, it returns the result to the server and requests the next task.
The server then includes the processed node in a new task for the next module in the CaptureFlow.

## Processing Levels

Modules can be configured to process tasks at different levels of the batch tree. Processing at
a lower level (closer to 0) means the module handles one page at a time, which is more granular.
Processing at a higher level means the module handles a document or the entire batch at once.

The level at which a module step triggers is defined in the process. Most image processing and
OCR steps run at level 0. Export steps typically run at level 7.

## IA Values

IA values are variables that carry data between module steps. A module reads its input IA values
when it receives a task and writes its output IA values when it finishes. Those output values
become the input values for the next step.

IA values fall into several categories:

- **Input and output file values** point to stage files (images, PDFs, etc.) on the server.
- **Module data values** carry metadata like operator name, scan time, and error information.
- **Batch values** are tied to specific nodes in the batch tree and can be created dynamically.
- **Trigger values** control when a module step activates. A step only fires when its trigger
  values are set to a non-zero state.
- **System values** carry environment data such as `$user`, `$machine`, and `$server`.
- **Setup values** carry per-task configuration like scanner settings or OCR language.

IA values are declared in a module's MDF. Production IA values are the ones a module exposes
to the process for use by other modules.

## Department Routing

Department routing lets you route tasks to specific module instances. This is useful when you
need certain tasks handled by specific operators, machines configured for a particular language,
or workgroups with different security clearances.

There are two routing modes:

**Task-level routing** uses the `IATaskRouting` IA value to route individual tasks. An operator
starts their module instance specifying a department name, and that instance only receives tasks
tagged with that department.

**Step-level routing** uses the `IADepartments` IA value or a static department name set at
the step level. This is commonly used for load balancing where you want certain instances to
handle high-priority work.

Department values are not persistent across steps. If you set a department for the Completion
step, that value does not automatically apply to the next step. You need to set it explicitly for
each step that requires it.

ACLs can be applied to departments to restrict which users can access tasks belonging to that
department.