# EMC Captiva 7.5 Web Client and Real-time Capture Services: New Concepts and Architecture
## Introduction

I've been working with Captiva since version 5.3, and I have to say that version 7.5 is the release I'm most excited about. The core idea behind it is simple but powerful: why should intelligent document capture be limited to a central processing room? Documents don't start their life at a server. They start at a bank counter, an insurance office, a branch lobby. That's where we need to capture them, and that's exactly what Captiva 7.5 sets out to do.

We're calling it "Capture at the Edges." The goal is to push the full power of the Captiva platform out to the very points where documents first enter an organization, and to make that experience as smooth and immediate as possible.

---

## What's New in Captiva 7.5

Before I get into the big new features, here's a quick look at everything that landed in this release on the server side and beyond:

- 64-bit server support
- Native XPP execution
- Enhanced identification and operator modules
- New scripting APIs
- Advanced Freeform Recognition
- Improved data formatting and fuzzy regular expressions
- Standard import via email and file system
- Distributed real-time REST services
- The new Captiva Capture Web Client

The last two are the ones I really want to talk about. Everything else is an improvement on what we already had. The REST services and the Web Client are genuinely new territory.

<!-- IMAGE PLACEHOLDER: Figure 1 - What's New in Captiva 7.5 feature overview diagram (Slide 6) -->

---

## How We Got Here: The Evolution of Distributed Capture

To understand why 7.5 matters, it helps to look at where we've been.

### Captiva 6.x: Remoting with ClickOnce

Back in 6.x, distributed capture worked through a classic remoting model. The central InputAccel Server did all the heavy lifting, and remote sites ran ScanPlus and IndexPlus clients deployed via ClickOnce. It worked, but it required a solid network connection back to the central server for almost everything. Remote workers had the eScan and eIndex desktop tools, which helped, but the architecture was clearly built around the assumption that "real" capture happened at headquarters.

<!-- IMAGE PLACEHOLDER: Figure 2 - Captiva 6.x distributed capture architecture (Slide 7) -->

### Captiva 7.0: A More Unified Desktop

Version 7.0 consolidated things into a cleaner Captiva Server and ScanPlus Desktop model. We simplified the remote deployment story quite a bit, and the eScan and eIndex desktop clients continued to serve remote sites alongside the newer ClickOnce ScanPlus. It was a step forward, but the fundamental architecture was still built around thick desktop clients on remote machines.

<!-- IMAGE PLACEHOLDER: Figure 3 - Captiva 7.0 distributed capture architecture (Slide 8) -->

### Captiva 7.1: The REST Foundation

This is where things started to get interesting. Version 7.1 added a REST-based services layer to the platform. It only supported one operation at the time (Create Batch), but the architectural foundation was there. Any application that could make an HTTP call could now talk to Captiva. That felt like a door opening.

<!-- IMAGE PLACEHOLDER: Figure 4 - Captiva 7.1 distributed capture with REST layer (Slide 9) -->

### Captiva 7.5: The Full Picture

And now here we are. Version 7.5 takes that REST foundation and builds a complete capture processing platform on top of it. Classification, extraction, OCR, image enhancement, population and validation... all of it is now accessible through REST. And sitting on top of that is a full browser-based Web Client that lets anyone do end-to-end capture from any modern browser, with no software installation required.

That's the story of how we got to "Capture at the Edges."

<!-- IMAGE PLACEHOLDER: Figure 5 - Captiva 7.5 full distributed architecture with Web Client (Slide 10) -->

---

## The Captiva Capture Web Client

This is the part I'm most proud of. The Capture Web Client brings the complete document capture and indexing workflow to any modern browser. No plugins, no thick client, no ClickOnce deployment to manage. Just a URL and a scanner.

### Two Real-World Use Cases

I always like to ground new features in real scenarios, because that's what makes them meaningful. Here are two that shaped how we built the Web Client.

#### Use Case 1: On-the-Spot Scanning at a Bank Branch

Das is applying for a loan and needs to bring pay stubs to prove her income. Here's what that experience looks like with and without the Web Client.

**Without the Web Client:**
1. Das brings her pay stubs to the bank
2. The clerk makes photocopies
3. The copies get mailed to central processing
4. Processing discovers a watermark is obscuring the gross pay figure
5. Das gets called back and has to come in again

**With the Web Client:**
1. Das brings her stubs to the bank
2. The clerk scans the documents right there at the counter
3. If the scan quality is bad, he tries again on the spot
4. The documents are uploaded immediately for processing

The difference is night and day. No mailing, no callbacks, no second trip for Das.

<!-- IMAGE PLACEHOLDER: Figure 6 - Use Case 1: On-the-spot scanning at a bank branch (Slide 12) -->

#### Use Case 2: Branch Office Scanning at an Insurance Agency

