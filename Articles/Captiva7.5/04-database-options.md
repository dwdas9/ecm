# Database Options in Captiva Capture

Captiva Capture gives you a choice between two types of databases for storing configuration
settings and system data. Choosing the right one for your environment depends on scale,
availability requirements, and operational complexity.

## External SQL Server Database

The external database option uses Microsoft SQL Server. This is the recommended choice for
any production environment. It is required if you want to use ScaleServer groups (multiple
InputAccel Servers working together).

A few things to keep in mind when planning an external database:

- The database server should have the highest-speed, lowest-latency network connection
  available. Database performance directly affects overall capture throughput.
- SQL Server must be configured with TCP/IP enabled on port 1433. SQL Server Express, in
  particular, disables TCP/IP by default. You must enable it before installing the InputAccel
  Database.
- Use SQL Server and Windows Authentication mode (mixed mode), not Windows Authentication
  only.
- The account used to access the database needs these permissions: Connect, Delete, Execute,
  Insert, Select, and Update. Using the `sa` account works but is generally not recommended.
- The InputAccel Database is case-insensitive, even if installed on a case-sensitive SQL Server.
- Enabling Audit Logging and Reporting writes significant amounts of data to the database.
  Plan your storage accordingly.

If you run both a test and a production InputAccel Database, use separate SQL Server instances
at minimum, and separate machines where possible. This prevents test workloads from affecting
production performance.

## File-Based Internal Database

The file-based internal database stores configuration data in files on the InputAccel Server
machine itself. It does not require a SQL Server installation. This makes it simpler to set up for
development, demonstration, or very small deployments.

The tradeoff is that it does not support ScaleServer. If you need multiple InputAccel Servers
working together, you must use the external SQL Server database.

## SQL Server Express Limitations

SQL Server Express can be used for development or low-volume environments, but it has
meaningful limitations that make it unsuitable for production:

- SQL Server 2012 Express has a 10 GB database size cap. Once you hit that limit, you must
  manually purge batches before the system can continue.
- SQL Server Express does not support failover or high availability configuration.
- It uses at most 1 GB of RAM and a single CPU core, regardless of what the machine has
  available.
- It creates a named instance by default. Named instances require the instance name in every
  database connection string. To avoid this, create an unnamed instance during installation.
- Dynamic ports are enabled by default. Disable dynamic ports and set a fixed port (1433) in
  SQL Server Configuration Manager.

## Multiple Server and Database Configurations

A single InputAccel Server only needs one InputAccel Database. Multiple independent
InputAccel Servers can share the same database or have separate databases depending on your
business requirements.

A ScaleServer group, however, must have exactly one shared InputAccel Database. All servers
in the group must access the same database instance.

Separate Captiva Capture deployments that need to remain fully isolated from each other
require separate InputAccel Databases.