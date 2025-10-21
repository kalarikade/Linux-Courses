# Create a Simulated Disk Image Using dd

In this step, you will create a forensic image from a simulated evidence file. In digital forensics, an "image" is a bit-for-bit copy of a source drive. We use the `dd` command, a powerful and versatile utility for copying and converting data. It's a standard tool in forensics because it can create an exact replica of a storage device, preserving all data, including deleted files and slack space.

First, let's navigate to our case directory and see the original evidence file that has been prepared for you.

```bash
cd ~/project/forensics_case
ls -lh
```

You should see a file named `original_evidence.dd`.

```plaintext
-rw-rw-r-- 1 labex labex 11M Aug  5 15:28 original_evidence.dd
```

Now, let's use `dd` to create an image of this file. We will name our image `evidence_image.img`.

- `if=original_evidence.dd`: Specifies the **i**nput **f**ile.
- `of=evidence_image.img`: Specifies the **o**utput **f**ile.
- `bs=4096`: Sets the **b**lock **s**ize to 4096 bytes. This is a common block size and can affect performance.

Execute the following command:

```bash
dd if=original_evidence.dd of=evidence_image.img bs=4096
```

The command will output the number of records in and out, and the total bytes copied.

```plaintext
2560+1 records in
2560+1 records out
10485809 bytes (10 MB, 10 MiB) copied, 0.0130138 s, 806 MB/s
```

Now, list the files again to see both the original and the newly created image.

```bash
ls -lh
```

You will see that `evidence_image.img` has the exact same size as `original_evidence.dd`, which is a good first sign that our copy was successful.

```plaintext
-rw-rw-r-- 1 labex labex 11M Aug  5 15:43 evidence_image.img
-rw-rw-r-- 1 labex labex 11M Aug  5 15:28 original_evidence.dd
```

# Calculate Hashes of Original and Image Files

In this step, you will verify the integrity of the forensic image you just created. Simply having a copy is not enough; we must be able to prove that the copy is an exact, unaltered replica of the original. We do this by calculating a cryptographic hash for both the original file and the image.

