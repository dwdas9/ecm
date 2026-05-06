# System Administration Essentials

Captiva Administrator is the single tool for managing all aspects of a running Captiva Capture
system. If you are the person keeping the system healthy day-to-day, most of your work happens
here.

## What Captiva Administrator Manages

Captiva Administrator gives you visibility and control over the following:

- CaptureFlow definitions deployed to the server
- Batch data in real time as it moves through processing
- User departments, roles, and permissions
- Servers and ScaleServer group configuration
- Web Services configuration (Coordinator and Hosting)
- Licensing
- System logs

Note that Captiva REST Service and Module Server licensing is managed separately through the
Captiva REST Services Licensing tool, not through Captiva Administrator.

## Monitoring Batch Traffic

The Batch Traffic view shows all batches currently on the server, their processing state, priority,
and which module steps they are waiting on. This is your first stop when something in production
appears stuck or delayed.

The Processes screen lists all deployed processes along with batch counts per state. A hidden
Legacy column indicates whether a process uses the older CodeBehind-based IPP format or the
newer CaptureFlow XPP format. This distinction matters when managing deployment operations.

## Managing Servers and ScaleServer Groups

From Captiva Administrator, you can view all connected InputAccel Servers, check their status,
and configure ScaleServer group membership. When adding a new server to a ScaleServer group,
the license codes for that server must include the ScaleServer feature code before the server can
participate.

Each InputAccel Server must have its own activation file (CAF file) installed and activated. The
activation step requires a one-time internet connection to the EMC Captiva Activation Portal.

## Managing Modules and Connections

The Modules view shows which modules are currently connected to the server and their
operational state. This is useful for confirming that unattended services came up after a restart,
or for identifying a module that has gone offline unexpectedly.

The Connections view shows active network connections to the server.

## Departments

Departments are configured under Systems in the navigation panel. Each department can have
an ACL controlling which users can access tasks belonging to it. Creating departments before
putting a process that uses department routing into production is a required setup step.

## Logging

Captiva Administrator exposes system logs for review. If Audit Logging is enabled, those records
are written to the InputAccel Database and can be queried with a third-party reporting tool.
Enabling full audit logging writes a significant volume of data, so plan your database storage
accordingly.

## The Legacy Administration Console

The older Administration Console is no longer supported. It may still connect and work for some
basic tasks, but it does not support any features introduced in Captiva Capture 7.x. All
administrative work should go through Captiva Administrator.