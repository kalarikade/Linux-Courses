# Introduction

In the world of computing, data is invaluable. Data loss, whether due to hardware failure, accidental deletion, or malicious attacks, can be catastrophic. Data resilience is the practice of ensuring that your data is protected, available, and can be recovered in the event of a failure.

This lab will introduce you to two fundamental concepts of data resilience on Linux systems: creating backups and verifying file integrity. You will use standard Linux command-line tools to perform these tasks.

You will learn how to:

- Use the `tar` command to create a compressed archive (backup) of important files.
- Simulate a data loss scenario by deleting files.
- Restore your data from the backup archive.
- Use checksums with `md5sum` to verify that your files have not been altered or corrupted.

By the end of this lab, you will have practical, hands-on experience with essential skills for any system administrator or developer.
# Create a Local Backup of Critical Files with Tar

In this step, you will learn how to create a compressed backup of a directory using the `tar` (Tape Archive) utility. `tar` is a powerful and widely used command for bundling files and directories into a single archive file, which can then be compressed to save space. This is a common method for creating local backups.

First, let's verify the files that were created for you by the setup script. All your work will be done in the `~/project` directory.

List the contents of the `~/project` directory:

```bash
ls -R ~/project
```

You should see the `critical_data` directory with its files and an empty `backups` directory.

```plaintext
/home/labex/project:
backups  critical_data

/home/labex/project/backups:

/home/labex/project/critical_data:
config.txt  user_data.csv
```

Now, let's create a compressed backup of the `critical_data` directory and store it in the `backups` directory. We will use the `tar` command with the following options:

- `c`: Create a new archive.
- `z`: Compress the archive with gzip. This is why the file will have a `.gz` extension.
- `v`: Verbose mode. List the files as they are being processed.
- `f`: Specifies the filename of the archive to create.
- `-C`: Change to the specified directory before creating the archive. This ensures we store relative paths instead of absolute paths, making the backup more portable and avoiding nested directory structures during extraction.

Run the following command in your terminal:

```bash
tar -czvf ~/project/backups/backup.tar.gz -C ~/project critical_data
```

The command will output the names of the files it is adding to the archive:

```plaintext
critical_data/
critical_data/config.txt
critical_data/user_data.csv
```

You have now successfully created a backup. To be sure, you can list the contents of the `backups` directory:

```bash
ls ~/project/backups
```

You should see your new backup file:

```plaintext
backup.tar.gz
```

---
# Simulate Data Loss and Restore from Backup

In this step, you will simulate a common disaster scenario: accidental data deletion. Then, you will use the backup file you created in the previous step to restore the lost data.

First, let's simulate the data loss by permanently deleting the `critical_data` directory. The `rm -rf` command is very powerful and should be used with caution; it removes directories and files recursively (`-r`) and forcefully (`-f`) without prompting.

Execute the following command to delete the directory:

```bash
rm -rf ~/project/critical_data
```

Now, verify that the directory is gone by listing the contents of your project directory:

```bash
ls ~/project
```

You will see that the `critical_data` directory is no longer there:

```plaintext
backups
```

Don't worry! This is why we have backups. Now, let's restore the directory from the `backup.tar.gz` archive. We will use the `tar` command again, but with different options:

- `x`: Extract files from an archive.
- `z`: Decompress the archive with gzip.
- `v`: Verbose mode, to see the files as they are extracted.
- `f`: Specifies the filename of the archive to use.
- `-C`: Specifies the directory to extract the files to. We'll extract them to `~/project`.

Run this command to restore your data:

```bash
tar -xzvf ~/project/backups/backup.tar.gz -C ~/project
```

You will see the output of the files being extracted:

```plaintext
critical_data/
critical_data/config.txt
critical_data/user_data.csv
```

Notice that `tar` preserved the original directory structure, but without the absolute path prefixes since we used the `-C` option during backup creation. Let's verify that the restoration was successful. List the contents of the `~/project` directory again:

```bash
ls -R ~/project
```

You should see that the `critical_data` directory and all its original files have been restored.

```plaintext
/home/labex/project:
backups  critical_data

/home/labex/project/backups:
backup.tar.gz

/home/labex/project/critical_data:
config.txt  user_data.csv
```

You have successfully recovered from a simulated data loss event.

---
# Monitor File Integrity with Checksums

In this step, you will learn how to monitor file integrity. A backup protects you from data loss, but it doesn't tell you if a file has been subtly corrupted or maliciously altered. A checksum (or hash) is a unique digital fingerprint of a file. If even one bit in the file changes, the checksum will change completely. This makes it an excellent tool for verifying file integrity.

We will use the `md5sum` command, which calculates an MD5 checksum. While SHA-256 (`sha256sum`) is more cryptographically secure, `md5sum` is simple and sufficient for this demonstration.

First, let's generate checksums for our critical files and save them to a reference file.

```bash
md5sum ~/project/critical_data/* > ~/project/checksums.md5
```

Now, view the contents of the `checksums.md5` file you just created:

```bash
cat ~/project/checksums.md5
```

The output will show a unique checksum for each file. Your checksum values will be identical to these:

```plaintext
d7118547a95c5470575962481c2e358c  /home/labex/project/critical_data/config.txt
b643407358b9de495715f13432577d33  /home/labex/project/critical_data/user_data.csv
```

Now, let's simulate an unauthorized modification to one of the files. We will append a comment to the `config.txt` file.

```bash
echo "# This is a change" >> ~/project/critical_data/config.txt
```

The file has been altered. To an observer just looking at the file list, nothing seems wrong. But we can use our checksums file to check for changes. The `-c` option tells `md5sum` to check files against a list.

```bash
md5sum -c ~/project/checksums.md5
```

The command will check each file listed in `checksums.md5` against its current state on the disk. The output will clearly show that `config.txt` has been modified:

```plaintext
/home/labex/project/critical_data/config.txt: FAILED
/home/labex/project/critical_data/user_data.csv: OK
md5sum: WARNING: 1 computed checksum did NOT match
```

This "FAILED" status immediately alerts you that the file is no longer in its original, trusted state. This is a powerful technique for detecting data corruption or security breaches.

---
