# Captiva Web Client

Captiva Web Client is the browser-based front-end for scanning, indexing, and validating
documents introduced in Captiva 7.5. It eliminates the need for browser plug-ins and replaces
the traditional thick-client scanning interface for knowledge worker scenarios.

## What It Does

Captiva Web Client lets operators scan, review, and validate documents from a standard web
browser. The interface runs entirely in HTML5, with no Active-X controls, no Java applets, and
no browser plugins required.

Supported browsers at release: Internet Explorer v10 and v11, Chrome v32, and Firefox v26.

Document classification and data extraction happen automatically in the background at scan
time using Captiva Real-Time Capture Services. By the time an operator finishes scanning, the
system has already attempted to identify the document type and populate index fields. Operators
review and correct rather than manually entering data from scratch.

## Requirements

Captiva Web Client requires:

- Captiva REST Services 2.0 (shipped with Captiva 7.5)
- Module Server (a separate Windows service installed on its own machine)
- IIS configured with HTTPS and a valid SSL certificate

The Module Server manages the interaction between the Web Client and the Captiva Capture
processing pipeline. Without the Module Server, Web Client cannot function.

## Installation and Configuration

Install Captiva Web Client and Captiva REST Service together on the same web server machine.
Configure the Captiva REST Service shared data folder during installation. All instances of
REST Service and Module Server in the environment must point to the same shared folder.

After installation, configure SSL in IIS Management Console. Add HTTPS bindings for both
the REST Service and Web Client web applications before deploying to users.

Deploy the Captiva Cloud Capture Toolkit to any machines from which users will access
Captiva Web Client. The toolkit enables the browser to communicate with local scanning
hardware.

A custom REST Service authentication plugin is optional but recommended for production
deployments where you need more control over how users authenticate. Refer to the Captiva
Scripting Guide for implementation details.

## Pass-Through Login

Captiva Web Client supports pass-through login, which allows users to authenticate to the Web
Client using their existing Windows credentials without entering them again. This can also be
configured to use the Jasig Central Authentication Service (CAS) for organizations already using
CAS for single sign-on.

## Operational Notes

Captiva Web Client is designed for knowledge workers who need occasional or distributed capture
capability, not for high-volume dedicated scanning operations. For high-volume environments
with dedicated scanners and operators running full shifts, the traditional ScanPlus model
remains more appropriate.

Web Client licensing is managed through the Captiva REST Services Licensing tool, separate
from the main Captiva Administrator licensing panel.