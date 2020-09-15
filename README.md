# Deployment Roadmap

## Initial Server Setup with Ubuntu 18.04 (based on [DigitalOcean's guide](https://www.digitalocean.com/community/tutorials/initial-server-setup-with-ubuntu-18-04))

This will increase the security and usability of your server and will give you a solid foundation for subsequent actions.

### Step 1 — Logging in as Root

To log into your server, you will need to know your server’s public IP address. You will also need the password or, if you installed an SSH key for authentication, the private key for the root user’s account.

If you are not already connected to your server, go ahead and log in as the root user using the following command (substitute the 'your_server_ip' word with your server’s public IP address):

```shell
$ ssh root@your_server_ip
```

Accept the warning about host authenticity if it appears. If you are using password authentication, provide your root password to log in. If you are using an SSH key that is passphrase protected, you may be prompted to enter the passphrase the first time you use the key each session. If this is your first time logging into the server with a password, you may also be prompted to change the root password.

### Step 2 — Creating a New User

Once you are logged in as root, we’re prepared to add the new user account that we will use to log in from now on.
(substitute the 'your_username' word with your desired username)

```shell
$ adduser your_username
```

You will be asked a few questions, starting with the account password.

Enter a strong password and, optionally, fill in any of the additional information if you would like. This is not required and you can just hit `ENTER` in any field you wish to skip.

### Step 3 — Granting Administrative Privileges

Now, we have a new user account with regular account privileges. However, we may sometimes need to do administrative tasks.

To avoid having to log out of our normal user and log back in as the root account, we can set up what is known as “superuser” or root privileges for our normal account. This will allow our normal user to run commands with administrative privileges by putting the word `sudo` before each command.

To add these privileges to our new user, we need to add the new user to the sudo group. By default, on Ubuntu 18.04, users who belong to the sudo group are allowed to use the `sudo` command.

As root, run this command to add your new user to the sudo group (substitute the 'your_username' word with your new user):

```shell
$ usermod -aG sudo your_username
```

Now, when logged in as your regular user, you can type `sudo` before commands to perform actions with superuser privileges.

### Step 4 — Setting Up a Basic Firewall

Ubuntu 18.04 servers can use the UFW firewall to make sure only connections to certain services are allowed. We can set up a basic firewall very easily using this application.

Different applications can register their profiles with UFW upon installation. These profiles allow UFW to manage these applications by name. OpenSSH, the service allowing us to connect to our server now, has a profile registered with UFW.

You can see this by typing:

```shell
$ ufw app list
```

```shell
# Output

Available applications:
  OpenSSH
```

We need to make sure that the firewall allows SSH connections so that we can log back in next time. We can allow these connections by typing:

```shell
$ ufw allow OpenSSH
```

Afterwards, we can enable the firewall by typing:

```shell
$ ufw enable
```

Type “y” and press `ENTER` to proceed. You can see that SSH connections are still allowed by typing:

```shell
$ ufw status
```

```shell
# Output

Status: active

To                         Action      From
--                         ------      ----
OpenSSH                    ALLOW       Anywhere
OpenSSH (v6)               ALLOW       Anywhere (v6)
```

As the **firewall is currently blocking all connections except for SSH**, if you install and configure additional services, you will need to adjust the firewall settings to allow acceptable traffic in.

### Step 5 — Enabling External Access for Your Regular User

Now that we have a regular user for daily use, we need to make sure we can SSH into the account directly.

The process for configuring SSH access for your new user depends on whether your server’s root account uses a password or SSH keys for authentication.

#### If the Root Account Uses Password Authentication

If you logged in to your root account using a password, then password authentication is enabled for SSH. You can SSH to your new user account by opening up a new terminal session and using SSH with your new username:

```shell
$ ssh your_username@your_server_ip
```

After entering your regular user’s password, you will be logged in. Remember, if you need to run a command with administrative privileges, type `sudo` before it. You will be prompted for your regular user password when using `sudo` for the first time each session (and periodically afterwards).

