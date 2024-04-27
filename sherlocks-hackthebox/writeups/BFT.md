# BFT Sherlock

## Some notes (in italian)

MFT (master file table) è essenzialmente un database che memorizza i metadati per ogni file e directory di un volume NTFS. Ognuna di queste è una entry della tabella e ha un identificatore unico.
La MFT stessa è strutturata come file su un volume NTFS.

Struttura in dettaglio dei campi della MFT nel writeup di HTB.

In the process of identifying the initial ZIP file, we aim to uncover the URL from which it was downloaded. When files are downloaded from the internet using a browser, Alternate Data Streams (ADS) are generated for the downloaded file, containing the URL of the download source. ADS can be used to store additional information associated with a file or directory.

```Created0x10: STANDARD_INFO created timestamp Created0x30: FILE_NAME created timestamp```

## Writeup
```In this very easy Sherlock, you will familiarize yourself with Unix auth.log and wtmp logs. We'll explore a scenario where a Confluence server was brute-forced via its SSH service. After gaining access to the server, the attacker performed additional activities, which we can track using auth.log. Although auth.log is primarily used for brute-force analysis, we will delve into the full potential of this artifact in our investigation, including aspects of privilege escalation, persistence, and even some visibility into command execution.```

`MFTECMD.exe -f "C:\Users\marco\Desktop\$MFT" --csv "C:\users\marco\Desktop"`

- **Simon Stark was targeted by attackers on February 13. He downloaded a ZIP file from a link received in an email. What was the name of the ZIP file he downloaded from the link?** => Click under the column there is a filter option (white row) => Set Created0x10 at 13/02/2024, Set Extension to .zip => 3 files, kape.zip is the artifact collector, stage-xx.zip and invoices.zip, invoices is inside the stage directory so stage-xx.zip is the answer.
- **Examine the Zone Identifier contents for the initially downloaded ZIP file. This field reveals the HostUrl from where the file was downloaded, serving as a valuable Indicator of Compromise (IOC) in our investigation/analysis. What is the full Host URL from where this ZIP file was downloaded?** => Filter with Stage-20240213T093324Z-001.zip in the File Name column, inside ZoneId there is the url.
- **What is the full path and name of the malicious file that executed malicious code and connected to a C2 server?** => Filter with keyword "stage" inside the parent path column => C:\Users\simon.stark\Downloads\Stage-20240213T093324Z-001\Stage\invoice\invoices\invoice.bat
- **Analyze the $Created0x30 timestamp for the previously identified file. When was this file created on disk?** => 2024-02-13 16:38:39
- **Finding the hex offset of an MFT record is beneficial in many investigative scenarios. Find the hex offset of the stager file from Question 3.** => invoices.bat => ENTRY NUMBER * 1024 (size of one record) = OFFSET in decimal => to hex => final answer => 16E3000.
This hexadecimal address is essential for forensic tools to directly navigate to and analyze the file's raw data within the MFT.
- **Each MFT record is 1024 bytes in size. If a file on disk has smaller size than 1024 bytes, they can be stored directly on MFT File itself. These are called MFT Resident files. During Windows File system Investigation, its crucial to look for any malicious/suspicious files that may be resident in MFT. This way we can find contents of malicious files/scripts. Find the contents of The malicious stager identified in Question3 and answer with the C2 IP and port.**

While there is no strict upper limit, typically files smaller than approximately 900 bytes can be fully contained within their MFT record.invoices.bat has a size of 286 bytes.

Open the MFT with an hex editor (HxD for example). Go to => 16E3000. Found the file and the C2 ip and port.
