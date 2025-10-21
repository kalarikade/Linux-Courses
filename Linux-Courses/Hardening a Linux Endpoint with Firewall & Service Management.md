# Introduction

Endpoint hardening is a critical process for securing any computer system. It involves reducing the system's "attack surface," making it less vulnerable to security threats. Two fundamental aspects of hardening a Linux system are managing running services and implementing a firewall.

In this lab, you will get hands-on experience with these essential security practices. You will learn how to:

- Identify and disable services that are not essential for the system's function.
- Install and configure the Uncomplicated Firewall (UFW), a user-friendly interface for managing network filtering rules on Linux.

By the end of this lab, you will have a practical understanding of how to apply basic hardening techniques to a Linux endpoint.


# Identify and Disable Unnecessary Services

In this step, you will learn how to identify active services on your Linux system and disable those that are not required. Reducing the number of running services minimizes the potential attack surface, as each service could potentially have vulnerabilities.

First, let's list all active services. In some Linux environments, especially containers, the `systemctl` command is not available. We can use the `service` command as an alternative.

Execute the following command in your terminal to see the status of all services:

```bash
service --status-all
```

You will see a list of services. A `[ + ]` next to a service name indicates it is running, while a `[ - ]` indicates it is stopped.

```plaintext
 [ - ]  acpid
 [ - ]  apparmor
 [ + ]  apache2
 [ - ]  atop
 [ - ]  atopacct
 [ + ]  cron
...
```

Notice that `apache2`, a web server, is running. For a general-purpose endpoint that doesn't need to host a website, this service is unnecessary and should be disabled.

To stop the `apache2` service, use the `service` command with `stop`. You need `sudo` privileges to manage system services.

```bash
sudo service apache2 stop
```

Now, verify that the service has been stopped by checking its status again:

```bash
service apache2 status
```

The output will confirm that the Apache2 service is no longer running.

```plaintext
○ apache2.service - The Apache HTTP Server
     Loaded: loaded (/lib/systemd/system/apache2.service; enabled; vendor preset: enabled)
     Active: inactive (dead) since Tue 2025-08-05 14:44:53 CST; 2s ago
       Docs: https://httpd.apache.org/docs/2.4/
    Process: 5892 ExecStop=/usr/sbin/apachectl graceful-stop (code=exited, status=0/SUCCESS)
   Main PID: 4800 (code=exited, status=0/SUCCESS)
        CPU: 50ms
```

By disabling unneeded services, you have taken a simple yet effective step to harden your system.



# Install and Configure Uncomplicated Firewall (UFW)

In this step, you will install the Uncomplicated Firewall (UFW). UFW is a user-friendly front-end for the powerful but complex `iptables` firewall. It provides a streamlined way to configure common firewall rules.

First, update your package list and install the `ufw` package.

```bash
sudo apt-get update && sudo apt-get install -y ufw
```

After the installation is complete, UFW will be installed but not yet active. You can check its status with the following command:

```bash
sudo ufw status
```

The output will show that the firewall is inactive:

```plaintext
Status: inactive
```

To activate the firewall, you need to enable it. This command will start the firewall and configure it to launch automatically at system startup.

> **Note:** When enabling a firewall on a remote server, you must ensure you have a rule that allows SSH access. Otherwise, you could be locked out. For this lab environment, it is safe to proceed.

Enable UFW now:

```bash
sudo ufw enable
```

You will be prompted to confirm the operation. The command will enable the firewall and may disrupt existing connections.

```plaintext
Firewall is active and enabled on system startup
```

Let's check the status again, this time with the `verbose` option to see more details, including the default policies.

```bash
sudo ufw status verbose
```

The output will now show the firewall is active and lists the default policies. By default, UFW denies all incoming traffic and allows all outgoing traffic, which is a secure starting point.

```plaintext
Status: active
Logging: on (low)
Default: deny (incoming), allow (outgoing), deny (routed)
New profiles: skip
```

