# Introduction

In this lab, you will learn how to use OpenSSL for file encryption and decryption, a crucial skill in cybersecurity. You will practice generating a symmetric key and applying AES encryption to protect sensitive data through hands-on exercises.

This lab walks you through installing OpenSSL, creating a test file, and performing a complete encryption-decryption cycle. This practical experience will help you understand fundamental data protection techniques used in real-world applications.

This is a Guided Lab, which provides step-by-step instructions to help you learn and practice. Follow the instructions carefully to complete each step and gain hands-on experience. Historical data shows that this is a beginner level lab with a 91% completion rate. It has received a 100% positive review rate from learners.


# Install and Verify OpenSSL

In this step, you will install OpenSSL, a powerful toolkit for implementing secure communications. While OpenSSL is often included in Linux distributions, this step ensures it is correctly installed and ready for use.

First, verify OpenSSL is installed by checking the installed version.

```bash
openssl version
```

You should see output similar to the following, indicating the version of OpenSSL on your system.

```plaintext
OpenSSL 3.0.2 15 Mar 2022 (Library: OpenSSL 3.0.2 15 Mar 2022)
```

Finally, you can display the help menu to see a list of available commands and confirm that OpenSSL is functioning correctly.

```bash
openssl help
```

This command will list the various cryptographic operations OpenSSL can perform, which we will explore in the next steps.

# Create a Sample File for Encryption

In this step, you will create a simple text file to use for our encryption and decryption exercises. Using a known file allows us to easily verify that the process works correctly by comparing the original and decrypted content.

Your terminal should already be in the `/home/labex/project` directory. We will create the file here.

First, use the `nano` text editor to create a new file named `sample.txt`.

```bash
nano sample.txt
```

Once `nano` opens, type the following text into the editor:

```plaintext
This is a sample text file for encryption testing.
LabEx Cybersecurity Lab - AES Encryption Demo.
```

Now, save the file and exit `nano`:

1. Press `Ctrl + O` to write the changes to the file.
2. Press `Enter` to confirm the filename (`sample.txt`).
3. Press `Ctrl + X` to exit the editor.

To confirm the file was created correctly, display its contents using the `cat` command.

```bash
cat sample.txt
```

The output should show the exact text you entered:

```plaintext
This is a sample text file for encryption testing.
LabEx Cybersecurity Lab - AES Encryption Demo.
```

This sample file is now ready for the encryption process in the upcoming steps.

# Generate a Symmetric Encryption Key

In this step, you will generate a symmetric key, which is a secret key used for both encrypting and decrypting data. We will use OpenSSL to create a strong, random key suitable for AES-256 encryption, a highly secure and widely adopted standard.

Ensure you are in your project directory, `/home/labex/project`.

```bash
cd ~/project
```

Use the `openssl rand` command to generate 32 bytes (256 bits) of random data and save it in hexadecimal format to a file named `symmetric_key.hex`.

```bash
openssl rand -hex 32 > symmetric_key.hex
```
The command you provided generates a random symmetric key using OpenSSL and saves it in a file named `symmetric_key.hex`. Here's a breakdown of the command:

- `openssl`: This is the command-line tool for using the OpenSSL library.
- `rand`: This subcommand is used to generate random data.
- `-hex`: This option specifies that the output should be in hexadecimal format.
- `32`: This indicates the number of bytes of random data to generate (32 bytes = 256 bits).
- `>`: This operator redirects the output to a file.
- `symmetric_key.hex`: This is the name of the file where the generated key will be stored.

In summary, this command creates a 256-bit random key and saves it in a hex format in the specified file.
View the generated key to see what it looks like.

```bash
cat symmetric_key.hex
```

The output will be a 64-character hexadecimal string, which represents your 256-bit key. It will look something like this (your key will be different):

```plaintext
2f8b5e9a1c3d4e6f8a9b0c1d2e3f4a5b6c7d8e9f0a1b2c3d4e5f6a7b8c9d0e1f
```

Since this key can unlock your data, it is critical to protect it. Use the `chmod` command to set the file permissions so that only the file owner (you) can read and write to it.

```bash
chmod 600 symmetric_key.hex
```

Finally, verify the file permissions with `ls -l`.

```bash
ls -l symmetric_key.hex
```

The output should show permissions as `-rw-------`, confirming that the file is secured.

```plaintext
-rw------- 1 labex labex 65 Nov 15 10:30 symmetric_key.hex
```

With the key generated and secured, you are now ready to encrypt the sample file.

