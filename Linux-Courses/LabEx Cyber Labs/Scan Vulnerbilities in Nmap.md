# Perform a Basic Network Scan

In this first step, you will get acquainted with Nmap by performing a basic scan. A basic scan is used to discover which ports are open on a target machine. An open port indicates that a service (like a web server or SSH) is running and listening for connections.

The lab environment has been pre-configured with several services running on `localhost` (your own virtual machine) for you to practice on. The `nmap` tool is already installed.

1. First, let's verify that Nmap is installed and check its version. Open the terminal and run the following command:
    
    ```bash
    nmap --version
    ```
    
    You should see output confirming the Nmap version, similar to this:
    
    ```plaintext
    Nmap version 7.80 ( https://nmap.org )
    Platform: x86_64-pc-linux-gnu
    Compiled with: liblua-5.3.3 openssl-1.1.1f libssh2-1.8.0 libz-1.2.11 libpcre-8.39 nmap-libpcap-1.9.1
    Compiled without:
    Available nsock engines: epoll poll select
    ```
    
2. Now, perform your first scan against `localhost`. This command tells Nmap to check for the most common open ports on your local machine.
    
    ```bash
    nmap localhost
    ```
    
3. Examine the output. Nmap will list the ports it found to be open, along with the state and the common service associated with that port. The output will look something like this, showing the services prepared for this lab:
    
    ```plaintext
    Starting Nmap 7.80 ( https://nmap.org ) at ...
    Nmap scan report for localhost (127.0.0.1)
    Host is up (0.000092s latency).
    Not shown: 995 closed ports
    PORT     STATE SERVICE
    22/tcp   open  ssh
    2121/tcp open  ccproxy-ftp
    2222/tcp open  EtherNetIP-1
    3001/tcp open  nessus
    8080/tcp open  http-proxy
    
    Nmap done: 1 IP address (1 host up) scanned in 0.04 seconds
    ```
    

This initial scan gives you a map of the running services, which is the first step in any network security assessment.

## To close open ports, you can follow these general steps:

1. **Identify Open Ports**: First, determine which ports are currently open on your system. You can use tools like Nmap or the `netstat` command. For example, to check open ports using Nmap, run:
    
    ```bash
    nmap localhost
    ```
    
2. **Stop the Service**: If a specific service is running on an open port that you want to close, you can stop that service. For example, if you want to stop an HTTP server running on port 80, you can use:
    
    ```bash
    sudo systemctl stop apache2  # For Apache
    sudo systemctl stop nginx     # For Nginx
    ```
    
3. **Disable the Service**: To prevent the service from starting again on boot, you can disable it:
    
    ```bash
    sudo systemctl disable apache2  # For Apache
    sudo systemctl disable nginx     # For Nginx
    ```
    
4. **Configure Firewall Rules**: Use a firewall to block access to specific ports. For example, using `ufw` (Uncomplicated Firewall) on Ubuntu, you can deny access to port 80 like this:
    
    ```bash
    sudo ufw deny 80
    ```
    
5. **Check Firewall Status**: After making changes, check the status of your firewall to ensure the rules are applied:
    
    ```bash
    sudo ufw status
    ```
    
6. **Re-scan for Open Ports**: Finally, re-scan your system to confirm that the ports are closed:
    
    ```bash
    nmap localhost
    ```
    

By following these steps, you can effectively close open ports and enhance the security of your system.

## Configuring Firewall Rules

Configuring firewall rules depends on the firewall software you are using. Below are examples for two common firewall management tools: `iptables` (Linux) and `ufw` (Uncomplicated Firewall, Ubuntu).

### Using `iptables`

1. **View Current Rules**:
    
    ```bash
    sudo iptables -L
    ```
    
2. **Allow Incoming Traffic on a Specific Port** (e.g., port 80):
    
    ```bash
    sudo iptables -A INPUT -p tcp --dport 80 -j ACCEPT
    ```
    