A hash function takes an input (in our case, the file's content) and produces a fixed-size string of characters, which is the "hash." Even a tiny change in the input file will result in a completely different hash. If the hash of the original file and the hash of the image file match, we can be confident that the copy is perfect. We will use the `sha256sum` command, which implements the SHA-256 hashing algorithm.

First, calculate the SHA256 hash of the original evidence file:

```bash
sha256sum original_evidence.dd
```

The output will be a long string of characters (the hash) followed by the filename.

```plaintext
55a290c58509790860da55a47256188865bdd8dd5cbf7cd5c4b95cb5264f109a  original_evidence.dd
```

Next, calculate the hash for the image file you created:

```bash
sha256sum evidence_image.img
```

```plaintext
55a290c58509790860da55a47256188865bdd8dd5cbf7cd5c4b95cb5264f109a  evidence_image.img
```

Compare the two hashes. They must be identical. This is the mathematical proof that your `evidence_image.img` is a true and accurate copy of `original_evidence.dd`.

For proper documentation, it's best practice to save these hashes to a log file. Let's do that now.

```bash
sha256sum original_evidence.dd evidence_image.img > hashes.txt
```

Now, view the contents of your hash log file.

```bash
cat hashes.txt
```

```plaintext
55a290c58509790860da55a47256188865bdd8dd5cbf7cd5c4b95cb5264f109a  original_evidence.dd
55a290c58509790860da55a47256188865bdd8dd5cbf7cd5c4b95cb5264f109a  evidence_image.img
```

This `hashes.txt` file is a crucial piece of documentation for your case.

# Extract Basic File Metadata such as Timestamps

In this step, we will examine the file's metadata. Metadata is "data about data," and it can provide crucial context in an investigation. For files, this includes information like creation time, last modification time, and last access time. These are often referred to as MAC times (Modify, Access, Change).

We will use the `stat` command to view this detailed information. The `stat` command displays file or file system status.

Let's first look at the metadata for the `original_evidence.dd` file.

```bash
stat original_evidence.dd
```

The output provides a wealth of information, but let's focus on the timestamps.

```plaintext
  File: original_evidence.dd
  Size: 10485809        Blocks: 20488      IO Block: 4096   regular file
Device: 7eh/126d        Inode: 11210686    Links: 1
Access: (0664/-rw-rw-r--)  Uid: ( 5000/   labex)   Gid: ( 5000/   labex)
Access: 2025-08-05 15:43:57.680473291 +0800
Modify: 2025-08-05 15:28:59.196596566 +0800
Change: 2025-08-05 15:28:59.196596566 +0800
 Birth: 2025-08-05 15:28:59.164595416 +0800
```

Now, let's do the same for our `evidence_image.img`.

```bash
stat evidence_image.img
```

```plaintext
  File: evidence_image.img
  Size: 10485809        Blocks: 20488      IO Block: 4096   regular file
Device: 7eh/126d        Inode: 11206338    Links: 1
Access: (0664/-rw-rw-r--)  Uid: ( 5000/   labex)   Gid: ( 5000/   labex)
Access: 2025-08-05 15:44:14.001048192 +0800
Modify: 2025-08-05 15:43:57.692473714 +0800
Change: 2025-08-05 15:43:57.692473714 +0800
 Birth: 2025-08-05 15:43:57.680473291 +0800
```

Notice that while the file content is identical (as proven by the hash), the metadata is not. The timestamps for `evidence_image.img` reflect when the image was _created_, not when the original file was created or modified. This is expected behavior and is important to document. It shows when you, the investigator, performed the acquisition.
The output you provided is the result of the `ls -l` command in a Unix-like operating system, which displays detailed information about a file. Here's a breakdown of each part:

- **File**: `evidence_image.img` - The name of the file.
- **Size**: `10485809` - The size of the file in bytes.
- **Blocks**: `20488` - The number of blocks allocated for the file on disk.
- **IO Block**: `4096` - The size of each block in bytes.
- **Type**: `regular file` - Indicates that this is a standard file.
- **Device**: `7eh/126d` - The device identifier where the file is stored.
- **Inode**: `11206338` - The inode number, which is a unique identifier for the file in the filesystem.
- **Links**: `1` - The number of hard links to the file.
- **Access**: `(0664/-rw-rw-r--)` - The file permissions. `0664` is the octal representation, and `-rw-rw-r--` indicates read and write permissions for the owner and group, and read-only for others.
- **Uid**: `( 5000/ labex)` - The user ID and username of the file owner.
- **Gid**: `( 5000/ labex)` - The group ID and group name of the file owner.
- **Access**: `2025-08-05 15:44:14.001048192 +0800` - The last access time of the file.
- **Modify**: `2025-08-05 15:43:57.692473714 +0800` - The last modification time of the file.
- **Change**: `2025-08-05 15:43:57.692473714 +0800` - The last status change time of the file.
- **Birth**: `2025-08-05 15:43:57.680473291 +0800` - The creation time of the file (if supported by the filesystem).

# Generate a Chain of Custody Log for Acquired Evidence

In this final step, you will create a simple Chain of Custody log. The Chain of Custody is one of the most critical documents in forensics. It is a chronological record that details the seizure, custody, control, transfer, analysis, and disposition of evidence. A well-maintained chain of custody proves that the evidence has been handled properly and has not been tampered with.

While a real chain of custody form is more complex, we can create a basic text log to document our actions. This log should include key information about the evidence and the acquisition process.

We will use the `echo` command to write information into a file named `chain_of_custody.log`. We'll need to gather a few pieces of information:

- **Case Number:** A unique identifier for the investigation. We'll use `CASE-001`.
- **Item Description:** What the evidence is.
- **Date/Time of Acquisition:** We can get this using the `date` command.
- **Acquired by:** The analyst who performed the acquisition. We'll use the output of the `whoami` command.
- **SHA256 Hash:** The hash of the acquired image, which we'll get from our `hashes.txt` file.

Let's create the log. We'll use `echo` with the `>` operator to create the file and add the first line, and then `>>` to append subsequent lines. First, let's grab the hash of our image file.

```bash
IMAGE_HASH=$(grep 'evidence_image.img' hashes.txt | cut -d ' ' -f 1)
```

Now, let's create the log file with all the necessary information.

```bash
echo "--- CHAIN OF CUSTODY ---" > chain_of_custody.log
echo "Case Number: CASE-001" >> chain_of_custody.log
echo "------------------------" >> chain_of_custody.log
echo "Item ID: 1" >> chain_of_custody.log
echo "Description: Forensic image of original_evidence.dd" >> chain_of_custody.log
echo "Acquired By: $(whoami)" >> chain_of_custody.log
echo "Acquisition Date: $(date)" >> chain_of_custody.log
echo "SHA256 Hash: $IMAGE_HASH" >> chain_of_custody.log
echo "--- END OF LOG ---" >> chain_of_custody.log
```

Finally, display your completed chain of custody log.

```bash
cat chain_of_custody.log
```

Your output should look similar to this:

```plaintext
--- CHAIN OF CUSTODY ---
Case Number: CASE-001
------------------------
Item ID: 1
Description: Forensic image of original_evidence.dd
Acquired By: labex
Acquisition Date: Tue Aug  5 15:44:43 CST 2025
SHA256 Hash: 55a290c58509790860da55a47256188865bdd8dd5cbf7cd5c4b95cb5264f109a
--- END OF LOG ---
```

You have now created a basic but essential piece of documentation that records your actions and preserves the integrity of the evidence.