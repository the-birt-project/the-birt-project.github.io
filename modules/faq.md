# <span class="center-text">B<span style="color: #EA3E5D;">IR</span>T Frequently Asked Questions (FAQ)</span>
<br><br>

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

## **Client and Server Application System Requirements**
Minimum System Requirements:
- Windows Native, Docker
- 4+ CPU
- 8+GB memory
- SSD or NVME, 10+GB

Suggested System Requirements:
- Docker
- 16 CPU (Zen 3+)
- 32GB memory
- NVME, 512GB+