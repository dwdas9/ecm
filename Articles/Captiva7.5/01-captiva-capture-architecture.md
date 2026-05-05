# Captiva Capture Architecture Overview

Captiva Capture is built around a server-centric architecture where a central server coordinates
all document capture activity by routing work to a set of client modules. Understanding this model
is key to designing, deploying, and troubleshooting any Captiva environment.

## The Server and Its Role

The core of the system is the InputAccel Server (also called the Captiva Capture server). Despite
the older name, it is the same component referenced throughout all Captiva documentation. The
server does not perform the actual capture work itself. Its job is to manage and coordinate: it
tracks batches, forms tasks, and routes those tasks to available client modules based on the
instructions in the active CaptureFlow.

The server stores all batch data, IA values, and stage files. It also handles task scheduling,
priority queuing, and task locking to prevent two modules from working on the same node at
the same time.

## Production Modules

Production modules run on client machines and perform the actual processing work: scanning,
image enhancement, OCR, indexing, classification, and export. The server pushes tasks to modules
as they become available, so modules do not poll the server. This push-based model keeps idle time
low.

Most production modules run as unattended Windows services. A few, such as ScanPlus and
Completion, require an operator to interact with them directly.

Captiva Capture uses TCP/IP for all server-to-module communication. Two modules, Web Services
Input and Web Services Output, can also communicate over the internet using SOAP, which enables
integration with external systems regardless of their platform or location.

Third-party certified modules can plug into the same architecture provided they follow the
Module Definition File (MDF) specification.

## Asynchronous Task Processing

Each task in a process is self-contained. A module picks up a task, processes it, returns it to the
server, and immediately picks up the next available task. The server then routes the completed
task forward to the next module in the CaptureFlow. This asynchronous model means modules
from different batches can run in parallel without waiting for each other.

If no module is available to handle a task, the server queues it until a module becomes free.

## Batch Structure

All data in Captiva Capture flows through collections called batches. Each batch is organized as
a tree with up to eight levels. Individual scanned pages sit at level 0, and the batch as a whole is
at level 7. Pages can be grouped into documents or folders at intermediate levels depending on
how the process is designed.

This tree structure lets modules process data at whatever level makes sense for the task. An image
enhancement step might work page by page at level 0, while an export step might handle the
entire batch at level 7.

## ScaleServer Groups

When a single InputAccel Server is not enough, you can configure multiple servers into a
ScaleServer group. All servers in a group share a single external SQL Server database. Each server
manages its own task scheduling independently without coordinating with other servers in the
group. Each batch lives on exactly one server at a time.

ScaleServer is the primary horizontal scaling mechanism in Captiva Capture.

## Captiva REST Services

Captiva 7.5 introduced REST Services, a web-tier component that exposes capture functionality
as REST endpoints. It sits between the client (browser or mobile app) and the application tier.
Supported services include document classification, data extraction, barcode recognition, OCR,
image processing, and validation. REST Services can run on multiple web servers that share a
common data folder.

## Summary

The Captiva Capture architecture separates coordination (the server) from processing (the
modules). This separation is what makes the system scalable, resilient to individual module
failures, and open to third-party module integration.