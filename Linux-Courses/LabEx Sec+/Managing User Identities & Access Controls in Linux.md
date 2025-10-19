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