3. **Deny Incoming Traffic on a Specific Port** (e.g., port 22):
    
    ```bash
    sudo iptables -A INPUT -p tcp --dport 22 -j DROP
    ```
    
4. **Save the Rules** (to persist after reboot):
    
    ```bash
    sudo iptables-save | sudo tee /etc/iptables/rules.v4
    ```
    
5. **Flush All Rules** (if needed):
    
    ```bash
    sudo iptables -F
    ```
    

### Using `ufw`

1. **Enable UFW** (if not already enabled):
    
    ```bash
    sudo ufw enable
    ```
    
2. **Allow Incoming Traffic on a Specific Port** (e.g., port 80):
    
    ```bash
    sudo ufw allow 80/tcp
    ```
    
3. **Deny Incoming Traffic on a Specific Port** (e.g., port 22):
    
    ```bash
    sudo ufw deny 22/tcp
    ```
    
4. **Check Status and Rules**:
    
    ```bash
    sudo ufw status
    ```
    
5. **Delete a Rule** (if needed):
    
    ```bash
    sudo ufw delete allow 80/tcp
    ```
    

### General Tips

- Always review your firewall rules to ensure they meet your security requirements.
- Test your configuration after making changes to ensure that the desired ports are open or closed as intended.
- Consider logging dropped packets for troubleshooting purposes.

By following these steps, you can effectively configure firewall rules to manage network traffic on your system.

## Common firewall management tools include:

1. **iptables**: A powerful command-line utility for configuring the Linux kernel firewall. It allows users to set up rules for filtering network traffic.
    
2. **ufw (Uncomplicated Firewall)**: A user-friendly front-end for `iptables` designed to simplify the process of managing firewall rules on Ubuntu and other Debian-based systems.
    
3. **firewalld**: A dynamic firewall management tool for Linux that provides a more user-friendly interface than `iptables`. It uses zones to define the trust level of network connections.
    
4. **pf (Packet Filter)**: A firewall tool used primarily on BSD operating systems (like FreeBSD and OpenBSD). It provides advanced features like stateful packet inspection and traffic shaping.
    
5. **Windows Firewall**: Built into Windows operating systems, it provides basic firewall protection and can be managed through the Control Panel or Windows PowerShell.
    
6. **Cisco ASA (Adaptive Security Appliance)**: A hardware-based firewall solution that provides advanced security features for enterprise networks.
    
7. **Check Point Firewall**: A comprehensive security solution that includes firewall capabilities along with intrusion prevention and VPN support.
    
8. **Sophos XG Firewall**: A next-generation firewall that combines traditional firewall features with advanced threat protection and web filtering.
    

These tools vary in complexity and features, allowing users to choose the one that best fits their needs and expertise level.

## Examples of common firewall actions 

### 1. **Allow**

- **Example**: Allowing web traffic.
    
    - **Rule**: Allow incoming TCP traffic on port 80 (HTTP).
    - **Scenario**: You want users to access your website, so you create a rule to allow traffic on port 80.
    
    ```bash
    sudo ufw allow 80/tcp
    ```
    

### 2. **Deny**

- **Example**: Denying SSH access.
    
    - **Rule**: Deny incoming TCP traffic on port 22 (SSH).
    - **Scenario**: You want to prevent remote access to your server via SSH for security reasons.
    
    ```bash
    sudo ufw deny 22/tcp
    ```
    

### 3. **Drop**

- **Example**: Dropping unwanted traffic.
    
    - **Rule**: Drop incoming UDP traffic from a specific IP address.
    - **Scenario**: You notice suspicious activity from a particular IP and want to block it without notifying the sender.
    
    ```bash
    sudo iptables -A INPUT -s 192.168.1.100 -j DROP
    ```
    

### 4. **Reject**

- **Example**: Rejecting specific traffic.
    
    - **Rule**: Reject incoming TCP traffic on port 23 (Telnet).
    - **Scenario**: You want to block Telnet access and inform users that the connection is rejected.
    
    ```bash
    sudo iptables -A INPUT -p tcp --dport 23 -j REJECT
    ```
    

