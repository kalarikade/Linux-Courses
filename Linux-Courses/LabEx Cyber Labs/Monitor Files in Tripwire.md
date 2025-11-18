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