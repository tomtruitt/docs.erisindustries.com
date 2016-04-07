---

layout: docs
title: "Tutorials | Getting Started With Eris in the Cloud"

---

This tutorial will cover the first step when seeking to install Eris on cloud providers. Covered in this tutorial are the following cloud providers:

* Digital Ocean
* Amazon Web Services

On all cloud providers a Ubuntu base will be assumed. While eris does run fine on non-Debian derivative Linux OS's, Ubuntu is a common starting point for learners and as such will be used as the base of this tutorial.

**N.B. 1** -- the network typography used for this tutorial is dead simple. This tutorial is not about how to set up your private network typography but rather about how to set up Eris within that typography.

**N.B. 2** -- what about on premise deployments? Generally speaking they will work the same way, but on premise is difficult to generalize and as such will not be covered. Eris is able to provide Tier 2 support for your on premise deployment of the Eris platform as part of our [Library Subscription and Managed Eris packages](https://erisindustries.com/products/); just [get in touch](mailto:contact@erisindustries.com).

# Dependencies

At Eris we use a lot of docker machine to do our work. We find it is a fast and convenient way to get started with various cloud providers folks end up needing to work with. If you are running eris from OSX or Windows machine then docker machine is required and you will certainly have it installed on your machine. Which we cover in our [docker machine tutorial](/tutorials/tool-specific/docker_machine/).

This tutorial assumes that you have docker-machine installed and that you have worked through our [docker machine tutorial](/tutorials/tool-specific/docker_machine/).

This tutorial also assumes that you have keys for Digital Ocean and AWS accounts. If you only have access to an account with one of these cloud providers the modifications you'd need to make to this tutorial should be self explanatory. Should you want to use other could providers which are not covered in this tutorial, but which do have [docker-machine drivers](https://docs.docker.com/machine/drivers/) then please do utilize that driver (if you're on premise, this is how we recommend you work with `eris`).

Should you have no credentials or prefer not to utilize **any** cloud provider, this tutorial can still be utilized. Just utilize the `--driver virtualbox` and run everything from your machine.

The final dependency is that this tutorial will assume that you have added the proper credentials for whatever docker-machine driver you are using as environment variables. If you have not added them as environment variables then please add the necessary flags for the driver you will be using.

For Digital Ocean:

```bash
export DIGITALOCEAN_ACCESS_TOKEN=your_key DIGITALOCEAN_REGION=your_region
```

For AWS:

```bash
export AWS_ACCESS_KEY_ID=your_id\
AWS_DEFAULT_REGION=your_region\
AWS_SECRET_ACCESS_KEY=your_key\
AWS_SECURITY_GROUP=your_group\
AWS_VPC_ID=your_vpc
```

To check check these variables:

`env | grep AWS` _or_ `env | grep DIGIT`

# Introduction

To make our advanced chain run over non-proprietary lines we are going to deploy this chain to four digital ocean nodes in four different data centers and three AWS machines in three data centers. Obviously you may not want to do something this complex, so you can mix and match data centers and cloud providers with relative ease if you have even a base understanding of docker machine.

There are only two steps necessary to get `eris` working properly in the cloud:

1. Provision Your Machines
2. Install Eris

# Provision Your Machines

Obviously, if we're going to work on remote cloud instances we should first provision them. Provisioning is the process where a cloud provider reserves space in their data center for "your computer" and then gives you access to that computer. While there are a number of ways in which any one cloud provider generally offers to allow you to provision your machines, since we already have docker machine installed, we will use that to provision our machines.

Docker machine not only is a super convenient (marmot approved) provisioning and connection device, it also allows us to easily share access to machines within our teams (we will see this later).

## Digital Ocean

First let's make our digital ocean machines. We'll choose the following data centers (but feel free to mix and match to suit your needs / compliance requirements):

* ams3 (Amsterdam)
* sgp1 (Singapore)
* sfo1 (San Francisco)
* tor1 (Toronto)

Then we will name these machines using the following paradigm: `my-advchain-val-000` ... `my-advchain-val-006`.

Before we make some machines let's look at the options available to us when we use the Digital Ocean driver:

```bash
docker-machine create --driver digitalocean -h
```

Note, especially, the `--digitalocean-region` flag. We'll be using that one now to create the machines.

```bash
docker-machine create --driver digitalocean --digitalocean-size 1gb --digitalocean-region ams3 my-advchain-val-000
docker-machine create --driver digitalocean --digitalocean-size 1gb --digitalocean-region sgp1 my-advchain-val-001
docker-machine create --driver digitalocean --digitalocean-size 1gb --digitalocean-region sfo1 my-advchain-val-002
docker-machine create --driver digitalocean --digitalocean-size 1gb --digitalocean-region tor1 my-advchain-val-003
```
As stated above, feel free to substitute your favorite cloud provider for digital ocean, or virtualbox even if you just wanted to run this tutorial locally; just note that for other cloud providers you would use the appropriate flags for that provider instead of the `--digitalocean-region` flag (as we will see in the next section with AWS there can be more than one additional flag required).

It will take some time to provision those machines. If you want to do it faster you can background the first three jobs and leave the region unspecified: 

```bash
for i in `seq 0 2`
do
  docker-machine create --driver digitalocean --digitalocean-size 1gb my-adv-chain-val-00$i &
done
docker-machine create --driver digitalocean --digitalocean-size 1gb my-adv-chain-val-003
```

Note that we use 1gb droplet sizes as go has a bit of trouble building `eris` on smaller boxes due to lower RAM capacity. Alternatively, you could install `eris` on smaller boxes from apt-get which is explained in the **Install Eris** section below. For local apt-get installation do this:

