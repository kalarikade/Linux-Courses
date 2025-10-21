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