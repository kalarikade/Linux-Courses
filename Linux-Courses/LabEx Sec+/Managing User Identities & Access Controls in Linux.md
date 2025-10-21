# Introduction

Welcome to the lab on Managing User Identities and Access Controls in Linux. Proper user and access management is a cornerstone of system security and administration. It ensures that users only have access to the resources necessary for their roles, preventing unauthorized actions and potential security breaches.

In this lab, you will get hands-on experience with the essential command-line tools for managing users, groups, and permissions. You will learn how to create user accounts, manage group memberships, control file access with `chmod` and `chown`, and grant specific administrative privileges using `sudo`. By the end of this lab, you will have a solid foundation for securing a Linux environment.


# Create and Manage User Accounts and Groups

In this step, you will learn how to create and manage user accounts and groups, which are fundamental tasks for any Linux system administrator. We will create a new user, a new group, and then add the user to that group.

First, let's create a new user named `alice`. We'll use the `useradd` command with the `-m` option, which tells the system to create a home directory for the user.

```bash
sudo useradd -m alice
```

Next, every user account should have a password. Use the `passwd` command to set a password for `alice`. You will be prompted to enter and confirm the new password.

```bash
sudo passwd alice
```

You can enter a simple password like `password` for this lab. Make sure to type the same password twice - if they don't match, you'll see an error and need to try again.

To verify that the user `alice` has been created, you can check the `/etc/passwd` file, which contains information about all user accounts.

```bash
grep alice /etc/passwd
```

You should see an output line containing details for the `alice` user.

```plaintext
alice:x:5001:5001::/home/alice:/bin/sh
```

Now, let's create a new group called `developers`. We use the `groupadd` command for this.

```bash
sudo groupadd developers
```

To verify the group creation, you can check the `/etc/group` file.

```bash
grep developers /etc/group
```

The output should show the new group.

```plaintext
developers:x:5003:
```

Finally, let's add our new user `alice` to the `developers` group. We use the `usermod` command with the `-aG` options. `-a` stands for append, and `-G` specifies the group(s). It's important to use `-a` to avoid removing the user from other groups they might belong to.

```bash
sudo usermod -aG developers alice
```

To confirm that `alice` is now a member of the `developers` group, use the `groups` command.

```bash
groups alice
```

The output will list all the groups `alice` belongs to, which should now include `developers`.

```plaintext
alice : alice developers
```


---
# Configure File and Directory Permissions (chmod, chown)

In this step, you will learn how to manage file and directory permissions using the `chmod` and `chown` commands. These are essential for controlling who can read, write, or execute files on your system. We will work with a file located at `~/project/reports/quarterly.txt`.

First, let's examine the current ownership and permissions of the file using the `ls -l` command.

```bash
ls -l reports/quarterly.txt
```

The output will look something like this, showing that the file is owned by the `labex` user and group.

```plaintext
-rw-rw-r-- 1 labex labex 20 Aug  5 10:34 reports/quarterly.txt
```

Now, let's change the ownership of this file. We'll make the user `alice` the owner and the `developers` group the group owner. We use the `chown` command for this. The syntax is `chown user:group filename`.

```bash
sudo chown alice:developers reports/quarterly.txt
```

Let's verify the change by running `ls -l` again.

```bash
ls -l reports/quarterly.txt
```

The output now shows the new owner and group.

```plaintext
-rw-rw-r-- 1 alice developers 20 Aug  5 10:34 reports/quarterly.txt
```

Next, we'll modify the file's permissions using the `chmod` command. We want to set the permissions so that:

- The owner (`alice`) has read and write access (`rw-`).
- The group (`developers`) has read-only access (`r--`).
- Others have no access at all (`---`).

In octal (numeric) notation, read is `4`, write is `2`, and execute is `1`. So, `rw-` is `4+2=6`, `r--` is `4`, and `---` is `0`. This gives us the permission code `640`.

Let's apply these permissions.

```bash
sudo chmod 640 reports/quarterly.txt
```

Finally, check the permissions one last time to confirm they are set correctly.

```bash
ls -l reports/quarterly.txt
```

The output should now reflect the new permissions `-rw-r-----`.

```plaintext
-rw-r----- 1 alice developers 20 Aug  5 10:34 reports/quarterly.txt
```


---
# Implement Sudo for Privileged Access Management

In this step, you will learn how to grant privileged access to users using `sudo`. Instead of giving full root access, `sudo` allows fine-grained control over which commands a user can run with elevated privileges. This is a much more secure practice.

The configuration for `sudo` is stored in the `/etc/sudoers` file. The safest way to edit this file is by using the `visudo` command, which locks the file and checks for syntax errors before saving.

Let's give user `alice` permission to run the `apt update` command without needing to enter a password.

Open the `sudoers` file for editing:

```bash
sudo EDITOR=nano visudo
```

This will open the file in a text editor (like `nano`). Scroll to the bottom of the file and add the following line:

```
alice ALL=(ALL) NOPASSWD: /usr/bin/apt update
```

This line means: user `alice` on `ALL` hosts can run commands as `ALL` users (`(ALL)`) without a password (`NOPASSWD:`) for the specified command `/usr/bin/apt update`.

After adding the line, save and exit the editor. In `nano`, you can do this by pressing `Ctrl+O`, then `Enter` to confirm the filename, and `Ctrl+X` to exit.

Now, let's test the new rule. We need to switch to the `alice` user account.

```bash
sudo su - alice
```

Your command prompt will change to indicate you are now logged in as `alice`. Now, try to run the allowed command.

```bash
sudo /usr/bin/apt update
```

The command should execute successfully without asking for a password. You will see the package lists being updated.

Next, let's try to run a command that `alice` is _not_ allowed to run with `sudo`, like `apt upgrade`.

```bash
sudo /usr/bin/apt upgrade
```

This command will fail, and you will see an `[sudo] password for alice:` message indicating that the user `alice` is not allowed to execute this command.

```plaintext
[sudo] password for alice:
```

Press `Ctrl+C` to exit the command.

This confirms our `sudo` rule is working as expected. Now, switch back to your original `labex` user.

```bash
exit
```

---
# Review Sudo Logs for Privileged Access Activity

In this step, you will learn how to review system logs to monitor privileged access. Auditing `sudo` usage is a critical security practice to track who is running commands as root and when.

On Debian-based systems like Ubuntu, `sudo` commands are logged to the `/var/log/auth.log` file. This file is protected, so you need to use `sudo` to read it.

We can use the `grep` command to filter the log and find entries related to the `sudo` commands executed by user `alice` in the previous step.

Run the following command to search the authorization log:

```bash
sudo grep 'sudo.*alice' /var/log/auth.log
```

The output will show several lines. You should be able to identify the logs for both the successful and the failed `sudo` attempts.

A successful attempt will look similar to this, showing the command that was executed:

```plaintext
... labex-vm sudo:    alice : TTY=pts/0 ; PWD=/home/alice ; USER=root ; COMMAND=/usr/bin/apt update
```

A failed attempt will be logged as well, clearly indicating that the command was not allowed:

```plaintext
... labex-vm sudo:    alice : user NOT allowed to execute /usr/bin/apt upgrade on labex-vm ; TTY=pts/0 ; PWD=/home/alice ; USER=root ; COMMAND=/usr/bin/apt upgrade
```

Reviewing these logs allows administrators to monitor for suspicious activity and ensure that privileged commands are being used appropriately.