Since the firewall is now active and blocks all incoming connections by default, we need to allow SSH access to ensure we can continue managing the system remotely.

Add a rule to allow SSH connections:

```bash
sudo ufw allow ssh
```

You will see confirmation that the rule has been added:

```plaintext
Rule added
Rule added (v6)
```

This ensures that SSH access remains available while the firewall is active.


# Create and Test Basic Firewall Rules

Now that UFW is active, you'll learn how to create rules to control network traffic. We will add rules to allow specific, necessary services while blocking everything else by default.

First, let's ensure our default policies are set correctly. It's good practice to define them explicitly.

```bash
sudo ufw default deny incoming
sudo ufw default allow outgoing
```

Since we already have SSH access configured from the previous step, let's add a rule for a custom application. Imagine we are running a custom web application on port `8080`. We need to create a rule to allow traffic to this specific port.

```bash
sudo ufw allow 8080/tcp
```

This command specifically allows incoming traffic on TCP port `8080`.

```plaintext
Rule added
Rule added (v6)
```

To see a list of the current rules, you can use `ufw status` with the `numbered` option. This makes it easier to manage or delete rules later.

```bash
sudo ufw status numbered
```

The output will display your new rules with a number next to each one.

```plaintext
Status: active

     To                         Action      From
     --                         ------      ----
[ 1] 22/tcp                     ALLOW IN    Anywhere
[ 2] 8080/tcp                   ALLOW IN    Anywhere
[ 3] 22/tcp (v6)                ALLOW IN    Anywhere (v6)
[ 4] 8080/tcp (v6)              ALLOW IN    Anywhere (v6)
```

You have now configured the firewall to allow only essential traffic, significantly improving the security of your system.


# Review System Logs for Firewall Activity

In this step, you will learn how to enable and review UFW's logging. Logs are essential for monitoring network traffic, identifying potential security threats, and troubleshooting connectivity issues.

UFW's logging is not enabled by default. You can enable it with a simple command.

```bash
sudo ufw logging on
```

You will see a confirmation that logging has been enabled.

```plaintext
Logging enabled
```

UFW logs are written to the `/var/log/ufw.log` file. You can view the log file in real-time using the `tail` command with the `-f` (follow) flag.

```bash
sudo tail -f /var/log/ufw.log
```

This command will display the last few lines of the log and wait for new entries. In a live environment, you would see log entries appear as the firewall blocks or allows connections. Since this lab environment doesn't have external traffic trying to access it, you may not see any new entries. Press `Ctrl+C` to stop following the log.

A typical log entry for a blocked packet looks like this:

```plaintext
Sep 10 12:00:00 ubuntu kernel: [UFW BLOCK] IN=eth0 OUT= MAC=01:02:03:04:05:06:07:08:09:0a:0b:0c SRC=192.168.1.10 DST=192.168.1.20 LEN=40 TOS=0x00 PREC=0x00 TTL=64 ID=12345 PROTO=TCP SPT=54321 DPT=23 WINDOW=0 RES=0x00 SYN URGP=0
```

Here's a quick breakdown of the important fields:

- `[UFW BLOCK]`: Indicates the action taken by the firewall.
- `SRC`: The source IP address of the packet.
- `DST`: The destination IP address.
- `PROTO`: The network protocol (e.g., TCP, UDP).
- `DPT`: The destination port.

You can also search the log file for specific events using `grep`. For example, to find all blocked connection attempts, you can run:

```bash
sudo grep "UFW BLOCK" /var/log/ufw.log
```

This command will filter the log and show only the lines containing the string "UFW BLOCK". This is extremely useful for identifying patterns of suspicious activity.



# Identify and Disable Unnecessary Services

In this step, you will learn how to identify active services on your Linux system and disable those that are not required. Reducing the number of running services minimizes the potential attack surface, as each service could potentially have vulnerabilities.

First, let's list all active services. In some Linux environments, especially containers, the `systemctl` command is not available. We can use the `service` command as an alternative.

Execute the following command in your terminal to see the status of all services:

