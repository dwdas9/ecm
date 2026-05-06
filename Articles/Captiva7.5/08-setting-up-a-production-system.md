# Setting Up a Production Captiva Capture System

A production Captiva Capture installation involves several components that need to be installed
in the right order. This article walks through the full sequence for a typical production
environment.

## Installation Order

Always install in this order. Reversing the sequence causes configuration problems that are
harder to fix than doing it right the first time.

1. InputAccel Database (if using an external SQL Server database)
2. InputAccel Server
3. Captiva Capture Web Client and Captiva REST Services (if applicable)
4. Client modules

## Step 1: Install the InputAccel Database

The InputAccel Database is optional in small deployments using the file-based internal database,
but it is required for ScaleServer groups and strongly recommended for all production systems.

Before running the installer, make sure SQL Server is already installed and configured with:
- TCP/IP enabled on port 1433
- Mixed-mode authentication (SQL Server and Windows Authentication)
- A user account with the `dbcreator` role

Run the Captiva Capture setup program, select **Install Product**, then select
**Step 1 - Install InputAccel Database**. EMC recommends disabling antivirus software and
Data Execution Prevention (DEP) during installation.

## Step 2: Install the InputAccel Server

Run the setup program again and select **Step 2 - Install InputAccel Server**. During this step
you will specify the SQL Server instance hosting the InputAccel Database (if using external
database), and the port the server will listen on (default 10099).

The server setup program creates the `InputAccel_Server_admin_group` Windows group and
configures the minimum permissions the server needs to run under a least-privileged account.

For high-volume or high-availability environments, you can install multiple instances of the
InputAccel Server on a single multi-core machine (side-by-side installation), or install servers
on separate machines and configure them as a ScaleServer group.

## Step 3: Install Captiva Capture Web Client and REST Services (Optional)

If you are deploying Captiva Web Client or building applications on top of Captiva REST
Services, install those components on a dedicated web server machine after the InputAccel
Server is up. Configure the REST Service shared data folder so that all REST Service instances
in your environment point to the same location. Add SSL/HTTPS bindings in IIS before exposing
these endpoints to users.

The Module Server (a separate Windows service) is required if you are deploying Captiva Web
Client or using Ad Hoc services. Install it on a dedicated machine after the REST Service is
configured.

## Step 4: Install Client Modules

Run the client components setup (**Step 4 - Install Client Components**) on each machine
designated for client modules according to your installation plan.

For attended modules (ScanPlus, RescanPlus, Completion), install them as applications. For
unattended modules (Image Processor, OCR, Export, etc.), install them as Windows services so
they start automatically and run without a logged-in user.

Web Services Input, Web Services Output, Web Services Coordinator, and Web Services Hosting
should be installed on a dedicated machine or machines if your deployment uses them.

ScanPlus and RescanPlus can optionally be deployed via ClickOnce from a web server or network
file share instead of a direct installation.

## Step 5: Configure the System

After installation, run Captiva Administrator and complete these configuration steps:

- Activate each InputAccel Server using its CAF file from the EMC Captiva Activation Portal.
- Install license codes on each server.
- If running a ScaleServer group, specify the group name and add each server to the group.
  Make sure the same users are in `InputAccel_Server_admin_group` on all servers in the group.
- Configure Web Services Coordinator and Hosting if applicable.
- Set up roles and permissions, and add users.
- Set the UI language for each component if needed.

## Step 6: Deploy Your Processes

Use Captiva Designer to deploy your CaptureFlows to the server. After deployment, use Captiva
Administrator to verify the process appears in the Processes list with the Legacy flag unchecked
(for CaptureFlow processes).

Run Captiva Administrator and confirm that modules connect and that a test batch can be
created, processed, and exported before going live.