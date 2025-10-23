# Understanding Ports and Creating a Listener

Now that we have our communication tool ready, let's learn about ports and set up a listener. In networking, ports are like different channels on a radio - they allow multiple communications to happen simultaneously on the same device. Think of them as numbered doors where each door leads to a different service or application on your computer.

1. Let's create a listener. We'll use port 12345 for our communication. Port numbers range from 0 to 65535, and we're choosing 12345 because it's easy to remember and typically not used by system services. In the terminal, type:
    
    ```bash
    nc -l 12345
    ```
    
    This command tells Netcat to listen (`-l` flag) on port 12345. The `-l` stands for "listen" mode, which means your computer will wait for incoming connections on this port.
    
2. The terminal will seem to hang. Don't worry! This is expected behavior because Netcat is now actively listening and waiting for someone to connect to port 12345. The cursor blinking means the program is running properly in listening mode.
    
3. To test our listener, we need to open another terminal window. You can do this by right-clicking on the terminal icon and selecting "New Terminal" or using the shortcut Ctrl+Shift+N. This second terminal will act as our "client" that connects to the listener.
    
4. In this new terminal, we'll connect to our listener. Type:
    
    ```bash
    nc localhost 12345
    ```
    
    This command tells Netcat to connect to `localhost` (which means your own computer) on port 12345. The `localhost` is a special hostname that always refers to your current machine.
    
5. Now, type a simple message like "Hello" in this second terminal and press Enter. You should see the message appear in the first terminal where the listener is running. This demonstrates how data travels from the connecting terminal to the listening terminal through the network port.
    

