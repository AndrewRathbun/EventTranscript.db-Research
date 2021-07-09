# EventTranscript.db Research

This repository serves to provide all currently known information about EventTranscript.db. 

[Forensically Unpacking EventTranscript.db: An Investigative Series](https://www.kroll.com/en/insights/publications/cyber/forensically-unpacking-eventtranscript) - articles at the bottom of this landing page

## What is EventTranscript.db?

EventTranscript.db is a SQLite database that appears to record lots of diagnostic-related information about events that occur on the Windows operating system in real-time. This database is not enabled by default and, if enabled, can be enormous in size and potentially serve as a treasure trove of data. 

## Where is EventTranscript.db located?

`C:\ProgramData\Microsoft\Diagnosis\EventTranscript\EventTranscript.db`

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

Preliminary research shows that EventTranscript.db was being recorded to by Windows [using DiagTrack.dll](https://docs.microsoft.com/en-us/windows/privacy/diagnostic-data-viewer-overview#microsoft-edge-diagnostic-data-appearing-as-a-blob-of-text) starting with [Windows 1709](https://docs.microsoft.com/en-us/windows/privacy/enhanced-diagnostic-data-windows-analytics-events-and-fields). Prior to that, Windows recorded to .rbs files that were hardcoded in filename as events00.rbs, events01.rbs, events10.rbs, and events11.rbs. These files were effectively compressed JSON through 1703 until 1709 changed to EventTranscript.db, which is a SQLite database. I personally compare this to the .evt to .evtx transition Microsoft made with Windows Vista, i.e. .rbs = .evtx, EventTranscript.db = .evtx. 

For more info on the aforementioned .rbs files, check out this research paper: [Forensic analysis of the Windows telemetry for diagnostics](https://arxiv.org/ftp/arxiv/papers/2002/2002.12506.pdf).

## What does Diagnostic Data Viewer allow the end user to do?

You can do filtering on events stored within this database in real-time using Diagnostic Data Viewer. Also, notice at the end of this GIF that the number of new events automatically updates.

![DiagnosticDataOverviewFilteringandNewEventsOverview](https://github.com/rathbuna/EventTranscript.db-Research/blob/main/Pictures/DiagnosticDataOverviewFilteringandNewEventsOverview.gif)

You can also view Problem Reports within Diagnostic Data Viewer relating to applications suddenly not working as expected. Please note that this reports are the same found in `C:\ProgramData\Microsoft\Windows\WER`. 

![DiagnosticDataViewerProblemReports](https://github.com/rathbuna/EventTranscript.db-Research/blob/main/Pictures/DiagnosticDataViewerProblemReports.jpg)

In the About Your Data section, you can view a graphical overview of the data that's being stored in the EventTranscript.db database on your system. 

![DiagnosticDataViewerAboutYourData](https://github.com/rathbuna/EventTranscript.db-Research/blob/main/Pictures/DiagnosticDataViewerAboutYourData.jpg)

## Is there any other data that Diagnostic Data Viewer stores?

Yes, Office Diagnostic Data is optional and can be turned on in the below settings:

![OfficeDiagnosticData](https://github.com/rathbuna/EventTranscript.db-Research/blob/main/Pictures/OfficeDiagnosticData.jpg)

## What are the next steps in regards to researching EventTranscript.db?

There's a lot of opportunity to exploit this database for potentially useful forensic artifacts. Given the sheer volume of events this database records, it may be like finding a needle in a haystack at times. I personally think it will come down to finding out which Full Event Names provide quick wins within the JSON Payload data. 

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

## Parsing EventTranscript.db with PowerShell

### Installation
EventTranscript.db can be parsed with PowerShell. To interact with the service and retrieve the database contents you to install the Microsoft.DiagnosticsDataViewer PowerShell module as outlined at (https://docs.microsoft.com/en-us/windows/privacy/microsoft-diagnosticdataviewer). 

```PowerShell
Install-Module -Name Microsoft.DiagnosticDataViewer
```

The module is also available at PSGallery (https://www.powershellgallery.com/packages/Microsoft.DiagnosticDataViewer/2.0.0.1). 

### Usage
Usage of the PowerShell module is farily straight-forward but has a few issues. It allows for control of the logging capabilities the provided by the DiagTrack service. However, it requires installation of the Diagnostic Data Viewer application from the Microsoft Store. Once that is installed, you need to enable diagnostic data viewing via the Enable-DiagnosticDataViewing cmdlet. 

```PowerShell
PS C:\WINDOWS\system32> Enable-DiagnosticDataViewing
Diagnostic Data Viewing is enabled now
```

Next, You can view the various categories of diagnostic data by using the Get-DiagnosticDataCategories. The documentation at (https://docs.microsoft.com/en-us/powershell/module/microsoft.diagnosticdataviewer/?view=windowsserver2019-ps) list the cmdlet as Get-DiagnosticDataTypes. As shown below, this is incorrect. 

```PowerShell
PS C:\WINDOWS\system32> Get-DiagnosticDataCategories

Id Name                                  Description
-- ----                                  -----------
-1 Incorrect Data Category               Event is incorrectly categorized.  Microsoft is working on fixing such events
 1 Browsing History                      Records of the web browsing history when using the capabilities of the appl...
11 Device Connectivity and Configuration Data that describes the connections and configuration of the devices connec...
17 Inking Typing and Speech Utterance    Record of the input data provided by the end user through an interaction me...
24 Product and Service Performance       Data collected about the measurement, performance and operation of the capa...
25 Product and Service Usage             Data provided or captured about the end user’s interaction with the service...
31 Software Setup and Inventory          Data that describes the installation, setup and update of software.


PS C:\WINDOWS\system32> Get-DiagnosticDataTypes
Get-DiagnosticDataTypes : The term 'Get-DiagnosticDataTypes' is not recognized as the name of a cmdlet, function,
script file, or operable program. Check the spelling of the name, or if a path was included, verify that the path is
correct and try again.
At line:1 char:1
+ Get-DiagnosticDataTypes
+ ~~~~~~~~~~~~~~~~~~~~~~~
    + CategoryInfo          : ObjectNotFound: (Get-DiagnosticDataTypes:String) [], CommandNotFoundException
    + FullyQualifiedErrorId : CommandNotFoundException
```

From this point we are able to extract various diagnostic data and apply filters. The output is typically provided as JSON, but we can also export the data as a CSV. 

```PowerShell
PS C:\WINDOWS\temp> Get-DiagnosticData -DiagnosticDataCategory 31 -StartTime (Get-Date).AddHours(-12) -EndTime (Get-Date).AddHours(0) | Export-Csv 'tmp.csv'
PS C:\WINDOWS\temp> head -n 3 .\tmp.csv
#TYPE DDVCmdlets.Containers.EventRecord
"Name","Timestamp","Payload","IsRequired","DiagnosticDataCategories"
"Microsoft.Windows.StoreAgent.Telemetry.InstallOperationRequest","5/18/2021 3:31:42 PM","{""ver"":""4.0"",""name"":""Microsoft.Windows.StoreAgent.Telemetry.InstallOperationRequest"",""time"":""2021-05-18T15:31:42.5337494Z"",""iKey"":""o:0a89d516ae714e01ae89c96d185e9ae3"",""ext"":{""utc"":{""eventFlags"":514,""pgName"":""WINCORE"",""flags"":905970180,""epoch"":""5901065"",""seq"":6310},""mscv"":{""cV"":""F9pa8KoqmUOiQBHY.10.2""},""os"":{""bootId"":58,""name"":""Windows"",""ver"":""10.0.18363.1440.amd64fre.19h1_release.190318-1202""},""app"":{""id"":""U:Microsoft.WindowsStore_12104.1001.1.0_x64__8wekyb3d8bbwe!App"",""ver"":""12104.1001.1.0_x64_!2021/04/13:01:49:52!0!winstore.app.exe"",""asId"":25724},""device"":{""localId"":""s:8FA50876-DF77-42F6-B2A1-CA2D1D6229F7"",""deviceClass"":""Windows.Desktop""},""protocol"":{""devMake"":""VMware, Inc."",""devModel"":""VMware Virtual Platform""},""user"":{""localId"":""j:00847540-42CD-5ED8-2C47-0F1896FC2BAF""},""loc"":{""tz"":""-00:00""}},""data"":{""ProductId"":""9N8WTRRSQ8F7"",""SkuId"":""0010"",""CatalogId"":"""",""BundleId"":"""",""VolumePath"":""""}}","True","System.Collections.Generic.List`1[System.Int32]"
```

###  Moar Logs!!
On several systems where we tested the logging functionality of EventTranscript.db and the DiagTrack service the Optional diagnostic data option was greyed out in the GUI. 

![Diagnostic and Feedback - Optional](https://github.com/rathbuna/EventTranscript.db-Research/blob/main/Pictures/Pasted%20image%2020210518161731.png)

To manually enable the service we can modify the following registry keys:

```PowerShell
HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\DataCollection\AllowTelemetry REG_DWORD 0x00000003
HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\DataCollection\MaxTelemetryAllowed REG_DWORD 0x00000003
#and
HKEY_LOCAL_MACHINE\SOFTWARE\WOW6432Node\Microsoft\Windows\CurrentVersion\Policies\DataCollection\AllowTelemetry REG_DWORD 0x00000003
HKEY_LOCAL_MACHINE\SOFTWARE\WOW6432Node\Microsoft\Windows\CurrentVersion\Policies\DataCollection\MaxTelemetryAllowed REG_DWORD 0x00000003
```


![Windows Registry DataCollection](https://github.com/rathbuna/EventTranscript.db-Research/blob/main/Pictures/Pasted%20image%2020210518165200.png)

Once it is enabled, we can toggle the optional data collection categories. 

![Improving Inking and Typing and Tailored Experiences](https://github.com/rathbuna/EventTranscript.db-Research/blob/main/Pictures/Pasted%20image%2020210518162055.png)

Optional data collection enables us to record web traffic visited by Internet Explorer and Edge. Unfortunately, collection of web traffic from Firefox and Google Chrome does not appear to be collected. 

```PowerShell
PS C:\WINDOWS\temp> Get-DiagnosticData -DiagnosticDataCategory 1 
...
Name                     : Microsoft.Windows.App.Browser.HJ_BeforeNavigateExtended
Timestamp                : 5/18/2021 5:08:46 PM
Payload                  : {"ver":"4.0","name":"Microsoft.Windows.App.Browser.HJ_BeforeNavigateExtended","time":"2021-0
                           5-18T17:08:46.6756009Z","iKey":"o:0a89d516ae714e01ae89c96d185e9ae3","ext":{"utc":{"popSample
                           ":50,"eventFlags":524546,"pgName":"WIN","flags":469762564,"epoch":"5901065","seq":6563},"met
                           adata":{"f":{"sessionID":8,"userInputID":8,"AppSessionGuid":8}},"os":{"bootId":58,"name":"Wi
                           ndows","ver":"10.0.18363.1440.amd64fre.19h1_release.190318-1202"},"app":{"id":"U:Microsoft.M
                           icrosoftEdge_44.18362.449.0_neutral__8wekyb3d8bbwe!MicrosoftEdge","ver":"44.18362.449.0_neut
                           ral_!2079/11/26:09:41:53!1E050!microsoftedgecp.exe","asId":26237},"device":{"localId":"s:8FA
                           50876-DF77-42F6-B2A1-CA2D1D6229F7","deviceClass":"Windows.Desktop"},"protocol":{"devMake":"V
                           Mware, Inc.","devModel":"VMware Virtual Platform"},"user":{"localId":"j:00847540-42CD-5ED8-2
                           C47-0F1896FC2BAF"},"loc":{"tz":"-00:00"}},"data":{"sessionID":"DFCAC27D-B7F9-11EB-B1D4-00505
                           6ABB8A0","userInputID":"2BE4207C-34CF-4A55-BAB6-F3A2836DE786","AppSessionGuid":"00001A70-000
                           2-003A-CF87-6B6C084CD701","tabId":402,"frameId":1348598048,"managerProcessId":1026,"navigati
                           onUrlBytes":"0x647777772E6D736E2E636F6D","navigationUrlRejectCode":0,"navigationLocationUrlB
                           ytes":"0x","navigationLocationUrlRejectCode":30,"isNavLocUrlEqualToUrl":0,"isNavUrlTopLevelU
                           rl":1,"deviceFeatureStatus":136,"isCortanaEnabled":0,"browserId":"{032D297E-FF55-488E-9307-C
                           53C43DC560B}"}}
IsRequired               : False
DiagnosticDataCategories : {1, 24}
...
```

As shown above, the navigationUrlBytes field contains the value 0x647777772E6D736E2E636F6D. Decoded to ASCII this value is dwww.msn.com. 

{032D297E-FF55-488E-9307-C53C43DC560B}

# TODO

Add spreadsheet of 2,500+ Full Event Names with Counts.
