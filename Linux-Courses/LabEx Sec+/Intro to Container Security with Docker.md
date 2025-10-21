# Introduction

Welcome to this hands-on lab on the basics of container security with Docker. Docker is a powerful platform for developing, shipping, and running applications in containers. While containers provide process isolation, securing them is crucial for protecting your applications and the host system.

In this lab, you will explore Docker container security fundamentals. You will first verify that Docker is properly configured in the LabEx environment. Then, you will run a basic Nginx web server container, inspect its running processes, and examine its resource consumption. Finally, you will learn how to apply fundamental security controls by setting resource limits on a container.

By the end of this lab, you will have a practical understanding of the initial steps involved in securing a Docker environment.

This is a Guided Lab, which provides step-by-step instructions to help you learn and practice. Follow the instructions carefully to complete each step and gain hands-on experience. Historical data shows that this is a beginner level lab with a 100% completion rate. It has received a 100% positive review rate from learners.


# Verify Docker Installation and Setup

In this step, you will verify that Docker is already installed in the LabEx environment and ensure you have the proper permissions to use it.

The LabEx virtual machine comes with Docker pre-installed. Let's first check the Docker version to confirm it's available.

```bash
docker --version
```

You should see an output similar to this, confirming that Docker is installed correctly:

```plaintext
Docker version 20.10.21, build 20.10.21-0ubuntu1~22.04.3
```

Now let's check if the Docker daemon is running:

```bash
sudo systemctl status docker
```

This command will show you the status of the Docker service. You should see that it's active and running.

To run Docker commands without `sudo`, your user needs to be in the `docker` group. In the LabEx environment, your user is already configured with the proper permissions. Let's verify this:

```bash
groups $USER
```

You should see `docker` in the output, confirming that your user has the necessary permissions:

```plaintext
labex : labex sudo docker ssl-cert
```

Since your user is already in the docker group, you can run Docker commands directly without needing `sudo`.

Let's test that Docker is working properly by running a simple command:

```bash
docker info
```

This command will display system-wide information about Docker, confirming that you can communicate with the Docker daemon successfully.

---
# Run a Basic Web Server Container

In this step, you will run your first container. We will use the official `nginx` image, which provides a lightweight and high-performance web server.

**To run a container, you use the `docker run` command. We will use a few common flags:

- `-d`: Runs the container in detached mode (in the background).
- `-p 8080:80`: Maps port 8080 on the host to port 80 inside the container. This allows you to access the Nginx server via `localhost:8080`.
- `--name webserver`: Assigns a memorable name to your container.
- `nginx`: The name of the image to use.

Execute the following command to start the Nginx container:

```bash
docker run -d -p 8080:80 --name webserver nginx
```

Docker will first check if the `nginx` image is available locally. If not, it will automatically download it from Docker Hub and then start the container. You can see the container is running using the `docker ps` command.

```bash
docker ps
```

The output will show your `webserver` container, its ID, the image it's based on, and the port mapping.

```plaintext
CONTAINER ID   IMAGE     COMMAND                  CREATED          STATUS          PORTS                  NAMES
a1b2c3d4e5f6   nginx     "/docker-entrypoint.…"   10 seconds ago   Up 9 seconds    0.0.0.0:8080->80/tcp   webserver
```

Now, let's test if the web server is working. Use `curl` to send a request to the server on the mapped port.

```bash
curl localhost:8080
```

You should see the default Nginx welcome page HTML as output, which confirms your container is running and accessible.

```html
<!DOCTYPE html>
<html>
  <head>
    <title>Welcome to nginx!</title>
    <style>
      html {
        color-scheme: light dark;
      }
      body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
      }
    </style>
  </head>
  <body>
    <h1>Welcome to nginx!</h1>
    <p>
      If you see this page, the nginx web server is successfully installed and
      working. For further configuration, refer to the
      <a href="http://nginx.org/">nginx.org</a> documentation.
    </p>

    <p><em>Thank you for using nginx.</em></p>
  </body>
</html>
```

---
# Examine Container Processes and Resource Usage

In this step, you will learn how to inspect the processes running inside a container and monitor its resource consumption. This is a fundamental skill for debugging and understanding how your containerized application behaves.

A key feature of containers is process isolation. From the host's perspective, you can see what's running inside a container without having to `exec` into it. The `docker top` command allows you to view the main processes of a container.

Run the following command to see the processes inside your `webserver` container:

```bash
docker top webserver
```

The output shows the Nginx master process running as root and its worker processes running as `systemd+` (a system user for improved security). Notice that there are multiple worker processes, which is typical for Nginx to handle concurrent requests efficiently.

```plaintext
UID                 PID                 PPID                C                   STIME               TTY                 TIME                CMD
root                4098                4078                0                   14:34               ?                   00:00:00            nginx: master process nginx -g daemon off;
systemd+            4161                4098                0                   14:34               ?                   00:00:00            nginx: worker process
systemd+            4162                4098                0                   14:34               ?                   00:00:00            nginx: worker process
```

Next, let's look at the container's resource usage. The `docker stats` command provides a live stream of a container's CPU, memory, network, and disk I/O. To get a static snapshot for this lab, we'll use the `--no-stream` flag.

```bash
docker stats --no-stream webserver
```

This command will display a table with the current resource metrics for the `webserver` container.

```plaintext
CONTAINER ID   NAME        CPU %     MEM USAGE / LIMIT     MEM %     NET I/O           BLOCK I/O     PIDS
bda2386621a1   webserver   0.00%     3.125MiB / 3.505GiB   0.09%     6.31kB / 1.27kB   0B / 12.3kB   3
```

From this output, you can see that the container is using very little CPU (0.00%), about 3.1 MiB of memory out of the available 3.5 GiB system memory (0.09%), and has 3 running processes (the Nginx master and 2 worker processes). This information is vital for identifying performance bottlenecks or potential resource leaks in your applications.

---
# Inspect Container Security Properties such as Resource Limits

In this step, you will explore one of the most basic but critical container security features: resource limits. By default, a container can use as much of the host's CPU and memory as it wants. Setting limits prevents a single container from consuming all available resources and impacting other containers or the host itself.

First, let's inspect the configuration of the `webserver` container we created earlier. The `docker inspect` command provides a detailed JSON output with all the container's settings.

```bash
docker inspect webserver
```

The output is very long. You can use the `--format` flag to extract specific fields. Let's check if any memory limit is set.

```bash
docker inspect --format='{{.HostConfig.Memory}}' webserver
```

The output will be `0`, which means there is no memory limit.

```plaintext
0
```

Now, let's create a new container with a memory limit. First, you need to stop and remove the existing container, as we will reuse the port mapping.

```bash
docker stop webserver
docker rm webserver
```

Next, run a new Nginx container, but this time add the `--memory="100m"` flag to limit its memory usage to 100 megabytes. We'll also give it a new name.

```bash
docker run -d -p 8080:80 --name webserver-limited --memory="100m" nginx
```

Now, inspect the memory limit of this new container.

```bash
docker inspect --format='{{.HostConfig.Memory}}' webserver-limited
```

This time, the output will be the memory limit in bytes (100 × 1024 × 1024).

```plaintext
104857600
```

You have successfully applied a resource constraint to a container, which is a foundational step in container security and resource management.

