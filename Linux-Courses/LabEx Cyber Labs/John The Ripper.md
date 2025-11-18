# Introduction

In this lab, you will learn how to crack a password-protected ZIP file using John the Ripper, a popular and powerful password recovery tool. This exercise will guide you through the fundamental steps of a password cracking process, from setting up the necessary tools to verifying the cracked password. By the end of this lab, you will understand how password hashes are extracted and used in dictionary and brute-force attacks, highlighting the importance of using strong passwords for file protection.

This is a Guided Lab, which provides step-by-step instructions to help you learn and practice. Follow the instructions carefully to complete each step and gain hands-on experience. Historical data shows that this is a beginner level lab with a 89% completion rate. It has received a 100% positive review rate from learners.

Continue

![labby](https://labex.io/images/labby/avatar-new.png)Labby

# Install Required Tools

Before we can begin cracking passwords, we need to install the necessary software. In this step, you will install John the Ripper (JtR), the primary tool for this lab, and the `zip` utility, which we will use to create our target file. We will use the `apt` package manager, which is the standard for Debian-based systems like Ubuntu.

First, install the required dependencies and `zip` utility by running the following commands in your terminal. The `-y` flag automatically confirms the installation.

```bash
sudo apt update
sudo apt install -y git build-essential libssl-dev zlib1g-dev zip
sudo apt install -y yasm pkg-config libgmp-dev libpcap-dev libbz2-dev
```

The standard `john` package from apt repositories doesn't include the `zip2john` utility we need. We'll clone and compile John the Ripper from the official repository to get the complete toolset.

```bash
cd ~/project
git clone --depth 1 https://github.com/openwall/john -b bleeding-jumbo john
cd john/src
./configure && make -s clean && make -sj4
```
Here's a breakdown of the commands you've provided:

1. **Change Directory**:
    
    ```bash
    cd ~/project
    ```
    
    This command navigates to the `project` directory in your home folder.
    
2. **Clone a Git Repository**:
    
    ```bash
    git clone --depth 1 https://github.com/openwall/john -b bleeding-jumbo john
    ```
    
    This command clones the `john` repository from GitHub. The `--depth 1` option creates a shallow clone with only the latest commit, which saves time and space. The `-b bleeding-jumbo` specifies that you want to clone the `bleeding-jumbo` branch, and the last `john` is the name of the directory where the repository will be cloned.
    
3. **Change Directory to Source**:
    
    ```bash
    cd john/src
    ```
    
    This command navigates into the `src` directory of the cloned `john` project.
    
4. **Configure and Build**:
    
    ```bash
    ./configure && make -s clean && make -sj4
    ```
    
    - `./configure`: Prepares the build environment for the project.
    - `make -s clean`: Cleans up any previous builds quietly (without output).
    - `make -sj4`: Compiles the project using 4 parallel jobs, speeding up the build process.

If you're new to Git or building software from source, I recommend checking out some introductory labs on LabEx to strengthen your understanding of these concepts!
> **Warning:** The compilation process may take 3~5 minutes. Please be patient.

After compilation is complete, create aliases for the John the Ripper tools so you can use them from anywhere:

```bash
echo 'alias john="$HOME/project/john/run/john"' >> ~/.zshrc
echo 'alias zip2john="$HOME/project/john/run/zip2john"' >> ~/.zshrc
source ~/.zshrc
```
Here's a breakdown of the commands you've provided:

1. **Create an Alias for `john`**:
    
    ```bash
    echo 'alias john="$HOME/project/john/run/john"' >> ~/.zshrc
    ```
    
    This command adds an alias named `john` to your `~/.zshrc` file, which allows you to run the command `$HOME/project/john/run/john` simply by typing `john` in the terminal.
    
2. **Create an Alias for `zip2john`**:
    
    ```bash
    echo 'alias zip2john="$HOME/project/john/run/zip2john"' >> ~/.zshrc
    ```
    
    Similarly, this command creates an alias for `zip2john`, enabling you to run `$HOME/project/john/run/zip2john` by just typing `zip2john`.
    
3. **Reload the Zsh Configuration**:
    
    ```bash
    source ~/.zshrc
    ```
    
    This command reloads your `~/.zshrc` file, applying the new aliases immediately without needing to restart the terminal.
    
Now verify that both `john` and `zip2john` are working correctly from any directory:

```bash
john
```

You should see output similar to the following, which indicates that John the Ripper is successfully installed:

```plaintext
John the Ripper 1.9.0-jumbo-1+bleeding-7a8c81abd9 2025-08-23 23:43:48 +0200 OMP [linux-gnu 64-bit x86_64 AVX-512 AC]
Copyright (c) 1996-2025 by Solar Designer and others
Homepage: https://www.openwall.com/john/

Usage: john [OPTIONS] [PASSWORD-FILES]

Use --help to list all available options.
```

Also verify that `zip2john` is available:

```bash
zip2john
```

This should show the usage information for the `zip2john` utility.

Now you can navigate back to your project directory for the rest of the lab:

```bash
cd ~/project
```

With the necessary tools installed, you are now ready to create a password-protected file for our cracking exercise.

# Create a Password-Protected ZIP File

To practice password cracking, we first need a target. In this step, you will create a simple text file and then archive it into a password-protected ZIP file. This will simulate a real-world scenario where sensitive files are secured with a password.

First, ensure you are in the project directory. Your terminal should already be in this directory by default.

```bash
cd ~/project
```

Next, create a simple text file named `secret.txt`. This file will be placed inside our protected ZIP archive.

```bash
echo "my secret data" > secret.txt
```

Now, use the `zip` command with the `-e` flag (for encryption) to create a password-protected archive named `secret.zip` containing `secret.txt`.

```bash
zip -e secret.zip secret.txt
```

After running the command, you will be prompted to enter and then verify a password. For this lab, use the weak password `password123`. This will make the cracking process faster for demonstration purposes.

**Note:** When typing the password, you won't see any characters displayed on the screen - this is a normal security feature, not a system freeze. Simply type the password and press Enter.

```plaintext
Enter password:
Verify password:
  adding: secret.txt (stored 0%)
```

Finally, you can list the files in the directory to confirm that `secret.zip` has been created.

```bash
ls -l
```

You should see `secret.txt` and `secret.zip` in the output.

```plaintext
total 8
-rw-rw-r-- 1 labex labex  15 Aug 29 09:57 secret.txt
-rw-rw-r-- 1 labex labex 213 Aug 29 09:57 secret.zip
```

You have now successfully created a password-protected ZIP file, which will serve as our target in the following steps.

# Extract the Hash from the ZIP File

John the Ripper does not work directly on the ZIP file itself. Instead, it works on the file's password hash. A hash is a unique string of characters that represents the password. In this step, you will use a utility included with John the Ripper called `zip2john` to extract this hash.

Run the `zip2john` command on your `secret.zip` file. We will redirect the output using the `>` operator to save the hash into a new file named `zip_hash.txt`:

```bash
zip2john secret.zip > zip_hash.txt
```

When you run this command, you'll see some verbose output on the terminal showing information about the ZIP file structure:

```plaintext
ver 1.0 efh 5455 efh 7875 secret.zip/secret.txt PKZIP Encr: 2b chk, TS_chk, cmplen=27, decmplen=15, crc=AF1563E8 ts=4F26 cs=4f26 type=0
```

This information shows details about the encrypted file, including compression and encryption parameters.

To see what the extracted hash looks like, use the `cat` command to display the contents of `zip_hash.txt`:

```bash
cat zip_hash.txt
```

The output will be a single line of text that looks similar to this:

```plaintext
secret.zip/secret.txt:$pkzip$1*2*2*0*1b*f*af1563e8*0*44*0*1b*4f26*50dbe9bcf15eafdb00ef774e1eaffef116bf5e0597fa55bb8b364c*$/pkzip$:secret.txt:secret.zip::secret.zip
```

The hash string includes the original filename, the encryption type (`$pkzip$`), and the encrypted data. With this hash file created, you are now ready to use John the Ripper to crack the password.

# Crack the Password

Now that you have the password hash, you can use John the Ripper to crack it. JtR will use its default settings, which include a dictionary of common passwords and rules to find a match for the hash.

To start the cracking process, simply run the `john` command and provide your hash file, `zip_hash.txt`, as the input:

```bash
john zip_hash.txt
```

Because we used a very weak password, John the Ripper should crack it quickly. The output will show the progress and the result. John will try different attack modes - first "single" mode, then wordlist mode:

```plaintext
Using default input encoding: UTF-8
Loaded 1 password hash (PKZIP [32/64])
Will run 4 OpenMP threads
Note: Passwords longer than 21 [worst case UTF-8] to 63 [ASCII] rejected
Proceeding with single, rules:Single
Press 'q' or Ctrl-C to abort, 'h' for help, almost any other key for status
Almost done: Processing the remaining buffered candidate passwords, if any.
0g 0:00:00:00 DONE 1/3 (2025-08-29 10:24) 0g/s 2619Kp/s 2619Kc/s 2619KC/s Txtsecret1900..Tzip1900
Proceeding with wordlist:/home/labex/project/john/run/password.lst
Enabling duplicate candidate password suppressor using 256 MiB
password123      (secret.zip/secret.txt)
1g 0:00:00:00 DONE 2/3 (2025-08-29 10:24) 8.333g/s 286541p/s 286541c/s 286541C/s 123456..nelly1
Use the "--show" option to display all of the cracked passwords reliably
Session completed
```

The line `password123 (secret.zip/secret.txt)` indicates that the password has been successfully cracked. Note that John first tries "single" mode (which didn't find the password), then moves to wordlist mode where it successfully finds the password.

Once a password has been cracked, John stores it in a file. To view any cracked passwords for a given hash file, you can use the `--show` option.

```bash
john --show zip_hash.txt
```

This command will display the cracked password in a clear format, linking it to the source file.

```plaintext
secret.zip/secret.txt:password123:secret.txt:secret.zip::secret.zip

1 password hash cracked, 0 left
```

You have successfully used John the Ripper to recover the password from the ZIP file's hash. The final step is to verify that this password works.

# Verify Access with the Cracked Password

The final step in this process is to confirm that the password you cracked is correct. You will do this by using the password to extract the contents of the `secret.zip` archive. This step validates the result of your password cracking effort.

Use the `unzip` command to extract the archive.

```bash
unzip secret.zip
```

The command will prompt you for the password. Enter `password123`, which is the password that John the Ripper recovered.

```plaintext
Archive:  secret.zip
[secret.zip] secret.txt password:
replace secret.txt? [y]es, [n]o, [A]ll, [N]one, [r]ename: yes
 extracting: secret.txt
```

You may see a prompt asking if you want to replace the existing `secret.txt` file (if it already exists from Step 2). Type `yes` to confirm the replacement. If the password is correct, the `unzip` command will extract `secret.txt` into your current directory without any errors.

To complete the verification, display the contents of the extracted file to ensure the data is intact.

```bash
cat secret.txt
```

The output should match the original text we created in Step 2.

```plaintext
my secret data
```

By successfully extracting the file and viewing its original content, you have confirmed that the password recovered by John the Ripper is correct. This completes the full cycle of creating a protected file, cracking its password, and verifying the result.