### 5. **Log**

- **Example**: Logging denied traffic.
    
    - **Rule**: Log all incoming traffic that is denied.
    - **Scenario**: You want to monitor attempts to access your server that are blocked by the firewall.
    
    ```bash
    sudo iptables -A INPUT -j LOG --log-prefix "Denied: "
    ```
    

### 6. **Limit**

- **Example**: Limiting connection attempts.
    
    - **Rule**: Limit incoming connections on port 80 to 10 per minute from a single IP.
    - **Scenario**: You want to prevent denial-of-service attacks by limiting the number of requests from a single source.
    
    ```bash
    sudo iptables -A INPUT -p tcp --dport 80 -m conntrack --ctstate NEW -m limit --limit 10/minute -j ACCEPT
    ```
    ### Breakdown of the Command

1. **`sudo`**:
    
    - This command is run with superuser privileges, which is necessary for modifying firewall rules.
2. **`iptables`**:
    
    - This is the command-line utility used to configure the Linux kernel's packet filtering rules.
3. **`-A INPUT`**:
    
    - `-A` stands for "append." This option adds a new rule to the `INPUT` chain, which is responsible for handling incoming traffic to the server.
4. **`-p tcp`**:
    
    - This specifies the protocol for the rule. In this case, it is set to `tcp`, which is commonly used for web traffic.
5. **`--dport 80`**:
    
    - This option specifies the destination port for the incoming traffic. Here, it is set to `80`, which is the standard port for HTTP traffic (web traffic).
6. **`-m conntrack`**:
    
    - This option loads the `conntrack` module, which allows the rule to track the state of connections.
7. **`--ctstate NEW`**:
    
    - This specifies that the rule should only apply to new connections. In other words, it will only match packets that are initiating a new connection.
8. **`-m limit`**:
    
    - This option loads the `limit` module, which allows you to set limits on the rate of matching packets.
9. **`--limit 10/minute`**:
    
    - This sets the limit for the rule to allow a maximum of 10 new connections per minute from a single source IP address. If the limit is exceeded, further connections will be dropped.
10. **`-j ACCEPT`**:
    
    - `-j` stands for "jump." This specifies the target action for the rule. In this case, it tells `iptables` to accept the packets that match the criteria defined in the rule.

### Summary

In summary, this command configures the firewall to allow a maximum of 10 new TCP connections to port 80 (HTTP) per minute from any single source IP address. This helps to prevent denial-of-service attacks by limiting the rate of incoming connections while still allowing legitimate traffic.

### 7. **Redirect**

- **Example**: Redirecting HTTP traffic.
    
    - **Rule**: Redirect incoming traffic from port 80 to port 8080.
    - **Scenario**: You want to run a web application on a different port while still allowing users to access it via the standard HTTP port.
    
    ```bash
    sudo iptables -t nat -A PREROUTING -p tcp --dport 80 -j REDIRECT --to-port 8080
    ```
    ### Breakdown of the Command

1. **`sudo`**:
    
    - This command is executed with superuser privileges, which are required to modify firewall rules.
2. **`iptables`**:
    
    - This is the command-line utility used to configure the Linux kernel's packet filtering rules.
3. **`-t nat`**:
    
    - The `-t` option specifies the table to use. In this case, `nat` refers to the Network Address Translation table, which is used for modifying packets that create new connections.
4. **`-A PREROUTING`**:
    
    - `-A` stands for "append." This option adds a new rule to the `PREROUTING` chain. The `PREROUTING` chain is used to alter packets as they arrive before any routing decisions are made.
5. **`-p tcp`**:
    
    - This specifies the protocol for the rule. Here, it is set to `tcp`, which is commonly used for web traffic.
6. **`--dport 80`**:
    
    - This option specifies the destination port for the incoming traffic. In this case, it is set to `80`, which is the standard port for HTTP traffic.
