# ScaleServer Groups and High Availability

Captiva Capture provides several mechanisms for scaling up and keeping the system available
during failures. Understanding what each one does and does not do helps you design the right
architecture for your environment.

## ScaleServer Groups

A ScaleServer group combines between two and eight InputAccel Servers into a single logical
capture system. From the perspective of a client module, the group appears as a single server.
Modules that are ScaleServer compatible can connect to all servers in the group simultaneously
and receive tasks from any of them.

Key points about ScaleServer:

- All servers in the group must share the same external SQL Server InputAccel Database. The
  file-based internal database cannot be used with ScaleServer.
- Each batch lives on exactly one server in the group. Once a batch is assigned to a server, that
  server manages it through its entire processing cycle.
- Each server does its own task scheduling independently. There is no inter-server coordination
  for task routing within a batch.
- ScaleServer provides load balancing and increased throughput. It does not provide data
  redundancy or automatic batch failover. If a server in the group goes down, the batches on
  that server cannot be processed until the server comes back up.
- Servers in a ScaleServer group share page count and connection licenses automatically.

To verify a ScaleServer group is working, start a ScaleServer-compatible module, select the
"Connect to server group" checkbox at login, then check Captiva Administrator to confirm the
module appears as connected to all servers in the group.

## Side-by-Side InputAccel Server Instances

On high-end server hardware with multiple CPU cores, you can install multiple instances of the
InputAccel Server on a single machine. This can improve parallel batch execution on
multi-processor hardware. Each instance requires its own license. This is also the basis for
Active/Active Microsoft Failover Clustering with two InputAccel Server instances on two nodes.

## Microsoft Failover Clustering

Captiva Capture supports Microsoft Failover Clustering for the InputAccel Server. Two modes
are supported:

**Active/Passive:** One server instance runs actively, the other is on standby. If the active
node fails, the cluster automatically starts the server on the standby node.

**Active/Active:** Two server instances run simultaneously, typically configured as a ScaleServer
group. If one node fails, the cluster moves its resources to the other node.

Note: Captiva Capture has been tested on two-node clusters. Other configurations may work but
are not officially supported.

## High Availability Best Practices

Beyond ScaleServer and clustering, these infrastructure practices improve overall availability:

- Connect the InputAccel Server machine and the database machine to an uninterruptible power
  supply as a minimum.
- Configure SQL Server for high availability using database mirroring or clustering.
- Run all unattended client modules as Windows services with automatic restart on failure
  enabled.
- Use high-performance RAID arrays for data storage redundancy.
- Use rack-mount or blade server hardware with redundant power supplies for key components.
- Consider VMware VMotion as an alternative to clustering for moving virtual machines between
  hosts during failures.
- New client modules can be brought online at any time to supplement or replace existing
  instances without disrupting production.

## Client Scalability

Adding more module instances is the primary way to scale throughput at the client layer. If a
particular step is the bottleneck (OCR, for example), add more machines running that module.
The server automatically distributes tasks across all available instances of a given module.

When planning client counts, consider: the volume of incoming documents, the processing time
each module needs per task (OCR takes far more time per page than export), and operator
throughput for attended modules.