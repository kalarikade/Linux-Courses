# Introduction

In the digital world, ensuring that data has not been tampered with (integrity) and that it comes from a trusted source (authenticity) is crucial. Cryptography provides the tools to achieve these goals.

This lab will introduce you to fundamental cryptographic practices on a Linux system. You will learn about:

- **Hashing**: Creating a unique, fixed-size "fingerprint" of a file. If the file changes even by a single bit, the hash will change completely. We will use the MD5 and SHA256 algorithms.
- **Digital Signatures**: Using a private key to "sign" a file, which allows anyone with the corresponding public key to verify that the file is authentic and has not been modified.
- **Encryption**: Scrambling the contents of a file so that it can only be read by someone who has the correct key to decrypt it, ensuring confidentiality.

We will use standard Linux command-line tools, including `md5sum`, `sha256sum`, and GnuPG (`gpg`), the GNU implementation of the Pretty Good Privacy (PGP) standard. By the end of this lab, you will be able to confidently hash, sign, verify, encrypt, and decrypt files.

This is a Guided Lab, which provides step-by-step instructions to help you learn and practice. Follow the instructions carefully to complete each step and gain hands-on experience. Historical data shows that this is a beginner level lab with a 87% completion rate. It has received a 100% positive review rate from learners.

![labby](https://labex.io/images/labby/avatar-new.png)Labby

Great! Let me prepare the lab VM for you. While it's being set up, you can start reviewing the first step of this lab.

![labby](https://labex.io/images/labby/avatar-new.png)Labby

# Calculate File Hashes (MD5 and SHA256)

In this step, you will learn how to calculate cryptographic hashes for a file. A hash function takes an input (like a file) and returns a fixed-size string of bytes, known as the hash value or digest. This value acts as a digital fingerprint.

First, we will use the `md5sum` command to calculate the MD5 hash of the `important_data.txt` file that has been pre-created in your `~/project` directory.

Execute the following command in your terminal:

```bash
md5sum important_data.txt
```

You will see an output consisting of the hash value followed by the filename:

```plaintext
d9e21981521545759153147864347199  important_data.txt
```

While MD5 is fast, it is now considered insecure for cryptographic purposes due to vulnerabilities. A more secure and commonly used alternative is the SHA-2 family, specifically SHA256.

Now, let's calculate the SHA256 hash using the `sha256sum` command:

```bash
sha256sum important_data.txt
```

The output will be a longer, more secure hash:

```plaintext
a39b2c414f234246a2535321238863141b1a4849443b9992994b4189317e8591  important_data.txt
```

If you were to change anything in `important_data.txt`, both the MD5 and SHA256 hashes would change completely, allowing you to easily detect any unauthorized modifications.


---


# Install and Configure GnuPG (GPG)

In this step, you will install GnuPG (GPG), the tool we will use for digital signatures and encryption. While GPG is often pre-installed on modern Linux distributions, it's good practice to ensure it's present and up-to-date.

First, update your package list using `apt-get update`. You need to use `sudo` because package management requires administrative privileges.

```bash
sudo apt-get update
```

Next, install the `gnupg` package. The `-y` flag automatically answers "yes" to any prompts, making the installation non-interactive.

```bash
sudo apt-get install -y gnupg
```

After the installation is complete, you can verify that GPG is installed correctly by checking its version.

```bash
gpg --version
```

You should see output similar to the following, confirming the installation. The version numbers and details may vary slightly.

```plaintext
gpg (GnuPG) 2.2.27
libgcrypt 1.9.4
Copyright (C) 2021 Free Software Foundation, Inc.
License GNU GPL-3.0-or-later <https://gnu.org/licenses/gpl.html>
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.

Home: /home/labex/.gnupg
Supported algorithms:
Pubkey: RSA, ELG, DSA, ECDH, ECDSA, EDDSA
Cipher: IDEA, 3DES, CAST5, BLOWFISH, AES, AES192, AES256, TWOFISH,
        CAMELLIA128, CAMELLIA192, CAMELLIA256
Hash: SHA1, RIPEMD160, SHA256, SHA384, SHA512, SHA224
Compression: Uncompressed, ZIP, ZLIB, BZIP2
```

Now that GPG is ready, we can proceed to create our own cryptographic keys.

# Generate a GPG Key Pair for Digital Signatures

In this step, you will generate your own GPG key pair, which consists of a private key and a public key.

- **Private Key**: Must be kept secret. It is used for decrypting messages and creating digital signatures.
- **Public Key**: Can be shared freely. It is used for encrypting messages intended for you and for verifying your digital signatures.

Normally, `gpg --full-generate-key` runs an interactive setup. To make this process simpler and non-interactive for the lab, we will use GPG's batch mode. First, create a parameter file that specifies the details for our key.

Use the following `cat` command with a "here document" to create the file `gen-key-params` instantly:

```bash
cat << EOF > gen-key-params
%echo Generating a basic key
Key-Type: RSA
Key-Length: 2048
Subkey-Type: RSA
Subkey-Length: 2048
Name-Real: LabEx User
Name-Email: user@labex.io
Expire-Date: 0
%no-protection
%commit
%echo done
EOF
```

This file instructs GPG to create a 2048-bit RSA key for a user named "LabEx User" with the email "[user@labex.io](mailto:user@labex.io)". The key will never expire, and we use `%no-protection` to create the key without a passphrase for simplicity (in a real-world scenario, you should always use a strong passphrase).

Now, generate the key using the parameter file:

```bash
gpg --batch --gen-key gen-key-params
```

GPG will use the parameters to generate the key pair. This may take a few moments. Once it's done, you can list your keys to confirm that the key pair was created successfully.

```bash
gpg --list-keys
```

The output should show your newly created public key, similar to this:

```plaintext
/home/labex/.gnupg/pubring.kbx
-------------------------------
pub   rsa2048 2023-10-27 [SC]
      XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
uid           [ultimate] LabEx User <user@labex.io>
sub   rsa2048 2023-10-27 [E]
```

You now have a GPG key pair ready for signing and encrypting files.

# Digitally Sign and Verify a File with GPG

In this step, you will use your private key to create a digital signature for a file. This signature proves that the file originated from you and has not been altered since you signed it.

We will create a "detached" signature, which is stored in a separate file. This is useful because the original file remains untouched.

To sign `important_data.txt`, run the following command. GPG will automatically use the default private key you just generated.

```bash
gpg --detach-sign important_data.txt
```

Since we created the key without a passphrase, you will not be prompted for one. Now, list the files in your directory to see the new signature file.

```bash
ls
```

You will see the original file and its signature:

```plaintext
gen-key-params  important_data.txt  important_data.txt.sig
```

Now, imagine you are a recipient who has received both `important_data.txt` and `important_data.txt.sig`, along with your public key. To verify the file's authenticity and integrity, they would run the `gpg --verify` command.

Let's perform the verification yourself:

```bash
gpg --verify important_data.txt.sig important_data.txt
```

The output will confirm that the signature is valid:

```plaintext
gpg: Signature made Mon Aug  4 16:39:30 2025 CST
gpg:                using RSA key 8765265B14E42532B9CBAE6DE2120C9784C69814
gpg: Good signature from "LabEx User <user@labex.io>" [ultimate]
```

The "Good signature" message confirms the file is authentic and unmodified.


# Encrypt and Decrypt a File with GPG

In this step, you will learn how to encrypt a file for confidentiality. Unlike signing (which uses your private key), encrypting a file for someone requires their public key. The recipient then uses their private key to decrypt it.

Here, we will encrypt `important_data.txt` for ourselves, so we will use our own public key as the recipient's key.

Use the following command to encrypt the file. The `--recipient` flag specifies whose public key to use for encryption, and `--output` defines the name of the encrypted file.

```bash
gpg --encrypt --recipient "user@labex.io" --output important_data.txt.gpg important_data.txt
```

If you try to view the contents of the new encrypted file, `important_data.txt.gpg`, you will see unreadable binary data.

```bash
cat important_data.txt.gpg
```

Now, let's decrypt the file. This action requires the corresponding private key. Since you have the private key, you can decrypt the message. We will save the decrypted content to a new file named `decrypted_data.txt`.

```bash
gpg --decrypt --output decrypted_data.txt important_data.txt.gpg
```

GPG will use your private key to decrypt the file. You will see some information from GPG on your terminal, including a confirmation that the file was encrypted with your key.

Finally, view the contents of the decrypted file to confirm that the process was successful.

```bash
cat decrypted_data.txt
```

The output should be the original, readable message:

```plaintext
This is a secret message that needs to be protected.
```

You have successfully encrypted a file to protect its contents and then decrypted it to retrieve the original information.