7. **`-j REDIRECT`**:
    
    - `-j` stands for "jump." This specifies the target action for the rule. In this case, it tells `iptables` to redirect the matching packets.
8. **`--to-port 8080`**:
    
    - This option specifies the port to which the traffic should be redirected. Here, it indicates that incoming traffic on port 80 should be redirected to port 8080.

### Summary

In summary, this command configures the firewall to redirect all incoming TCP traffic on port 80 (HTTP) to port 8080. This is useful when you want to run a web application on a different port while still allowing users to access it via the standard HTTP port.

### Conclusion

These examples illustrate how different firewall actions can be applied to manage network traffic effectively. Each action serves a specific purpose, helping to enhance security and control over your network environment.

# Detect Service Versions

Knowing which ports are open is useful, but knowing the exact software and version running on those ports is much more powerful for a security analyst. Outdated software is a primary source of vulnerabilities. In this step, you will use Nmap to perform service and version detection.

We will use the `-sV` flag, which instructs Nmap to probe open ports to determine detailed service and version information.

1. Run a version detection scan against `localhost`. To make the scan more efficient, we'll target the specific ports we discovered in Step 1 (22, 2121, 2222, 3001, 8080) rather than scanning all ports.
    
    ```bash
    nmap -sV -p 22,8080 localhost
    ```
    
    > **Pro Tip**: Targeting specific ports dramatically reduces scan time. A full port range scan with `-sV` can take several minutes, while this targeted approach typically completes in seconds.
    
2. Compare the output with the basic scan from Step 1. You will now see an additional column, `VERSION`, which provides details about the software running on each port.
    
    The output will be more detailed, similar to this:
    
    ```plaintext
    Starting Nmap 7.80 ( https://nmap.org ) at ...
    Nmap scan report for localhost (127.0.0.1)
    Host is up (0.00011s latency).
    Not shown: 995 closed ports
    PORT     STATE SERVICE VERSION
    22/tcp   open  ssh     OpenSSH 8.9p1 Ubuntu 3ubuntu0.13 (Ubuntu Linux; protocol 2.0)
    8080/tcp open  http    nginx 1.18.0 (Ubuntu)
    Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
    
    Nmap done: 1 IP address (1 host up) scanned in 2.15 seconds
    ```
    

This information is critical. For example, if the scan revealed an old version of `nginx` with a known critical vulnerability, you would know exactly where to focus your remediation efforts.

# Use the Nmap Scripting Engine (NSE)

The Nmap Scripting Engine (NSE) is one of Nmap's most powerful features. It allows you to automate a wide variety of networking tasks using a library of scripts. These scripts can be used for more advanced discovery, vulnerability detection, and even exploitation.

In this step, you will use the `-sC` flag, which runs a set of default scripts that are considered safe and useful for discovery.

1. Run an Nmap scan with the default scripts enabled against `localhost`. The `-sC` flag is a convenient way to get more information without specifying individual scripts.
    
    ```bash
    nmap -sC localhost
    ```
    
2. Review the output. You will see additional information indented under each port. This is the output from the NSE scripts. For example, the `http-title` script might grab the title of the web page on port `8080`, and SSL scripts might report on certificate details.
    
    The output will be even more verbose:
    
    ```plaintext
    Starting Nmap 7.80 ( https://nmap.org ) at ...
    Nmap scan report for localhost (127.0.0.1)
    Host is up (0.000099s latency).
    Other addresses for localhost (not scanned): ::1
    Not shown: 995 closed ports
    PORT     STATE SERVICE
    22/tcp   open  ssh
    2121/tcp open  ccproxy-ftp
    2222/tcp open  EtherNetIP-1
    3001/tcp open  nessus
    | ssl-cert: Subject: commonName=localhost
    | Subject Alternative Name: DNS:localhost
    | Not valid before: 2024-07-18T03:37:05
    |_Not valid after:  2034-07-16T03:37:05
    8080/tcp open  http-proxy
    |_http-open-proxy: Proxy might be redirecting requests
    |_http-title: Site doesn't have a title (text/html).
    
    Nmap done: 1 IP address (1 host up) scanned in 0.62 seconds
    ```
    

