# Installing Wireshark

In this step, we'll install Wireshark on our Ubuntu system. Wireshark is a network protocol analyzer that lets you capture and interactively browse network traffic. It's available in the default Ubuntu repositories, making the installation process straightforward for beginners.

1. First, let's open the terminal. On your desktop, locate and open the Xfce Terminal. The terminal is where we'll execute all our commands for installing and configuring Wireshark.

![Opening Xfce Terminal window](https://file.labex.io/namespace/df87b950-1f37-4316-bc07-6537a1f2c481/cybersecurity/lab-network-analysis-with-wireshark/assets/screenshot-20240924-AphfKRdJ@2x.png)

> Note: Free users can't connect to the internet, so Wireshark is already pre-installed in the lab environment, You can jump to item 5 in the list (check if Wireshark is installed). [Upgrade to a pro](https://labex.io/pricing?utm_source=labby) user to practice installing Wireshark by yourself.

Pro Users Only

2. Before installing any software, it's good practice to update the package lists. This ensures we're installing the latest available version of Wireshark. Run the following command:

```bash
sudo apt update
```

3. Now we'll install Wireshark itself. The `-y` flag automatically answers "yes" to any prompts during installation, making the process smoother:

```bash
sudo apt install wireshark -y
```

4. During installation, you'll encounter an important security prompt about allowing non-superusers to capture packets. This is a key configuration choice - selecting "Yes" means you won't need root privileges to run Wireshark, which is safer for everyday use. Use the arrow keys to select "Yes" and press Enter.

![Wireshark installation prompt](https://file.labex.io/namespace/df87b950-1f37-4316-bc07-6537a1f2c481/cybersecurity/lab-network-analysis-with-wireshark/assets/20240925-09-07-31-8vyHUZiD.png)

5. After installation completes, let's verify Wireshark was installed correctly. The version check confirms the installation was successful and shows which version you're running:

```bash
wireshark --version
```

6. To capture network traffic, we need to configure proper permissions. This involves adding your user to the `wireshark` group and setting up Dumpcap (Wireshark's packet capture tool) with the right permissions:

First, check if the `wireshark` group exists (it should have been created during installation):

```bash
getent group wireshark
```

If for some reason the group doesn't exist, create it with:

```bash
sudo groupadd wireshark
```

These commands configure Dumpcap to run with elevated privileges while keeping Wireshark itself running as a normal user - a security best practice:

```bash
sudo chgrp wireshark /usr/bin/dumpcap
sudo chmod 4755 /usr/bin/dumpcap
sudo gpasswd -a $USER wireshark
```

After running these commands, you'll need to log out and back in for the group changes to take effect. You've now successfully installed and configured Wireshark, ready to start analyzing network traffic!

The commands are used to configure permissions for the `dumpcap` utility, which is part of the Wireshark network protocol analyzer. Here's a brief explanation of each command:

1. **Change Group Ownership**:
    
    ```bash
    sudo chgrp wireshark /usr/bin/dumpcap
    ```
    
    This command changes the group ownership of the `dumpcap` file to the `wireshark` group. This allows users in the `wireshark` group to access the file with the appropriate permissions.
    
2. **Set Setuid Permission**:
    
    ```bash
    sudo chmod 4755 /usr/bin/dumpcap
    ```
    
    This command sets the permissions of `dumpcap` to `4755`. The `4` at the beginning sets the Setuid bit, which allows users to run the program with the permissions of the file owner (usually root). The `755` means the owner can read, write, and execute, while the group and others can read and execute.
    
3. **Add User to Group**:
    
    ```bash
    sudo gpasswd -a $USER wireshark
    ```
    
    This command adds the current user (represented by `$USER`) to the `wireshark` group. This is necessary for the user to have the permissions granted by the previous commands.
    

### Summary

These commands collectively allow a user to run `dumpcap` with elevated privileges, which is often required for capturing network packets.

# Capturing Network Traffic

Now that we have Wireshark installed, let's capture some network traffic. Think of this like setting up a microphone to record all conversations happening on your network. Every time your computer sends or receives data online, it's broken down into small pieces called packets. Wireshark lets us see and analyze these packets.

1. Open Wireshark by typing `wireshark` in the terminal and pressing Enter. This launches the graphical interface where we'll do our packet capturing.
    
2. When Wireshark opens, you'll see a list of network interfaces (the connections your computer uses to talk to networks). Look for an interface named "eth0" or "eth1" - these are typically your wired Ethernet connections. If you're using WiFi, you might see "wlan0" instead.
    

![Wireshark interface selection](https://file.labex.io/namespace/df87b950-1f37-4316-bc07-6537a1f2c481/cybersecurity/lab-network-analysis-with-wireshark/assets/20240925-09-09-24-QXcGTn64.png)

3. Double-click on the active interface (the one showing packet counts increasing) to start capturing. The main window will now display a live stream of packets - each line represents a different network conversation happening right now.

![Wireshark packet capture window](https://file.labex.io/namespace/df87b950-1f37-4316-bc07-6537a1f2c481/cybersecurity/lab-network-analysis-with-wireshark/assets/20240925-09-22-28-JnYLbYJp.png)

4. Let's create some simple network activity to capture. Open a new terminal window and enter:
    
    ```bash
    curl http://example.com
    ```
    
    This command fetches the homepage of [example.com](http://example.com/), generating HTTP packets (web traffic) that Wireshark can capture. You'll see new packets appear in Wireshark immediately after running this.
    
5. After seeing several packets (about 5-10 seconds is enough), click the red "Stop" button at the top. This freezes the capture so we can examine the packets without new ones constantly arriving.
    
6. Congratulations! You've captured your first network traffic. Each line represents a packet with details like source/destination addresses, protocol type, and timing. The three main panels show: the packet list (summary), packet details (technical breakdown), and raw packet data (hexadecimal view).
    
7. To save your work, go to File > Save and name the file `myfirstcapture.pcapng` in `/home/labex`. The .pcapng format preserves all the packet details for future analysis.
    

![Saving network capture file](https://file.labex.io/namespace/df87b950-1f37-4316-bc07-6537a1f2c481/cybersecurity/lab-network-analysis-with-wireshark/assets/20240925-09-28-46-8Vlt1NZ6.png)

This basic capture process is the foundation of network analysis. In upcoming steps, we'll learn how to filter and interpret these packets to understand network behavior and troubleshoot issues.

# Analyzing Packet Data

Now that we've captured some network traffic, let's examine it step by step. Think of this like reading a digital conversation - we'll learn how to interpret the "language" computers use to communicate.

1. **Opening the Capture File**:  
    Start Wireshark and open your saved capture file (`myfirstcapture.pcapng`) either through File > Open or by double-clicking the file. This file contains all the network traffic we previously recorded, similar to a recording of phone calls.
    
2. **Understanding the Packet List**:  
    The top pane shows all captured packets in chronological order. Each line represents one network message, showing:
    
    - Who sent it (Source IP)
    - Who received it (Destination IP)
    - What type of communication it was (Protocol)
    - A brief summary (Info)
3. **Examining Packet Details**:  
    When you click any packet, the middle pane reveals its internal structure through protocol layers:
    
    - Frame: Physical network details
    - Ethernet: Hardware addressing
    - IP: Internet routing information
    - TCP/UDP: Transport details
    - Application data (like HTTP)
4. **Filtering HTTP Traffic**:  
    In the filter bar (top), type `http` and press Enter. This isolates web traffic, just like searching for only text messages in a phone log. HTTP is the protocol web browsers use to communicate with servers.
    

![Wireshark HTTP packet filter](https://file.labex.io/namespace/df87b950-1f37-4316-bc07-6537a1f2c481/cybersecurity/lab-network-analysis-with-wireshark/assets/20240925-09-31-34-jI4MU27I.png)

5. **Identifying Web Requests**:  
    Look for packets with "GET" in the Info column. These represent your browser asking for web pages. Click one to see:
    - The exact URL requested
    - Which website was contacted
    - Technical details about the request

![HTTP GET packet details](https://file.labex.io/namespace/df87b950-1f37-4316-bc07-6537a1f2c481/cybersecurity/lab-network-analysis-with-wireshark/assets/20240925-09-34-19-IDmZjeuI.png)

6. **Analyzing HTTP Headers**:  
    Expand the "Hypertext Transfer Protocol" section to see:
    
    - Request method (GET/POST)
    - Requested resource (/index.html)
    - Browser information
    - Cookies and other metadata
7. **Finding Server Responses**:  
    Search for "HTTP/1.1 200 OK" packets. These show successful webpage deliveries. Notice:
    
    - Response codes (200=success, 404=not found)
    - Server type (Apache/Nginx)
    - Content type (HTML, images, etc.)

![HTTP server successful response packet](https://file.labex.io/namespace/df87b950-1f37-4316-bc07-6537a1f2c481/cybersecurity/lab-network-analysis-with-wireshark/assets/20240925-09-34-49-KAGGltxM.png)

8. **Viewing Web Content**:  
    In successful responses, check the "Line-based text data" section to see actual webpage content. This shows the HTML code the server sent to your browser.

This analysis reveals the hidden conversations happening across networks. By understanding these patterns, you can troubleshoot issues, identify suspicious activity, and comprehend how applications communicate.

# Using Filters

Wireshark's real power comes from its ability to filter and analyze large amounts of network data quickly. In this step, we'll learn how to use filters to focus on specific types of traffic. Think of filters as search terms that help you find exactly what you're looking for in a sea of network packets.

1. With your capture file open in Wireshark, let's start by using some simple display filters. The filter bar is located just below the main toolbar:
    
    - To show only TCP traffic (the most common transport protocol), type `tcp` in the filter bar and press Enter. This will hide all non-TCP packets from view.
    - To show traffic to or from a specific IP address, type `ip.addr == 93.184.215.14` (this is the IP address of [example.com](http://example.com/), but you can replace it with any IP address you see in your capture). The double equals sign (==) means "exactly matches".
    
    ![Wireshark TCP filter example](https://file.labex.io/namespace/df87b950-1f37-4316-bc07-6537a1f2c481/cybersecurity/lab-network-analysis-with-wireshark/assets/20240925-09-38-40-oJLQdOg3.png)
    
    - To show all HTTP GET requests (the most common type of web request), type `http.request.method == "GET"`. This helps you see what web pages are being requested.
2. Now let's create a more complex filter by combining conditions. We'll look for all HTTP GET requests specifically going to [example.com](http://example.com/):
    
    - In the filter bar, enter: `http.request.method == "GET" && http.host contains "example.com"`
    - The `&&` means "AND", so both conditions must be true
    - The `contains` operator lets us match any domain with "[example.com](http://example.com/)" in it
    
    ![Wireshark HTTP GET filter](https://file.labex.io/namespace/df87b950-1f37-4316-bc07-6537a1f2c481/cybersecurity/lab-network-analysis-with-wireshark/assets/20240925-09-38-04-CqHyK0lt.png)
    
    - This filter shows all GET requests to any domain containing "[example.com](http://example.com/)", which is useful for monitoring traffic to specific websites.
3. Since you'll likely use certain filters repeatedly, Wireshark lets you save them. Let's save our HTTP GET filter:
    
    - Click on the plus sign ("+") next to the filter bar. This opens the filter management window.
    - Name the filter "HTTP GETs" and click Save. Choose a descriptive name you'll remember.
    - You can now quickly apply this filter anytime by selecting it from the Saved Filters list (the bookmark icon next to the filter bar).
4. Finally, let's export some of our findings for documentation or further analysis:
    
    - Go to Statistics > HTTP > Requests to see a summary of all HTTP requests
    - Click on "Save as" to export the list to a file
    - Choose `/home/labex` as the location and save the file as `http_requests.txt`
    
    ![Exporting HTTP requests list](https://file.labex.io/namespace/df87b950-1f37-4316-bc07-6537a1f2c481/cybersecurity/lab-network-analysis-with-wireshark/assets/20240925-09-44-07-FDDRS0Q9.png)
    

Using filters in this way allows you to quickly sift through large amounts of network data and focus on what's important. It's like having a super-powered magnifying glass that can instantly show you specific types of network traffic. Mastering filters will save you hours when troubleshooting network issues or analyzing security events.