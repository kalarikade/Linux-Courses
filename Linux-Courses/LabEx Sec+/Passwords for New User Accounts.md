# Enforce Strong Passwords for New User Accounts

In this step, you will verify that the password complexity and aging policies you configured in the previous step are being enforced for new user accounts. The best way to test this is to create a new user and attempt to set a password that violates the rules. This will demonstrate how the `pam_pwquality` module and the system's default settings work together to secure new accounts from the moment they are created.

We will use the `adduser` command, which is a user-friendly, interactive script for creating users in Debian-based systems. Let's create a new user named `testuser`.

```bash
sudo adduser testuser
```

The system will immediately prompt you to set a password for the new user. This is where our policy will be tested.

First, let's try to set a simple, weak password that violates our policy. When prompted for the new password, type `password` and press `Enter`.

```plaintext
Adding user `testuser' ...
Adding new group `testuser' (1001) ...
Adding new user `testuser' (1001) with home directory `/home/testuser' ...
Copying files from `/etc/skel' ...
New password:
```

Because `password` violates our rules (it's too short and contains no uppercase letters or digits), the system will reject it. Let's see the reason it gives.

```plaintext
BAD PASSWORD: The password contains less than 1 digits
```

The system rejects it, pointing out the lack of digits. It violates multiple policies, but this is the first one it reports.

The system will prompt you again. Now, let's try a password that meets the length requirement but still violates the complexity rules. Type `12345` and press `Enter`.

```plaintext
BAD PASSWORD: The password contains less than 1 uppercase letters
New password:
```

> Tips: If you accidentally create a user, you can use `sudo deluser testuser` to delete the user and try again.

The system rejects it again. This time, it's because the password lacks a digit. It also lacks an uppercase letter, so it does not meet our complexity requirements. Now, let's provide a password that meets all our criteria. When prompted, type `StrongPass2025` and press `Enter`. You will be asked to retype it for confirmation.

```plaintext
New password:
Retype new password:
passwd: password updated successfully
```

Success! The system accepted the strong password. It will now ask for additional information about the user. You can simply press `Enter` for each prompt to accept the defaults.

```plaintext
Changing the user information for testuser
Enter the new value, or press ENTER for the default
        Full Name []:
        Room Number []:
        Work Phone []:
        Home Phone []:
        Other []:
Is the information correct? [Y/n] Y
```

The user `testuser` has now been created. You can verify that the user exists by using the `id` command.

```bash
id testuser
```

This will show the user's ID, group ID, and group memberships.

```plaintext
uid=1001(testuser) gid=1001(testuser) groups=1001(testuser)
```

Next, let's confirm that the password aging policy we set in `/etc/login.defs` was automatically applied to this new user. Use the `chage` command to view the aging information for `testuser`.

```bash
sudo chage -l testuser
```

The output will show that the `Maximum number of days between password change` is set to `900`, exactly as we configured.

```plaintext
Last password change                                    : Jul 22, 2023
Password expires                                        : Oct 20, 2023
Password inactive                                       : never
Account expires                                         : never
Minimum number of days between password change          : 7
Maximum number of days between password change          : 900
Number of days of warning before password expires       : 14
```

Finally, to keep our system clean, let's remove the test user. The `deluser` command will remove the user account.

```bash
sudo deluser testuser
```

The system will confirm that the user has been removed.

```plaintext
Removing user `testuser' ...
Warning: group `testuser' has no more members.
Done.
```

You have successfully created a new user, tested the password policy enforcement, and verified that default aging rules were applied.