# Perform a Localhost Port Scan to Identify Open Services

In this step, you will perform a basic port scan on your local machine to identify any open ports and the services associated with them. Scanning `localhost` (which has the IP address `127.0.0.1`) is a safe way to practice using Nmap, as you are only scanning your own system.

A port scan checks for open "doors" on a computer that services use to communicate over the network. By default, Nmap scans the 1000 most common TCP ports.

To run a basic scan on your local machine, execute the following command in your terminal:

```bash
nmap localhost
```

Nmap will now scan your machine and report its findings. The output will look something like this. Note that the lab environment has been pre-configured to have a service running on port 8000.

```plaintext
Starting Nmap 7.80 ( https://nmap.org ) at 2023-10-27 10:00 UTC
Nmap scan report for localhost (127.0.0.1)
Host is up (0.000087s latency).
Not shown: 998 closed ports
PORT     STATE SERVICE
22/tcp   open  ssh
3001/tcp open  nessus
8000/tcp open  http-alt       SimpleHTTPServer 0.6 (Python 3.10.12)

Nmap done: 1 IP address (1 host up) scanned in 0.08 seconds
```

Let's break down this output:

- `PORT`: The port number and protocol (e.g., `8000/tcp`).
- `STATE`: The status of the port. `open` means a service is actively accepting connections on this port.
- `SERVICE`: The common name for the service typically found on that port (e.g., `ssh` for port 22).

From this scan, you've identified that ports 22 and 8000 are open on your machine.


---
# Interpret Nmap Scan Results for Potential Vulnerabilities

In this step, you will learn how to interpret the Nmap scan results to gather more detailed information, which is crucial for identifying potential vulnerabilities. Knowing a port is open is useful, but knowing the exact software and version running on that port is much more powerful.

An open port itself is not a vulnerability. The risk comes from the service running on that port. If the service is outdated or misconfigured, it could be exploited.

To find out more about the services, you can use Nmap's version detection feature with the `-sV` flag. This will probe the open ports to determine the service and version information.

Run the following command to perform a version scan on `localhost`:

```bash
nmap -sV localhost
```

The output will now be more detailed. Pay close attention to the `VERSION` column.

```plaintext
Starting Nmap 7.80 ( https://nmap.org ) at 2023-10-27 10:05 UTC
Nmap scan report for localhost (127.0.0.1)
Host is up (0.00012s latency).
Not shown: 998 closed ports
PORT     STATE SERVICE VERSION
22/tcp   open  ssh         OpenSSH 8.9p1 Ubuntu 3ubuntu0.10 (Ubuntu Linux; protocol 2.0)
3001/tcp open  ssl/nessus?
8000/tcp open  http        SimpleHTTPServer 0.6 (Python 3.10.12)

Nmap done: 1 IP address (1 host up) scanned in 6.54 seconds
```

As you can see, Nmap has now identified the specific version of the services running. For example, on port 8000, it's not just a generic web service (`http-alt`), but specifically `SimpleHTTPServer 0.6 (Python 3.10.12)`.

With this version information, a security analyst (or an attacker) could search online vulnerability databases (like CVE) for known exploits affecting "SimpleHTTPServer 0.6 (Python 3.10.12)". This is the fundamental process of basic vulnerability scanning.


---
