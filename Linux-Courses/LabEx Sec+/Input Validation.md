# Introduction

In this lab, you will explore two fundamental security concepts: input validation and code integrity. Input validation is the practice of ensuring that any input a program receives is safe and well-formed before it is processed. This is a critical defense against a wide range of attacks, including command injection. Code integrity involves verifying that your application code has not been altered or corrupted by an unauthorized party.

You will learn these concepts through a hands-on approach. First, you will create a simple shell script that is intentionally vulnerable to command injection. Next, you will exploit this vulnerability to understand the risk. Then, you will secure the script by implementing proper input sanitization. Finally, you will learn how to use cryptographic hashes to verify the integrity of your script, ensuring it hasn't been tampered with.

# Create a Simple Script with User Input

In this step, you will create a simple Bash script that prompts the user for a filename and then displays detailed information about that file using the `ls -l` command. This initial version of the script will not have any security checks.

First, use the `nano` editor to create a new file named `check_file.sh` in the `~/project` directory.

```bash
nano ~/project/check_file.sh
```

Now, add the following code to the file. This script will ask for a filename, read the input into a variable named `filename`, and then execute `ls -l` on that filename using the `eval` command. Note that using `eval` with user input is a serious security vulnerability.

```bash
#!/bin/bash

echo "Please enter a filename to check:"
read filename
echo "Checking details for: $filename"
eval "ls -l $filename"
```

Press `Ctrl+X`, then `Y`, and `Enter` to save the file and exit `nano`.

Next, you need to make the script executable so you can run it. Use the `chmod` command to add execute permissions.

```bash
chmod +x ~/project/check_file.sh
```

Now, let's run the script to see it in action. We will use `testfile.txt`, which was created for you in the lab environment.

```bash
./check_file.sh
```

The script will prompt you for input. Type `testfile.txt` and press `Enter`.

```plaintext
Please enter a filename to check:
testfile.txt
Checking details for: testfile.txt
-rw-rw-r-- 1 labex labex 0 Aug  4 15:19 testfile.txt
```

You have successfully created and run a simple script that takes user input. In the next step, you will see how this script can be exploited.

---
# Simulate a Basic Command Injection Attack

In this step, you will see how the script created in the previous step is vulnerable to a command injection attack. Command injection occurs when an attacker is able to execute arbitrary commands on the host operating system through a vulnerable application.

Our script is vulnerable because it uses `eval` to execute a command that includes user input (`$filename`) without first checking if the input is safe. The `eval` command treats the entire string as a command to be executed, making it particularly dangerous. An attacker can exploit this by providing input that includes additional shell commands.

Let's run the script again.

```bash
./check_file.sh
```

This time, when prompted for a filename, enter the following string. The semicolon (`;`) is a special character in the shell that separates commands.

```plaintext
testfile.txt; whoami
```

After you press `Enter`, you will see the following output:

```plaintext
Please enter a filename to check:
testfile.txt; whoami
Checking details for: testfile.txt; whoami
-rw-rw-r-- 1 labex labex 0 Aug  4 15:19 testfile.txt
labex
```

Notice what happened. The script first executed `ls -l testfile.txt` as expected. However, because of the semicolon and the use of `eval`, the shell then executed the second command, `whoami`, which printed the current username (`labex`). This demonstrates a successful command injection. The `eval` command made this possible by treating the entire input as executable code. An attacker could use this vulnerability to run much more dangerous commands.

**Note:** If you used `ls -l $filename` without `eval` (as in the original version), the attack would not work because the shell would treat `testfile.txt; whoami` as separate arguments to `ls`, resulting in error messages like "cannot access 'testfile.txt;'" and "cannot access 'whoami'". The `eval` command is what makes the command injection possible in this example.

---
# Implement Input Sanitization to Prevent Injection

In this step, you will modify the script to prevent command injection attacks by sanitizing the user input and removing the dangerous `eval` command. Input sanitization is the process of cleaning or filtering input to remove or neutralize potentially malicious characters.

Our strategy will be to:

1. Remove the dangerous `eval` command and use direct command execution with proper quoting
2. Check if the input filename contains only allowed characters. For a typical filename, we can create a whitelist of characters, such as letters, numbers, underscores, hyphens, and periods. We will reject any input that contains other characters, like the semicolon.

Open the `check_file.sh` script again with `nano`.

```bash
nano ~/project/check_file.sh
```

Modify the script to include a validation check using a regular expression. Replace the existing content with the following code:

```bash
#!/bin/bash

echo "Please enter a filename to check:"
read filename

# Input sanitization: only allow alphanumeric characters, underscores, hyphens, and dots.
if [[ "$filename" =~ ^[a-zA-Z0-9_.-]+$ ]]; then
  echo "Checking details for: $filename"
  ls -l "$filename"
else
  echo "Error: Invalid filename. Malicious input detected."
fi
```

The key changes are:

1. Removal of the dangerous `eval` command
2. Addition of proper quoting around `"$filename"` in the `ls` command
3. The `if` statement with input validation. The expression `[[ "$filename" =~ ^[a-zA-Z0-9_.-]+$ ]]` checks if the `filename` variable contains only the characters in the specified set from beginning (`^`) to end (`$`).

Press `Ctrl+X`, `Y`, and `Enter` to save the changes.

Now, let's try the same attack again. Run the script:

```bash
./check_file.sh
```

Enter the malicious input `testfile.txt; whoami` and press `Enter`.

```plaintext
Please enter a filename to check:
testfile.txt; whoami
Error: Invalid filename. Malicious input detected.
```

As you can see, the script now detects the invalid characters and prints an error message instead of executing the malicious `whoami` command. The attack is successfully prevented.

---
# Verify Script Integrity with Hashing

In this step, you will learn how to verify the integrity of your script using a cryptographic hash. This ensures that the script has not been modified or tampered with by an attacker after you have secured it. We will use the `sha256sum` utility, which calculates a SHA-256 hash.

First, let's generate a hash for our secure `check_file.sh` script and save it to a file. This file will act as our "known good" signature.

```bash
sha256sum ~/project/check_file.sh > ~/project/check_file.sha256
```

This command computes the SHA-256 hash of `check_file.sh` and redirects the output to a new file named `check_file.sha256`. Let's view the contents of this file.

```bash
cat ~/project/check_file.sha256
```

You will see a long string of characters followed by the filename. This string is the unique hash of your script.

```plaintext
e3b0c44298fc1c149afbf4c8996fb92427ae41e4649b934ca495991b7852b855  /home/labex/project/check_file.sh
```

_(Note: Your hash value will be different.)_

Now, let's simulate an unauthorized modification. We will add a simple comment to the end of the script. Even a small change like this should alter the hash completely.

```bash
echo "# A harmless comment" >> ~/project/check_file.sh
```

The script has now been modified. To check its integrity, we can use `sha256sum` with the `-c` (check) option, which reads the hash from our signature file and compares it to the current hash of the script.

```bash
sha256sum -c ~/project/check_file.sha256
```

The command will report a failure because the file has changed.

```plaintext
/home/labex/project/check_file.sh: FAILED
sha256sum: WARNING: 1 computed checksum did NOT match
```

This confirms that the script's integrity has been compromised. This technique is essential for ensuring that the code you are running is the code you trust.