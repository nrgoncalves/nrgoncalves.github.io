---
title: "Setting up a Spark Standalone Cluster"
date: 2022-07-14T15:09:12+01:00
draft: false
categories: ["Infrastructure"]
tags: ["Spark", "PySpark"]
---

This is a quick note on how to set up a Spark cluster in standalone mode. This
is useful if you want to setup a cluster for your own development purposes or
if you just want to do it for fun—for more serious use cases, Spark clusters
should be setup on top of YARN or Kubernetes.

You will need to create a few VMs: one VM for the cluster manager and then
one or more VMs where the executors will be running. For instance:

- manager.myproject.example.com
- node01.myproject.example.com
- node02.myproject.example.com

Note: this setup would be better done via a configuration management tool such
as Puppet, mainly to make it really easy to add new nodes to the cluster. I've
written a very quick intro to setting up Puppet
[here](https://nrgoncalves.github.io/posts/puppetserver/). However, we will be
doing this manually here as it might not be worth the overhead of setting up
Puppet.

First, we need to install Spark. Here I am using Debian 11 but Spark should work
regardless of the OS provided that Java is installed. Let's do the following on
all VMs:

```bash
# Install Java, if not installed
sudo apt install default-jre

# Install Spark (note: pick a recent version here)
wget https://downloads.apache.org/spark/spark-3.3.0/spark-3.3.0-bin-hadoop3.tgz
tar xvf spark-3.3.0-bin-hadoop3.tgz
sudo mv spark-3.3.0-bin-hadoop3 /opt/spark
```

Then we need to add the following variables to `.profile`:

```
export SPARK_HOME=/opt/spark
export PATH=$PATH:$SPARK_HOME/bin:$SPARK_HOME/sbin
export PYSPARK_PYTHON=/usr/bin/python3
```

and `source .profile` to get those variables set.

On the master node, generate an SSH key pair to allow the master to communicate
with the workers via SSH

```bash
# Generate an ssh key pair
ssh-keygen -t rsa
```

Then add the `id_rsa.pub` public key to each of the nodes `.ssh/authorized_keys`
file. Finally, return to the master node to configure Spark:

```bash
# Spark configuration
cp $SPARK_HOME/conf/spark-env.sh.template $SPARK_HOME/conf/spark-env.sh
cp $SPARK_HOME/conf/workers.template $SPARK_HOME/conf/workers

# Get the path to java (value of java.home) from
java -XshowSettings:properties -version

# Add java home and master host to spark-env.sh
# export JAVA_HOME=/usr/lib/jvm/java-11-openjdk-amd64
# export SPARK_MASTER_HOST=manager.myproject.example.com
sudo nano $SPARK_HOME/conf/spark-env.sh

# Add the worker hosts to the workers file
# i.e. node01.myproject.example.com, node02.myproject.example.com, one in each line
sudo nano $SPARK_HOME/conf/workers

# Start the cluster
$SPARK_HOME/sbin/start-all.sh

# Verify that the worker exists by fetching the Spark UI page
curl http://127.0.0.1:8080/
```

That is it—the Spark cluster is now ready! You may now want to setup an SSH
tunnel to allow you to access the master VM conveniently from your local desktop
environment.
