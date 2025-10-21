# Define a Simple Linux Security Baseline for User Defaults

In this step, you will define a simple security baseline by setting default password policies for all new users created on the system. We will edit the `/etc/login.defs` file, which controls these default settings.

First, let's open the configuration file using `nano` with `sudo` privileges, as this is a system-level file.

```bash
sudo nano /etc/login.defs
```

Inside the `nano` editor, scroll down to find the following three parameters related to password aging and modify their values. You can use `Ctrl + W` to search for text within `nano`.

- `PASS_MAX_DAYS`: The maximum number of days a password may be used. We will set this to `90`.
- `PASS_MIN_DAYS`: The minimum number of days allowed between password changes. We will set this to `7`.
- `PASS_WARN_AGE`: The number of days warning is given before a password expires. We will set this to `14`.

Find these lines and change them to look like this:

```
#
# Password aging controls:
#
#	PASS_MAX_DAYS	Maximum number of days a password may be used.
#	PASS_MIN_DAYS	Minimum number of days allowed between password changes.
#	PASS_WARN_AGE	Number of days warning is given before a password expires.
#
PASS_MAX_DAYS	90
PASS_MIN_DAYS	7
PASS_WARN_AGE	14
```

After making the changes, save the file and exit `nano` by pressing `Ctrl + X`, then `Y`, and finally `Enter`.

You have now established a simple security baseline for password policies on your system. Any new user created from this point forward will inherit these settings.

# Verify Baseline Compliance with System Commands

In this step, you will verify that the baseline you defined in the previous step is being correctly applied. The most direct way to do this is to create a new user and inspect their account properties.

First, let's create a new user named `compliance_user`. The `useradd` command will automatically apply the default settings from `/etc/login.defs`. The `-m` flag ensures that a home directory is created for the user.

```bash
sudo useradd -m compliance_user
```

This command won't produce any output if successful. Now, we can check the password aging settings for our new user using the `chage` (change age) command with the `-l` (list) flag.

```bash
sudo chage -l compliance_user
```

You should see an output similar to the following. Notice how the values for "Maximum number of days between password change", "Minimum number of days between password change", and "Number of days of warning before password expires" match the baseline we set in `/etc/login.defs`.

```plaintext
Last password change                                    : Aug 05, 2025
Password expires                                        : Nov 03, 2025
Password inactive                                       : never
Account expires                                         : never
Minimum number of days between password change          : 7
Maximum number of days between password change          : 90
Number of days of warning before password expires       : 14
```

This output confirms that our baseline is effective and the system is compliant with our new policy for new users.

# Configure Auditd for System Call Monitoring

In this step, you will configure the Linux Audit Daemon (`auditd`) to monitor our baseline configuration file, `/etc/login.defs`, for any changes. This is a critical compliance control, as it provides a log of who changed the file and when.

The `auditd` service uses a set of rules to determine what events to log. We will add a new rule to watch for any write or attribute change operations on our file.

We will add our rule to the `/etc/audit/rules.d/audit.rules` file. Using `echo` and `tee` is a safe way to append a line to a system file.

Execute the following command to add the audit rule:

```bash
sudo echo "-w /etc/login.defs -p wa -k login_defs_changes" | sudo tee -a /etc/audit/rules.d/audit.rules
```

Let's break down this rule:

- `-w /etc/login.defs`: Specifies the file path to watch.
- `-p wa`: Sets the permissions to audit. `w` is for write access, and `a` is for attribute changes.
- `-k login_defs_changes`: Assigns a custom key `login_defs_changes` to these audit events, making them easy to search for later.

For the new rule to take effect, you must restart the `auditd` service.

```bash
sudo systemctl restart auditd
```

Now, `auditd` is actively monitoring `/etc/login.defs` for any modifications.

# Generate and Review Audit Logs for Policy Enforcement

In this step, you will test the audit rule by triggering an event and then reviewing the generated logs. This demonstrates that your compliance control is working as expected.

To trigger the audit rule, we need to modify the `/etc/login.defs` file. A simple way to do this without changing the content is to use the `touch` command, which updates the file's access and modification timestamps. This action will be detected by `auditd` as an attribute change.

```bash
sudo touch /etc/login.defs
```

This action should have created an event in the audit log. The logs are stored in `/var/log/audit/audit.log`, but they can be extensive. A better way to find specific events is to use the `ausearch` command with the key we defined in the previous step.

Let's search for all events tagged with our key, `login_defs_changes`:

```bash
sudo ausearch -k login_defs_changes
```

The output will contain one or more records related to the `touch` command. It will look something like this (timestamps and IDs will vary):

```plaintext
----
time->Tue Dec 12 08:35:15 2023
type=PROCTITLE msg=audit(1702370115.321:101): proctitle=746F756368002F6574632F6C6F67696E2E64656673
type=PATH msg=audit(1702370115.321:101): item=0 name="/etc/login.defs" inode=131409 dev=00:1e mode=0100644 ouid=0 ogid=0 rdev=00:00 nametype=NORMAL cap_fp=0 cap_fi=0 cap_fe=0 cap_fver=0 cap_frootid=0
type=CWD msg=audit(1702370115.321:101): cwd="/home/labex/project"
type=SYSCALL msg=audit(1702370115.321:101): arch=c000003e syscall=257 success=yes exit=3 a0=ffffff9c a1=7ffc62e212d0 a2=90800 a3=0 items=1 ppid=1102 pid=1125 auid=1000 uid=0 gid=0 euid=0 suid=0 fsuid=0 egid=0 sgid=0 fsgid=0 tty=pts0 ses=3 comm="touch" exe="/usr/bin/touch" key="login_defs_changes"
```

Look for the `key="login_defs_changes"` at the end of the `SYSCALL` record. This confirms that `auditd` successfully logged the modification event for our critical configuration file, proving our monitoring control is active and effective.

