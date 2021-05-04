# EventTranscript.db Research

This repository serves to provide all currently known information about EventTranscript.db. 

(Kroll Blog Post link here)

## What is EventTranscript.db?

EventTranscript.db is a SQLite database that appears to record lots of diagnostic-related information about events that occur on the Windows operating system in real-time. This database is not enabled by default and, if enabled, can be enormous in size and potentially serve as a treasure trove of data. 

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

## How much data does EventTranscript.db record?

The answer to everything in DFIR: "It depends". The user can specify the size and scope of the database, as seen below:

![DiagnosticDataSettings](https://github.com/rathbuna/EventTranscript.db-Research/blob/main/Pictures/DiagnosticDataSettings.gif)

## How can I parse EventTranscript.db?

SQLECmd has a Map that'll parse EventTranscript.db into 6 separate CSVs, one for each Tag Description. From there, it's strongly suggested to filter on Full Event Name column for potentially relevant findings.

### Parsing Considerations

Full Event Name is a column within the EventTranscript.db database which appears to give a high level summary of the event, similar to the description of an event provided in Windows Event Logs. For each event entry in this database, there is a JSON Payload that appears to differ between each Full Event Name. What that means is likely no "one size fits all" SQLite query will work for ALL events that exist within this database.

I've compiled a deduplicated list of Full Event Names I observed on my own system [here](https://github.com/rathbuna/EventTranscript.db-Research/tree/main/FullEventNames). Please feel free to add ones that my system didn't happen to record so a more complete list can be maintained for the benefit of the community. 

### Writing Your Own SQLite Queries to Parse EventTranscript.db

Since the JSON Payload appears to be different for each Full Event Name, you'll want to leverage `json_extract` for parsing out data from the JSON Payload column. 

It appears every event has the following names and corresponding values
* `ver`
* `name`
* `time`
* `iKey`
* `ext`
* `data`

The `data` name is where data differentiates between each Full Event Name. If you want to parse the SessionID value from the `data` node, it would look something like this:

`json_extract ( payload, '$.data.sessionID' ) AS SessionID,`

To illustrate there, here are some more examples of what the SQLite query would look like for parsing a particular value that's nested within the JSON Payload column:

![JSONExtractExamples](https://github.com/rathbuna/EventTranscript.db-Research/blob/main/Pictures/JSONExtractExamples.jpg)

More documentation can be found [here](https://www.sqlite.org/json1.html) on extracting JSON using SQLite queries.

## I don't see EventTranscript.db on my own system/a client's system, what's the deal?

Open the Windows start menu and start typing 'Diagnostics and Feedback Settings'. Within that menu, enable these options.

![DiagnosticFeedbackandSettings](https://github.com/rathbuna/EventTranscript.db-Research/blob/main/Pictures/DiagnosticsandFeedbackSettingsMarkedUp.jpg)

## What is the data stored within EventTranscript.db used for?

This database appears to serve as a backend for the Diagnostic Data Viewer application within Windows.

![DiagnosticDataViewer](https://github.com/rathbuna/EventTranscript.db-Research/blob/main/Pictures/DiagnosticDataViewer.jpg)

## How long has EventTranscript.db existed within Windows?

Preliminary research shows that EventTranscript.db was being recorded to by Windows [using DiagTrack.dll](https://docs.microsoft.com/en-us/windows/privacy/diagnostic-data-viewer-overview#microsoft-edge-diagnostic-data-appearing-as-a-blob-of-text) [starting with Windows 1709](https://docs.microsoft.com/en-us/windows/privacy/enhanced-diagnostic-data-windows-analytics-events-and-fields). Prior to that, Windows recorded to .rbs files that were hardcoded in filename as events00.rbs, events01.rbs, events10.rbs, and events11.rbs. These files were effectively compressed JSON through 1703 until 1709 changed to EventTranscript.db, which is a SQLite database. I personally compare this to the .evt to .evtx transition Microsoft made with Windows Vista, i.e. .rbs = .evtx, EventTranscript.db = .evtx. 

For more info on the aforementioned .rbs files, check out this research paper: [Forensic analysis of the Windows telemetry for diagnostics](https://arxiv.org/ftp/arxiv/papers/2002/2002.12506.pdf).

## What does Diagnostic Data Viewer allow the end user to do?

You can do filtering on events stored within this database in real-time using Diagnostic Data Viewer. Also, notice at the end of this GIF that the number of new events automatically updates.

![DiagnosticDataOverviewFilteringandNewEventsOverview](https://github.com/rathbuna/EventTranscript.db-Research/blob/main/Pictures/DiagnosticDataOverviewFilteringandNewEventsOverview.gif)

You can also view Problem Reports within Diagnostic Data Viewer relating to applications suddenly not working as expected.

![DiagnosticDataViewerProblemReports](https://github.com/rathbuna/EventTranscript.db-Research/blob/main/Pictures/DiagnosticDataViewerProblemReports.jpg)

In the About Your Data section, you can view a graphical overview of the data that's being stored in the EventTranscript.db database on your system. 

![DiagnosticDataViewerAboutYourData](https://github.com/rathbuna/EventTranscript.db-Research/blob/main/Pictures/DiagnosticDataViewerAboutYourData.jpg)

## Is there any other data that Diagnostic Data Viewer stores?

Yes, Office Diagnostic Data is optional and can be turned on in the below settings:

![OfficeDiagnosticData](https://github.com/rathbuna/EventTranscript.db-Research/blob/main/Pictures/OfficeDiagnosticData.jpg)

## What are the next steps in regards to researching EventTranscript.db?

There's a lot of opportunity to exploit this database for potentially useful forensic artifacts. Given the sheer volumne of events this database records, it may be like finding a needle in a haystack at times. I personally think it will come down to finding out which Full Event Names provide quick wins within the JSON Payload data. 

For instance, minimal research has been done on the following, but there appears to be potential in the following Full Event Names:

* [Microsoft.Windows.ClipboardHistory.Service.AddItemActivity](https://github.com/rathbuna/EventTranscript.db-Research/blob/f1f648fb8ae4f46bc4719395b9063704ebec238c/FullEventNames/Product%20and%20Service%20Performance/ProductandServicePerformanceFullEventNames.txt#L237)
* [Microsoft.Windows.FileSystem*](https://github.com/rathbuna/EventTranscript.db-Research/blob/f1f648fb8ae4f46bc4719395b9063704ebec238c/FullEventNames/Product%20and%20Service%20Performance/ProductandServicePerformanceFullEventNames.txt#L375)
* [Local Session Manager Events](https://github.com/rathbuna/EventTranscript.db-Research/blob/f1f648fb8ae4f46bc4719395b9063704ebec238c/FullEventNames/Product%20and%20Service%20Usage/ProductandServiceUsageFullEventNames.txt#L8)
* [Microsoft.Windows.Apps.Photos.Analysis.OneDriveStorageStatistics](https://github.com/rathbuna/EventTranscript.db-Research/blob/f1f648fb8ae4f46bc4719395b9063704ebec238c/FullEventNames/Product%20and%20Service%20Usage/ProductandServiceUsageFullEventNames.txt#L71)

These are just a few that jumped out to me as potentially having forensic value. For each Full Event Name, the JSON Payload will have to be examined for forensic value, documented, and shared with the community. 

## Supplemental Documention

EventTranscript.db isn't named by name in any of the below documentation, but all the below links provide invaluable insight into how Windows utilizes and records diagnostic data that resides in this database.

[Diagnostics, feedback, and privacy in Windows 10](https://support.microsoft.com/en-us/windows/diagnostics-feedback-and-privacy-in-windows-10-28808a2b-a31b-dd73-dcd3-4559a5199319)

[Diagnostic Data Viewer Overview](https://docs.microsoft.com/en-us/windows/privacy/diagnostic-data-viewer-overview)

[Feedback & Diagnostics Settings](https://answers.microsoft.com/en-us/windows/forum/windows_10-other_settings-winpc/feedback-diagnostics-settings/c300bfe3-8562-45f6-9341-d7373cc85d9c)

[Forensic analysis of the Windows telemetry for diagnostics](https://arxiv.org/ftp/arxiv/papers/2002/2002.12506.pdf)

[Windows 10, version 1709 and newer optional diagnostic data](https://docs.microsoft.com/en-us/windows/privacy/windows-diagnostic-data)

[Windows 10 diagnostic data events and fields collected through the limit enhanced diagnostic data policy](https://docs.microsoft.com/en-us/windows/privacy/enhanced-diagnostic-data-windows-analytics-events-and-fields)

# TODO
Add links to SQLECmd Map.

Add section about using SQLECmd with KAPE.