Das works at an insurance agency and collects new-policy forms throughout the day. The old workflow processed them in the evening via fax to a central facility with a 4-hour SLA.

**Without the Web Client:**
1. Das faxes documents to central processing
2. Captiva extracts what it can centrally
3. An operator at the central office keys in the rest
4. Documents trigger a business process within the 4-hour SLA

**With the Web Client:**
1. Das scans the documents himself at the branch
2. The Web Client extracts what it can in real time
3. Das keys in anything remaining
4. He submits the batch and the business process triggers immediately

The SLA goes from hours to minutes, and Das is in control of the quality of his own submissions.

<!-- IMAGE PLACEHOLDER: Figure 7 - Use Case 2: Branch office scanning at an insurance agency (Slide 13) -->

### How the Architecture Works

The system architecture is intentionally simple. A standard web server hosts the HTML and JavaScript application. The browser talks to the Captiva server through the REST services layer. For scanner access, an optional lightweight Captiva Cloud Toolkit runs on the client machine. All the heavy processing (classification, extraction, OCR) happens on the Captiva server. The browser just orchestrates it.

<!-- IMAGE PLACEHOLDER: Figure 8 - Web Client system architecture diagram (Slide 14) -->

### The User Interface

The interface has three main areas: the Batch panel (your documents and pages), the Image viewer (the scanned page), and the Form panel (for data entry and validation). Where these panels live is configurable, which I'll get to in the user preferences section.

<!-- IMAGE PLACEHOLDER: Figure 9 - Web Client UI showing Batch, Image, and Form panels (Slide 15) -->

### Core Features

Here's what the Web Client can do right out of the box:

- Scan or import documents and images directly from the browser
- Automatic document assembly with intelligent separation
- Optional real-time classification and data extraction via image pipelining
- Document data validation with field-level rules
- User-driven image processing (rotate, crop, enhance, and more)
- Fully customizable user experience with branding and localization support
- No browser plugins required
- Works in Internet Explorer, Chrome, and Firefox

### Creating a Batch: The Basic Workflow

The steps are straightforward and designed to mirror how people naturally think about scanning documents:

1. Select a scan profile
2. Scan documents or import existing image files
3. Assemble documents by defining the folder, document, and page structure
4. Fill in the data-entry forms for each document
5. Submit the batch

During this process, images are optionally pipelined through server-side processing as they come in. Data gets validated in real time. When everything looks good, the images and metadata upload securely to the Captiva Server.

<!-- IMAGE PLACEHOLDER: Figure 10 - Steps to create a batch workflow diagram (Slide 18) -->

### Image Pipelining

This is one of the features I'm most excited about from a user experience standpoint. Instead of waiting for the entire batch to be scanned before any processing happens, pipelining lets us process each image as it comes off the scanner.

Here's how it works:

- As each page is scanned or imported, post-processing kicks off automatically
- The Captiva Cloud Toolkit handles blank page detection and barcode reading on the client side
- The Captiva Server applies server-side operations: image enhancement, document classification, and data extraction
- The client uses the returned results to assemble documents automatically

The practical effect is that by the time the user finishes scanning, the documents are already classified and a lot of the data is already extracted. The user just reviews, corrects anything that needs it, and submits.

<!-- IMAGE PLACEHOLDER: Figure 11 - Image pipelining flow diagram (Slide 19) -->

### Scan Profiles

Everything about how the Web Client behaves is driven by scan profiles, which are designed in Captiva Designer and deployed to the capture server. A scan profile controls:

- **Batch handling:** which CaptureFlow to use and the batch naming schema
- **Scanner settings:** paper size, resolution, orientation, and image format
- **Document assembly rules:** how to handle barcodes, blank pages, and which document types are allowed
- **Pipeline steps:** which image enhancements, classification profiles, and extraction profiles to run

### Batch Structuring

The document model is consistent with the rest of Captiva: folders, documents, and pages. Automatic separation can happen based on blank pages, a defined page count, or barcode separator sheets. Users can also manually cut, copy, paste, split, and merge documents as needed.

### Supported File Types

| File Type | Behavior |
|-----------|----------|
| TIFF, JPEG, PNG | Displayed inline; fully editable |
| PDF | Shown in the native browser PDF viewer; read-only |
| Other formats | Not displayed, but still uploaded |

### Image Processing

Users can do a lot of quality control right in the browser before they ever submit a batch. The available operations are:

- Rotate and crop
- Color conversion (grayscale or black and white)
- Straighten (deskew)
- Remove noise and overscan
- Remove lines
- Adjust brightness and contrast

### Document Types and Forms

Classification and data entry can be either fully automatic or manually driven.

In automatic mode, the system identifies the document type, extracts relevant data fields, and performs initial population and validation without the user needing to do anything. In manual mode, the user picks the document type from a list, field values carry over from the previous document as a starting point, and population and validation are triggered when the user is ready.

