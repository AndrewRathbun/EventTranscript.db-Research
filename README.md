# EventTranscript.db Research

This repository serves to provide all currently known information about EventTranscript.db. 

(Kroll Blog Post link here)

## What is EventTranscript.db?

EventTranscript.db is a SQLite database that appears to record lots of diagnostic-related information about events that occur on the Windows operating system. This database is not enabled by default and, if enabled, can be enormous in size and potentially serve as a treasure trove of data. 

## Where is EventTranscript.db located?

C:\ProgramData\Microsoft\Diagnosis\EventTranscript\EventTranscript.db

## What does EventTranscript.db record?

There is a table within EventTranscript.db that provides the following information within the Tag Descriptions table.

### Tag Descriptions

| Tag ID | Locale Name | Tag Name | Description |
|-|-|-|-|
| 1 | en-US | Browsing History | Records of the web browsing history when using the capabilities of the application or cloud service, stored in either the service or the application. |
| 11 | en-US |Device Connectivity and Configuration | Data that describes the connections and configuration of the devices connected to the service and the network, including device identifiers (e.g. IP addresses) configuration, setting and performance. |
| 17 | en-US | Inking Typing and Speech Utterance | Record of the input data provided by the end user through an interaction method or action such as inking, typing, speech utterance or gesture. |
| 24 | en-US | Product and Service Performance | Data collected about the measurement, performance and operation of the capabilities of the product or service.  This data represents information about the capability and its use, with a focus on providing the capabilities of the product or service. |
| 25 | en-US | Product and Service Usage | Data provided or captured about the end user’s interaction with the service or products by the cloud service provider.  Captured data includes the records of the end user’s preferences and settings for capabilities, the capabilities used and commands provided to the capabilities. |
| 31 | en-US | Software Setup and Inventory | Data that describes the installation, setup and update of software. |

## How can I parse EventTranscript.db?

SQLECmd has a Map that'll parse EventTranscript.db into 6 separate CSVs, one for each Tag Description.

### Parsing Considerations

There is a JSON Payload in each event entry that appears to differ between each Full Event Name. 

## I don't see EventTranscript.db on my own system/a client's system, what's the deal?

Open the Windows start menu and start typing 'Diagnostics and Feedback Settings'. Within that menu, enable these options.

![DiagnosticFeedbackandSettings](https://github.com/rathbuna/EventTranscript.db-Research/blob/main/Pictures/DiagnosticsandFeedbackSettingsMarkedUp.jpg)

## What is the data stored within EventTranscript.db used for?

This database appears to serve as a backend for the Diagnostic Data Viewer application within Windows.

![DiagnosticDataViewer](https://github.com/rathbuna/EventTranscript.db-Research/blob/main/Pictures/DiagnosticDataViewer.jpg)

## How long has EventTranscript.db existed within Windows?

Preliminary research shows that EventTranscript.db was being recorded to by Windows using DiagTrack.dll starting with Windows 1709. Prior to that, Windows recorded to .rbs files that were hardcoded in filename as events00.rbs, events01.rbs, events10.rbs, and events11.rbs. These files were effectively compressed JSON through 1703 until 1709 changed to EventTranscript.db, which is a SQLite database.

## Supplemental Documention

EventTranscript.db isn't named by name in any of the below documentation

[Diagnostics, feedback, and privacy in Windows 10](https://support.microsoft.com/en-us/windows/diagnostics-feedback-and-privacy-in-windows-10-28808a2b-a31b-dd73-dcd3-4559a5199319)

[Diagnostic Data Viewer Overview](https://docs.microsoft.com/en-us/windows/privacy/diagnostic-data-viewer-overview)

[Feedback & Diagnostics Settings](https://answers.microsoft.com/en-us/windows/forum/windows_10-other_settings-winpc/feedback-diagnostics-settings/c300bfe3-8562-45f6-9341-d7373cc85d9c)

# TODO
Add section about Full Event Name and how JSON Payload is different for each one.
