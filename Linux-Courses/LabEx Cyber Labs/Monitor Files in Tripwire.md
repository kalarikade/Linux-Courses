# Configure Tripwire Keys

Welcome to the lab. The `tripwire` package has been pre-installed in your environment to save time. Your first task is to configure the cryptographic keys that Tripwire uses to protect its database and configuration files. These keys ensure that an attacker cannot tamper with Tripwire's own files to hide their tracks.

1. First, verify that Tripwire is installed by checking its version.
    
    ```bash
    tripwire --version
    ```
    
    You should see output displaying the installed version of Tripwire.
    
    ```plaintext
    Tripwire(R) 2.4.3.7
    (i686-pc-linux-gnu)
    ...
    ```
    
2. Next, generate the cryptographic keys manually using `twadmin`. You need to create both a _site key_ and a _local key_. The site key protects the policy and configuration files, while the local key protects the database on the local machine.
    
    First, generate the site key:
    
    ```bash
    sudo twadmin --generate-keys --site-keyfile /etc/tripwire/site.key
    ```
    
    If the key file already exists, you'll be prompted to overwrite it. Type `y` and press Enter to confirm.
    
    When prompted for the site key passphrase, enter `labex-password` and press Enter. Then confirm the passphrase by entering it again.
    
    **Note**: The passphrase input is hidden for security - you won't see any characters as you type, but the system is still receiving your input.
    
3. Next, generate the local key:
    
    ```bash
    sudo twadmin --generate-keys --local-keyfile /etc/tripwire/$(hostname)-local.key
    ```
    
    Again, if prompted to overwrite an existing key file, type `y` and press Enter.
    
    When prompted for the local key passphrase, enter `labex-password` and press Enter. Confirm the passphrase by entering it again. Remember, the passphrase input is hidden.
    
    After you have entered the passphrases, both keys will be generated and saved in the `/etc/tripwire/` directory. The key generation process may take several minutes to complete.

# Create a Simplified Monitoring Policy

Before creating the initial database (the baseline), you need to configure which files and directories Tripwire should monitor. Due to compatibility issues with the default policy file that may cause segmentation faults during initialization, you will create a simplified policy that focuses on essential system files and your project directory.

1. First, create a backup of the original policy file.
    
    ```bash
    sudo cp /etc/tripwire/twpol.txt /etc/tripwire/twpol.txt.bak
    ```
    
2. Create a new simplified policy file that avoids problematic virtual filesystems.
    
    ```bash
    sudo nano /etc/tripwire/twpol-simple.txt
    ```
    
3. Add the following content to the file. This simplified policy monitors essential system files and your project directory while avoiding paths that may cause initialization errors:
    
    ```plaintext
    #
    # Simplified Tripwire Policy File for Lab Environment
    #
    
    @@section GLOBAL
    TWBIN = /usr/sbin;
    TWETC = /etc/tripwire;
    TWVAR = /var/lib/tripwire;
    
    @@section FS
    
    SEC_CRIT      = $(IgnoreNone)-SHa ;
    SEC_BIN       = $(ReadOnly) ;
    SEC_CONFIG    = $(Dynamic) ;
    SEC_LOG       = $(Growing) ;
    SEC_INVARIANT = +tpug ;
    SIG_LOW       = 33 ;
    SIG_MED       = 66 ;
    SIG_HI        = 100 ;
    
    (
      rulename = "Tripwire Binaries",
      severity = $(SIG_HI)
    )
    {
        $(TWBIN)/siggen         -> $(SEC_BIN) ;
        $(TWBIN)/tripwire       -> $(SEC_BIN) ;
        $(TWBIN)/twadmin        -> $(SEC_BIN) ;
        $(TWBIN)/twprint        -> $(SEC_BIN) ;
    }
    
    (
      rulename = "Tripwire Data Files",
      severity = $(SIG_HI)
    )
    {
        $(TWVAR)/$(HOSTNAME).twd    -> $(SEC_CONFIG) -i ;
        $(TWETC)/tw.pol             -> $(SEC_BIN) -i ;
        $(TWETC)/tw.cfg             -> $(SEC_BIN) -i ;
        $(TWETC)/$(HOSTNAME)-local.key  -> $(SEC_BIN) ;
        $(TWETC)/site.key           -> $(SEC_BIN) ;
        $(TWVAR)/report             -> $(SEC_CONFIG) (recurse=0) ;
    }
    
    (
      rulename = "Critical system files",
      severity = $(SIG_HI)
    )
    {
        /bin            -> $(SEC_BIN) ;
        /sbin           -> $(SEC_BIN) ;
        /etc/passwd     -> $(SEC_CONFIG) ;
        /etc/shadow     -> $(SEC_CONFIG) ;
    }
    
    (
      rulename = "Lab Project Files",
      severity = $(SIG_HI)
    )
    {
      /home/labex/project        -> $(SEC_BIN) ;
    }
    ```
    
