# Official [Cyber Range](http://joshmadakor.tech/cyber-range) Project

<img width="400" src="https://github.com/user-attachments/assets/44bac428-01bb-4fe9-9d85-96cba7698bee" alt="Tor Logo with the onion and a crosshair on it"/>

# Threat Hunt Report: Unauthorized TOR Usage
- [Scenario Creation](https://github.com/joshmadakor0/threat-hunting-scenario-tor/blob/main/threat-hunting-scenario-tor-event-creation.md)

## Platforms and Languages Leveraged
- Windows 10 Virtual Machines (Microsoft Azure)
- EDR Platform: Microsoft Defender for Endpoint
- Kusto Query Language (KQL)
- Tor Browser

##  Scenario

Management suspects that some employees may be using TOR browsers to bypass network security controls because recent network logs show unusual encrypted traffic patterns and connections to known TOR entry nodes. Additionally, there have been anonymous reports of employees discussing ways to access restricted sites during work hours. The goal is to detect any TOR usage and analyze related security incidents to mitigate potential risks. If any use of TOR is found, notify management.

### 1. Searched the `DeviceFileEvents` Table

Searched the DeviceFileEvents table for ANY files that had the string “tor” in it and discovered what looked like the user “zub-admin” downloaded a tor installer, did something that resulted in many tor-related files being copied to the desktop and the creation of a file called “tor-shopping-list.ext” on the desktop. These events began: 2025-09-04T02:08:00.4886444Z

<img width="979" height="443" alt="image" src="https://github.com/user-attachments/assets/0eeaf8ca-e64b-40df-8fc0-7dae614e5be5" />

<img width="1101" height="248" alt="image" src="https://github.com/user-attachments/assets/55fea40c-97c5-44d7-84cc-4daa8aaabe79" />

### 2. Process Execution - TOR Browser Installation

Searched the Device Process Events table for any ProcessCommandLine that contained “tor-browser-windows-x86_64-portable-14.5.6.exe” and based on the logs returned, At 2:08 AM UTC on September 4, 2025, a computer named lab-endpoint-za logged that the user account zubadmin created a new process. The process launched the file tor-browser-windows-x86_64-portable-14.5.6.exe from the Downloads folder. It was run with the command line tor-browser-windows-x86_64-portable-14.5.6.exe /S, which means the Tor Browser portable installer was started in silent mode, installing or extracting itself without showing any pop-up windows or prompts. The system also recorded the file’s unique SHA-256 hash fingerprint for verification.

<img width="742" height="446" alt="image" src="https://github.com/user-attachments/assets/c1c5cdb0-9f74-4e02-8d0c-f7c76a2c7672" />

### 3. Searched the `DeviceProcessEvents` Table for TOR Browser Execution

Searched DeviceFileEvents for any indication that user “zubadmin” actually opening the tor browser. There was evidence that they did open it at 2025-09-04T02:11:34.032905Z . There were also several instance of firefox.exe (Tor) as well as tor.exe spawned afterwards. 

<img width="1097" height="470" alt="image" src="https://github.com/user-attachments/assets/47ab63d1-ba11-4e92-b5ff-daebb7e66727" />

### 4. Searched the `DeviceNetworkEvents` Table for TOR Network Connections

Searched the DeviceNetworkEvents table for any indication the tor browser was sued to establish a connection using any of the known Tor RemotePorts "9001", "9030", "9050", "9051", "9150", "9151") and discovered that at 2:11 AM UTC on September 4, 2025, the computer lab-endpoint-za recorded that the user account zubadmin successfully made a TCP connection. The process firefox.exe connected to the local machine at 127.0.0.1 on port 9151. This port is typically used by the Tor Browser’s control service, which strongly suggests that the Firefox process associated with Tor Browser was communicating with its built-in Tor component to manage or control the connection. There were also other port connections as well.

<img width="1110" height="309" alt="image" src="https://github.com/user-attachments/assets/1c611af0-3d83-4b1c-a424-e8760e217847" />

- ## a. There were a couple of other connections to sites over port 443


## Summary

The investigation shows that on September 4, 2025, the user zubadmin on host lab-endpoint-za downloaded and executed the Tor Browser portable installer. The installer was launched in silent mode, which extracted the Tor Browser files onto the user’s Desktop without prompting. Shortly after, both firefox.exe (the Tor Browser front-end) and tor.exe (the Tor daemon) were observed running. Firefox established local loopback connections to the Tor control and SOCKS ports, while tor.exe reached out to known Tor relay nodes over ports 9001 and 443. These outbound connections indicate that the system successfully built Tor circuits and was capable of anonymized browsing activity.
In addition to this activity, the user zubadmin created a file named tor-shopping-list, which potentially indicates a set of notes or content related to their Tor Browser activities.
Overall, the evidence confirms that Tor Browser was installed, launched, and actively connected to the Tor network during the observed timeframe, with the presence of the “tor-shopping-list” file suggesting possible documentation of related activity.


## Response Taken

TOR usage was confirmed on endpoint lab-enpoint-za. The device was isolated and the user's direct manager was notified



