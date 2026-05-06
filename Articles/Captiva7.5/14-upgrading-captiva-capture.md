# Upgrading Captiva Capture

Upgrading Captiva Capture requires careful sequencing. Getting the order right avoids
compatibility issues that are tedious to undo.

## Supported Upgrade Paths

Upgrades to version 7.5 are supported from: 6.0 SP3, 6.5, 6.5 SP1, 6.5 SP2, 7.0, and 7.1.

Versions prior to 6.0 SP3 cannot upgrade directly to 7.5.

## Upgrade Sequence

Install or upgrade components in this order. Do not skip or reverse steps.

1. InputAccel Database (required for users upgrading from 6.0 SP3 or 6.5.x with an external
   database)
2. InputAccel Servers
3. Captiva Capture Web Client and Captiva REST Services (if applicable)
4. Existing client modules
5. New client modules (optional, can be added post-upgrade)

## Before You Start

**Complete in-process batches** if either of the following is true: any modules will use a
different code page after the upgrade, or you are using a custom module that reads or writes
binary IA values.

**Stop all running components:** client modules running as applications, client module services,
and all InputAccel Server instances.

**Archive irreplaceable files.** See the Disaster Recovery article for the full list. This step is
not optional. The upgrade cannot be rolled back reliably without these archives.

**Back up custom modules.xml:** If you added custom modules in previous versions of
CaptureFlow Designer, back up `modules.xml` and copy its contents to
`src\MDF\User\UserModules.xml` before upgrading.

**Test first.** Run the full upgrade in a test environment before touching production. Use the
IAMigrate application to migrate configurations from test to production after a successful test
upgrade.

## Automatic Backups During Upgrade

The setup programs automatically create backup directories during upgrade:

- `C:\Program Files\InputAccel\Server\$InputAccelServer<version>$` - Server file backups
- `C:\Program Files\InputAccel\Client\$InputAccelClient<version>$` - Client file backups

Keep these backup directories until you are confident the upgraded system is running correctly
and you have ruled out any possibility of rollback.

## Client Compatibility Notes

Not all client modules need to be upgraded. Some older versions can connect to a 7.5 server
under specific conditions:

- 6.0 SP3, 6.5, 6.5 SP1, and 6.5 SP2 clients can connect to a 7.5 server only if it has an
  upgraded external database (not a newly installed database). Server and client must also be
  in the same locale using the same code page.
- 7.0 and 7.1 clients can connect to a 7.5 server if the database type (external or internal)
  matches what was used in the previous version.
- Clients from 6.0 and earlier cannot connect to a 7.5 server at all.

**Important:** Once you modify a process with 7.5 setup data or create a new 7.5 process,
that process can only be used with 7.5 modules. Do not configure any module setup using 7.5
modules or create 7.5 processes until all components have been upgraded to 7.5.

## Upgrading the InputAccel Database

Before upgrading the production database, create a full backup and test the upgrade steps in
a separate environment. If the database upgrade fails, restore the backup, resolve the issue,
and try again.

Shut down all client modules, the InputAccel Remoting Server, and all InputAccel Servers
connected to the database before running the database upgrade installer.

## Upgrading the InputAccel Server

Run the client setup program on the Administration Console host machine to upgrade Captiva
Administrator automatically. Note that Administration Console is no longer supported and
cannot connect to the InputAccel Database in 7.5.

If upgrading InputAccel Servers in a Microsoft Failover Clustering environment, follow the
cluster-specific upgrade procedure in the Install Guide.

## Upgrading Captiva REST Services

You only need to upgrade REST Services if you want to use Captiva Web Client or the Ad Hoc
services feature from REST Services 2.0. To upgrade, run the Captiva Capture installer on the
machines where REST Services is currently installed. Note that the upgraded REST Service uses
a different default URL, so existing clients need to be reconfigured to the new URL after upgrade.

## Scheduling the Upgrade

A few scheduling recommendations:

- Determine which components to upgrade in each phase and locate all installation media
  before starting.
- Choose upgrade timing carefully. Upgrading the night before the last production day of the
  week gives you a full production day to catch issues, followed by a non-production weekend
  to resolve them.
- Contact EMC Support if you hit major issues during upgrade.

## Migrating Process Developer Processes

If you have existing Process Developer (IPP-based) processes, you can continue using them or
migrate them to CaptureFlow Designer. Migration to CaptureFlow Designer is recommended
because CaptureFlow processes are easier to maintain, support deployment directly from
Designer, and compile to .NET rather than the deprecated VBA runtime.

When redesigning a Process Developer process in CaptureFlow Designer, start with a technical
design documenting control flows, trigger levels, and dependencies. Check module guides to
understand whether any old module steps should be replaced with newer equivalents.