# Exploring the Target Website

In this step, we'll examine the website we'll be testing and introduce the concept of brute-force attacks. Before we start using automated tools, it's important to understand how basic authentication systems work and what we're trying to accomplish.

Open the `Web 8080` tab in your lab environment. This will display a web browser interface where you'll see a simple login page with fields for entering a username and password. This is similar to login pages you encounter on many websites every day.

![Login page screenshot](https://file.labex.io/namespace/df87b950-1f37-4316-bc07-6537a1f2c481/cybersecurity/lab-using-hydra-to-crack-passwords/assets/20240923-14-04-44-wJXP8Mcm.png)

Let's try some basic login attempts to understand how the system responds:

1. First attempt:
    
    - Username: `test`
    - Password: `password123`
2. Second attempt:
    
    - Username: `admin`
    - Password: `admin`

After each attempt, you'll receive an "Invalid username or password" message. This generic response is intentional - it doesn't tell you whether the username exists or if just the password was wrong. This is called "security through obscurity" and helps prevent attackers from gathering information about valid accounts.

Notice that you can keep trying different combinations without any restrictions. In production systems, security measures like account lockouts or CAPTCHAs would typically be implemented after several failed attempts to prevent automated attacks. The absence of these protections in our lab environment makes it suitable for demonstrating brute-force techniques.

What you've just done manually - trying different username and password combinations - is essentially a manual version of a brute-force attack. In real attacks, hackers use automated tools to try thousands of combinations per second. This exercise shows why simple passwords like "password123" or "admin" are extremely vulnerable, and why systems need proper security measures to prevent such attacks.


# Examining the Password List

Now that we understand the basic concept of a brute-force attack, let's examine a list of common passwords that attackers might use. Before we proceed, it's important to know that brute-force attacks work by systematically trying all possible password combinations until the correct one is found. Using pre-compiled password lists makes this process much faster.

Switch back to the **Desktop Tab** in your lab environment. This is where we'll perform all our commands and view the results.

Open the Xfce Terminal on the Desktop by double-clicking its icon. The terminal is our primary tool for running commands in this lab.

![Xfce Terminal on Desktop](https://file.labex.io/namespace/df87b950-1f37-4316-bc07-6537a1f2c481/cybersecurity/lab-using-hydra-to-crack-passwords/assets/20240927-15-02-25-O9LEkNIo.png)

In real-world scenarios, attackers often use extensive lists of passwords from past data breaches. These lists contain millions of passwords that were exposed in security incidents. For our lab, we'll use a smaller list of common weak passwords found on the internet to demonstrate the concept. This list has already been prepared for you and is located at `/home/labex/project/500-worst-passwords.txt`.

Let's examine the contents of the file using the `head` command, which shows the first few lines of a file:

```bash
head -n 10 500-worst-passwords.txt
```

This command will display the first 10 passwords from the list. The `-n 10` option tells the command to show exactly 10 lines.

```plaintext
123456
password
12345678
1234
12345
dragon
qwerty
567sjej
mustang
letmein
```

These passwords are common choices that many people unfortunately still use. Notice how simple and predictable they are - mostly numeric sequences or common words. This is precisely why attackers often try these first in their attempts to gain unauthorized access.

By using a list like this, an attacker can greatly speed up their brute-force attack. Instead of trying every possible combination of characters (which could take years), they start with the most likely passwords, significantly increasing their chances of quick success. In cybersecurity, we call this a "dictionary attack" - a smarter version of brute-forcing that uses known passwords rather than random guesses.

# Setting Up Hydra

Now that we have our list of passwords, let's set up Hydra, the tool we'll use to automate our brute-force attack simulation. Hydra is a powerful password-cracking tool that helps security professionals test the strength of passwords by systematically trying different combinations.

First, we'll create a simple text file containing common usernames that we want to test. This is important because many systems use predictable default usernames that attackers might try first. Run these commands:

```bash
cd ~/project
echo -e "admin\nuser\nroot" > ~/project/usernames.txt
cat ~/project/usernames.txt
```

The first command (`cd ~/project`) ensures we're in the correct working directory. The second command creates a file called `usernames.txt` containing three common usernames (admin, user, and root), each on a new line. The third command displays the contents of the file so we can verify it was created correctly.

Now, let's make sure Hydra is installed. In a real-world scenario, you would typically install it yourself, but in this lab environment:

> Note: Free users can't connect to the internet, so Hydra is already pre-installed in the lab environment, you can skip this command. [Upgrade to a pro](https://labex.io/pricing?utm_source=labby) user to practice installing Hydra by yourself.

Pro Users Only

```bash
sudo apt-get update
sudo apt-get install hydra -y
```

Hydra works by taking lists of potential usernames and passwords, then attempting to authenticate with each combination against a target service. It supports many protocols including HTTP, FTP, SSH, and more. The tool is particularly useful for penetration testers to identify weak passwords in a system.

To verify Hydra is working correctly, we can check its version and available options with:

```bash
hydra -h
```

This command displays Hydra's help menu, showing all the available commands and options. You'll see information about different attack modes, supported protocols, and how to customize your password cracking attempts. This is useful for understanding what Hydra can do before we start using it.

# Using Hydra for Password Cracking

Now that we have Hydra installed and our password list ready, let's use it to simulate a brute-force attack on our practice website. Before we begin, it's important to understand that Hydra is an automated tool that systematically tries different username and password combinations against a login page - this is called a brute-force attack.

Run the following Hydra command:

```bash
hydra -L ~/project/usernames.txt -P ~/project/500-worst-passwords.txt localhost -s 8080 http-post-form "/:username=^USER^&password=^PASS^:Invalid username or password" -o ~/project/hydra_results.txt
```

Let's break down this command piece by piece to understand what each part does:

- `hydra`: This is the command to start the Hydra tool.
- `-L ~/project/usernames.txt`: Specifies the file containing potential usernames to try.
- `-P ~/project/500-worst-passwords.txt`: Points to the file with common passwords we'll test against each username.
- `localhost -s 8080`: The target web server address (localhost) and port (8080) we're testing.
- `"/:username=^USER^&password=^PASS^:Invalid username or password"`: This tells Hydra:
    - The login page URL (/)
    - How the login form fields are named (username and password)
    - What the error message looks like when login fails
- `-o ~/project/hydra_results.txt`: Saves all successful attempts to this output file.

Hydra will start running, and you'll see output as it tries each username and password combination. This process might take a few minutes as Hydra systematically works through all possible combinations in the files we provided.

![Hydra command execution output](https://file.labex.io/namespace/df87b950-1f37-4316-bc07-6537a1f2c481/cybersecurity/lab-using-hydra-to-crack-passwords/assets/20240923-16-51-16-KVRVDQDH.png)

After Hydra finishes, we can check the results to see if any combinations worked:

```bash
cat ~/project/hydra_results.txt
```

![Hydra results output](https://file.labex.io/namespace/df87b950-1f37-4316-bc07-6537a1f2c481/cybersecurity/lab-using-hydra-to-crack-passwords/assets/20240923-16-52-29-cYnKmbGJ.png)

Successful logins will appear as lines like "login: [username] password: [password] found". These represent credentials that would grant access to the system.

This exercise demonstrates several important security concepts:

- Hydra tested hundreds of password combinations in minutes, showing how quickly automated tools can work.
- The successful logins prove that weak or common passwords are easily cracked.
- In real-world scenarios, attackers can use dictionaries with millions of passwords, making strong, unique passwords essential.
- The difference between manual attempts (Step 1) and automated tools highlights why password policies matter.

Notice how the tool's efficiency makes weak credentials extremely vulnerable. This is why security professionals emphasize using complex passwords and implementing account lockout policies.

