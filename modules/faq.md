# <span class="center-text">B<span style="color: #EA3E5D;">IR</span>T Frequently Asked Questions (FAQ)</span>
<br><br>
[Back](../index.md)
## **What is BIRT?**
A collaborative Incident Response forensics platform designed for rapid tactical deployment

## **What does it do?**
- Parses forensic artifacts or exported SIEM events into normalized timestamped events.
- Re-orders the events and fuses them into a single collection, the Endpoint.
- Reconstructs the endpoint's state over time while applying Micro-Technique rules.
- Micro-Techniques collect and label evidence and behaviors event-by-event.
- Evidence is combined with contextual information to produce a view into the endpoint's most interesting behaviors.
- Investigators use evidence or individual events to create an Investigation, an automated container for an investigations' events and evidence.
- Reports are generated from Investigations and can be viewed in real-time, or exported as a PDF.

## **What artifact types are supported?**

Every file processed is broken down into one or more events with a timestamp.  The files that do not match any recognized patterns fall into a catch-all artifact event type that uses the processing time as the time stamp.  This can be overridden if the artifact is added to an Investigation.

Events are hierarchically linked, where applicable, and these structures are useful when reconstructing the endpoint and building Micro-Techniques.  EDR events, Registry, $MFT and other File Tables retain their hierarchies, for example.

File Type | Event Types | Comment
-- | -- | -- 
Windows XML Event Log | EDR, Windows EVTX | Creates EDR events for Sysmon + Security 4688.  Recognizes 800+ event sources and event ID's to add descriptive text.
Windows Registry | Windows Registry | A related tree/graph of keys/values sorted by their last-access timestamps for **any** Windows Registry format file (AmCache, UsrClass, System, User, etc).
Windows Master File Table | MFT | A related tree/graph of the $MFT with full $FN and $SI MACE attributes.
Windows PE | PE file | Windows PE files are parsed and searched for additional executables including XOR encoded files.
Linux ELF | ELF file | Linux Executable and Linking Format files are parsed and searched for executables.
Windows Scheduled Tasks and Jobs | Execution Artifact | Evidence of execution with forensically significant timestamp.
Windows Prefetch Files | Execution Artifact | Evidence of execution with forensically significant timestamp.
Windows SRUM DB | Execution Artifact | Evidence of execution with forensically significant timestamp and network metadata.
Linux UTMP/WTMP/BTMP |  Execution Artifact | Evidence of execution with forensically significant timestamp and a remote host, if any.
Linux Auditd | EDR, Audit Message | Creates proc/net/file/etc events for relevant syscall events and parses other messages into a singular event type.
Timestamped Messages | Event Log Message | Linux style logs with a leading timestamp can be naively parsed into a utc/message event, such as the majority of logs in /var/log/.
Dictionary, CSV, JSON | **any** | Opportunistically parse lines into recognized event types, including Sysmon (Linux, Windows), Crowdstrike and Carbonblack events.  This can come from tool output or SIEM/log store exports.
Text and Other Files | Artifact | Text files are parsed line-by-line, and all other files have strings extracted up to a configurable limit (64kb).  Artifacts are searched for executables.
Forensic Images | **any** | .E01 and .dd files can be automatically processed for artifacts, Linux and Mac OS have the file systems walked to produce a tree/hierarchy similar to the Windows $MFT, to aid in analysis.
PCAP and PCAPNG | PCAP Connection | Source/Dest, Protocol information for discovered connections.  Connections are searched for executables.

## **What is a Micro-Technique?**
Micro-Techniques are specific implementations of a MITRE ATT&CK Sub-Technique. They are finite-state machines, where each state matches an event and persists until the states are completed or expire.  A state match can also save attributes from an event that can be compared against in subsequent states.  MT are intended to describe any endpoint behavior that can be deduced from a sequence of events. 

- **Tactic** TA0002 Execution
- **Technique** TA0002.T1204 User Execution
- **Sub-Technique** TA0002.T1204.002 Malicious File
- **Micro-Technique** TA0002.T1204.002.765d9a Executing out of downloads folder

## **What is the Micro-Technique Engine?**
The MTE handles the Micro-Technique rules and their state-transitions while reconstructing the endpoint state.  The MTE tracks power state cycles, IP addresses and users.  When a Micro-Technique closes and registers evidence the contextual data is pulled from the current endpoint reconstruction and merged into the evidence graph.  

The evidence graph contains all of the labelled evidence and contextual data and is broken up into process groups.  Each process group is a logical partition an endpoint.  For example, evidence that includes children of the Windows GUI (explorer.exe) are grouped together.

## **What is an Investigation?**
An Investigation is a named container of evidence.  When events or evidence is found to be part of the narrative forming during the course of analysis, an investigator can add these events to the currently selected Investigation.  The Investigation object will gather additional metadata and context when events and/or evidence is added.  When behavioral data such as EDR, Sysmon, Auditd, etc are present, the process tree is walked back to it's root for additional context.  This creates a rich and intuitive view into intrusions that reduces cognitive workload and accelerates response efforts.

The Investigation creates a global and individual timelines for each endpoint, process/event hierarchies, a graphical timeline and an exportable report.  Investigators can leave comments on individual events or groups, modify evidence timestamps to better align small offsets and prune unnecessary events.  Reporting and summary analysis can be generated with LLM's such as ChatGPT. or locally hosted alternatives.

The Investigation intends to serve an offering at each understanding "altitude" of the stakeholders.  From technical investigator to security management and then to CTO/CISO.

## **Does BIRT have an API for my SOAR or other Orchestration tools?**
Yes! BIRT has a python API class that can:

- Create Endpoints
- Upload Files
- Create Investigations
- Update Investigation Title and Summary Description
- Monitor MT or Connect jobs

## **Does BIRT have an agent?**
No.  BIRT has a tool and service called DART that can retrieve forensic artifacts from Windows and Linux systems.

Velociraptor as a data source is also configurable.  With the triage feature, **many** endpoints can be analyzed at the same time.

## **Client and Server Application System Requirements**
Minimum System Requirements:
- Windows Native, Docker
- 4+ CPU
- 8+GB memory
- SSD or NVME, 10+GB

Suggested System Requirements:
- Docker
- 8+ CPU (Zen 3+)
- 32+GB memory
- NVME, 128GB+