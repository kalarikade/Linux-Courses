# Introduction

Welcome to the Basic File Operations in Linux lab! In Linux, almost everything is treated as a file, which makes file operations fundamental to using the system. This lab will guide you through the most common commands for managing files and directories, helping you become more proficient in navigating and organizing your Linux system.

This is a Guided Lab, which provides step-by-step instructions to help you learn and practice. Follow the instructions carefully to complete each step and gain hands-on experience. Historical data shows that this is a intermediate level lab with a 67% completion rate. It has received a 99% positive review rate from learners.


# Understanding Your Working Environment

In Linux, each user typically has a "home directory," represented by `~`. However, in this lab environment, we'll start in the `/home/labex/project` directory, which is our default working directory.

First, please open a terminal on the Desktop **OR** switch to the terminal tab in the lab environment.

![Terminal interface screenshot](https://file.labex.io/namespace/df87b950-1f37-4316-bc07-6537a1f2c481/linux/lab-basic-files-operations/assets/20240727-20-38-20-xoA2i66E@2x.png)

Let's begin by understanding our current location:

```bash
pwd
```

`pwd` stands for "print working directory". It displays your current location in the file system. This command is crucial for orienting yourself in the Linux file structure. You should see `/home/labex/project` as the output.

Now, let's explore the relationship between the current directory and the home directory:

```bash
echo ~
```

> Note: If you cannot type the `~` symbol in the Desktop terminal due to keyboard layout differences in certain countries, you can try switching to the standalone Terminal tab in the upper left corner of the VM interface.

This command will display the path to your home directory, which should be `/home/labex`.

To see the contents of your current directory, use:

```bash
ls
```

This will list the files and directories in your current working directory (`/home/labex/project`).

Let's also check the contents of your home directory:

```bash
ls ~
```

This command lists the contents of your home directory, which may be different from your current working directory.

Understanding the distinction between your current working directory and your home directory is important for navigating the Linux file system effectively.


# Navigating the File System

Linux uses what we call a "hierarchical file system". Think of it like a big tree with branches. The main trunk is called the "root directory", represented by a single forward slash `/`. All other directories and files branch out from this root.

Let's explore how to move around in this tree-like structure:

1. Check your current location:

```bash
pwd
```

This should show `/home/labex/project`. If it doesn't, you might be in a different directory. Use `cd /home/labex/project` to get back to the starting point.

2. View the contents of your current directory:

```bash
ls
```

This lists all files and folders in your current location. `/home/labex/project` is empty, so you won't see anything.

3. Move up one level to the parent directory:

```bash
cd ..
```

The `..` means "the directory above". After this command, do `pwd` again. You should now be in `/home/labex`.

4. Return to your project directory:

```bash
cd project
```

This takes you back to `/home/labex/project`.

5. Go to your home directory:

```bash
cd ~
```

The `~` is a shortcut for your home directory. Do `pwd` to confirm you're in `/home/labex`.

6. Return to the project directory using an absolute path:

```bash
cd /home/labex/project
```

This is called an "absolute path" because it starts from the root (`/`) and gives the full location.


# Creating Files and Listing Directory Contents

Now that we know how to navigate, let's create some files and explore how to list directory contents.

First, make sure you're in the `/home/labex/project` directory:

```bash
cd /home/labex/project
```

1. Let's create a few files:

```bash
touch file1.txt
```

The `touch` command is used to create an empty file. If the file already exists, it updates the file's timestamp without changing its content. It's a simple way to create new, empty files.

```bash
echo "Hello, Linux" > file2.txt
```

This command does two things:

- `echo` is a command that prints text.
- The `>` symbol redirects the output of `echo` into a file named `file2.txt`. If the file doesn't exist, it's created. If it does exist, its content is replaced.

```bash
echo "Hidden file" > .hiddenfile
```

This creates a hidden file. In Linux, any file or directory name that starts with a dot (.) is considered hidden.

2. Now, let's create a directory:

```bash
mkdir testdir
```

The `mkdir` command (short for "make directory") creates a new directory named `testdir`.

3. Basic listing:

```bash
ls
```

![Directory contents listing output](https://file.labex.io/namespace/df87b950-1f37-4316-bc07-6537a1f2c481/linux/lab-basic-files-operations/assets/20240909-13-35-48-tD0mGo9k.png)

This shows the contents of your current directory. You should see `file1.txt`, `file2.txt`, and `testdir`.

4. Detailed listing:

```bash
ls -l
```

The `-l` option (that's a lowercase L, not the number 1) provides a "long" format listing. You'll see additional details like file permissions, owner, size, and modification date.

5. Show hidden files:

```bash
ls -a
```

This will show all files, including the hidden `.hiddenfile` we created.

6. Combine options:

```bash
ls -la
```

This combines the long format (`-l`) with showing all files (`-a`).

7. List contents of a specific directory:

```bash
ls -l testdir
```

This lists the contents of the `testdir` directory (which should be empty at this point).

# Copying Files and Directories

Now that we have some files to work with, let's learn how to copy them:

1. Copy a file:

```bash
cp file1.txt file1_copy.txt
```

This creates a copy of `file1.txt` named `file1_copy.txt` in the current directory.

Let's verify the copy:

```bash
ls
```

2. Copy a file to another directory:

```bash
cp file2.txt testdir/
```

This copies `file2.txt` into the `testdir` directory.

3. Copy a directory:

```bash
cp -r testdir testdir_copy
```

The `-r` option stands for "recursive". It's necessary when copying directories to ensure all contents are copied.

4. Verify our copies:

```bash
ls
ls testdir
ls testdir_copy
```

![Terminal showing copied files](https://file.labex.io/namespace/df87b950-1f37-4316-bc07-6537a1f2c481/linux/lab-basic-files-operations/assets/20240909-13-36-42-EGSUMnzx.png)


# Moving and Renaming Files and Directories

The `mv` command is used for both moving and renaming in Linux:

1. Rename a file:

```bash
mv file1.txt newname.txt
```

This renames `file1.txt` to `newname.txt`.

2. Move a file to a directory:

```bash
mv newname.txt testdir/
```

This moves `newname.txt` into the `testdir` directory.

3. Rename a directory:

```bash
mv testdir_copy new_testdir
```

This renames `testdir_copy` to `new_testdir`.

4. Move and rename in one command:

```bash
mv testdir/newname.txt ./original_file1.txt
```

This moves `newname.txt` out of `testdir` and renames it to `original_file1.txt` in the current directory.

5. Verify our changes:

```bash
ls
ls testdir
```

![Command execution results display](https://file.labex.io/namespace/df87b950-1f37-4316-bc07-6537a1f2c481/linux/lab-basic-files-operations/assets/20240909-13-37-29-enkh85AS.png)


# Removing Files and Directories

Removing files and directories is a powerful operation. Unlike graphical interfaces, the command line often doesn't have a "Recycle Bin" or "Trash". Deletions made with `rm` are usually permanent. Always double-check your commands before executing them!

Let's clean up the files and directories we created. Make sure you are in the `/home/labex/project` directory.

```bash
pwd
ll -a
```

You should see files like `original_file1.txt`, `.hiddenfile`, `file2.txt`, and directories like `testdir`, `new_testdir`.

```plaintext
-rw-rw-r-- 1 labex labex   12 May  3 08:44 .hiddenfile
-rw-rw-r-- 1 labex labex    0 May  3 08:45 file1_copy.txt
-rw-rw-r-- 1 labex labex   13 May  3 08:44 file2.txt
drwxrwxr-x 2 labex labex   23 May  3 08:45 new_testdir
-rw-rw-r-- 1 labex labex    0 May  3 08:44 original_file1.txt
drwxrwxr-x 2 labex labex   23 May  3 08:45 testdir
```

1. **Remove a single file:**
    
    ```bash
    rm original_file1.txt
    ```
    

The `rm` command (short for "remove") deletes files. Let's check:

```bash
ls
```

- `original_file1.txt` should be gone.
    
- **Remove interactively (safer):**
    
    Let's try to remove `file2.txt`, but this time using the interactive flag `-i`:
    
    ```bash
    rm -i file2.txt
    ```
    

The `-i` option prompts you for confirmation before deleting each file. Type `y` (for yes) and press Enter to confirm the deletion. If you type `n` or anything else, the file will not be deleted.

```bash
ls
```

- If you confirmed, `file2.txt` will be gone.
    
- **Remove an empty directory:**
    
    Remember the `new_testdir` we created by renaming `testdir_copy`? Let's check if it's empty:
    
    ```bash
    ls new_testdir
    ```
    

If it's empty (shows no files), we can remove it using `rmdir`:

```bash
rmdir new_testdir
```

`rmdir` (remove directory) only works on **empty** directories.

```bash
ls
```

- `new_testdir` can't be removed because it's not empty.
    
- **Attempt to remove a non-empty directory:**
    
    Now, let's try `rmdir` on `testdir`, which still contains `file2.txt` (copied in Step 4):
    
    ```bash
    ls testdir
    rmdir testdir
    ```
    
- You will likely see an error message like `rmdir: failed to remove 'testdir': Directory not empty`. This is expected because `rmdir` cannot remove directories that contain files or other directories.
    
- **Remove a directory and its contents (recursively):**
    
    To remove a directory that is _not_ empty, we need to use `rm` with the `-r` (recursive) option:
    
    ```bash
    rm -r testdir
    ```
    

This command removes the `testdir` directory and everything inside it. Use this command with caution.

```bash
ls
```

- `testdir` should now be gone.
    
- **Force removal (use with extreme caution):**
    
    Sometimes, you might want to remove files without being prompted, even if they are write-protected (though we don't have any here). The `-f` (force) option does this.
    
    Let's remove our hidden file:
    
    ```bash
    rm .hiddenfile
    ls -a
    ```
    

Now, let's combine `-r` and `-f`. The `rm -rf` command is extremely powerful and potentially dangerous. It removes directories recursively (`-r`) and forces removal without prompting (`-f`).

**!!! DANGER ZONE !!!**  
Be **ABSOLUTELY SURE** you know what you are deleting before running `rm -rf`. A small typo could delete critical system files or your personal data. There is no undo. For example, `rm -rf /` could attempt to delete your entire system (if you have permissions). **Always double-check the path.**

Let's create a temporary directory and file to demonstrate (safely):

```bash
mkdir temp_dir
touch temp_dir/temp_file.txt
ls -R temp_dir
```

**Note: You might notice we used `-R` (uppercase) with `ls` instead of `-r` (lowercase) like we did with `cp` and `rm`. This is not just a case difference - they are completely different options! For `ls`, `-R` means "recursive listing" (list subdirectories), while `-r` means "reverse sort order". For `cp` and `rm`, the recursive option is `-r` (lowercase). Always check the manual (`man command`) to understand each command's specific options.**

Now, let's remove it forcefully:

```bash
rm -rf temp_dir
```

Verify the removal:

```bash
ls
```

1. `temp_dir` should be gone.
    

Remember: In Linux command line, deleted files are generally gone forever. Use `rm` carefully!

---

# Introduction

In this challenge, you will delve into the world of Linux file management by practicing essential commands used to navigate the file system and manipulate files and directories. Specifically, you will learn to copy, move, and delete files and directories – fundamental operations for any Linux user or system administrator. Mastering these commands is crucial for efficiently managing your data and system in a Linux environment.

## Achievements

Upon completing this challenge, you will demonstrate proficiency in using:

- `cp` - to copy files and directories
- `mv` - to move and rename files and directories
- `rm` - to remove files and directories

This is a Challenge, which differs from a Guided Lab in that you need to try to complete the challenge task independently, rather than following the steps of a lab to learn. Challenges are usually a bit difficult. If you find it difficult, you can discuss with Labby or check the solution. Historical data shows that this is a beginner level challenge with a 89% pass rate. It has received a 99% positive review rate from learners.



# Copy Files and Directories

This initial step focuses on mastering the `cp` command, a cornerstone of file system manipulation. You will learn how to duplicate both individual files and entire directory structures, a common task when backing up data or setting up new environments.

## Tasks

1. Copy the `~/.zshrc` file to `~/Desktop/zshrc-copy`.
2. Copy the entire `~/Code` directory to `~/Desktop`.

## Requirements

- Use the `cp` command to complete both tasks.
- The `-r` option is essential when copying directories recursively to ensure all contents are included.

## Example

After successfully executing these tasks, you should observe a new file named `zshrc-copy` and a directory named `Code` residing on your Desktop. You can easily verify the successful copy operation by listing the contents of your Desktop using the following command:

```bash
ls -l ~/Desktop
```

Sample Output:

```plaintext
drwxr-xr-x 2 labex labex    6 Jan 19 09:25 Code
-rwxr-xr-x 1 labex labex  636 Jan 15 10:47 code.desktop
-rwxr-xr-x 1 labex labex  749 Jan 15 10:47 gedit.desktop
-rwxr-xr-x 1 labex labex 5106 Jan 15 10:47 gvim.desktop
-rwxr-xr-x 1 labex labex 8499 Jan 15 10:47 xfce4-terminal.desktop
-rw-r--r-- 1 labex labex 4752 Jan 19 09:25 zshrc-copy
```

# Rename Files and Directories

Having mastered copying, the next crucial skill is renaming files and directories. In this step, you will utilize the `mv` command, a versatile tool that serves dual purposes: moving files and directories, and, as you will practice here, renaming them. We will now work with the copies you created in the previous step.

## Tasks

1. Rename the `~/Desktop/zshrc-copy` file to `~/Desktop/zshrc-move`.
2. Rename the `~/Desktop/Code` directory to `~/Desktop/Code-move`.

## Requirements

- Use the `mv` command exclusively for both renaming operations.
- Remember that the `mv` command's behavior for renaming is essentially moving an item to the same location but with a new name.

## Example

Upon completion of these renaming tasks, your Desktop should now display `zshrc-move` and `Code-move` instead of their previous names. Verify the changes by listing the contents of your Desktop once more:

```bash
ls -l ~/Desktop
```

Sample Output:

```plaintext
drwxr-xr-x 2 labex labex    6 Jan 19 09:25 Code-move
-rwxr-xr-x 1 labex labex  636 Jan 15 10:47 code.desktop
-rwxr-xr-x 1 labex labex  749 Jan 15 10:47 gedit.desktop
-rwxr-xr-x 1 labex labex 5106 Jan 15 10:47 gvim.desktop
-rwxr-xr-x 1 labex labex 8499 Jan 15 10:47 xfce4-terminal.desktop
-rw-r--r-- 1 labex labex 4752 Jan 19 09:25 zshrc-move
```

# Remove Files and Directories

Having learned to copy and rename, the final essential file management skill is removal. This step focuses on the `rm` command, used for deleting files and directories. It's crucial to exercise caution with `rm`, as deleted items are typically permanently removed from the file system. In this step, you will clean up the files and directories you've been working with on your Desktop.

## Tasks

1. Remove the `~/Desktop/zshrc-move` file.
2. Remove the `~/Desktop/Code-move` directory.

## Requirements

- Use the `rm` command to remove both the file and the directory.
- Be especially cautious when using `rm` with directories. You might need to use options like `-r` or `-R` for recursive removal of directories and their contents, but for this specific task, consider if it's necessary. (Hint: Consider the default behavior of `rm` when dealing with directories).

## Example

After successfully removing these items, neither `zshrc-move` nor `Code-move` should be visible on your Desktop anymore. Verify the removal by listing the Desktop's contents one last time:

```bash
ls -l ~/Desktop
```

Sample Output:

```plaintext
-rwxr-xr-x 1 labex labex  636 Jan 15 10:47 code.desktop
-rwxr-xr-x 1 labex labex  749 Jan 15 10:47 gedit.desktop
-rwxr-xr-x 1 labex labex 5106 Jan 15 10:47 gvim.desktop
-rwxr-xr-x 1 labex labex 8499 Jan 15 10:47 xfce4-terminal.desktop
```