```bash
service --status-all
```

You will see a list of services. A `[ + ]` next to a service name indicates it is running, while a `[ - ]` indicates it is stopped.

```plaintext
 [ - ]  acpid
 [ - ]  apparmor
 [ + ]  apache2
 [ - ]  atop
 [ - ]  atopacct
 [ + ]  cron
...
```

Notice that `apache2`, a web server, is running. For a general-purpose endpoint that doesn't need to host a website, this service is unnecessary and should be disabled.

To stop the `apache2` service, use the `service` command with `stop`. You need `sudo` privileges to manage system services.

```bash
sudo service apache2 stop
```

Now, verify that the service has been stopped by checking its status again:

```bash
service apache2 status
```

The output will confirm that the Apache2 service is no longer running.

```plaintext
○ apache2.service - The Apache HTTP Server
     Loaded: loaded (/lib/systemd/system/apache2.service; enabled; vendor preset: enabled)
     Active: inactive (dead) since Tue 2025-08-05 14:44:53 CST; 2s ago
       Docs: https://httpd.apache.org/docs/2.4/
    Process: 5892 ExecStop=/usr/sbin/apachectl graceful-stop (code=exited, status=0/SUCCESS)
   Main PID: 4800 (code=exited, status=0/SUCCESS)
        CPU: 50ms
```

By disabling unneeded services, you have taken a simple yet effective step to harden your system.


# Summary

Congratulations! You have successfully completed this lab on hardening a Linux endpoint.

In this lab, you gained practical experience with fundamental security tasks. You have learned how to:

- Identify running services on a Linux system using the `service` command.
- Stop and disable unnecessary services to reduce the system's attack surface.
- Install, enable, and configure the Uncomplicated Firewall (UFW).
- Create specific `allow` rules for essential services like SSH and custom applications.
- Enable and review UFW logs to monitor firewall activity and detect potential threats.

These are foundational skills for any system administrator or developer responsible for maintaining secure and robust Linux environments. By applying these techniques, you can significantly improve the security posture of your systems.



# Review System Logs for Firewall Activity

In this step, you will learn how to enable and review UFW's logging. Logs are essential for monitoring network traffic, identifying potential security threats, and troubleshooting connectivity issues.

UFW's logging is not enabled by default. You can enable it with a simple command.

```bash
sudo ufw logging on
```

You will see a confirmation that logging has been enabled.

```plaintext
Logging enabled
```

UFW logs are written to the `/var/log/ufw.log` file. You can view the log file in real-time using the `tail` command with the `-f` (follow) flag.

```bash
sudo tail -f /var/log/ufw.log
```

This command will display the last few lines of the log and wait for new entries. In a live environment, you would see log entries appear as the firewall blocks or allows connections. Since this lab environment doesn't have external traffic trying to access it, you may not see any new entries. Press `Ctrl+C` to stop following the log.

A typical log entry for a blocked packet looks like this:

```plaintext
Sep 10 12:00:00 ubuntu kernel: [UFW BLOCK] IN=eth0 OUT= MAC=01:02:03:04:05:06:07:08:09:0a:0b:0c SRC=192.168.1.10 DST=192.168.1.20 LEN=40 TOS=0x00 PREC=0x00 TTL=64 ID=12345 PROTO=TCP SPT=54321 DPT=23 WINDOW=0 RES=0x00 SYN URGP=0
```

Here's a quick breakdown of the important fields:

- `[UFW BLOCK]`: Indicates the action taken by the firewall.
- `SRC`: The source IP address of the packet.
- `DST`: The destination IP address.
- `PROTO`: The network protocol (e.g., TCP, UDP).
- `DPT`: The destination port.

You can also search the log file for specific events using `grep`. For example, to find all blocked connection attempts, you can run:

```bash
sudo grep "UFW BLOCK" /var/log/ufw.log
```

This command will filter the log and show only the lines containing the string "UFW BLOCK". This is extremely useful for identifying patterns of suspicious activity.