<!-- IMAGE PLACEHOLDER: Figure 12 - Document type and form handling (Slide 25) -->

### Population and Validation

I want to spend a moment on this because it changed quite a bit between 7.1 and 7.5.

In Captiva 7.1, if you wanted to automatically calculate a field value based on other fields (say, multiplying a subtotal by a tax rate), you had to write custom C# code in an event handler. It worked, but it required a developer every time someone needed a new calculation rule.

In Captiva 7.5, we introduced population rules in Captiva Designer. You can define expressions, query-based lookups, or scripts (in the full Completion client) declaratively, without touching code. That's a much better experience for everyone involved.

<!-- IMAGE PLACEHOLDER: Figure 13 - Population rules interface in Captiva 7.5 (Slide 27) -->

### Barcode Handling

Barcodes are automatically extracted from each scanned page by the Captiva Cloud Toolkit. The values land in well-known document fields (Barcode1, Barcode2, and so on) and show up in the form if the document type is configured to use them. This makes barcode-driven workflows like automatic document identification and data pre-population work seamlessly in the Web Client.

<!-- IMAGE PLACEHOLDER: Figure 14 - Barcode handling diagram (Slide 29) -->

### User Preferences

Operators can tailor the interface to how they like to work:

- Move the data-entry form to the right, bottom, or beneath the image
- Pop the image out into a separate window for dual-monitor setups
- Adjust thumbnail size in the batch panel
- Preview scanned pages while scanning is still in progress

### Customization

Administrators have a few options for making the Web Client feel like it belongs to the organization:

- Change the company name and application title
- Provide a custom download link or installation instructions for the Captiva Cloud Toolkit
- Add custom language packs on top of the 10 languages that ship out of the box

### Web Client vs. Completion: Key Differences

I get this question a lot, so let me put it plainly. The Web Client is for distributed, ad hoc capture at the edge. The Completion client is the full-featured central operator station. Here's how they compare:

| Feature | Completion | Web Client |
|---------|------------|------------|
| Control types | All types | Editable only |
| Layout | Designer-defined | Designer-defined |
| Date formats | Any | No word-based dates |
| Data type validation | On field change | On field change |
| Range checks | On field change | On field change |
| Rule-based population | On field change | On user button click |
| Scripting | All events | None |
| Doc type changes | Copy values via scripting | Values copied automatically |

<!-- IMAGE PLACEHOLDER: Figure 15 - Completion vs. Web Client comparison table (Slide 32) -->

### Scalability and Security

A few things worth calling out here:

- Scalability is limited only by the capacity of the real-time services tier, which scales independently
- Pipeline processing is distributed across module servers, so adding capacity is straightforward
- Most local image processing happens on the browser machine, which reduces server load significantly
- Authentication runs through the real-time services layer
- All browser-to-server communication is over HTTPS
- The Captiva Cloud Toolkit stores images locally while the batch is open, and deletes them securely when the batch is submitted

<!-- IMAGE PLACEHOLDER: Figure 16 - Scalability and security architecture (Slide 33) -->

---

## Captiva Real-time Capture Services

The Real-time Capture Services are what power the Web Client under the hood, and they can also be used independently by any application that needs to embed enterprise-grade capture processing. This is where things get really interesting from a developer perspective.

### What's New Since 7.1

In Captiva 7.1, the REST services could do exactly one thing: create a batch. That was useful, but limited. In 7.5, we turned the REST layer into a full capture processing platform:

| Service | Captiva 7.1 | Captiva 7.5 |
|---------|-------------|-------------|
| Create Batch | Supported | Supported |
| Image Processor | Not available | Available |
| Image Converter | Not available | Available |
| Classification | Not available | Available |
| Extraction | Not available | Available |
| Full-page OCR | Not available | Available |
| Population / Validation | Not available | Available |

<!-- IMAGE PLACEHOLDER: Figure 17 - Captiva 7.1 vs 7.5 REST services comparison (Slide 35) -->

### How the Architecture Is Structured

The system is designed for high availability and horizontal scalability from the ground up:

- A pool of REST servers sits behind a firewall or load balancer
- Shared REST storage holds session data, images, and partial batch state, accessible by any server in the pool
- Captiva Servers handle batch creation and profile synchronization
- Module Servers run the actual processing workloads (image processing, OCR, classification, extraction)
- External applications connect through the same REST interface as the Web Client

<!-- IMAGE PLACEHOLDER: Figure 18 - Real-time Capture Services system architecture (Slide 36) -->

### Configuring the System

We designed configuration to be as simple as possible. You configure one REST server fully, then point additional servers at the shared configuration file. That shared file covers module type counts and recycle intervals, the Captiva server pool and connection settings, and the REST server address, port, timeouts, and security configuration.

