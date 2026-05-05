# Security Configuration in Captiva Capture

Captiva Capture security works in layers. Windows provides the authentication foundation,
Captiva roles handle action-based authorization, and ACLs handle object-level access control.
Getting these layers right before go-live is important because fixing them after users are already
running in production is more disruptive.

## Authentication

Captiva Capture uses Windows user accounts for all authentication. In all environments except
single-machine development or demo installations, these must be domain accounts. Supported
authentication providers are NTLM, Kerberos, and Negotiate (the default).

Kerberos is supported but requires some setup: it must be enabled on both client and server
machines, and a unique Service Principal Name (SPN) must be registered for each InputAccel
Server. The default `SecurityPackage` setting is `Negotiate`, which works with Kerberos
automatically once SPN registration is done correctly.

## Roles and ACLs

Security in Captiva Capture follows two tracks:

**Roles** define what a user can do. Each role has a set of permissions and a list of members.
Members inherit all permissions assigned to the role. Permissions cover actions like logging
into the server, reading and writing module data, and accessing batch and process data.

**ACLs** define access to specific objects: modules, batches, departments, and processes.
ACLs add a finer layer of control on top of roles.

Think of it this way: roles say "this user can run modules and read batches." ACLs say "but
only these specific batches or departments."

Captiva Capture ships with predefined default roles in Captiva Administrator. Review and
adjust these before adding users.

## Least-Privileged User Account (LUA)

The InputAccel Server is designed to run under a least-privileged user account. The installer
creates a dedicated group (`InputAccel_Server_admin_group`) and grants it only the specific
rights the server needs:

- Impersonate a client after authentication
- Load and unload device drivers
- Create global objects
- Full control of the InputAccel Server data directory (`c:\ias` by default) and its subkeys in
  the registry

If the group is accidentally deleted, the Administration Guide provides instructions for recreating it.

## Minimum Windows Permissions by Component

**InputAccel Database:** A SQL Server account with SQL Authentication enabled and Connect,
Delete, Execute, Insert, Select, and Update permissions.

**InputAccel Server:** Must be a member of `InputAccel_Server_admin_group` on the server
machine.

**Captiva REST Services:** The Application Pool identity for each REST Service web application
needs read/write/delete/create access to the shared data folder, and must be a member of the
`IIS_IUSRS` and `Performance Log Users` Windows groups on the web server machine.

**Client modules:** The user account running the module must have access to the relevant
directories (temp folders, watched folders, output directories) and must be assigned to a Captiva
role with at least the minimum production permissions.

**ClickOnce Deployment Utility:** The user deploying modules must be a member of the
local Administrators group on the target web server.

## Web Component Security

The following components are hosted by IIS and should be configured to use SSL:

- ClickOnce deployment
- Captiva REST Service
- Captiva Capture Web Client

Access to these components is controlled by the web server, Windows ACLs, licensing, and
Captiva Administrator roles working together. Configure HTTPS bindings in IIS Management
Console before exposing these components to users.

## Firewall Configuration

The InputAccel Server listens on port 10099 by default. You can change this after installation
via the TcpIpPort setting in Captiva Administrator under Server Settings. The InputAccel
Database communicates on TCP port 1433 (SQL Server default). Both ports need to be open
on any firewalls between the relevant components.

## Hardened Environments

Captiva Capture is designed to run in hardened Windows environments. This means it has been
tested with standard hardening practices: current security patches, disabled unnecessary services,
enabled firewalls, and blocked unused ports. Not every possible hardened configuration has been
tested, but the common ones are supported.

On Windows 8, 8.1, Server 2012, and Server 2012 R2, you must set User Account Control to
"Run all administrators in Admin Approval Mode: Disabled" in Local Security Policy for the
InputAccel Server to function correctly.

## Multi-Domain Environments

In a multi-domain environment, you need to establish domain trusts so cross-machine
authentication succeeds. The minimum required is a "Nontransitive One-Way External Trust"
from the domain containing the clients that need to authenticate, pointing toward the domain
containing the servers that perform the authentication.

Any user who logs into an InputAccel Server must also have the "Windows Login" privilege on
the machine hosting that server.

## Practical Tips

- Match Windows user groups to Captiva roles where possible to simplify permissions
  management.
- Consider a "Supervisors" role with Captiva Administrator access and full module rights, and a
  separate "Operators" role with module run rights only.
- Never use blank passwords. They are not supported in any configuration.
- When running modules as services under Network Service, add the machine account
  (`domain\machinename$`) to the appropriate Captiva role.
- Group all Network Service machine accounts into a domain group and add that group to the
  role. This makes it much easier to manage as the environment grows.