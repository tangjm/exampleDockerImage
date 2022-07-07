# Build and run a Docker container image on an EC2 instance [^1]

We will first configure and launch an EC2 instance. Then we use Docker to build and run an image created from a Dockerfile inside a public GitHub repository before pushing the image to the Docker Hub container registry.
### Prerequisites

- [x] Docker account
- [x] AWS Account
- [x] PuTTY on Windows or OpenSSH on MacOS and Linux (SSH is probably already installed)

### Setup EC2 instance

If you have access to AWS Academy, login to [AWS Academy](https://www.awsacademy.com/LMS_Login) and start the Sandbox Environment to access AWS EC2.

**For Windows Systems**:
1. Create a key pair using the .ppk private key format, this works with PuTTY
2. Create a security group with inbound rules set to allow SSH from your IP address and outbound rules to allow HTTP(S) to the internet (0.0.0.0/0).[^2]
3. Launch an EC2 instance with the key pair and security group you created.
4. Use PuTTY to connect to your EC2 instance. [^3]
   Two things to do: 
    1. Set the host name of your EC2 instance
        - In the 'Category' pane, click 'Session' and set Host Name to "ec2-user" + "@" + Public IPv4 DNS of EC2 instance. 
        - For example: ec2-user@ec2-54-91-206-43.compute-1.amazonaws.com
        (The Public DNS should appear once your EC2 instance is in the "running" state.)
    1. Open the private key paired with the EC2 instance's public key. 
       - Open 'Connection', then 'SSH' and click 'Auth':
         Open the private key you downloaded when you created the key pair.

From this point onwards, we will be working inside the PuTTY client.


**For MacOS and Linux**
If you're on MacOS or Linux, using OpenSSH:
 1. When creating your AWS instance key-pair, make sure you choose .pem for the private key format, this is the one     that works with OpenSSH
 2. Open a terminal session
 3. Make sure the private key from the key pair you created is saved to your `~/.ssh` folder
 4. Change the permissions of the private key to read-only by your current user using `chmod 400 key-pair.pem`
 5. Type `ssh -i /path/to/key-pair.pem  ec2-user@[your EC2 instance public DNS name]` and press enter. For example:

  `ssh -i ~/.ssh/key-pair-name.pem ec2-user@ec2-12-123-12-12.eu-west-2.compute.amazonaws.com`

   N.B. If you didn't use an Amazon AMI, then the username will most likely be `ubuntu` for an Ubuntu AMI, or potentially another name if you used a different instance.

 6. You should get a response asking you if you'd like to continue, select yes.
 7. You should now be connected.
 8. If you got an error because OpenSSH isn't installed, install OpenSSH through your package manager (Brew for Mac; APT, DNF, etc for Linux).


### 1. Setup

Update packages on EC2 instance

```bash
sudo yum update -y
```

Install Docker

```bash
sudo yum install docker -y
```

Install Git

```bash
sudo yum install git -y
```

### 2. Build a container image from a Dockerfile

Docker needs to be running before we can build an image from a Dockerfile.

Check if the docker daemon is running. A daemon is just a program that runs continuously and exists for the purpose of handling periodic service requests that a computer system expects to receive.

```bash
sudo service docker status
```

Start docker if docker is not running with one of these two commands:

```bash
sudo service docker start
```

```bash
sudo systemctl start docker
```

Build a container image from a Git repository. Either use the one I made or create your own.

```bash
sudo docker build -t my-app:v1 https://github.com/tangjm/testDockerOnEC2#main
```

The argument to `docker build` is the build context. This is a set of files used for generating your image. They should reside within the same directory as your Dockerfile. In this case, we're passing a URL to use a remote directory for our build context. The suffix `#main` indicates that we want the contents of our respository on our main branch. See [docs](https://docs.docker.com/engine/reference/commandline/build/#git-repositories) for more on this.

The build context is first sent to the Docker daemon. Then the instructions within your Dockerfile are executed in order, adding an immutable layer each time. Finally, the container image is built.

Check that the image was built successfully

```bash
sudo docker images
```

Try out the `docker tag` command by creating another tag

```bash
sudo docker tag my-app:v1 second-app:v1
```

Run the image in a container. This should print "Hello world!" to the terminal.

```bash
sudo docker run my-app:v1
```

### 3. Pushing images to container registries

To push to Docker Hub, you will need a docker account, register for one [here](https://hub.docker.com/)

Before pushing your image, make sure to login first

```bash
sudo docker login
```

Then tag the image you want to push ensuring that the repository name of the new tag has the prefix `your_username/`. Container image names have the format `hostname/repository:tag` and we are simply renaming the `repository` part to meet the formatting requirements of Docker Hub.

```bash
sudo docker tag my-app:v1 <your_docker_username>/my-app:v1
```

For example, for me, it would be this:

```bash
sudo docker tag my-app:v1 tangjm5/my-app:v1
```

Finally, push the tagged image:

```bash
sudo docker push <your_username>/my-app:v1
```

Verify the push by going back to [docker hub](https://hub.docker.com/).

A new repo should have appeared!

[^1]: Follow the EC2 section from [this guide](https://www.jenkins.io/doc/tutorials/tutorial-for-installing-jenkins-on-AWS/) if you have trouble setting up and connecting to an EC2 instance

[^2]: We allow SSH from your IP address so that you can connect to your EC2 instance via SSH using PuTTY. The outbound rules are there so that you can access the internet to install Docker and Git.

[^3]: [Detailed instructions here](https://www.jenkins.io/doc/tutorials/tutorial-for-installing-jenkins-on-AWS/#using-putty-to-connect-to-your-instance)
