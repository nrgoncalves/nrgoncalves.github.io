---
title: "Puppet with a Agent-Master architecture"
date: 2022-07-13T11:21:15+01:00
draft: false
categories: ["Infrastructure"]
tags: ["Puppet", "Configuration management"]
---

There are many resources out there which—rightly—focus on teaching Puppet using
simple architectures, e.g. a single node, masterless setup. However, I found
less info on how to set up Puppet with an Agent-Master architecture.

This short article gives a very high-level overview of the main steps. It is not
a self-sufficient tutorial by any means—but I am hoping it will still be useful
for those trying to understand the big picture.

Briefly, we need multiple VMs. One VM will act as the master VM and will hold
the Puppet manifests. Other VMs will be nodes—they will be running the Puppet
Agent periodically to get the desired configuration from the master and
execute the required tasks to make the node's state aligned with what is
defined in the Puppet manifests.

## Puppet master

We first need to create a virtual machine to act as our puppet master. Here
we're using a VM running CentOS 7. Let's assume its FQDN is
`puppet.myproject.example.com`.

```sh
sudo su -
rpm -Uvh https://yum.puppet.com/puppet6/puppet6-release-el-7.noarch.rpm
yum install puppetserver
systemctl enable --now puppetserver
```

The `puppetserver` service should now be running on your puppet master VM. To
verify that the `puppetserver` CLI has been installed correctly, run

```sh
puppetserver -v
```

on a new shell. If this does not work, it is probably because `puppetserver` has
not been added to your `PATH`, so you'll need to investigate why that was the
case.

You should also test that the puppet agent can run on the master:

```sh
sudo su -
puppet agent --test --server {FQDN}
```

If the above agent run completed successfully, we are pretty much done with the
initial master configuration.

## Puppet agent on a node VM

Now we need to create at least one node VM to test if things really are working.
Let's assume the VM is running CentOS 7 and that the FQDN is
`node01.myproject.example.com`.

Let's start by installing the agent and enabling it as a service

```sh
sudo yum install puppet-agent
sudo /opt/puppetlabs/bin/puppet resource service puppet ensure=running enable=true
```

Now we need to add `puppet` to our PATH and configure the server

```sh
source /etc/profile.d/puppet-agent.sh
puppet config set server puppet.myproject.example.com --section main
```

No we need to get the node talking to the master. To do that, we first run

```sh
puppet ssl bootstrap
```

on the node VM. This should have requested a certificate signature to the master VM,
which we now need to sign. SSH into the master VM, change to root and run

```sh
puppetserver ca sign --certname node01.myproject.example.com
```

This should have successfully signed the certificate for the node VM, which means
that the puppet master and the node VM should now be able to work together.

We need to repeat this section for every node you want to manage with puppet, e.g.
`node02.myproject.puppet.com`, `node03.myproject.puppet.com` and so on.

At a high-level, this is pretty much it. The next step is to actually start
writing some puppet manifests.

## Writing Puppet manifests

Puppet's installation process should have created a set of files and folders on
the master VM. They are located under `/etc/puppetlabs/code/`. This is where
Puppet expects to find manifests. A standard production environment is created
by default, with a standard structure for organising Puppet manifests. These can
be found under `/etc/puppetlabs/code/environments/production`.

It is a good idea to track changes in these files using a version control tool
like `git`, and pull those changes onto the master VM once you are ready to test
or apply them.

I am going to start by creating a simple file resource just for testing purposes
and I will apply it to all nodes. I create a `manifests/site.pp` file with the
following contents

```
node default {
  file {'/tmp/test.txt':
    content => 'test file content',
  }
}
```

This asks Puppet to create a file `/tmp/test.txt` with the content `test file content`
for all nodes.

To test that this is working, ssh into the node VM and run:

```bash
puppet agent --test
```

The agent will then retrieve the desired system state from the master, and execute
the corresponding tasks to achieve such state. If all worked well, the file
`/tmp/test.txt` should now exist on the node VM.

This is an embarrassingly simple example of what we can do with Puppet—for a proper
treatment of Puppet's features, take a look at Puppet's documentation.

## Conclusion

The aim of this short article was to provide a very high-level overview of how the
basic steps that are required to set up Puppet with an Agent-Master architecture.

I have skipped over lots of details (e.g. networking), but the goal here is to
focus on the "big picture" rather than setup details, which will differ
significantly depending on the OS platform or cloud provider of choice.
