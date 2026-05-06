# Language and Locale Support in Captiva Capture

Captiva Capture is designed for global deployments. You can run a single system that processes
documents in multiple languages simultaneously, as long as the environment is set up correctly.

## What Is Supported

- Production module user interfaces are available in multiple languages.
- Modules can run in most system locales even when a UI translation is not available for that
  locale.
- Dynamic IA values can store Unicode text data.
- NuanceOCR, ABBYY, and Extraction engines can extract text from documents in many languages.
- NuanceOCR can output Unicode text files, which is the recommended format for multi-language
  deployments because Unicode handles characters from many languages without encoding issues.

## Before You Install

Locale decisions need to be made before installation, not after. A few things to get right upfront:

The language specified by the locale setting on any machine running the InputAccel Server or
client modules must be supported by the code page selected on that machine. Mixing locale
settings and code pages incorrectly leads to data corruption that is difficult to diagnose.

User-specified information entered during setup must only include characters from the code page
of the machine running the installer. Non-code page values entered during installation will cause
data corruption.

## Multi-Language Deployments and Code Pages

When your process handles documents in multiple languages that span different code pages, you
cannot safely run all modules on a single machine with a single code page. Department routing
is the standard solution for this. Route tasks to modules running on machines configured with
the appropriate code page for the language being processed.

For example, if your process handles both English and Chinese documents, English tasks route
to modules on machines using a Western code page, and Chinese tasks route to modules on
machines using a Chinese code page.

## Setting the UI Language

After installation, you can set the UI language for each Captiva Capture component separately.
This is done through the setup program under the language settings step. Refer to the
Administration Guide for the full matrix of components and their supported UI languages.

If you are upgrading from a previous version, review locale compatibility before upgrading. If
any modules will use a different code page after the upgrade, all in-process batches going through
those modules must be completed before the upgrade begins.

## Unicode Best Practices

Use CaptureFlow Designer rather than Process Developer to design processes that need to handle
multiple languages and code pages. Processes designed in CaptureFlow Designer, following
current recommendations, can run across InputAccel Servers and client machines using any
combination of code pages and regional settings.

For the Invoice Capture component specifically, do not use non-code page Unicode characters
(such as Chinese) in the Invoice Capture database name during installation. If you need to use
such characters, create the database using the Database Manager utility after installation and
update `IAISettings.ini` manually.