4. Save the file and exit `nano` by pressing `Ctrl + O`, then `Enter`, and finally `Ctrl + X`.
    
5. Replace the original policy file with your simplified version.
    
    ```bash
    sudo cp /etc/tripwire/twpol-simple.txt /etc/tripwire/twpol.txt
    ```
    

You have now created a simplified monitoring policy that will work reliably in the lab environment. In the next step, you will use this policy to create the initial database.

# Initialize the Tripwire Database

With your simplified policy in place, you are now ready to create the Tripwire database. This database serves as the "known good" baseline snapshot of your system. Tripwire will compare the current state of your files against this baseline to detect any changes.

1. First, recreate the configuration file with your new site key to ensure compatibility.
    
    ```bash
    sudo twadmin --create-cfgfile --site-keyfile /etc/tripwire/site.key /etc/tripwire/twcfg.txt
    ```
    
    You will be prompted for your **site key** passphrase. Enter `labex-password` and press Enter.
    
2. Convert the simplified policy file into the signed binary format that Tripwire uses.
    
    ```bash
    sudo twadmin --create-polfile /etc/tripwire/twpol.txt
    ```
    
    Again, enter your **site key** passphrase (`labex-password`) when prompted.
    
3. Initialize the database. This command scans all files and directories specified in your policy and records their cryptographic signatures and attributes.
    
    ```bash
    sudo tripwire --init
    ```
    
    You will be prompted for your **local key** passphrase. Enter `labex-password` and press Enter.
    
4. During initialization, you may see a warning about the database file not existing initially:
    
    ```plaintext
    ### Warning: File system error.
    ### Filename: /var/lib/tripwire/68b1139d06fc790d9895e107.twd
    ### No such file or directory
    ### Continuing...
    ```
    
    This is normal and expected for the first initialization. The process will continue and create the database file.
    
5. Once the process is complete, verify that the database file has been created.
    
    ```bash
    ls -l /var/lib/tripwire/
    ```
    
    You should see a file named after your machine's hostname with a `.twd` extension (e.g., `68b1139d06fc790d9895e107.twd`). The presence of this file confirms that your baseline database has been successfully established.
    
    ```plaintext
    total 8
    -rw-r--r-- 1 root root 4164 Aug 29 11:00 68b1139d06fc790d9895e107.twd
    drwxr-xr-x 2 root root    6 Nov 11  2021 report
    ```
    

Your Tripwire system is now properly initialized with a baseline database. Any future changes to the monitored files will be detected when you run integrity checks.

# Detect and Report File Changes

Now that Tripwire has a baseline, you can test its ability to detect changes. You will create a new file in your project directory to see how Tripwire detects modifications to monitored areas.

1. First, create a new file in the `project` directory. This is a directory you explicitly added to the monitoring policy.
    
    ```bash
    touch ~/project/test_file.txt
    ```
    
