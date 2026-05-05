# User Roles in Captiva Capture

Captiva Capture uses a role-based access model. Roles are managed in Captiva Administrator and
control what users can do across the entire system.

## The Three Core Roles

The system is designed around three main roles:

**Designer** creates CaptureFlows and capture profiles. A designer defines how documents move
through the system, what modules run at each step, and what IA values pass between them.
Most environments have only one or a small number of designers.

**Administrator** handles day-to-day system operations. This includes managing servers,
assigning user roles, reviewing logs, monitoring batch traffic, and keeping the system running.
Like the designer, most environments have only one or a small number of administrators.

**Operator** uses one or more production modules to perform manual tasks. This includes
scanning documents in ScanPlus, reviewing and correcting data in Completion, and handling
rescan tasks. Operators make up the majority of users in a production environment.

A single person can hold more than one role, which is common in smaller deployments.

## Role Permissions and ACLs

Roles in Captiva Capture have two components: a set of permissions and a set of members (users
or groups). Members of a role inherit all permissions assigned to that role.

Permissions are for actions. ACLs are for objects. Role permissions control what a user can
do (start a module, read batch data, modify a process). ACLs control access to specific things
(a particular batch, department, module instance, or process).

Use roles as your first layer of security and ACLs for finer-grained control where needed.

## Minimum Permissions to Run a Module

For a user to run a module in production mode, their role must include at minimum:

- `Server.Login`
- `Server.Read.Module.Data`
- `Server.Write.Module.Data`
- `System.BatchRead`
- `System.BatchModify`
- `System.ProcessRead`

Some modules require additional permissions beyond this base set. Check the specific module's
documentation for details.

## The Default Administrator Account

During installation, the setup program adds the administrative user account to the Administrator
role. This role has full permissions across all components. That administrator is responsible for
setting up all other roles and adding users to them before anyone else can run client modules in
production.

Captiva Administrator ships with default predefined roles. Review these defaults before going
live. Adjust permissions and add new roles to match your organization's security requirements.

## Running Modules as Services

When a module runs as a Windows service under a domain user account, that account must be
a member of the appropriate Captiva role. When a module runs under the Network Service
account, the machine account (in the form `domain\machinename$`) must be added to the
relevant role.

A practical tip: add all machine accounts running under Network Service to a single domain
group, then add that group to the role. This simplifies ongoing maintenance as machines are
added or removed.

## Passwords and Account Requirements

Captiva Capture requires all user accounts to have passwords. Blank passwords are not
supported in any scenario, including single-machine installations. Passwords must not contain
the `@` symbol, as it is used as a delimiter in command-line arguments.

Except for single-machine development or demonstration installations, user accounts must be
domain accounts.