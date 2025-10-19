# CompTIA Network+ N10-009 4.2 Study Guide: Common Network Attacks

[#networking](https://zeroday.forem.com/t/networking) [#comptia](https://zeroday.forem.com/t/comptia) [#beginners](https://zeroday.forem.com/t/beginners) [#cybersecurity](https://zeroday.forem.com/t/cybersecurity)

## [](https://zeroday.forem.com/andrew_despres/comptia-network-n10-009-42-study-guide-common-network-attacks-k1o#understanding-network-threats-and-vulnerabilities)**Understanding Network Threats and Vulnerabilities**

This guide synthesizes critical information regarding common network attacks, providing the foundational knowledge required for the CompTIA Network+ (N10-009) exam. The focus is on understanding the mechanics of each attack, its purpose, and the recommended mitigation strategies.

---

### [](https://zeroday.forem.com/andrew_despres/comptia-network-n10-009-42-study-guide-common-network-attacks-k1o#1-spoofing-the-art-of-impersonation)**1. Spoofing: The Art of Impersonation**

[![Visualization of Spoofing attacks](https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fxg9k5av3viwvosz0x22x.png)](https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fxg9k5av3viwvosz0x22x.png)_Screenshot from [https://www.geeksforgeeks.org/ethical-hacking/what-is-arp-spoofing-attack/](https://www.geeksforgeeks.org/ethical-hacking/what-is-arp-spoofing-attack/)_

Spoofing is the fundamental act of one person or device pretending to be another. It is a core technique that enables many other, more complex attacks. Think of it like receiving a phone call where the Caller ID shows a familiar number, but the person on the other end is an impostor—that's **Caller ID spoofing**. Similarly, an email that appears to be from a colleague but is actually from an attacker is **email address spoofing**. The ultimate goal of spoofing is to gain access or trust that would not normally be granted, often to position an attacker for an on-path attack.

### [](https://zeroday.forem.com/andrew_despres/comptia-network-n10-009-42-study-guide-common-network-attacks-k1o#arp-poisoning-ip-spoofing)**ARP Poisoning (IP Spoofing)**

The Address Resolution Protocol (ARP) is the mechanism used on a local network to translate a known IP address into its corresponding physical MAC address. This process has a critical vulnerability: it lacks any authentication. ARP poisoning exploits this weakness.

- **How it Works:**
    1. A legitimate device (e.g., a workstation at 192.168.1.9) needs to send data to the router (192.168.1.1). It sends a broadcast ARP request asking, "Who has 192.168.1.1?"
    2. The router responds with its MAC address. The workstation stores this IP-to-MAC mapping in its local ARP cache.
    3. An attacker on the same network (e.g., at 192.168.1.14) sends an unsolicited ARP response to the workstation. This malicious response claims, "I am 192.168.1.1, and my MAC address is [attacker's MAC address]."
    4. The workstation's ARP cache is "poisoned"—it overwrites the legitimate router's MAC address with the attacker's MAC address.
- **The Result:** All traffic the workstation intends to send to the router is now sent to the attacker's machine first. The attacker can monitor, modify, or drop the traffic before forwarding it to the legitimate router, making them a "man in the middle" without the victim or the router knowing.

### [](https://zeroday.forem.com/andrew_despres/comptia-network-n10-009-42-study-guide-common-network-attacks-k1o#dns-poisoning-dns-spoofing)**DNS Poisoning (DNS Spoofing)**

DNS poisoning involves modifying DNS data to redirect users to malicious sites. This can be achieved by compromising the DNS server itself or by intercepting and altering DNS responses in transit.

- **How it Works:**
    1. A user's computer requests the IP address for example.com from its configured DNS server.
    2. An attacker, often using an on-path attack like ARP poisoning, intercepts this communication.
    3. Alternatively, the attacker hacks the DNS server and changes the legitimate IP address for example.com to the IP address of a malicious server they control.
    4. When a user requests the site, the DNS server (or the attacker) provides the fake, "poisoned" IP address.
    5. The user's computer caches this incorrect information and, when attempting to visit the website, is redirected to the attacker's fake web server.
- **Local Modification:** An attacker can also achieve DNS poisoning by modifying the hosts file on a client's computer. The hosts file has a higher priority than DNS, so any entries in it will override responses from a DNS server.

### [](https://zeroday.forem.com/andrew_despres/comptia-network-n10-009-42-study-guide-common-network-attacks-k1o#2-denial-of-service-dos-and-distributed-denial-of-service-ddos)**2. Denial of Service (DoS) and Distributed Denial of Service (DDoS)**

[![Visualization of DoS Attacks](https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fgvwf623we16o1txbw9jt.png)](https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fgvwf623we16o1txbw9jt.png)_Screenshot from [https://www.geeksforgeeks.org/computer-networks/what-is-ddosdistributed-denial-of-service/](https://www.geeksforgeeks.org/computer-networks/what-is-ddosdistributed-denial-of-service/)_

A Denial of Service (DoS) attack is any action that causes a service to fail by making it unavailable to legitimate users. This is often accomplished by overwhelming a system with more traffic or requests than it can handle.

- **Attack Goals:**
    - **Competitive Advantage:** A company might launch a DoS attack against a competitor to make their services unavailable.
    - **Distraction:** Attackers can use a DoS attack on one server as a smokescreen while they target a different, more valuable part of the network.
- **Types of DoS Attacks:**
    - **Overloading:** Sending a flood of traffic to exhaust a server's network bandwidth, CPU, or memory.
    - **Exploiting Vulnerabilities:** Using a known flaw in an application or OS to cause it to crash or become unresponsive. This is why keeping systems updated with security patches is crucial.
    - **Physical Attacks:** A simple DoS attack can be as low-tech as unplugging the power to a building or a water leak damaging servers in a data center.
    - **Accidental DoS:** A network loop created by misconfigured switches (without Spanning Tree Protocol) can quickly overwhelm the network, causing an accidental DoS. Similarly, a large download on a low-bandwidth connection can deny service to other users.

### [](https://zeroday.forem.com/andrew_despres/comptia-network-n10-009-42-study-guide-common-network-attacks-k1o#distributed-denial-of-service-ddos)**Distributed Denial of Service (DDoS)**

A DDoS attack uses multiple devices, often geographically dispersed, to launch a coordinated DoS attack against a single target. These attacking devices are typically compromised computers that form a **botnet**, controlled by a central command and control server.

- **Asymmetric Threat:** A DDoS attack is an asymmetric threat because an attacker with very few resources can command a botnet of thousands or millions of devices to take down a target with vastly more resources.

### [](https://zeroday.forem.com/andrew_despres/comptia-network-n10-009-42-study-guide-common-network-attacks-k1o#ddos-reflection-and-amplification)**DDoS Reflection and Amplification**

This is an advanced technique that makes DDoS attacks more powerful and harder to trace. The attacker spoofs the victim's IP address and sends small requests to third-party servers that generate large responses.

- **How it Works:**
    1. The attacker's command and control server instructs the botnet to send requests to public, open DNS resolvers.
    2. Crucially, the source IP address of these requests is **spoofed** to be the victim's IP address.
    3. The request is specially crafted to elicit a very large response. For example, a DNS query using the ANY parameter can return large DNS key records, amplifying a small 28-byte query into a 1,300-byte response.
    4. The open resolvers send these massive, amplified responses not to the botnet devices, but to the spoofed source IP—the victim.
    5. The victim's server is inundated with a flood of large, unsolicited response packets from thousands of resolvers, overwhelming its resources and causing a denial of service. Protocols like DNS, NTP, and ICMP are commonly used for amplification.

### [](https://zeroday.forem.com/andrew_despres/comptia-network-n10-009-42-study-guide-common-network-attacks-k1o#3-layer-2-attacks-manipulating-the-data-link-layer)**3. Layer 2 Attacks: Manipulating the Data Link Layer**

[![Visualization of Mac Flooding](https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fldbtyukndxdj4yflwrzr.png)](https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fldbtyukndxdj4yflwrzr.png)_Screenshot from [https://medium.com/@gowthamraj.pgr/mac-flooding-attack-433e12e3cd1c](https://medium.com/@gowthamraj.pgr/mac-flooding-attack-433e12e3cd1c)_

These attacks target the fundamental operations of network switches.

### [](https://zeroday.forem.com/andrew_despres/comptia-network-n10-009-42-study-guide-common-network-attacks-k1o#mac-address-fundamentals)**MAC Address Fundamentals**

A Media Access Control (MAC) address is a unique 48-bit hardware address burned into a network interface card (NIC). It consists of two parts:

- **Organizationally Unique Identifier (OUI):** The first 3 bytes, identifying the manufacturer.
- **Network Interface Controller-specific:** The last 3 bytes, acting as a unique serial number from that manufacturer.

Switches learn the MAC addresses of connected devices and map them to specific switch ports in a MAC address table. This allows the switch to forward frames directly to the intended recipient instead of broadcasting them to everyone.

### [](https://zeroday.forem.com/andrew_despres/comptia-network-n10-009-42-study-guide-common-network-attacks-k1o#mac-flooding)**MAC Flooding**

A MAC flooding attack exploits the limited memory of a switch's MAC address table.

- **How it Works:**
    1. An attacker connects to a switch and sends a massive number of Ethernet frames, each with a different, random source MAC address.
    2. The switch attempts to learn each new MAC address, quickly filling its MAC address table to capacity.
    3. Once the table is full, the switch can no longer store new addresses or look up existing ones efficiently.
    4. In this "fail-open" state, the switch reverts to behaving like a hub. It broadcasts all incoming frames to every port on the switch.
- **The Result:** The attacker, running a packet sniffer, can now capture all traffic passing through the switch, including data not originally destined for them.
- **Mitigation:** Modern switches offer **port security**, which can limit the number of MAC addresses that can be learned on a single port, effectively preventing a flood from one interface.

### [](https://zeroday.forem.com/andrew_despres/comptia-network-n10-009-42-study-guide-common-network-attacks-k1o#vlan-hopping)**VLAN Hopping**

Virtual Local Area Networks (VLANs) are used to segment a network, separating traffic for security and performance. For example, the Accounting department's devices can be on one VLAN, and Marketing on another, preventing them from communicating directly. VLAN hopping is an attack that allows an attacker on one VLAN to send traffic to another VLAN.

- **Switch Spoofing:**
    - **Vulnerability:** Some switches have ports that can auto-negotiate whether they are a standard access port (for a PC) or a trunk port (for connecting to another switch). A trunk port is configured to carry traffic for multiple VLANs.
    - **Attack:** An attacker configures their machine to pretend to be a switch. When they connect to the auto-negotiating port, the switch establishes a trunk link with the attacker's machine.
    - **Result:** The attacker now has access to all VLANs allowed on that trunk and can send traffic to any of them.
    - **Mitigation:** Disable automatic trunk negotiation and manually configure ports as either access or trunk ports. Explicitly define which VLANs are allowed on trunk links.
- **Double Tagging:**
    - **Vulnerability:** This attack exploits the way switches process frames on their "native VLAN." A native VLAN is a special VLAN on a trunk where traffic is sent untagged.
    - **Attack:** An attacker on one VLAN (e.g., VLAN 10, which is also the native VLAN) crafts a special frame with two VLAN tags. The outer tag is for their own VLAN (VLAN 10), and the inner tag is for the victim's VLAN (e.g., VLAN 20).
    - **Process:**
        1. The attacker sends the double-tagged frame. The first switch sees the outer tag (VLAN 10). Because this is the native VLAN, it strips this tag off and forwards the frame across the trunk link.
        2. The frame, now with only the inner tag (VLAN 20), arrives at the second switch. The second switch sees the VLAN 20 tag and forwards the frame to the victim on VLAN 20.
    - **Result:** The attacker successfully sends a packet from VLAN 10 to a device on VLAN 20. This is a one-way attack, often used for DoS.
    - **Mitigation:** Change the native VLAN ID from the default (usually 1) to an unused VLAN ID. Ensure that no user devices are placed on the native VLAN.

### [](https://zeroday.forem.com/andrew_despres/comptia-network-n10-009-42-study-guide-common-network-attacks-k1o#4-malware-malicious-software)**4. Malware: Malicious Software**

[![Visualization of Different Malware Types](https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fv8kjn4hmslbaw5scshcm.png)](https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fv8kjn4hmslbaw5scshcm.png)_Screenshot from [https://www.geeksforgeeks.org/ethical-hacking/malware-and-its-types/](https://www.geeksforgeeks.org/ethical-hacking/malware-and-its-types/)_

Malware is a broad term for any software designed to disrupt computer operations, gather sensitive information, or gain unauthorized access to systems.

- Virus: A generic term for malware that attaches to an application and spreads to other computers. Requires human intervention to spread, such as a user running an infected program or opening an infected email attachment.
- Worm: Self-replicating malware that spreads across networks without any human assistance. Exploits vulnerabilities in operating systems or applications to move from system to system. Considered more dangerous than viruses due to their ability to spread automatically.
- Randsomeware: Malware that encrypts a victim's personal or organizational data and demands a payment (ransom), often in cryptocurrency, for the decryption key. A user downloads and runs what they believe is a game or utility. While the legitimate program may run, the malware is installed in the background.
- Rootkit: Malware that embeds itself deep within an operating system, making it extremely difficult to detect and remove. Modifies core system files and can hide its presence from standard anti-malware tools.
- Keylogger: Software that records every keystroke a user makes. Can be installed by other malware. Its primary goal is to capture credentials like usernames and passwords during login.
- Adware: Malware designed to display unwanted advertisements to the user. Its purpose is to generate revenue for the malware author through ad clicks or views.
- Spyware: Malware that secretly monitors a user's activity, such as browsing history or webcam feeds, and reports it back to an attacker. Often works in conjunction with adware. Logic Bomb: Malware that lies dormant until a specific condition is met, such as a particular date or time, or the occurrence of a specific event. Once triggered, it executes its malicious payload.
- Botnet: A network of compromised computers (bots or zombies) controlled by a single attacker (the "bot herder"). Worms and Trojans are often used to infect systems and enlist them into a botnet, which can then be used for DDoS attacks, spam, etc.
- Bloatware: Unwanted, pre-installed software on a new computer that consumes storage space and system memory. While not directly malicious, it can degrade system performance and create potential security vulnerabilities.

**Malware Prevention:** The most effective defense is proactive.

- **Keep Systems Patched:** Regularly update your operating system and all applications to close vulnerabilities that worms and other malware exploit.
- **User Vigilance:** Do not click links in unsolicited emails or pop-ups. Be wary of drive-by downloads, where malware is downloaded simply by visiting a compromised website.

### [](https://zeroday.forem.com/andrew_despres/comptia-network-n10-009-42-study-guide-common-network-attacks-k1o#5-rogue-services-and-onpath-attacks)**5. Rogue Services and On-Path Attacks**

[![Visualization of On-Path or Man-in-the-Middle attacks](https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F7ep2mdpmdqd635vt0ds9.png)](https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F7ep2mdpmdqd635vt0ds9.png)_Screenshot from [https://www.wallarm.com/what/what-is-an-on-path-attacker](https://www.wallarm.com/what/what-is-an-on-path-attacker)_

Rogue services are unauthorized devices or servers set up on a network to intercept traffic or disrupt services.

### [](https://zeroday.forem.com/andrew_despres/comptia-network-n10-009-42-study-guide-common-network-attacks-k1o#rogue-dhcp-server)**Rogue DHCP Server**

The Dynamic Host Configuration Protocol (DHCP) automatically assigns IP address configurations to clients. Since the protocol has no built-in security, any device can respond to a DHCP request.

- **Threat:** An attacker can set up a rogue DHCP server to hand out invalid IP addresses, causing connectivity issues, or to assign a malicious default gateway and DNS server, forcing all client traffic through the attacker's machine.
- **Mitigation:**
    - **DHCP Snooping:** An enterprise switch feature that allows administrators to designate trusted ports where legitimate DHCP servers are connected. The switch will block DHCP responses from any other "untrusted" port.
    - **Active Directory Authorization:** In Microsoft environments, DHCP servers can be configured to only operate if they are authorized within Active Directory.

### [](https://zeroday.forem.com/andrew_despres/comptia-network-n10-009-42-study-guide-common-network-attacks-k1o#rogue-access-point-ap-amp-wireless-evil-twin)**Rogue Access Point (AP) & Wireless Evil Twin**

- **Rogue AP:** An unauthorized wireless access point plugged into the wired network. It could be installed by a well-meaning employee trying to improve Wi-Fi coverage or by an attacker. It creates a significant security hole, bypassing network security controls.
- **Wireless Evil Twin:** A malicious rogue AP configured to look exactly like a legitimate one. It mimics the SSID (network name), security settings, and even the captive portal of a trusted network (e.g., "Airport_Free_WiFi"). These APs often boost their signal strength to overpower legitimate APs, tricking users into connecting.
- **Mitigation:**
    - **Periodic Scans:** Regularly scan the wireless spectrum and walk the facility with a wireless analyzer to detect unauthorized APs.
    - **Network Access Control (802.1X):** Require users and devices to authenticate before gaining access to the network. This prevents an unauthorized AP from being able to pass traffic onto the wired network, even if it's plugged in.

### [](https://zeroday.forem.com/andrew_despres/comptia-network-n10-009-42-study-guide-common-network-attacks-k1o#onpath-attack-maninthemiddle)**On-Path Attack (Man-in-the-Middle)**

This is a general term for an attack where the perpetrator secretly positions themself between two communicating parties. They can then intercept, read, and even modify the data being exchanged without either party's knowledge.

- **Examples:** ARP poisoning and wireless evil twins are common methods for establishing an on-path position.
- **Mitigation:** The single most effective defense against on-path attacks is **encryption**. If data is encrypted end-to-end (e.g., using HTTPS or a VPN), an attacker in the middle can intercept the traffic, but they cannot read or make sense of it.

### [](https://zeroday.forem.com/andrew_despres/comptia-network-n10-009-42-study-guide-common-network-attacks-k1o#6-social-engineering-the-human-element)**6. Social Engineering: The Human Element**

[![Diagram of the aspects of Social Engineering](https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fjcsnnv17jzpht59p8kur.png)](https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fjcsnnv17jzpht59p8kur.png)_Screenshot from [https://www.geeksforgeeks.org/computer-networks/types-of-cyber-attacks/](https://www.geeksforgeeks.org/computer-networks/types-of-cyber-attacks/)_

Social engineering is the art of manipulating people into divulging confidential information or performing actions they shouldn't.

- Phishing: Sending deceptive emails (with spoofed sender addresses) that appear to be from a legitimate source. The goal is to trick the user into clicking a malicious link and entering credentials on a fake website. Scrutinize sender email addresses, check for spelling/grammar errors, and never click links in unexpected emails. Always navigate to sensitive websites by typing the URL directly.
- Shoulder Surfing: Directly observing someone's screen or keyboard to obtain sensitive information like passwords or financial data. This can be done in person or from a distance using binoculars or telescopes. Be aware of your surroundings. Use privacy filters on screens, which black out the display for anyone not viewing it directly from the front. Position monitors away from windows.
- Tailgating: An unauthorized person following an authorized person into a secure area without their knowledge. E.g., slipping through a door before it closes. All employees should be trained not to hold the door for strangers. Enforce a "one scan, one entry" policy.
- Piggybacking: An unauthorized person tricking an authorized person into holding the door for them. E.g., carrying a large box of donuts and asking for help with the door. Same as tailgating. Train employees to challenge anyone without a visible badge. Issue visitor badges and require escorts.
- Dumpster Diving: Sifting through an organization's trash to find sensitive information like contact lists, internal memos, or financial documents that have been improperly discarded. Implement a shredding policy for all sensitive documents. Use locked dumpsters and secure trash disposal areas with fences and cameras.

---

The attacks detailed here represent a fraction of the threats that network professionals must understand and defend against. From the subtle manipulation of ARP tables to the brute force of a DDoS attack, the common theme is the exploitation of protocols, hardware, and the human element. The defensive measures, from port security and DHCP snooping to encryption and user training, highlight that robust security is a layered, multi-faceted strategy.

The landscape of cybersecurity is constantly evolving. Attackers develop new techniques, and defenders create new safeguards. Your journey doesn't end with certification; it begins there. Commit to a career of continuous learning, stay vigilant, and become the proactive defender that every network needs.