# Encrypt the File with AES

Now you will use the symmetric key to encrypt `sample.txt`. We will use the AES-256-CBC encryption cipher. AES (Advanced Encryption Standard) with a 256-bit key is a strong encryption algorithm, and CBC (Cipher Block Chaining) is a mode of operation that enhances security by making each encrypted block dependent on the previous one.

The `openssl enc` command handles encryption. We will specify the cipher, input file, output file, and the key file.

Run the following command to encrypt `sample.txt` and save the output to `sample.enc`.

```bash
openssl enc -aes-256-cbc -in sample.txt -out sample.enc -pass file:symmetric_key.hex
```

- `-aes-256-cbc`: Specifies the encryption cipher.
- `-in sample.txt`: The input file to encrypt.
- `-out sample.enc`: The name of the output (encrypted) file.
- `-pass file:symmetric_key.hex`: Tells OpenSSL to use the content of `symmetric_key.hex` as the password (key).

After running the command, list the files to see the new encrypted file, `sample.enc`.


You can use the `-iter` and `-pbkdf2` options with the `openssl enc` command to enhance the security of the encryption process. Here's how you can incorporate them:

1. **`-pbkdf2`**: This option enables the use of the PBKDF2 key derivation function, which is more secure than the default method.
2. **`-iter`**: This option specifies the number of iterations for the key derivation function. A higher number of iterations increases security but may slow down the process.

Here’s how you can modify your command to include these options:

```bash
openssl enc -aes-256-cbc -in sample.txt -out sample.enc -pass file:symmetric_key.hex -pbkdf2 -iter 100000
```

In this example:

- `-iter 100000` sets the number of iterations to 100,000, which is a common practice for enhancing security.

This command will encrypt `sample.txt` using AES-256-CBC with the key derived from the contents of `symmetric_key.hex`, applying PBKDF2 with the specified number of iterations.
```bash
ls -l
```

You will notice that `sample.enc` is slightly larger than `sample.txt` due to encryption overhead like padding and metadata.

```plaintext
total 12
-rw-r--r-- 1 labex labex   96 Nov 15 10:35 sample.enc
-rw-r--r-- 1 labex labex   89 Nov 15 10:20 sample.txt
-rw------- 1 labex labex   65 Nov 15 10:30 symmetric_key.hex
```

If you try to view the contents of the encrypted file, you will see unreadable binary data.

```bash
cat sample.enc
```

The garbled output confirms that the file has been successfully encrypted.

# Decrypt the File and Verify Integrity

In this final step, you will decrypt `sample.enc` to get the original text back. This will confirm that your key works and the entire encryption-decryption cycle was successful.

To decrypt the file, you will use the `openssl enc` command again, but this time with the `-d` flag to specify decryption.

Run the following command to decrypt `sample.enc` and save the result to a new file, `sample.dec`.

```bash
openssl enc -d -aes-256-cbc -in sample.enc -out sample.dec -pass file:symmetric_key.hex
```
The command you provided is used to decrypt a file that was previously encrypted using OpenSSL. Here's a breakdown of the command:

- `openssl`: The command-line tool for using the OpenSSL library.
- `enc`: This subcommand is for encoding and decoding (encryption and decryption).
- `-d`: This option specifies that you want to decrypt the input file.
- `-aes-256-cbc`: This indicates the encryption algorithm used (AES with a 256-bit key in CBC mode).
- `-in sample.enc`: This specifies the input file to decrypt (the encrypted file).
- `-out sample.dec`: This specifies the output file where the decrypted content will be saved.
- `-pass file:symmetric_key.hex`: This option tells OpenSSL to read the password (or key) from the specified file (`symmetric_key.hex`).

In summary, this command decrypts the `sample.enc` file using the AES-256-CBC algorithm and saves the decrypted content to `sample.dec`.
Now, let's verify that the decryption worked. The best way to check is to compare the decrypted file (`sample.dec`) with the original file (`sample.txt`). The `diff` command is perfect for this.

```bash
diff sample.txt sample.dec
```

If the files are identical, this command will produce no output, which means the decryption was successful.

For a final visual confirmation, display the contents of the decrypted file.

```bash
cat sample.dec
```

The output should be the exact same as your original `sample.txt` file:

```plaintext
This is a sample text file for encryption testing.
LabEx Cybersecurity Lab - AES Encryption Demo.
```

You have now successfully encrypted a file and decrypted it back to its original form using OpenSSL.