```bash
apt-key adv --keyserver hkp://pool.sks-keyservers.net --recv-keys DDA1D0AB && echo "deb https://apt.eris.industries trusty main" > /etc/apt/sources.list.d/eris.list && apt-get update && apt-get install eris && eris init --yes
```

Nothing within this tutorial requires that Digital Ocean be used; the beauty of the docker-machine approach is that it normalizes working with docker engines running in the cloud. As we will see later, once one of these machines is "in scope" Eris can easily connect to it and run `eris` commands against the remote docker engine in the cloud. Pretty neat!

## Amazon Web Services

AWS is, admittedly, a bit more complicated to get started with. You will need to make sure that you have the appropriate VPC's and security groups set up for three different data centers. For this tutorial we're going to use the following data centers:

* eu-west-1 (Ireland)
* eu-central-1 (Frankfurt)
* us-east-1 (Northern Virginia)

We will continue our same naming paradigm. Before we make some machines let's look at the options available to us when we use the Digital Ocean driver:

```bash
docker-machine create --driver amazonec2 -h
```

Note that there is a lot more fine tuning you can do with AWS than can be done with Digital Ocean.

```bash
docker-machine create --driver amazonec2 --amazonec2-region eu-west-1 ----amazonec2-vpc-id {{vpcID1}} --amazonec2-security-group {{secGrp1}} my-advchain-val-004
docker-machine create --driver amazonec2 --amazonec2-region eu-central-1 ----amazonec2-vpc-id {{vpcID2}} --amazonec2-security-group {{secGrp2}} my-advchain-val-005
docker-machine create --driver amazonec2 --amazonec2-region us-east-1 ----amazonec2-vpc-id {{vpcID3}} --amazonec2-security-group {{secGrp3}} my-advchain-val-006
```

Again, this will take a few minutes to provision all of these machines. If you are unfamiliar with the nuances of AWS's VPCs and Security Groups then please use another cloud provider. Eris is not in a position to debug problems around your network configuration so if there are problems provisioning AWS boxes, please see docker-machines documentation and issues.

## Finalizing Provisioning

Now let us check that all the machines are on and running:

```bash
docker-machine ls
```

If you do not see any errors in the output and you see all seven validator nodes, then you're a-OK.

# Install Eris

Strictly speaking, we do not really need eris installed on the remote boxes. The way that `eris` tooling is designed is that it is built to run locally on the machine you are currently using and to connect, via docker's API, into a remote box. However, sometimes it is helpful to be able to SSH into a remote machine to debug something. Also, if you do not provision machines via docker machine then you may not be able to connect directly into the docker engine on the remote box.

Since this is a tutorial, we want to cover how easy it is to install eris on cloud providers.

What we are going to do is to pipe a shell file into a shell. Some folks get freaked out about this, and if you are of that calibre then you can accomplish what this script will do in any event! The script we will be using is [available here](https://github.com/eris-ltd/common/blob/master/cloud/setup/setup.sh). It is eris' cloud setup script. The script assumes that docker is installed. If you have provisioned a box via a web interface or otherwise and docker is not installed, you can set an environment variable and the script will automtically install Docker for you.

Since we provisioned these instances using docker-machine we do not need to do that.

```bash
for i in `seq 0 6`
do
  docker-machine ssh "my-advchain-val-00$i" "curl -sSL --ssl-req https://raw.githubusercontent.com/eris-ltd/common/master/cloud/setup/setup.sh | sudo bash"
done
```

That's it! Eris is now installed on all the machines we just made!

**N.B.** -- The script is only meant to work against Ubuntu. It is not tested against other operating systems. Should you be using RHEL, Fedora, CentOS or another Linux distribution you should follow our normal [getting started](/tutorials/getting-started) sequence.

**Troubleshooting**

Sometimes when using extremely small cloud instances eris has trouble building. If you get an error that looks like this:

```irc
Building eris.
# github.com/eris-ltd/eris-cli/cmd/eris
/usr/local/go/pkg/tool/linux_amd64/link: running gcc failed: fork/exec /usr/bin/gcc: cannot allocate memory
```

That means you don't have enough RAM on the machine to build eris. Sometimes this can be fixed by restarting a machine and rebuilding eris. Sometimes it requires migration to a larger machine.

**Removing Docker-Machines**

`docker-machine rm -f your_machine_name` will stop and force remove a named machine. Optionally you can use a loop to remove all machines by name starting with certain prefixes:

```bash
for i in `seq 0 6`
do
  docker-machine rm -f "my-adv-chain-val-00$i"
done
```
Or remove all Docker machines with `docker-machine rm -f $(docker-machine ls -q)`

# The Advantages of Using Docker Machine To Build Collaborative Software

OK, let's get real for a minute. This technology is all about providing a framework for business process automation for the information age. But what that means is that we have to be able to work together. Gated access to machines is one of the best ways docker machine helps us at Eris.

The reason why the cloud provider + docker-machine paradigm is interesting is that it is capable of provisioning nodes, starting, stopping, restarting nodes, and automating the docker connection from your local machine. This means, functionally, that one can be working from a laptop and in one command connect and deploy docker containers without having to ssh into the boxes or do very much at all (bear with us and we'll show you how it works over the course of this tutorial).

All the files you need to share for another colleague to have access to the box are available in `~/.docker/machine/machines/...` and in `~/.docker/certs`. We package these up into a docker image and push that to a private repository (quay.io for us) and share that with colleagues.

# Where to next?

**Next you'll want to make a [more complex chain](/tutorials/advanced/chain-making/).**
