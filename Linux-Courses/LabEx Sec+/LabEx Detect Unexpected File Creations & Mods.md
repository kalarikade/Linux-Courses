# Detect Unexpected File Creations and Modifications

In this step, you will learn how to detect suspicious file system activity, such as the unexpected creation or modification of files. Attackers often create or alter files to establish persistence, store malicious payloads, or tamper with system configurations. We will use two powerful command-line tools: `find` to search for files based on their modification times, and `inotifywait` to perform real-time monitoring of directory events. All operations will take place in your `~/project` directory.

First, let's use the `find` command to locate files that have been recently modified. This is useful for periodic security audits. Let's create a sample configuration file to monitor.

```bash
echo "SERVER_IP=192.168.1.1" > ~/project/app.conf
```

The `find` command can search for files based on various criteria. We'll use the `-mmin` flag, which stands for "modified minutes ago". The following command will find all files in the current directory (`.`) that were modified within the last 5 minutes.

```bash
find . -mmin -5
```

The output will list the file you just created.

```plaintext
./app.conf
```

Now, let's simulate an attacker modifying this configuration file.

```bash
echo "MALICIOUS_PAYLOAD=..." >> ~/project/app.conf
```

If you run the `find` command again, it will once again show `app.conf` because its modification time has been updated. This is a simple way to spot recent changes. Now, let's simulate an attacker dropping a new file, a common tactic.

```bash
touch ~/project/hidden_script.sh
```

Run the `find` command one more time.

```bash
find . -mmin -5
```

Now it lists both the modified file and the newly created one, demonstrating how you can quickly identify recent file system activity.

```plaintext
./app.conf
./hidden_script.sh
```

While `find` is useful for audits, it doesn't provide real-time alerts. For that, we can use `inotify-tools`. First, you must install the package.

```bash
sudo apt-get update
sudo apt-get install -y inotify-tools
```

The core command is `inotifywait`, which can monitor a file or directory for specific events. Let's run it in monitor mode (`-m`) on our `~/project` directory. The `&` will run it as a background process so you can continue to use your terminal.

```bash
inotifywait -m ~/project &
```

The command is now silently watching for changes. Let's perform a few actions to trigger it. First, create a file.

```bash
echo "new file" > ~/project/new_file.txt
```

You will immediately see output from `inotifywait` reporting the `CREATE` event.

```plaintext
/home/labex/project/ CREATE new_file.txt
```

Now, modify the file.

```bash
echo "modified" >> ~/project/new_file.txt
```

The tool reports a `MODIFY` event.

```plaintext
/home/labex/project/ MODIFY new_file.txt
```

Finally, delete the file.

```bash
rm ~/project/new_file.txt
```

You will see a `DELETE` event. This real-time feedback is invaluable for a Security Operations Center (SOC) or an automated intrusion detection system.

```plaintext
/home/labex/project/ DELETE new_file.txt
```

To stop the background monitoring process, you can use the `pkill` command.

```bash
pkill inotifywait
```

You have successfully used both `find` and `inotifywait` to detect file system changes.