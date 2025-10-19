# Configure Local Linux Password Policy for Complexity and Age

In this step, you will learn how to strengthen the security of a Linux system by configuring a robust password policy. A strong password policy is a fundamental security control that enforces two critical aspects: **complexity** and **age**. Complexity rules ensure that passwords are not easily guessable by requiring a mix of character types, while aging rules force users to change their passwords periodically, reducing the risk of compromised credentials. You will use standard Linux utilities to modify system-wide configuration files and apply these policies.

First, let's explore password complexity. On modern Debian-based systems like Ubuntu, password quality is managed by the `pam_pwquality` module. Its configuration is stored in the `/etc/security/pwquality.conf` file.

Let's begin by examining the default settings. Use the `cat` command to view the contents of the configuration file. We'll use `grep` to filter out commented lines and empty lines to make the output cleaner.

```bash
grep -vE '^#|^$' /etc/security/pwquality.conf
```

You might see some default settings or the file might be empty of active configurations. Now, we will enforce a stronger policy. We need to edit this file with administrator privileges, so we'll use `sudo` with the `nano` editor.

```bash
sudo nano /etc/security/pwquality.conf
```

Add the following lines to the file to enforce our new policy. These settings require a password to be at least 10 characters long, and contain at least one digit, one uppercase letter, and one lowercase letter.

```
minlen = 10
dcredit = -1
ucredit = -1
lcredit = -1
```

Let's break down these options:

- `minlen = 10`: Sets the minimum acceptable length for the password to 10 characters.
- `dcredit = -1`: Requires at least one digit. The negative number means "at least one".
- `ucredit = -1`: Requires at least one uppercase character.
- `lcredit = -1`: Requires at least one lowercase character.

After adding these lines, save the file and exit `nano` by pressing `Ctrl+X`, then `Y`, and `Enter`.

Now, let's move on to configuring password aging. These settings define the maximum and minimum lifetime of a password. The default values for new user accounts are stored in `/etc/login.defs`.

Use `grep` to find the relevant settings in this file.

```bash
grep -E 'PASS_MAX_DAYS|PASS_MIN_DAYS|PASS_WARN_AGE' /etc/login.defs
```

You will see the default values, which are often set to a very high number for `PASS_MAX_DAYS`, effectively disabling expiration.

```plaintext
#       PASS_MAX_DAYS   Maximum number of days a password may be used.
PASS_MAX_DAYS   99999
#       PASS_MIN_DAYS   Minimum number of days allowed between password changes.
PASS_MIN_DAYS   0
#       PASS_WARN_AGE   Number of days warning is given before a password expires.
PASS_WARN_AGE   7
```

Let's enforce a 900-day expiration policy. Open the file with `sudo nano`.

```bash
sudo nano /etc/login.defs
```

Find the `PASS_MAX_DAYS` line and change its value from `99999` to `900`. You can also set `PASS_MIN_DAYS` to `7` to prevent users from changing their password too frequently.

```
#       PASS_MAX_DAYS   Maximum number of days a password may be used.
PASS_MAX_DAYS   900
#       PASS_MIN_DAYS   Minimum number of days allowed between password changes.
PASS_MIN_DAYS   7
#       PASS_WARN_AGE   Number of days warning is given before a password expires.
PASS_WARN_AGE   14
```

Save the file and exit `nano`. These settings in `/etc/login.defs` apply to users created _after_ this change. To apply the policy to an existing user, like our `labex` user, we use the `chage` command.

First, check the current aging information for the `labex` user.

```bash
sudo chage -l labex
```

The output will show that the password never expires.

```plaintext
Last password change                                    : Jul 22, 2023
Password expires                                        : never
Password inactive                                       : never
Account expires                                         : never
Minimum number of days between password change          : 0
Maximum number of days between password change          : 99999
Number of days of warning before password expires       : 7
```

Now, use `chage` with the `-M` flag to set the maximum number of days to `900` for the `labex` user.

```bash
sudo chage -M 900 labex
```

Verify the change by checking the user's aging information again.

```bash
sudo chage -l labex
```

You will now see that the `Password expires` date has been updated, and the maximum number of days is set to 900, successfully applying the aging policy.

```plaintext
Last password change                                    : Jul 22, 2023
Password expires                                        : Oct 20, 2023
Password inactive                                       : never
Account expires                                         : never
Minimum number of days between password change          : 0
Maximum number of days between password change          : 900
Number of days of warning before password expires       : 7
```

You have now successfully configured both password complexity and aging policies on your Linux system.


---
