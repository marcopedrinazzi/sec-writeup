# Unit42 Sherlock
## Some notes (in italian)
Windows Event logs vengono estesi da Sysmon fornendo informazioni piu dettagliate sul comportamento del sistema e delle attività.

Event ID utili (sysmon):
1. event id 1: process creation
2. event id 2: file creation (modification) time changed
3. event id 3: network connection
4. event id 5: process termination
5. event id 11: file created 
6. event id 22: dns query

# Write up

```In this Sherlock, you will familiarize yourself with Sysmon logs and various useful EventIDs for identifying and analyzing malicious activities on a Windows system. Palo Alto's Unit42 recently conducted research on an UltraVNC campaign, wherein attackers utilized a backdoored version of UltraVNC to maintain access to systems. This lab is inspired by that campaign and guides participants through the initial access stage of the campaign.```

- **How many Event logs are there with Event ID 11?** Filter current registry => event 11 => apply => Below the header line there is a line starting with "filtered"
- **Whenever a process is created in memory, an event with Event ID 1 is recorded with details such as command line, hashes, process path, parent process path, etc. This information is very useful for an analyst because it allows us to see all programs executed on a system, which means we can spot any malicious processes being executed. What is the malicious process that infected the victim's system?** check for sus names in the logs filtering with event id 1 (process creation), turns out there is a weird file with a double extension inside the Downloads. A quick google search + hash lookup on virus total confirms it is the malicious binary.
- **Which Cloud drive was used to distribute the malware?** Filter for event 22 (dns query), check for the oldest entry and move forward. It seems dropbox, correlate that rules with event id 11 (file creation event) to see if it is correct (edit the current filter with 22,11) => it is => it is possible to see the .part files of firefox that downloads the file
- **The initial malicious file time-stamped (a defense evasion technique, where the file creation date is changed to make it appear old) many files it created on disk. What was the timestamp changed to for a PDF file?** Image: C:\Users\CyberJunkie\Downloads\Preventivo24.02.14.exe.exe ;; TargetFilename: C:\Users\CyberJunkie\AppData\Roaming\Photo and Fax Vn\Photo and vn 1.1.2\install\F97891C\TempFolder\~.pdf ;;CreationUtcTime: 2024-01-14 08:10:06.029 => Creation UTC time is the answer (filter for event 2)
- **The malicious file dropped a few files on disk. Where was "once.cmd" created on disk? Please answer with the full path along with the filename.** Filter for event 11, use "Find" with the filename => Image: C:\Users\CyberJunkie\Downloads\Preventivo24.02.14.exe.exe; TargetFilename: C:\Users\CyberJunkie\AppData\Roaming\Photo and Fax Vn\Photo and vn 1.1.2\install\F97891C\WindowsVolume\Games\once.cmd
- **The malicious file attempted to reach a dummy domain, most likely to check the internet connection status. What domain name did it try to connect to?** Filter for event 22, we see example.com (post infection from dropbox)
- **Which IP address did the malicious process try to reach out to?** Filter for event id 3
- **The malicious process terminated itself after infecting the PC with a backdoored variant of UltraVNC. When did the process terminate itself?** Filter for event id 5