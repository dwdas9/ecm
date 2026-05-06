# Captiva Real-Time Capture Services

Real-Time Capture Services is the REST-based layer that exposes Captiva's core capture
capabilities as discrete, callable services. It enables web and mobile applications to invoke
specific capture operations without the overhead of the traditional batch-processing model.

## What It Is

Rather than submitting a document to a full batch pipeline and waiting for it to work through
all processing steps, an application using Real-Time Capture Services calls individual operations
directly. For example, an application can submit a scanned image, call the Document
Classification service to identify the document type, call the Data Extraction service to pull
field values, and then call the Validation service, all in a single user interaction.

This is the technical foundation for "capture at the edge": getting validated documents and
data into a business process in real time without a mailroom queue.

## Available Services

The following discrete services are available:

- **DocType Enumeration** - Returns the list of available document types configured in the
  system.
- **Image Conversion** - Converts images between formats.
- **Image Processing** - Applies image enhancement operations (cleanup, deskew, etc.).
- **Barcode Recognition** - Reads barcodes from images.
- **OCR and PDF Creation** - Performs optical character recognition and creates searchable PDFs.
- **Document Classification** - Identifies the document type of a submitted image.
- **Data Extraction** - Extracts indexed field values from a classified document.
- **Validation** - Validates extracted data against configured rules.

## Architecture

REST Services sits in the web tier, between client applications (browsers, mobile apps, custom
integrations) and the application tier (InputAccel Server and Captiva Capture system). Multiple
REST Service instances can run on separate web servers, all pointing to the same shared data
storage folder. This shared data folder must have read/write/delete/create access from all
instances.

Version 2.0 of REST Services (shipped with Captiva 7.5) added the Module Server, a Windows
service required for Captiva Web Client and Ad Hoc services. The Module Server can also run
on multiple machines, all sharing the same data folder.

## Compatibility

REST Services 2.0 is compatible with both InputAccel Server 7.1 and 7.5. Applications built
on REST Services 1.0 (released with Captiva 7.1) continue to work with both server versions.

## Security

REST Service web applications run under an IIS Application Pool identity. That identity needs:
- Read/write/delete/create access to the shared data folder.
- Membership in the `IIS_IUSRS` Windows group on the web server.
- Membership in the `Performance Log Users` Windows group on the web server.
- Membership in the Captiva Administrators role in Captiva Administrator.

Configure SSL certificates and HTTPS bindings in IIS Management Console before exposing any
REST Service endpoints to external clients.

A custom authentication plugin can be implemented to provide more flexibility for how users
authenticate to REST Services. This is documented in the Captiva Scripting Guide.

## Licensing

REST Service and Module Server licensing is managed through the Captiva REST Services
Licensing tool, not through Captiva Administrator.

## Upgrading from REST Services 1.0

REST Services 2.0 installs on top of the existing 1.0 installation. After upgrade, the new
service uses a different default URL, so any existing clients pointing to the old URL need to
be reconfigured. Alternatively, you can install REST Services 2.0 on separate machines while
keeping the 1.0 installation running for legacy clients.