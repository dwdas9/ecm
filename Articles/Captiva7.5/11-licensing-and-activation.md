# Licensing and Activation in Captiva Capture

Captiva Capture uses a server-based licensing system. All licenses live on the InputAccel
Server, and the server enforces them when client modules connect.

## How Licensing Works

Each license code is tied to a specific Server ID, which the InputAccel Server retrieves from its
security key. A license code specifies a single module and controls four things: how many
concurrent copies can connect, how many pages the module can process, how long the license
is valid, and what extra features are unlocked.

When a client module connects to the server, the server checks for a valid license before
allowing the module to operate. If no valid license exists for that module, the connection is
refused.

As of Captiva Capture 7.5, hardware security keys are deprecated. Licensing uses software
activation files (CAF files) instead.

## Activation

Each InputAccel Server requires a one-time internet activation step. You activate through the
EMC Captiva Activation Portal, which issues a CAF file keyed to that server. Keep your CAF
files in a safe location. If you ever need to reactivate a server (for example, after a hardware
change), you will need the original CAF file.

The default location for activation files on the server is `C:\ias\activation\*.*`. Back these
up before any server upgrade or hardware migration.

## ScaleServer Licensing

ScaleServer technology requires a specific feature code (feature code S) on each InputAccel
Server that participates in the group. A standard server license does not include this.

Within a ScaleServer group, servers share page count and connection licenses. This means that
if one server in the group reaches its daily page limit, it can draw from the remaining page
capacity of other servers in the group rather than going offline. The sharing happens
automatically without any intervention.

Example: if two servers are each licensed for 50,000 pages per day, the group has a shared
capacity of 100,000 pages per day. If Server 1 reaches its individual 50,000 limit three hours
before the end of the day, it borrows remaining capacity from Server 2 automatically.

ScaleServer licensing is included in certain license levels and available as an add-on in others.
Check with your account manager if unsure.

## Microsoft Failover Clustering Licensing

Running InputAccel Servers in a Microsoft Failover Clustering environment requires a separate
clustering license feature code. A standard InputAccel Server license does not permit clustering.
This applies to both Active/Passive and Active/Active cluster configurations.

## Disaster Recovery Licensing

Certain licensing tiers include disaster recovery licenses that cover implementing, testing, and
using a disaster continuation system. This licensing allows you to periodically test your DR
environment without counting those test runs against your production page limits. Contact your
account manager to confirm whether your current license level includes this.

## Upgrade Considerations

When upgrading Captiva Capture, you need to obtain new license codes for any new client
modules being introduced. Existing license codes for previously licensed modules typically
carry over, but verify this with EMC before upgrading in production.

Licenses, activation files, and security keys should all be archived as part of your pre-upgrade
backup. Identify each activation file by the server it belongs to.