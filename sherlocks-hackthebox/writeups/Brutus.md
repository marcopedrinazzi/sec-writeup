# Brutus
```In this very easy Sherlock, you will familiarize yourself with Unix auth.log and wtmp logs. We'll explore a scenario where a Confluence server was brute-forced via its SSH service. After gaining access to the server, the attacker performed additional activities, which we can track using auth.log. Although auth.log is primarily used for brute-force analysis, we will delve into the full potential of this artifact in our investigation, including aspects of privilege escalation, persistence, and even some visibility into command execution.```

1. Analyzing the auth.log, can you identify the IP address used by the attacker to carry out a brute force attack? `grep ssh auth.log` or better `grep password auth.log`
2. The brute force attempts were successful, and the attacker gained access to an account on the server. What is the username of this account? From `grep password auth.log` of question 1, it's possible to see the failed password attemps but also the accepted ones. The accepted one is related to the `root` account.
3. Can you identify the timestamp when the attacker manually logged in to the server to carry out their objectives? `utmpdump wtmp > out.txt` look for the root account paired with the identified IP address. => 2024-03-06 06:32:45
4. SSH login sessions are tracked and assigned a session number upon login. What is the session number assigned to the attacker's session for the user account from Question 2? `grep session auth.log` => Removed session 37. => the answer
5. The attacker added a new user as part of their persistence strategy on the server and gave this new user account higher privileges. What is the name of this account? `grep useradd auth.log` `grep usermod auth.log` `grep groupadd auth.log`
6. What is the MITRE ATT&CK sub-technique ID used for persistence? Enterprise Matrix => Persistence => Create Account => Local Account
7. How long did the attacker's first SSH session last based on the previously confirmed authentication time and session ending within the auth.log? (seconds) `grep session auth.log` => `Removed session 37.` at 06:37:24. Start of the session 2024-03-06 06:32:45. Difference in seconds: 279
8. The attacker logged into their backdoor account and utilized their higher privileges to download a script. What is the full command executed using sudo? ```Hint : Although auth.log is not an artifact used to track command executions like auditd etc, when a command is executed with sudo keyword, the command is logged in auth.log since the system needs to authenticate the accounts privileges to get root level permissions for that command. You can search the keyword "COMMAND" to find commands executed using the sudo command.``` To solve the question `grep sudo auth.log`