To enhance your server’s security, **we strongly recommend setting up SSH keys instead of using password authentication**.
(Follow DigitalOcean's guide on [setting up SSH keys on Ubuntu 18.04](https://www.digitalocean.com/community/tutorials/how-to-set-up-ssh-keys-on-ubuntu-1804) to learn how to configure key-based authentication.)

#### If the Root Account Uses SSH Key Authentication

If you logged in to your root account using SSH keys, then password authentication is disabled for SSH. You will need to add a copy of your local public key to the new user’s `~/.ssh/authorized_keys` file to log in successfully.

Since your public key is already in the root account’s `~/.ssh/authorized_keys` file on the server, we can copy that file and directory structure to our new user account in our existing session.

The simplest way to copy the files with the correct ownership and permissions is with the `rsync` command. This will copy the root user’s `.ssh` directory, preserve the permissions, and modify the file owners, all in a single command. Make sure to change the highlighted portions of the command below to match your regular user’s name:

```shell
$ rsync --archive --chown=your_username:your_username ~/.ssh /home/your_username
```

Now, open up a new terminal session and using SSH with your new username:

```shell
$ ssh your_username@your_server_ip
```

You should be logged in to the new user account without using a password. Remember, if you need to run a command with administrative privileges, type `sudo` before it. You will be prompted for your regular user password when using sudo for the first time each session (and periodically afterwards).

## How To Install Docker on Ubuntu 18.04

The Docker installation package available in the official Ubuntu repository may not be the latest version. To ensure we get the latest version, we’ll install Docker from the official Docker repository. To do that, we’ll add a new package source, add the GPG key from Docker to ensure the downloads are valid, and then install the package.

First, update your existing list of packages:

```shell
$ sudo apt update
```

Next, install a few prerequisite packages which let apt use packages over HTTPS:

```shell
$ sudo apt install apt-transport-https ca-certificates curl software-properties-common
```

Then add the GPG key for the official Docker repository to your system:

```shell
$ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
```

Add the Docker repository to APT sources:

```shell
$ sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu bionic stable"
```

Next, update the package database with the Docker packages from the newly added repo:

```shell
$ sudo apt update
```

Make sure you are about to install from the Docker repo instead of the default Ubuntu repo:

```shell
$ apt-cache policy docker-ce
```

You’ll see output like this, although the version number for Docker may be different:

```shell
docker-ce:
  Installed: (none)
  Candidate: 18.03.1~ce~3-0~ubuntu
  Version table:
     18.03.1~ce~3-0~ubuntu 500
        500 https://download.docker.com/linux/ubuntu bionic/stable amd64 Packages
```

Notice that `docker-ce` is not installed, but the candidate for installation is from the Docker repository for Ubuntu 18.04 (`bionic`).

Finally, install Docker:

```shell
$ sudo apt install docker-ce
```

Docker should now be installed, the daemon started, and the process enabled to start on boot. Check that it’s running:

```shell
$ sudo systemctl status docker
```

The output should be similar to the following, showing that the service is active and running:

```shell
● docker.service - Docker Application Container Engine
   Loaded: loaded (/lib/systemd/system/docker.service; enabled; vendor preset: enabled)
   Active: active (running) since Thu 2018-07-05 15:08:39 UTC; 2min 55s ago
     Docs: https://docs.docker.com
 Main PID: 10096 (dockerd)
    Tasks: 16
   CGroup: /system.slice/docker.service
           ├─10096 /usr/bin/dockerd -H fd://
           └─10113 docker-containerd --config /var/run/docker/containerd/containerd.toml
```

Installing Docker now gives you not just the Docker service (daemon) but also the `docker` command line utility, or the Docker client.

#### Executing the Docker Command Without Sudo (Optional)

By default, the `docker` command can only be run the **root** user or by a user in the **docker** group, which is automatically created during Docker’s installation process. If you attempt to run the `docker` command without prefixing it with `sudo` or without being in the **docker** group, you’ll get an output like this:

```shell
docker: Cannot connect to the Docker daemon. Is the docker daemon running on this host?.
See 'docker run --help'.
```

If you want to avoid typing `sudo` whenever you run the `docker` command, add your username to the **docker** group:

```shell
$ sudo usermod -aG docker ${USER}
```

To apply the new group membership, log out of the server and back in, or type the following:

```shell
$ su - ${USER}
```

You will be prompted to enter your user’s password to continue.

Confirm that your user is now added to the **docker** group by typing:

```shell
$ id -nG
```

```shell
# Output

your_username sudo docker
```

If you need to add a user to the **docker** group that you’re not logged in as, declare that username explicitly using:

```shell
$ sudo usermod -aG docker username
```

## How To Install Docker Compose on Ubuntu 18.04

Although we can install Docker Compose from the official Ubuntu repositories, it is several minor version behind the latest release, so we’ll install Docker Compose from the Docker’s GitHub repository. The command below is slightly different than the one you’ll find on the [Releases](https://github.com/docker/compose/releases) page. By using the `-o` flag to specify the output file first rather than redirecting the output, this syntax avoids running into a permission denied error caused when using `sudo`.

We’ll check the [current release](https://github.com/docker/compose/releases) and if necessary, update it in the command below:

```shell
$ sudo curl -L https://github.com/docker/compose/releases/download/1.21.2/docker-compose-`uname -s`-`uname -m` -o /usr/local/bin/docker-compose
```

Next we’ll set the permissions:

```shell
$ sudo chmod +x /usr/local/bin/docker-compose
```

Then we’ll verify that the installation was successful by checking the version:

```shell
$ docker-compose --version
```

This will print out the version we installed:

```shell
# Output

docker-compose version 1.21.2, build a133471
```
