# Introduction

In this lab, you will learn fundamental techniques for identifying potential malware indicators on a Linux system. Through a series of hands-on exercises, you will use standard command-line utilities to investigate common signs of a compromise, gaining practical skills in system monitoring and basic forensic analysis.

You will begin by observing suspicious process activity, learning how to spot unusual CPU and memory spikes that can signal a malicious process. Next, you will focus on the file system to detect unexpected file creations and modifications, a key indicator of unauthorized activity. Finally, you will practice analyzing system logs to find anomalous entries that could point to a security breach.

This is a Guided Lab, which provides step-by-step instructions to help you learn and practice. Follow the instructions carefully to complete each step and gain hands-on experience. Historical data shows that this is a beginner level lab with a 89% completion rate. It has received a 100% positive review rate from learners.

![labby](https://labex.io/images/labby/avatar-new.png)Labby

Great! Let me prepare the lab VM for you. While it's being set up, you can start reviewing the first step of this lab.

![labby](https://labex.io/images/labby/avatar-new.png)Labby

# Observe Suspicious Process Activity for CPU and Memory Spikes

In this step, you will learn how to use standard Linux utilities to monitor system processes for unusual spikes in CPU and memory usage. A sudden, high consumption of resources is a common indicator of malicious software (malware), a compromised process, or a misbehaving application. We will use the `top` command for real-time monitoring and the `stress` tool to safely simulate a high-load scenario. All commands will be run from your `~/project` directory.

First, you need to install the `stress` utility, as it is not included in the base system. We'll use the `apt-get` package manager. The first command updates your package list, and the second installs `stress`.

```bash
sudo apt-get update
sudo apt-get install stress -y
```

Once the installation is complete, let's establish a baseline by viewing the current processes on the system. The `top` command provides a real-time, dynamic view of the running system.

```bash
top
```

You will see a full-screen interface that continuously updates. Pay attention to the columns `%CPU` (CPU usage) and `%MEM` (memory usage). Under normal conditions, these values should be relatively low for most processes. Press the `q` key to exit `top` and return to your command prompt.

```plaintext
top - 16:20:42 up 10 min,  1 user,  load average: 0.00, 0.01, 0.00
Tasks: 100 total,   1 running,  99 sleeping,   0 stopped,   0 zombie
%Cpu(s):  0.1 us,  0.1 sy,  0.0 ni, 99.8 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
MiB Mem :   1987.2 total,    845.5 free,    450.1 used,    691.6 buff/cache
MiB Swap:      0.0 total,      0.0 free,      0.0 used.   1355.2 avail Mem

    PID USER      PR  NI    VIRT    RES    SHR S  %CPU  %MEM     TIME+ COMMAND
      1 root      20   0  169404  13120   8484 S   0.0   0.6   0:01.55 systemd
      2 root      20   0       0      0      0 S   0.0   0.0   0:00.00 kthreadd
...
```

Now, let's simulate a suspicious process that consumes a high amount of CPU. We'll use `stress` to start one worker that will perform CPU-intensive calculations for 60 seconds. The `&` at the end runs the command in the background, allowing you to continue using the terminal.

```bash
stress --cpu 1 --timeout 60 &
```

You will see a process ID (PID) for the background job. Now, quickly run `top` again to observe the effect.

```bash
top
```

This time, you should see the `stress` process at or near the top of the list, with its `%CPU` value close to `100.0`. This is exactly what you would look for when hunting for a process that is hogging the CPU. After 60 seconds, the `stress` process will automatically terminate. Press `q` to exit `top`.

Next, let's simulate a process that consumes a large amount of memory. This command will start one worker that allocates 256 Megabytes of memory and holds it for 60 seconds.

```bash
stress --vm 1 --vm-bytes 256M --timeout 60 &
```

Again, run `top` immediately to see the impact.

```bash
top
```

In the `top` output, find the `stress` process. This time, its `%CPU` will be low, but its `%MEM` value will be significantly higher than other processes, demonstrating a memory leak or a memory-intensive attack. This technique helps you identify anomalous resource consumption that warrants further investigation. Press `q` to exit.