![Netcat listener and connection](https://file.labex.io/namespace/df87b950-1f37-4316-bc07-6537a1f2c481/cybersecurity/lab-using-netcat-for-simple-network-communication/assets/20240927-16-02-03-HFtwoalz.png)

6. You can continue typing messages in either terminal, and they will appear in the other. This shows Netcat's full-duplex capability - meaning communication can flow both ways simultaneously. Try sending a few messages back and forth to see how it works.
    
7. To end the connection, press Ctrl+C in both terminals. This keyboard shortcut sends an interrupt signal that closes the Netcat sessions cleanly. It's important to close connections properly when you're done testing.

# File Transfer with Netcat

Now that we've learned how to establish basic network communication with Netcat, let's explore one of its most practical applications: transferring files between systems. This simple yet powerful capability makes Netcat a valuable tool for system administrators and network engineers.

1. First, we'll create a sample file to transfer. In your terminal, type:
    
    ```bash
    cd ~/project
    echo "Top Secret: The cake recipe is actually a lie" > secret.txt
    ```
    
    This command does two things: changes to your project directory and creates a new file called `secret.txt` containing our example message. The `>` symbol redirects the output of the `echo` command into a file.
    
2. Now we'll set up the receiving end of our file transfer. Open a terminal window and run:
    
    ```bash
    cd ~/project
    nc -l 12345 > received_secret.txt
    ```
    
    Here, `nc -l 12345` starts Netcat in listen mode on port 12345. The `>` redirects any incoming data to a new file called `received_secret.txt`. This terminal will now wait silently for incoming connections.
    
3. In a separate terminal window (while keeping the first one running), we'll send our file:
    
    ```bash
    cd ~/project
    nc localhost 12345 < secret.txt
    ```
    
    This command uses Netcat to connect to our listener (`localhost` means "this same computer") on port 12345. The `<` symbol tells Netcat to use `secret.txt` as its input, effectively sending the file's contents through the network connection.
    
4. The transfer happens instantly because both ends are on the same machine. To verify the successful transfer, check the received file's contents:
    
    ```bash
    cat received_secret.txt
    ```
    
    The `cat` command displays the file's contents, which should match our original secret message exactly.
    

![Netcat file transfer screenshot](https://file.labex.io/namespace/df87b950-1f37-4316-bc07-6537a1f2c481/cybersecurity/lab-using-netcat-for-simple-network-communication/assets/screenshot-20240924-VasTvjVD@2x.png)

5. Congratulations! You've successfully transferred a file using Netcat. While we used `localhost` for this demonstration, the same process works between different computers on a network - just replace `localhost` with the receiving computer's IP address. This simple method can be incredibly useful for quick file transfers when more complex protocols aren't available.
# Implementing Encrypted Communication

In our final step, let's add a layer of security to our communication by implementing encryption using OpenSSL and integrating it with Netcat. Encryption is essential for protecting sensitive information from being intercepted and read by unauthorized parties during network transmission.

We have already learned how to use OpenSSL in the previous labs. OpenSSL provides robust cryptographic functions that we'll leverage to secure our messages. Specifically, we'll use the `openssl enc` command to encrypt and decrypt messages using the AES-256-CBC cipher, which is a strong symmetric encryption algorithm. We'll also use a passphrase to derive the encryption key - think of this as a shared secret between the sender and receiver.

1. First, let's create the sender script. This script will handle encrypting and sending messages. Open a new file named `secure_sender.sh`:
    
    ```bash
    nano secure_sender.sh
    ```
    
2. Add the following content to the file (You can copy and paste the content):
    
    ```bash
    #!/bin/bash
    
    echo "Secure Sender - Enter messages to send. Press Ctrl+C to exit."
    
    while true; do
      echo "Enter message:"
      read message
      encrypted=$(echo "$message" | openssl enc -aes-256-cbc -salt -base64 -pbkdf2 -iter 10000 -pass pass:secretpassword 2> /dev/null)
      echo "$encrypted" | nc -N localhost 12345
    done
    ```
    
    This script works by:
    
    - Continuously prompting you for messages in a loop
    - Taking your input and piping it through OpenSSL for encryption
    - Using Netcat (`nc`) to send the encrypted data to port 12345 on localhost
    - The encryption parameters include salt for added security, base64 encoding for safe transmission, and PBKDF2 key derivation with 10,000 iterations
    
    ![Secure sender script example](https://file.labex.io/namespace/df87b950-1f37-4316-bc07-6537a1f2c481/cybersecurity/lab-using-netcat-for-simple-network-communication/assets/20240927-16-21-45-pARhMhDM.png)
    
3. Press `Ctrl+X`, then `Y`, and `Enter` to save and exit the file. Make the script executable:
    
    ```bash
    chmod +x secure_sender.sh
    ```
    
4. Now, let's create the receiver script. This script will listen for and decrypt incoming messages. Open a new file named `secure_receiver.sh`:
    
    ```bash
    nano secure_receiver.sh
    ```
    
5. Add the following content:
    
    ```bash
    #!/bin/bash
    
    echo "Secure Receiver - Waiting for messages. Press Ctrl+C to exit."
    
    while true; do
      encrypted=$(nc -l -p 12345)
      if [ ! -z "$encrypted" ]; then
        decrypted=$(echo "$encrypted" | openssl enc -aes-256-cbc -d -salt -base64 -pbkdf2 -iter 10000 -pass pass:secretpassword 2> /dev/null)
        echo "Received message: $decrypted"
      fi
    done
    ```
    
    This script:
    
    - Continuously listens on port 12345 for incoming data
    - When data is received, pipes it through OpenSSL for decryption
    - Uses the same encryption parameters as the sender to properly decrypt the message
    - Displays the original plaintext message
6. Press `Ctrl+X`, then `Y`, and `Enter` to save and exit the file. Make the script executable:
    
    ```bash
    chmod +x secure_receiver.sh
    ```
    
7. Now, let's test our encrypted communication system. You'll need two terminal windows open simultaneously - one for the receiver and one for the sender.
    
8. In the first terminal, start the receiver:
    
    ```bash
    ./secure_receiver.sh
    ```
    
    You'll see a message indicating that the receiver is waiting for messages. The receiver is now actively listening on port 12345.
    
9. In the second terminal, start the sender:
    
    ```bash
    ./secure_sender.sh
    ```
    
    You'll be prompted to enter messages. Anything you type here will be encrypted before being sent over the network.
    
10. In the sender terminal, type a message and press Enter. You should see the encrypted message being sent (though you'll only see the encrypted version briefly as it's transmitted).
    

![Encrypted Message](https://file.labex.io/namespace/df87b950-1f37-4316-bc07-6537a1f2c481/cybersecurity/lab-using-netcat-for-simple-network-communication/assets/screenshot-20240924-U4V5uuEr@2x.png)

11. In the receiver terminal, you should see the decrypted message appear exactly as you typed it in the sender terminal.
    
12. You can continue sending messages from the sender terminal, and they will be automatically encrypted, sent, received, and decrypted in the receiver terminal. This demonstrates a complete secure communication loop.
    
13. To end the communication, press Ctrl+C in both terminals. This safely terminates both scripts.
    

This setup demonstrates a simple yet effective encrypted communication system. While we're using localhost for testing, this same approach works across networks. The messages are automatically encrypted before transmission and decrypted upon receipt, providing a secure channel for communication. Remember that in real-world applications, you would want to use more secure methods for key exchange than a hardcoded password.