As you can see, the default scripts discovered that anonymous FTP login is allowed on port `2121` and retrieved the title of the web page on port `8080`. This is valuable intelligence gathered automatically.

# Run a Vulnerability Scan

Now it's time to hunt for vulnerabilities. The NSE includes a category of scripts specifically designed to check for known security flaws. You can run all scripts in the `vuln` category to perform a broad vulnerability scan.

It is best practice to always save the output of long or important scans to a file. We will use the `-oN` flag to save the output in Nmap's normal format.

1. First, let's combine what you've learned. Run a scan that includes service detection (`-sV`) and executes all vulnerability scripts (`--script vuln`). Save the output to a file named `vuln_scan.txt`.
    
    ```bash
    nmap -sV --script vuln -oN vuln_scan.txt localhost
    ```
    
    This scan may take a few minutes as it is running many scripts against each open port.
    
2. Once the scan is complete, a file named `vuln_scan.txt` will be created in your current directory (`/home/labex/project`). You can view its contents using the `cat` command:
    
    ```bash
    cat vuln_scan.txt
    ```
    
3. The output file is long, so it's more efficient to search for keywords. Use `grep` to look for lines that indicate a vulnerability. The term "VULNERABLE" is a strong indicator.
    
    ```bash
    grep "VULNERABLE" vuln_scan.txt
    ```
    
    You should see output showing any vulnerabilities found. In this lab environment, you'll likely see a vulnerability related to Apache's byterange filter:
    
    ```plaintext
    |   VULNERABLE:
    |     State: VULNERABLE
    ```
    
    To see the full vulnerability details, you can search for the specific CVE or look at the complete scan output. For example, you might find an Apache DoS vulnerability (CVE-2011-3192) on port 8080. Note that you may also see some script errors (like `clamav-exec: ERROR`) which are normal and can be ignored - these occur when certain vulnerability scripts cannot execute properly in the lab environment.
    

This step demonstrates how to actively probe for weaknesses, moving from simple discovery to a targeted security audit.

## Useful Nmap Scripts
Nmap has a wide range of scripts available through its Nmap Scripting Engine (NSE) that can be useful for various tasks. Here are some categories and examples of useful Nmap scripts:

### 1. **Discovery Scripts**

- **`discovery`**: Scripts in this category help identify hosts and services on a network.
    - **Example**: `http-enum` - Enumerates directories and files on a web server.

### 2. **Vulnerability Scanning Scripts**

- **`vuln`**: These scripts check for known vulnerabilities in services.
    - **Example**: `smb-vuln-ms17-010` - Checks for the EternalBlue vulnerability in SMB.

### 3. **Auth Scripts**

- **`auth`**: Scripts that help with authentication-related tasks.
    - **Example**: `http-auth` - Checks for HTTP authentication methods supported by a web server.

### 4. **Exploit Scripts**

- **`exploit`**: Scripts that attempt to exploit vulnerabilities.
    - **Example**: `http-shellshock` - Tests for the Shellshock vulnerability in web servers.

### 5. **Malware Detection Scripts**

- **`malware`**: Scripts that help identify malware-related issues.
    - **Example**: `http-malware-host` - Checks if a web server is hosting known malware.

### 6. **Information Gathering Scripts**

- **`info`**: Scripts that gather information about services.
    - **Example**: `http-title` - Retrieves the title of a web page.

### 7. **DNS Scripts**

- **`dns`**: Scripts that perform DNS-related tasks.
    - **Example**: `dns-zone-transfer` - Checks if a DNS zone transfer is allowed.

### 8. **SSL/TLS Scripts**

- **`ssl`**: Scripts that check SSL/TLS configurations.
    - **Example**: `ssl-enum-ciphers` - Enumerates supported SSL/TLS ciphers on a server.

