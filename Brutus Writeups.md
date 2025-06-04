# Scenario
This is a very easy Sherlock, in which we familiarize with Unix `auth.log` and `wtmp logs`. 
We'll explore a scenario where a Confluence server **was brute-forced via its SSH service**. The `auth.log` file is essential for the brute-forced attack analysis and to **track the additional activities performed by the attacker** after gaining access to the server, we'll delve into the full potential of this artifact in our investigation, including aspects of privilege escalation, persistence, and even some visibility into command execution.
## Skills Learnt

- UNIX log analysis
- wtmp analysis
- BruteForce activity analysis
- Timeline creation
- Contextual Analysis
- Post Exploitation Analysis

## Initial Analysis
We have been provided with two artifacts, the Linux authentication logs and the WTMP output. Lets kick off with a brief explanation of those files.
### auth.log
This file is used for track any action that requires authentication.  Tasks like log in, switch user and others will generate an entry in this log file. This include activities involving `sshd`, `sudo` actions, and `cron` jobs requiring authentication.
#### Fields in auth.log
Entries in `auth.log` typically include the following fields:

- **Date and Time**: The timestamp when the event occurred.
- **Hostname**: The name of the system on which  the event occurred.
- **Service**: The name of the service or daemon reporting the event.
- **PID**: The process ID of the event when was logged.
- **User**: The username involved in the authentication process.
- **Authentication Status**: Details whether the authentication attempt was successful or failed.
- **ID Address/Hostname**: For remote connections, the IP address or hostname of the client attempting to connect.
- **Message**: A detailed message about the event, including any specific error message or codes associated with the authentication attempt.

Below is shows an entry example of a failed password attempt for user named "admin" on exampleserver from a source IP of 192.168.10.101 over port 22 (SSH).

```
Mar 10 10:34:21 exampleserver sshd[19360]: Failed password for invalid user admin from 192.168.10.101 port 22 ssh2
```

### wtmp
The `wtmp` file logs all login and logout events on the system. It's a binary file, **typically located at** `/var/log/wtmp` and present the following information.

- **username**: The name of the user logging in or out.
- **terminal**: The tty device name or terminal 
- **IP address/Hostname**: The IP address or hostname of the user's machine.
- **Login time**: The date and time the user logged in
- **Logout time**: The date and time the user logged out
- **Duration**: Duration of the session

Below is shown an example where the user sebh24 logged in from 192.168.10.101 and the session lasted for a total of 1 minute.

```
sebh24 pts/0 192.168.10.101 Sat Mar 10 10:34:21 - 10:35 (00:01)
```

As told before, `wtmp` is a binary file so we use different tools suhc as `last` or `utmpdump`, in this case we have a tool called `utmp.py` to aid the investigation 