In this lab, you will gain hands-on experience with the foundational principles of information security, known as the CIA Triad: Confidentiality, Integrity, and Availability. Working within a Linux environment, you will learn practical skills to protect data and ensure system reliability. The lab guides you through configuring file permissions, demonstrating file integrity with cryptographic hashes, and simulating basic service monitoring to understand how these core concepts are applied in real-world scenarios.

You will begin by manipulating Linux file permissions using the `chmod` command to enforce confidentiality and integrity on a sensitive file, ensuring it is only accessible and modifiable by authorized users. Next, you will explore file integrity by generating MD5 and SHA256 hashes to create a baseline and then verify that the file remains unaltered. Finally, you will simulate a simple service and use basic monitoring commands to understand the concept of availability and how to detect service disruptions.

This is a Guided Lab, which provides step-by-step instructions to help you learn and practice. Follow the instructions carefully to complete each step and gain hands-on experience. Historical data shows that this is a intermediate level lab with a 75% completion rate. It has received a 98% positive review rate from learners.

![labby](https://labex.io/images/labby/avatar-new.png)Labby

Great! Let me prepare the lab VM for you. While it's being set up, you can start reviewing the first step of this lab.

![labby](https://labex.io/images/labby/avatar-new.png)Labby

# Configure Linux File Permissions for Confidentiality and Integrity

In this step, you will learn how to use fundamental Linux file permissions to enforce two key principles of information security: **Confidentiality** and **Integrity**. Confidentiality ensures that data is accessible only to authorized users, while Integrity ensures that data is not altered in an unauthorized manner. You will create a file, examine its default permissions, and then use the `chmod` command to restrict access and prevent modifications. All operations will be performed in your home project directory, `~/project`.

Now, inside the `~/project` directory, create a sample file that represents sensitive data. We'll use the `echo` command to place some text inside it.

```bash
echo "Top Secret Details" > confidential_data.txt
```

Let's verify that the file was created and view its default permissions using the `ls -l` command, which provides a long listing format.

```bash
ls -l confidential_data.txt
```

You will see output similar to this. The exact time and size may vary.

```plaintext
-rw-rw-r-- 1 labex labex 19 Jul 21 15:09 confidential_data.txt
```

**Let's break down the permission string `-rw-rw-r--`:

- The first character `-` indicates it's a regular file.
- The next three characters `rw-` are the permissions for the **owner** (user `labex`). `r` means read, `w` means write.
- The next three `rw-` are for the **group** (group `labex`). The group can read and write the file.
- The final three `r--` are for **others** (all other users on the system). They can also only read the file.

This default setting violates confidentiality because any user on the system can read our "Top Secret" file. Let's fix this by changing the permissions so that only the owner can read and write. We will use the `chmod` command with numeric (octal) codes: `4` for read, `2` for write, and `1` for execute. To give the owner read/write access (`4+2=6`) and remove all access for the group (`0`) and others (`0`), we use the code `600`.

```bash
chmod 600 confidential_data.txt
```

Check the permissions again:

```bash
ls -l confidential_data.txt
```

The output now reflects the new, more secure permissions, enforcing confidentiality.

```plaintext
-rw------- 1 labex labex 19 Jul 21 15:09 confidential_data.txt
```

Next, let's protect the file's **integrity** by making it read-only, even for the owner. This prevents accidental modification. We'll set the permissions to `400`, which means read-only for the owner only, maintaining both confidentiality and integrity.

```bash
chmod 400 confidential_data.txt
```

Now, let's try to modify the file by appending new text to it.

```bash
echo "This should not be added." >> confidential_data.txt
```

The operation will fail, and you will see a "Permission denied" error. This demonstrates that the file's integrity is protected from changes.

```plaintext
zsh: permission denied: confidential_data.txt
```

Finally, check the permissions one last time to see the read-only state.

```bash
ls -l confidential_data.txt
```

```plaintext
-r-------- 1 labex labex 19 Jul 21 15:09 confidential_data.txt
```

You have successfully configured file permissions to protect both its confidentiality and integrity.


---
# Demonstrate File Integrity with Hashing (MD5/SHA256)

In this step, you will learn how to use cryptographic hashing to verify file integrity. While file permissions can prevent unauthorized users from modifying a file, they don't protect against accidental changes by an authorized user or more sophisticated attacks. A **hash function** creates a unique digital fingerprint (a fixed-length string of characters) for a file. Even a one-bit change in the file will produce a completely different hash, making it an excellent tool for detecting tampering. We will use the `sha256sum` command, a standard and secure hashing tool in Linux.

First, ensure you are in the correct directory. If you are continuing from the previous step, you should already be in `~/project`.

In the last step, we made `confidential_data.txt` read-only to protect its integrity. To demonstrate how hashing works, we first need to make the file writable again for the owner. We'll use `chmod` to grant write permissions back to the user (`u+w`).

```bash
chmod u+w confidential_data.txt
ls -l confidential_data.txt
```

The permissions should now be `-rw-------`, indicating the owner can write to the file again while maintaining confidentiality.

```plaintext
-rw------- 1 labex labex 19 Jul 21 15:09 confidential_data.txt
```

Now, let's generate the SHA256 hash for our original, trusted file. This hash will serve as our baseline for integrity checks.

```bash
sha256sum confidential_data.txt
```

The command will output a long string of characters, which is the hash, followed by the filename.

```plaintext
106b4d0169831f239fec7a53b072640a2c0b815e00b088bbe44e6689e92cb96d  confidential_data.txt
```

To make verification easier later, it's common practice to save this trusted hash value to a separate file. Let's redirect the output of the command into a new file named `hashes.sha256`.

```bash
sha256sum confidential_data.txt > hashes.sha256
```

You can view the contents of this new file to confirm the hash was saved.

```bash
cat hashes.sha256
```

```plaintext
106b4d0169831f239fec7a53b072640a2c0b815e00b088bbe44e6689e92cb96d  confidential_data.txt
```

Now, let's simulate an unauthorized modification. We will append a new line of text to our sensitive file.

```bash
echo "This is an unauthorized modification." >> confidential_data.txt
```

The file `confidential_data.txt` has now been altered. To verify its integrity, we can use the `-c` (or `--check`) flag with `sha256sum`, telling it to compare the file's current hash against the trusted value stored in `hashes.sha256`.

```bash
sha256sum -c hashes.sha256
```

Because the file content has changed, the new hash will not match the stored hash. The command will report a failure, proving that the file's integrity has been compromised.

```plaintext
confidential_data.txt: FAILED
sha256sum: WARNING: 1 computed checksum did NOT match
```

This powerful technique allows you to quickly and reliably check if any files have been changed from their original state.

---

# Simulate Service Availability and Basic Monitoring

In this step, you will explore the principle of **Availability**, the third component of the CIA security triad. Availability ensures that systems and services are operational and accessible when needed. You will simulate a simple web service, learn how to check if it's running, simulate a service failure, and then write a basic monitoring script to automate the availability check.

First, ensure you are in the `~/project` directory.

We will use Python's built-in web server to simulate a running service. This command will start a web server on port `8000` and serve the files in the current directory. ==The `&` at the end runs the process in the **background**, so you can continue using your terminal.==

```bash
python3 -m http.server 8000 &
```

You will see a message indicating the process has started, along with its Process ID (PID).

```plaintext
[1] 12345
```

Press `Enter` to continue.

Your service is now running. A basic way to check if a service is running is to see if its process exists. We can use the `pgrep` command to find the PID of a process by name.

```bash
pgrep -f http.server
```

This should return the PID of the server process, confirming it's running in memory.

```plaintext
12345
```

However, a running process doesn't guarantee the service is working correctly. A better check is to try and connect to it, just like a user would. We'll use the `curl` command to send a request to our local server.

```bash
curl http://localhost:8000/confidential_data.txt
```

Since the service is available, it will respond with the contents of the file you created in the previous steps.

```plaintext
Top Secret Details
This is an unauthorized modification.
```

Now, let's simulate a service failure. We will use the `kill` command to terminate the web server process. You'll need the PID you found earlier.

```bash
# Replace 12345 with the actual PID from the pgrep command
kill 12345
```

After running `kill`, you might see a "Terminated" message for the background job. Now, let's try to access the service again with `curl`.

```bash
curl http://localhost:8000/confidential_data.txt
```

This time, the command will fail because the service is no longer running and cannot accept the connection. This demonstrates a lack of availability.

```plaintext
curl: (7) Failed to connect to localhost port 8000 after 0 ms: Connection refused
```

Finally, let's create a simple monitoring script to automate this check. Create a new file named `monitor.sh` using `nano`.

```bash
nano monitor.sh
```

Enter the following script into the editor. This script uses `curl` to check the service. If it gets a successful response (`200 OK`), it reports the service is "UP"; otherwise, it reports "DOWN".

```bash
#!/bin/bash

# Check if the service at localhost:8000 is responding
if curl -s --head http://localhost:8000 | grep "200 OK" > /dev/null; then
  echo "Service Status: UP"
else
  echo "Service Status: DOWN"
fi
```

Save the file and exit `nano` by pressing `Ctrl+X`, then `Y`, and `Enter`.

Make the script executable using `chmod`:

```bash
chmod +x monitor.sh
```

Now, run your monitoring script. Since the service is stopped, it should report "DOWN".

```bash
./monitor.sh
```

```plaintext
Service Status: DOWN
```

You have now learned how to start a service, check its availability, simulate a failure, and create a basic script to monitor it.