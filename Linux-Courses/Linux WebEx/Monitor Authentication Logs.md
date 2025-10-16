# Monitor Authentication Logs for Failed Login Attempts

In this step, you will learn how to monitor system authentication logs to detect potential security threats, such as brute-force attacks. A key part of system administration is regularly reviewing logs for suspicious activity. Failed login attempts are a critical indicator of unauthorized access attempts. You will simulate a failed login and then use standard Linux commands to find and analyze the corresponding log entries.

On Debian-based systems like Ubuntu, authentication events (both successful and failed) are typically recorded in the `/var/log/auth.log` file. This file requires administrator privileges to read.

First, let's generate a failed login attempt. We can do this safely by using the `su` (substitute user) command to try to switch to our own user account, `labex`, but intentionally providing the wrong password.

```bash
su labex
```

The system will prompt you for a password. Type any incorrect password, like `wrongpassword`, and press `Enter`.

```plaintext
Password:
su: Authentication failure
```

Let's do it one more time to create a pattern of failures.

```bash
su labex
```

Again, enter an incorrect password. Now that we have generated some log data, let's examine the authentication log file. We'll use the `tail` command with `sudo` to view the last few lines of `/var/log/auth.log`, which is where the newest events are recorded.

```bash
sudo tail /var/log/auth.log
```

You will see several new lines related to your failed `su` attempts. The output will look similar to this, showing the timestamp, the process (`su`), and the failure message.

```plaintext
Jul 22 16:45:01 labex-vm su[12345]: pam_unix(su:auth): authentication failure; logname=labex uid=1000 euid=0 tty=/dev/pts/0 ruser=labex rhost=  user=labex
Jul 22 16:45:01 labex-vm su[12345]: FAILED SU (to labex) labex on /dev/pts/0
Jul 22 16:45:15 labex-vm su[12346]: pam_unix(su:auth): authentication failure; logname=labex uid=1000 euid=0 tty=/dev/pts/0 ruser=labex rhost=  user=labex
Jul 22 16:45:15 labex-vm su[12346]: FAILED SU (to labex) labex on /dev/pts/0
```

Manually searching through logs can be time-consuming. A more efficient method is to use `grep` to filter for specific keywords. Let's search for all lines containing "authentication failure".

```bash
sudo grep "authentication failure" /var/log/auth.log
```

This command will display only the lines that match the pattern, making it easy to spot the failed attempts you just created.

Now, let's create a simple shell script to automate this monitoring process. This script will check the log for failures and report a summary. In your `~/project` directory, create a new file named `log_monitor.sh` using `nano`.

```bash
nano log_monitor.sh
```

Enter the following Bash script into the editor. This script uses `grep` with the `-c` flag to count the number of failed login attempts and then prints a status message.

```bash
#!/bin/bash

LOG_FILE="/var/log/auth.log"
FAILURE_COUNT=$(sudo grep -c "authentication failure" $LOG_FILE)

echo "--- Authentication Log Monitor ---"
if [ "$FAILURE_COUNT" -gt 0 ]; then
  echo "WARNING: Found $FAILURE_COUNT failed login attempts."
else
  echo "OK: No failed login attempts found."
fi
echo "--------------------------------"
```

Save the file and exit `nano` by pressing `Ctrl+X`, then `Y`, and `Enter`.

Next, make your new script executable using the `chmod` command.

```bash
chmod +x log_monitor.sh
```

Finally, run your monitoring script to see the result.

```bash
./log_monitor.sh
```

Since you generated failed logins, the script will detect them and display a warning message with the exact count.

```plaintext
--- Authentication Log Monitor ---
WARNING: Found 2 failed login attempts.
--------------------------------
```

You have now learned how to simulate, detect, and create a basic monitoring script for failed authentication attempts, a crucial skill for maintaining system security.