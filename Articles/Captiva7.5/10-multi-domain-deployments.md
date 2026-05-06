# Captiva Capture in Multi-Domain Environments

Captiva Capture is optimized for single-domain deployments, where the setup program handles
most configuration automatically. Multi-domain environments are supported but require manual
configuration of domain trusts and some additional planning.

## Why Multi-Domain Adds Complexity

Every time a client module communicates with the InputAccel Server across domain boundaries,
a security authentication check happens. If that check fails, the module cannot connect. The setup
program cannot configure cross-domain trust relationships for you since these are handled at the
Windows infrastructure level.

## Trust Relationship Requirements

The minimum trust relationship you need is a **Nontransitive One-Way External Trust** from
the domain containing the clients (the ones that need to authenticate) toward the domain
containing the servers (the ones performing the authentication).

Creating these trusts is an IT infrastructure task, not a Captiva task. It uses Windows operating
system tools and must be completed before attempting to connect any client modules across
domain boundaries.

## Additional Windows Login Requirement

Any user who logs into an InputAccel Server must have the Windows Login privilege on the
machine hosting that server. In a single-domain environment, members of the Domain\Users
group get this by default because they are added to the machine's local Users group. In a
multi-domain environment, you may need to explicitly grant this privilege to cross-domain users.

## Authentication Providers

Captiva Capture supports three Windows authentication providers: NTLM, Kerberos, and
Negotiate. The default is Negotiate, which automatically selects Kerberos or NTLM depending
on what both sides support. In most multi-domain setups, this default works correctly as long
as the trust relationships are properly established.

## Kerberos-Specific Requirements

If you want to enforce Kerberos authentication, additional setup is needed:

- Kerberos must be enabled on both client and server machines.
- A unique Service Principal Name (SPN) must be registered for each InputAccel Server.
- Only one SPN may be registered per InputAccel Server within a domain.
- The format is: `IAServer/<servername>:<port>`
- Adding SPNs requires write permission to arbitrary SPNs in the domain, which by default
  only domain administrators have.

The `SecurityPackage` setting on both the server and all clients defaults to `Negotiate`, which
works with Kerberos automatically once SPNs are registered correctly.

## SQL Server in a Multi-Domain Environment

If the InputAccel Database is hosted on a SQL Server in a different domain from the InputAccel
Server, make sure firewalls between them pass traffic on TCP port 1433, and that the SQL Server
user account used by Captiva has appropriate cross-domain access.

## Practical Tips

- Test cross-domain authentication thoroughly in a dev or staging environment before any
  production deployment.
- Use matching Windows user groups and Captiva roles where possible to simplify ongoing
  permissions management across domains.
- When modules connect to a ScaleServer group in a multi-domain environment, users must
  specify the InputAccel Server machine name, not `localhost` or an IP address.