2. With the change made, run an integrity check. Tripwire will scan the system and compare it against the baseline database.
    
    ```bash
    sudo tripwire --check
    ```
    
    You will be prompted for your **local key** passphrase. Enter `labex-password` and press Enter.
    
3. After the scan completes, Tripwire will generate a report and print a summary to the console. The summary will highlight the objects that were added or modified. You should see output similar to this:
    
    ```plaintext
    ===============================================================================
    Rule Summary:
    ===============================================================================
    
      Rule Name                       Severity Level    Added    Removed  Modified
      ---------                       --------------    -----    -------  --------
      Tripwire Binaries               100               0        0        0
      Critical system files           100               0        0        0
    * Tripwire Data Files             100               1        0        0
    * Lab Project Files               100               1        0        1
      (/home/labex/project)
    
    Total objects scanned:  16
    Total violations found:  3
    
    ===============================================================================
    Object Summary:
    ===============================================================================
    
    -------------------------------------------------------------------------------
    Rule Name: Lab Project Files (/home/labex/project)
    Severity Level: 100
    -------------------------------------------------------------------------------
    
    Added:
    "/home/labex/project/test_file.txt"
    
    Modified:
    "/home/labex/project"
    ```
    
4. The report shows that Tripwire detected:
    
    - **Added**: The new `test_file.txt` in your project directory
    - **Modified**: The `/home/labex/project` directory itself (because its contents changed)
    - **Tripwire Data Files**: The database file was also flagged as "added" because this is the first check since initialization

This output confirms that Tripwire has successfully detected the file system changes you made.

# Review the Integrity Report

The summary from the `--check` command provides a good overview, but for detailed analysis, you can view the full report file. These reports are stored in a binary format and must be read using the `twprint` utility.

1. First, list the contents of the report directory to find the name of the most recent report. Reports are timestamped in their filenames.
    
    ```bash
    sudo ls -lt /var/lib/tripwire/report/
    ```
    
    The output will list the report files, with the newest one at the top. Note its filename, which typically follows the format `hostname-YYYYMMDD-HHMMSS.twr`.
    
    ```plaintext
    total 4
    -rw------- 1 root root 3456 Aug 29 11:02 68b1139d06fc790d9895e107-20250829-110252.twr
    ```
    
2. Use the `twprint` command to view the report in a human-readable format. Replace `[REPORT_FILENAME]` with the actual filename you noted from the previous command.
    
    ```bash
    sudo twprint --print-report --twrfile /var/lib/tripwire/report/[REPORT_FILENAME]
    ```
    
    For example:
    
    ```bash
    sudo twprint --print-report --twrfile /var/lib/tripwire/report/68b1139d06fc790d9895e107-20250829-110252.twr
    ```
    
3. The detailed report shows the same information as the summary but with additional metadata:
    
    ```plaintext
    Report created on:            Fri Aug 29 11:02:52 2025
    Database last updated on:     Never
    
    Host name:                    68b1139d06fc790d9895e107
    Host IP address:              172.18.0.8
    Policy file used:             /etc/tripwire/tw.pol
    Configuration file used:      /etc/tripwire/tw.cfg
    Database file used:           /var/lib/tripwire/68b1139d06fc790d9895e107.twd
    Command line used:            tripwire --check
    ```
    
    The report provides comprehensive details about each change, including timestamps, file properties, and checksums. This level of detail is crucial for forensic analysis to understand exactly what changes occurred on your system.
    
4. You can also view a more detailed version with file attributes by using the `--verbose` flag:
    
    ```bash
    sudo twprint --print-report --verbose --twrfile /var/lib/tripwire/report/[REPORT_FILENAME]
    ```
    
    This will show additional information like file permissions, ownership, size, and various cryptographic hashes for each detected change.
    

Understanding how to read these reports is essential for system administrators to investigate potential security incidents and maintain system integrity.