### How to Use Nmap Scripts

You can run these scripts using the `--script` option followed by the script name or category. For example:

```bash
nmap --script http-enum -p 80 localhost
```

### Conclusion

These are just a few examples of the many useful Nmap scripts available. You can explore more scripts by listing them with:

```bash
ls /usr/share/nmap/scripts/
```

## How to write your own Nmap Scripts
Writing your own Nmap script involves using the Lua programming language, which is the scripting language used by the Nmap Scripting Engine (NSE). Here’s a step-by-step guide to help you create a simple Nmap script:

### Step 1: Set Up Your Environment

1. **Install Nmap**: Ensure you have Nmap installed on your system.
2. **Familiarize with Lua**: Basic knowledge of Lua will help you write scripts effectively. You can find resources online to learn Lua.

### Step 2: Create Your Script File

1. **Create a New File**: Use a text editor to create a new file with a `.nse` extension. For example, `my_script.nse`.

### Step 3: Write the Script

Here’s a simple example of an Nmap script that checks if a web server is running on a specified port:

```lua
description = [[
  A simple script to check if a web server is running.
]]

---
-- @usage
-- nmap -p 80 --script my_script
-- 
-- @output
-- PORT   STATE SERVICE
-- 80/tcp open  http
-- 
-- @args port The port to check (default is 80).
--

local port = 80

action = function(host, port)
  local socket = nmap.new_socket()
  socket:set_timeout(5000)

  local status, err = socket:connect(host.ip, port)
  if status then
    return "Web server is running on port " .. port
  else
    return "No web server found on port " .. port
  end
end
```

### Step 4: Save the Script

Save your script file in the Nmap scripts directory, typically located at `/usr/share/nmap/scripts/`.

### Step 5: Run Your Script

You can run your script using the following command:

```bash
nmap --script my_script -p 80 localhost
```

### Step 6: Debugging and Testing

- **Debugging**: If your script doesn’t work as expected, check the Nmap output for errors. You can also add print statements to help debug.
- **Testing**: Test your script against different hosts and ports to ensure it behaves as intended.

### Conclusion

Creating your own Nmap script allows you to extend its functionality for specific tasks. Start with simple scripts and gradually incorporate more complex logic as you become comfortable with Lua and the Nmap scripting framework.

# Save and Format Scan Findings

Properly documenting and reporting your findings is a critical skill for any security professional. Nmap supports several output formats suitable for different purposes. In this final step, you will learn how to save your results in multiple formats and create a user-friendly HTML report.

1. First, create a dedicated directory to keep your reports organized.
    
    ```bash
    mkdir -p ~/project/reports
    ```
    
2. Now, run the scan again, but this time save the output in two formats simultaneously: normal text (`-oN`) and XML (`-oX`). XML is a structured format that is ideal for processing with other tools.
    
    ```bash
    nmap -sV -p 8080 --script vuln -oN ~/project/reports/scan_report.txt -oX ~/project/reports/scan_report.xml localhost
    ```
    
3. The XML format is not very human-readable. Nmap provides a utility called `xsltproc` to convert the XML file into a clean HTML report. Run the following command to generate `scan_report.html`.
    
    ```bash
    xsltproc ~/project/reports/scan_report.xml -o ~/project/reports/scan_report.html
    ```
    
4. Let's verify that all your report files have been created in the `~/project/reports` directory. Use the `ls -l` command to list the files and their details.
    
    ```bash
    ls -l ~/project/reports
    ```
    
    You should see your three report files:
    
    ```plaintext
    total 40
    -rw-rw-r-- 1 labex labex 14276 Aug 28 15:12 scan_report.html
    -rw-rw-r-- 1 labex labex  5686 Aug 28 15:11 scan_report.txt
    -rw-rw-r-- 1 labex labex 14924 Aug 28 15:11 scan_report.xml
    ```
    

You now have a plain text file for quick review, an XML file for machine processing, and an HTML file for easy sharing and presentation.

