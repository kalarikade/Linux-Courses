# Introduction

Welcome to this lab on Network Traffic Analysis and Secure Remote Access in Linux. Understanding how to monitor network activity and secure communications is a critical skill for any system administrator, developer, or security enthusiast.

In this lab, you will get hands-on experience with a suite of powerful command-line tools. You will learn how to:

- Inspect active network connections and listening services using `netstat` and its modern successor, `ss`.
- Capture and analyze live network packets with `tcpdump`.
- Establish a secure remote connection to your own machine using `ssh`.
- Verify the integrity of DNS responses using `dig` and DNSSEC.

By the end of this lab, you will have a practical understanding of these essential networking utilities and their roles in managing and securing a Linux system.

![labby](https://labex.io/images/labby/avatar-new.png)Labby

Great! Let me prepare the lab VM for you. While it's being set up, you can start reviewing the first step of this lab.

![labby](https://labex.io/images/labby/avatar-new.png)Labby

# Inspect Active Network Connections with Netstat/SS

In this step, you will learn how to view active network connections and listening ports on your system. This is fundamental for understanding which services are running and accessible over the network. We will use two common tools: `netstat` and `ss`.

First, let's use `netstat`, a classic and widely-known utility. The `net-tools` package, which contains `netstat`, was installed for you during the lab setup.

Run the following command to list all listening TCP (`-t`) and UDP (`-u`) ports in a numeric format (`-n`) without resolving names, focusing only on listening sockets (`-l`):

```bash
netstat -tuln
```

You should see an output similar to this. Note the entries for the SSH server (port 22) and the Python web server (port 8000) that were started for you, along with additional services on ports 3001 and 3002:

```plaintext
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN
tcp        0      0 0.0.0.0:3001            0.0.0.0:*               LISTEN
tcp        0      0 0.0.0.0:3002            0.0.0.0:*               LISTEN
tcp        0      0 0.0.0.0:8000            0.0.0.0:*               LISTEN
tcp        0      0 127.0.0.11:37199        0.0.0.0:*               LISTEN
tcp6       0      0 :::22                   :::*                    LISTEN
udp        0      0 127.0.0.11:60120        0.0.0.0:*
udp        0      0 0.0.0.0:3001            0.0.0.0:*
```

Now, let's use `ss`, which is the modern replacement for `netstat`. It is generally faster and provides more detailed information. The command and flags are very similar.

```bash
ss -tuln
```

The output will be slightly different in format but will show the same essential information: the services listening on ports 22, 3001, 3002, and 8000, along with some internal Docker services:

```plaintext
Netid                        State                         Recv-Q                        Send-Q                                               Local Address:Port                                                 Peer Address:Port                        Process
udp                          UNCONN                        0                             0                                                       127.0.0.11:60120                                                     0.0.0.0:*
udp                          UNCONN                        0                             0                                                          0.0.0.0:3001                                                      0.0.0.0:*
tcp                          LISTEN                        0                             128                                                        0.0.0.0:22                                                        0.0.0.0:*
tcp                          LISTEN                        0                             5                                                          0.0.0.0:3001                                                      0.0.0.0:*
tcp                          LISTEN                        0                             128                                                        0.0.0.0:3002                                                      0.0.0.0:*
tcp                          LISTEN                        0                             5                                                          0.0.0.0:8000                                                      0.0.0.0:*
tcp                          LISTEN                        0                             4096                                                    127.0.0.11:37199                                                     0.0.0.0:*
tcp                          LISTEN                        0                             128                                                           [::]:22                                                           [::]:*
```

By using these commands, you can quickly get a snapshot of your system's network-facing services.

---

# Capture Local Network Traffic with Tcpdump

In this step, you will use `tcpdump`, a powerful command-line packet analyzer, to capture and inspect network traffic in real-time. This is invaluable for debugging network issues or understanding how protocols work.

First, let's see which network interfaces are available for capturing traffic.

```bash
sudo tcpdump -D
```

The output will list the available interfaces. `lo` is the "loopback" interface, which is used for network communication within the same machine (e.g., connecting to `localhost`). You'll see various interfaces including the primary network interface `eth1`:

```plaintext
1.eth1 [Up, Running, Connected]
2.any (Pseudo-device that captures on all interfaces) [Up, Running]
3.lo [Up, Running, Loopback]
4.bluetooth-monitor (Bluetooth Linux Monitor) [Wireless]
5.nflog (Linux netfilter log (NFLOG) interface) [none]
6.nfqueue (Linux netfilter queue (NFQUEUE) interface) [none]
7.dbus-system (D-Bus system bus) [none]
8.dbus-session (D-Bus session bus) [none]
```

We will capture traffic on the loopback interface (`lo`) to see the requests to our local web server. To do this, you will need two terminals.

In your current terminal, run the following `tcpdump` command. It will listen on the `lo` interface (`-i lo`) and stop after capturing 5 packets (`-c 5`).

```bash
sudo tcpdump -i lo -c 5
```

Now, **click the `+` icon in the terminal tab bar to open a new terminal**. In this new terminal, generate some network traffic by making a request to the local web server using `curl`.

```bash
curl http://localhost:8000
```

Switch back to your first terminal. You will see that `tcpdump` has captured the packets related to your `curl` request. The output will look something like this, showing the TCP handshake and data transfer:

```plaintext
tcpdump: verbose output suppressed, use -v[v]... for full protocol decode
listening on lo, link-type EN10MB (Ethernet), snapshot length 262144 bytes
14:57:14.158058 IP localhost.57272 > localhost.8000: Flags [S], seq 2025143228, win 65495, options [mss 65495,sackOK,TS val 1006286113 ecr 0,nop,wscale 7], length 0
14:57:14.158072 IP localhost.8000 > localhost.57272: Flags [S.], seq 403368374, ack 2025143229, win 65483, options [mss 65495,sackOK,TS val 1006286113 ecr 1006286113,nop,wscale 7], length 0
14:57:14.158083 IP localhost.57272 > localhost.8000: Flags [.], ack 1, win 512, options [nop,nop,TS val 1006286113 ecr 1006286113], length 0
14:57:14.158129 IP localhost.57272 > localhost.8000: Flags [P.], seq 1:79, ack 1, win 512, options [nop,nop,TS val 1006286113 ecr 1006286113], length 78
14:57:14.158133 IP localhost.8000 > localhost.57272: Flags [.], ack 79, win 511, options [nop,nop,TS val 1006286113 ecr 1006286113], length 0
5 packets captured
24 packets received by filter
0 packets dropped by kernel
```

Note that if you run `tcpdump` without generating specific traffic, you may see background DNS queries and other system communications, as shown in your logs. This is normal system behavior and demonstrates that even when you're not actively browsing, your system maintains various network communications.

You have successfully captured and viewed live network traffic.

---
# Demonstrate Secure Remote Access Using SSH

In this step, you will learn to use SSH (Secure Shell) for secure remote access. We will configure key-based authentication, which is more secure than using passwords, to log into your own machine (`localhost`). The `openssh-server` was already started for you.

First, you need to generate an SSH key pair (a private key and a public key). We will create an RSA key with a bit length of 2048. The `-f` flag specifies the file to save the key, and `-N ""` sets an empty passphrase for convenience in this lab.

```bash
ssh-keygen -t rsa -b 2048 -f ~/.ssh/id_rsa -N ""
```

The command you provided is used to generate a new RSA SSH key pair. Here's a breakdown of the components:

- `ssh-keygen`: The command to create a new SSH key.
- `-t rsa`: Specifies the type of key to create, in this case, RSA.
- `-b 2048`: Sets the number of bits in the key to 2048, which is a common size for security.
- `-f ~/.ssh/id_rsa`: Defines the file path where the private key will be saved. The public key will be saved with the same name but with a `.pub` extension.
- `-N ""`: Sets the passphrase for the key. An empty string means no passphrase is set.

This command will create a private key (`id_rsa`) and a public key (`id_rsa.pub`) in the `~/.ssh` directory.
This command creates `id_rsa` (your private key) and `id_rsa.pub` (your public key) inside the `~/.ssh` directory.

Next, to allow your own user to log in using this key, you must add the public key to the `authorized_keys` file. This file lists all public keys that are permitted to log in.

```bash
cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
```

For security, SSH requires strict permissions on the `.ssh` directory and its files. Let's ensure they are set correctly. The setup script already created these files with the right permissions, but it's good practice to know these commands.

```bash
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys
```

Now everything is set up. You can test the secure connection to `localhost`. The command will execute `echo "Hello from SSH"` on the remote end (which is your own machine) and print the result.

```bash
ssh labex@localhost 'echo "Hello from SSH"'
```

The first time you connect, you may be asked to verify the host's fingerprint. Type `yes` and press Enter. You should then see the output from the echo command.

```plaintext
The authenticity of host 'localhost (127.0.0.1)' can't be established.
ED25519 key fingerprint is SHA256:xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx.
This key is not known by any other names
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added 'localhost' (ED25519) to the list of known hosts.
Hello from SSH
```

You have successfully configured and used SSH for secure, key-based authentication.

---
# Verify DNSSEC Resolution using Dig

In this step, you will use `dig` (Domain Information Groper) to perform DNS queries and learn about DNSSEC (DNS Security Extensions). DNSSEC helps protect against DNS spoofing by adding cryptographic signatures to DNS data.

First, let's perform a standard DNS query for `labex.io` to see its IP address.

```bash
dig labex.io
```

The output will show you the query details and, most importantly, the `ANSWER SECTION` with the A records (IP addresses). Notice that `labex.io` has multiple IP addresses for load balancing:

```plaintext
; <<>> DiG 9.18.24-0ubuntu0.22.04.1-Ubuntu <<>> labex.io
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 9789
;; flags: qr rd ra; QUERY: 1, ANSWER: 3, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 65494
;; QUESTION SECTION:
;labex.io.                      IN      A

;; ANSWER SECTION:
labex.io.               10      IN      A       104.26.10.219
labex.io.               10      IN      A       104.26.11.219
labex.io.               10      IN      A       172.67.72.162

;; Query time: 0 msec
;; SERVER: 127.0.0.11#53(127.0.0.11) (UDP)
;; WHEN: Tue Aug 05 14:58:16 CST 2025
;; MSG SIZE  rcvd: 85
```

Now, let's perform the same query but ask the DNS resolver to provide DNSSEC-related data by adding the `+dnssec` option.

```bash
dig labex.io +dnssec
```

The output includes additional DNSSEC-related information in the OPT PSEUDOSECTION, showing that DNSSEC was requested (the `do` flag means "DNSSEC OK"). However, note that the `flags` section does not contain an `ad` (Authenticated Data) flag:

```plaintext
; <<>> DiG 9.18.24-0ubuntu0.22.04.1-Ubuntu <<>> labex.io +dnssec
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 1042
;; flags: qr rd ra; QUERY: 1, ANSWER: 3, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags: do; udp: 65494
; OPT=5: 05 07 08 0a 0d 0e 0f (".......")
; OPT=6: 01 02 04 ("...")
; OPT=7: 01 (".")
;; QUESTION SECTION:
;labex.io.                      IN      A

;; ANSWER SECTION:
labex.io.               5       IN      A       172.67.72.162
labex.io.               5       IN      A       104.26.11.219
labex.io.               5       IN      A       104.26.10.219

;; Query time: 0 msec
;; SERVER: 127.0.0.11#53(127.0.0.11) (UDP)
;; WHEN: Tue Aug 05 14:58:21 CST 2025
;; MSG SIZE  rcvd: 108
```

The absence of the `ad` flag indicates that while DNSSEC information was requested, the resolver either couldn't validate the signatures or the domain doesn't have DNSSEC enabled.

Let's try another well-known domain that uses DNSSEC, `icann.org`, to see it again.

```bash
dig icann.org +dnssec
```

Again, inspect the header in the output:

```plaintext
; <<>> DiG 9.18.24-0ubuntu0.22.04.1-Ubuntu <<>> icann.org +dnssec
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 54437
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags: do; udp: 65494
; OPT=5: 05 07 08 0a 0d 0e 0f (".......")
; OPT=6: 01 02 04 ("...")
; OPT=7: 01 (".")
;; QUESTION SECTION:
;icann.org.                     IN      A

;; ANSWER SECTION:
icann.org.              10      IN      A       192.0.43.7

;; Query time: 72 msec
;; SERVER: 127.0.0.11#53(127.0.0.11) (UDP)
;; WHEN: Tue Aug 05 14:58:26 CST 2025
;; MSG SIZE  rcvd: 77
```

Similar to the previous query, there's no `ad` flag in the response. This indicates that the DNS resolver in this environment is not performing DNSSEC validation, which is common in containerized or virtualized environments where the DNS resolver might be configured differently than a standard system resolver.

---
