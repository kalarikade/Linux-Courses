# Analyze System Logs for Anomalous Entries

In this step, you will learn how to analyze system logs to find evidence of suspicious activity. System logs are records of events that occur on the system, and they are a critical source of information for security monitoring and forensic analysis. We will focus on authentication logs, which track user logins and privilege escalations, and use standard Linux tools like `grep`, `tail`, and `journalctl` to find anomalous entries.

Most system logs on a Linux system are stored in the `/var/log` directory. One of the most important for security is `/var/log/auth.log`, which records authentication-related events, including SSH logins and `sudo` command usage. Let's start by viewing the last few lines of this file using the `tail` command. You need `sudo` because this file is protected.

```bash
sudo tail /var/log/auth.log
```

You will see a series of timestamped entries. The format typically includes the date, time, hostname, the process that generated the log, and the event message.

```plaintext
Jul 22 10:30:01 labex-vm sudo:    labex : TTY=pts/0 ; PWD=/home/labex/project ; USER=root ; COMMAND=/usr/bin/tail /var/log/auth.log
Jul 22 10:30:01 labex-vm sudo: pam_unix(sudo:session): session opened for user root(uid=0) by (uid=1000)
Jul 22 10:30:01 labex-vm sudo: pam_unix(sudo:session): session closed for user root
```

Now, let's simulate a common suspicious event: a failed login attempt. Attackers often try to guess passwords for valid or common usernames. We can simulate this by trying to SSH into our own machine (`localhost`) with a username that doesn't exist, like `baduser`.

```bash
ssh baduser@localhost
```

The system will prompt you for a password (you can use `labex` as the password). Since the user doesn't exist, any password will fail. Simply press `Ctrl+C` to cancel the password prompt and return to the terminal. The connection attempt will fail, but it will be logged.

Now, let's hunt for the evidence. We can use the `grep` command to filter the `auth.log` file for lines containing the word "failed". This quickly isolates unsuccessful events.

```bash
sudo grep "Failed" /var/log/auth.log
```

The output will now clearly show the log entry for our failed SSH login attempt, which is a strong indicator of a potential intrusion attempt.

```plaintext
Jul 22 10:35:15 labex-vm sshd[12345]: Failed password for invalid user baduser from 127.0.0.1 port 54321 ssh2
```

Modern Linux systems using `systemd` also have a centralized logging system managed by `journald`. The `journalctl` command is a powerful tool to query these logs. Let's use it to find the same failed login event by searching for messages from the `sshd` process. Note that you must use `sudo` to view system-level logs.

```bash
sudo journalctl | grep sshd
```

Scroll through the output (using the arrow keys, and press `q` to exit) and you will find the same "Failed password" entry. To make it even easier, you can filter by message priority. Let's ask `journalctl` to show only entries with a priority of "error" (`err`) or higher for the `sshd` service.

```bash
sudo journalctl -p err | grep sshd
```

This command gives you a much more concise view of only the error conditions, making it very efficient for spotting problems.

```plaintext
-- Logs begin at ... --
Jul 22 10:35:15 labex-vm sshd[12345]: Failed password for invalid user baduser from 127.0.0.1 port 54321 ssh2
-- End of logs --
```

Finally, you can also manually inject messages into the logs using the `logger` command. This is useful for custom scripts to report security events. Let's create a custom alert.

```bash
logger "SECURITY-ALERT: Unauthorized file access detected in /etc"
```

Now, search for this specific alert in the system logs.

```bash
sudo grep "SECURITY-ALERT" /var/log/syslog
```

You will see your custom message, demonstrating how you can integrate your own monitoring with the system's logging facility.

```plaintext
Jul 22 10:40:00 labex-vm labex: SECURITY-ALERT: Unauthorized file access detected in /etc
```

You have now learned the basics of navigating system logs to find evidence of anomalous and potentially malicious activity.