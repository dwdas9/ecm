# Deploying Captiva Modules with ClickOnce

ClickOnce is an option for distributing ScanPlus, RescanPlus, and IndexPlus to operator
workstations without having to visit each machine for installation. When set up correctly,
users install the module by clicking a URL or running a file from a network share, and updates
are delivered automatically on the next launch.

## When to Use ClickOnce

ClickOnce makes sense when your ScanPlus or RescanPlus operators are spread across locations
and you want centralized deployment without remote administration overhead. If operators need
to specify command-line arguments like `-department` at startup, you must deploy from an IIS
web server rather than a file share, because file-share-deployed ClickOnce shortcuts cannot
accept command-line arguments.

## Prerequisites

Before running the ClickOnce Deployment Utility, have these ready:

- A valid SSL certificate in PFX format for signing the deployment manifest. Any authorized
  signing authority (like VeriSign) works.
- ClickOnce publishing experience. Review the Microsoft MSDN articles on ClickOnce technology
  before choosing this deployment method.
- If deploying via IIS web server, the web server needs a virtual directory configured for each
  module package with these IIS settings:
  - Virtual directory access permissions: Read, Run Scripts, and Write. Do not enable Execute.
  - Security settings: the connecting user account must have Read, Write, and Modify access.
  - WebDAV Web Service Extension status: set to Allowed.

## Installing the ClickOnce Packages

Install the ScanPlus ClickOnce Package, RescanPlus ClickOnce Package, and IndexPlus ClickOnce
Package from the Captiva Capture client installer on the machine where you will run the utility.

## Running the Deployment Utility

Launch the ClickOnce Deployment Utility from **Start > Programs > EMC Captiva Capture >
Tools (Standard) > ClickOnce Deployment Utility**.

Key settings to configure:

**Application settings:** Select the module to deploy, set the publisher name, product name,
and a support URL.

**Deployment settings:**
- Application URL: the URL or network share path users will use to access the module. Do not
  include the manifest filename in this path.
- Installation URL: the location where application files are copied. If left blank, files go to
  the Application URL location.
- Enable "Use a .deploy file name extension" (on by default).
- Keep "Allow URL parameters to be passed to the application" enabled. Disabling this
  prevents setup mode and login parameter passing.

**Signing:** Sign the deployment manifest with your SSL certificate file. Set the Timestamp URL
to a time-stamping service so the certificate remains valid for deployed applications even after
it expires.

After configuring all settings, click **Deploy**. The utility confirms success and displays the
URLs users need to install the module.

## Workaround: Windows Network File Share

Direct HTTP protocol deployment is not supported for ScanPlus or RescanPlus. If you need to
deploy to a web server, use a Windows network file share as the installation target with a virtual
directory mapping to it:

1. On the web server, create a Windows network file share and a virtual directory pointing to it.
2. In the Deployment Utility, set the Application URL to the virtual directory HTTP path.
3. Set the Installation URL to the Windows network file share path.
4. Complete the deployment steps normally.

## Workaround: 64-bit Operating Systems

Deploying ScanPlus, RescanPlus, or IndexPlus on a 64-bit OS produces a 503 error if IIS is not
configured to run 32-bit worker processes. Use either of these approaches:

**Option 1:** Deploy to a Windows network file share instead of directly to IIS.

**Option 2:** Configure IIS to enable 32-bit application support:
1. Open a command window and run:
   `CSCRIPT %SYSTEMDRIVE%\Inetpub\AdminScripts\adsutil.vbs SET W3SVC/AppPools/Enable32bitAppOnWin64 1`
2. Restart the Application Pool.
3. Complete the deployment.

## Common Deployment Errors

- **401 Unauthorized:** Virtual directory permissions are not set correctly.
- **404 Not Found:** The virtual directory grants full Execute permissions. Change to
  Execute Scripts Only.
- **501 Not Implemented:** WebDAV Web Service Extension is not set to Allowed.
- **503 Server Unavailable:** Deploying to a 64-bit OS without the 32-bit workaround applied.

## Note on Event Log Messages

ClickOnce-deployed modules cannot write complete module information to the Windows Event
Log due to how they are registered. Event Log messages will not exactly match what appears in
Captiva Administrator. This is expected behavior, not an error.