<!-- IMAGE PLACEHOLDER: Figure 19 - System configuration overview (Slide 37) -->

### Setting Up Capture Profiles

All capture processing profiles for the Real-time Services are created in Captiva Designer and deployed to the capture server. This covers Image Processor profiles, Image Converter profiles, Advanced Recognition profiles, Document Types, and Import profiles. Once deployed, the Real-time Capture modules sync those profiles automatically. The only time you need a live connection to the Captiva Server is during design, sync, and batch creation.

### Session State

This is important if you're thinking about deploying this at scale. The Real-time Services are stateless at the request level, which means:

- Users authenticate to any server in the pool to get a session ticket
- Session content (images and partial batch data) lives in shared storage, not in any one server's memory
- Each individual REST request can be handled by any server in the pool
- Server restarts and failures don't lose session data

That's a pretty robust architecture for something running over potentially unreliable branch office networks.

<!-- IMAGE PLACEHOLDER: Figure 20 - Session state and shared storage architecture (Slide 39) -->

### Efficiency Is Built In

One thing I want to highlight is how much leaner the Real-time Services are compared to the legacy WS Modules approach. Here's a side-by-side:

| Step | WS Modules (Legacy) | Real-time Services |
|------|---------------------|--------------------|
| 1 | Send page to WS Input | Send page to REST server |
| 2 | Server sends task to module | REST server sends task to module server |
| 3 | Module returns task to server | Module server returns task |
| 4 | Server sends task to WS Output | (No separate output step) |
| 5 | WS Output sends results | REST server returns results directly |

Fewer hops, less overhead, faster response times.

<!-- IMAGE PLACEHOLDER: Figure 21 - WS Modules vs Real-time Services efficiency comparison (Slide 40) -->

### Image Management

The REST API gives you a lot of flexibility around how images move through the system. You can upload multiple images in a single request to reduce round-trips, or split a large image across smaller requests if you're working with limited bandwidth. Crucially, the image ID returned by an upload can be reused for downstream operations without downloading and re-uploading the image.

A practical example: upload one image, convert it to a different format, and add the converted version to a batch. Three API calls, one image transfer. That kind of efficiency matters a lot when you're running over a branch office internet connection.

### Authentication Options

There are two ways to authenticate with the Real-time Services:

**Windows-based authentication:** The services delegate to the Captiva Server for user verification. User-level security can restrict which CaptureFlows a given user can access.

**Custom plug-in authentication:** A plug-in validates the user against any identity provider you choose. The plug-in returns one or more Captiva roles, and the user gets access to all CaptureFlows. This is the option to use if you're integrating with a non-Windows identity system.

### User Roles

| Role | What They Can Do |
|------|-----------------|
| Administrator | Access the licensing UI; requires the Captiva Server Administrators role |
| Operator | Use all other real-time services; requires Server.Login, Server.Create.Batch, System.BatchRead, System.BatchModify, System.ProcessRead |

### Licensing

Real-time Services are licensed separately from the Captiva Server. Here's how the licensing model breaks down:

| License Type | What It Covers |
|--------------|---------------|
| On/Off (unlimited use) | Document type processing, Image processing |
| Page count (limited use) | Classification, Extraction, Full-page OCR and text-searchable PDF creation |
| Concurrent connections | Captiva Capture Web Client user sessions |

Admins can manage licenses through the licensing UI, add new ones, review existing ones, and expired or consumed licenses get cleaned up automatically. The Create Batch service specifically requires Captiva Server feature code "W."

<!-- IMAGE PLACEHOLDER: Figure 22 - Licensing structure (Slides 44 and 45) -->

### Scalability and Security

Same story as the Web Client, but the levers are different here:

- Capacity scales by adding new module server instances pointed at the shared configuration
- Multiple authentication options give you flexibility for different deployment environments
- All network traffic is secured with HTTPS
- The REST data directory should be secured at the OS or network level to protect session images

<!-- IMAGE PLACEHOLDER: Figure 23 - Real-time Services scalability and security (Slide 46) -->

---

## Wrapping Up

If I had to summarize Captiva 7.5 in one sentence, it would be this: we took everything Captiva could do and made it available everywhere.

The Capture Web Client lets any branch employee do intelligent, real-time capture from a browser without any software setup. The Real-time Capture Services let any developer embed that same capture intelligence into their own applications through a clean REST API.

Together, they turn Captiva from a centralized capture system into a distributed capture network. Whether you're a bank teller scanning loan documents, an insurance agent processing new policies, or a developer building capture into a line-of-business application, version 7.5 has something meaningful for you.

I'm genuinely excited about where this takes us. Capture at the edges is just the beginning.

<!-- IMAGE PLACEHOLDER: Figure 24 - Summary diagram: Web Client and Real-time Services together (Slide 48) -->

---

*© Das. All rights reserved.*