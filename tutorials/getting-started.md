---

layout: wide
title: "Getting Started With Eris"

---

# Get Started With Eris In Three Easy Steps

There are three steps need to get moving with Eris:

1. **Install** the Eris platform.
2. **Roll** your own Blockchain in seconds.
3. **Build** (and run) your distributed application using smart contract templates and a simple, web-based user interface.

# Step 1: Install the Eris Platform

**Dependencies**: `eris` has 1 dependency: [Docker](https://www.docker.com/). Docker is a run anywhere container solution which makes development, deployment, testing, and running of distributed applications a breeze.

Currently we consider the most workable setup to be (what our tests consider authoritative). We are working steadily toward making eris available for a wide variety of host environments.

* HOST_OS = {{ site.data.coding["os"].authoritative }}
* DOCKER = {{ site.data.coding["docker"].authoritative }}

## [Install Docker](http://docs.docker.com/installation/).

Installation requires that Docker be installed. Please see the [Docker](https://docs.docker.com/installation/) documentation for how to install.

At the current time, `eris` requires `docker` >= {{ site.data.coding["docker"].minimum }}. You can check your docker version with `docker version`. We do not test against older versions of docker. `eris` may or may not work against earlier versions of docker and we can make no guarantees of usability there.

### Docker Installation for Linux

Follow the link above for the official way to install Docker on your Linux box.

**Essential**! After you install docker, then you must make sure that the user you are using to develop with `eris` has access to the docker socket (which is accessible via the docker Linux usergroup). When you are logged in as the user you can do this:

```bash
sudo usermod -a -G docker $USER
```

That command will add the current user to the docker group which will mean that docker will not need to be called from `sudo`.

After you run that command, then please log out of the current shell and open a new shell. After that `eris` will then be able to connect to docker.

The above command *may* be skipped, but if you do, then you will need to run all `eris` commands either as the root user or using `sudo`.

**Note** If you are using a Linux or a Mac OSX box, please **make sure** you are installing Docker for the 64 bit architecture.

### Docker Installation for OSX & Windows

If you are on OSX or Windows, the [Docker Toolbox](https://www.docker.com/toolbox) is a graphical way of installing Docker, Virtualbox and Docker Machine. The Toolbox will build all of the functionality which Docker requires so that `eris` is able to connect into the Docker daemon.

**Note** that the Docker Toolbox will install VirtualBox by default. Because Docker runs by connecting into a **Linux** kernel, Docker only runs on Linux OS's at this time. While Docker is reportedly working on native solutions for other operating systems, at this time a Linux operating system is required.

By default the Docker Toolbox will use VirtualBox to create a very minimal Linux virtual machine with the Docker daemon preinstalled. The toolbox will install Docker Machine (`docker-machine`) which is used to start, stop, reboot, remove, etc. virtual machines via VirtualBox's API.

`eris` has been built to be able to connect into the Docker daemon running within a VirtualBox minimal Linux virtual machine by default and without any effort on your part (other than installing the Docker Toolbox).

If VirtualBox gives you trouble, then use [docker machine's plugins](https://github.com/docker/machine/blob/master/docs/AVAILABLE_DRIVER_PLUGINS.md) and create your `eris` machine [in the cloud](/tutorials/advanced/chain-deploying/).

### OSX Only

If you're a brew and brew cask user then:

```bash
brew cask install virtualbox
brew install docker docker-machine
```

### Windows Only

**N.B.** No matter whether you follow the below or not, you'll want to run `eris` commands either from `git bash` or from the `Docker Quickstart Terminal` (which is really just `git bash`) window. 

If you prefer to use the `cmd` as a terminal, you still can, every command should work as expected, though all the tutorials will assume that you are using the `Docker Quickstart Terminal` and are structured to support **only** that environment.

If you're a [Chocolatey](https://chocolatey.org) user then:

```bash
choco install virtualbox docker docker-machine
```

### All Platforms

Make sure that everything is set up with Docker by running:

```bash
docker version
```

**Note** you will need to make sure that you perform the above command for the *user* which will be running Eris.

## Install Eris

`eris` can be easily installed using our convenient binary releases.

We distribute binaries via our [Github Releases Page](https://github.com/eris-ltd/eris-cli/releases). You will simply need to download the proper zip or tarball for your architecture and then extract that into a place in your `PATH`.

### Debian Package Installation

We have (beta) `apt-get` support for most current versions of Debian and Ubuntu. If you wish to use apt-get to install `eris` then you will simply perform the following:

```bash
sudo apt-key adv --keyserver hkp://pool.sks-keyservers.net --recv-keys DDA1D0AB
sudo su -c "echo deb https://apt.eris.industries DIST main > /etc/apt/sources.list.d/eris.list"
```

**Note** in the above command you have to replace `DIST` with the distribution codename for your version of Debian or Ubuntu (`vivid`, `trusty`, `jessie`, etc.). We intend to make this easier in coming releases. We do not use the major-minor distribution pattern (e.g., `ubuntu-trusty`) as Docker does, but rather we just use (`trusty`) as the DIST codename.

Once the apt repository is added to your sources then:

```bash
sudo apt-get update
sudo apt-get install eris
```

### RPM Package Installation

We now have (experimental) RPM support for most current versions of Fedora, CentOS, RHEL, etc. If you wish to use `yum` to install `eris` then you will perform the following:

```bash
sudo su -c "curl -sSL https://yum.eris.industries/eris.repo > /etc/yum.repos.d/eris.repo"
yum install eris-cli
```

RPM support is still quite experimental, so please do let us know of any issues you have with this installation and we will make sure to address those as quickly as we can.

### Building From Source

If you would like to build from source [see our documentation](/tutorials/install-source/).

### All Platforms

Check that everything installed correctly with:

```bash
eris init
```

The command will begin setting you up.

## Troubleshooting Your Install

If you have any errors which arise during the installation process, please see our [trouble shooting page](/tutorials/install-troubleshooting/) or visit our [support forums](https://support.erisindustries.com)

# Step 2: Roll Your Own Blockchain in Seconds

If you want to create your blockchain it is very easy.

```bash
eris services start keys
eris chains make test_chain --chain-type simplechain
eris chains new test_chain
```

That `test_chain` can be whatever name you would like it to be. This simple command will create a permissioned, smart contract enabled blockchain suitable for testing.

To check that your chain is running type:

```bash
eris chains ls
```

Stop and remove your chain:

```bash
eris chains rm -xf test_chain
```

Obviously, you will want an ability to make chains which you properly parameterize. As such you can always type:

```bash
eris chains new --help
```

To see the various ways which you can give to chains new for it to be instantiated properly.

Eris does not only work with permissioned smart contract networks. It works just as well with existing blockchains. Want to run bitcoin?

```bash
eris services start btcd
```

Want to run Ethereum?

```bash
eris services start eth
```

That's it! Your chain is rolled!

# Step 3: Build (and run) your Distributed Application

{% image 'CEu7odJWMAA3VSa.jpg' %}

Let's remove all of the eris "stuff" before we move on to the next portion of the tutorials:

```bash
eris clean 
```

Now you're ready to build and run your distributed application!

There are a lot of ways you can go from here!

**To continue this getting started tour please see our next tutorial in this series on [Making Your Own Permissioned Chain](/tutorials/chain-making).**

Please go here if you are interested in [learning about smart contracts](/explainers/smart_contracts/).

Please go here if you are interested in [learning more about blockchains](/explainers/blockchains/).
