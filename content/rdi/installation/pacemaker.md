---
Title: High availability deployment using Pacemaker
linkTitle: High availability deployment 
description: Shows how to set up a Replica Of cluster using Pacemaker.
weight: 40
alwaysopen: false
categories: ["redis-di"]
aliases: 
---

High availability with Redis Data Integration can be achieved using [Pacemaker](https://wiki.clusterlabs.org/wiki/Pacemaker), as shown in the following image:

{{<image filename="/images/rdi/redis-di-high-availability.png" alt="High availability topology for Debezium Server" >}}{{</image>}}

The following steps show how to deploy a three node Active/Passive cluster using Debezium Server virtual machines (VMs) and Pacemaker.

You can use this guide with the following operating systems:

- [CentOS Linux 7](https://centos.org/centos-linux/)
- [CentOS Stream 8](https://www.centos.org/centos-stream/#centos-stream-8)
- [Red Hat Enterprise Linux (RHEL) 7 and 8](https://www.redhat.com/en/technologies/linux-platforms/enterprise-linux)
- [Rocky Linux 8](https://rockylinux.org/)
- [Ubuntu 18.04](https://releases.ubuntu.com/18.04/)
- [Ubuntu 20.04](https://releases.ubuntu.com/focal/)

For other Linux versions, use the [Pacemaker documentation](https://clusterlabs.org/quickstart.html) to determine the required binary packages and configurations.

## Prerequisites

The following ports must be enabled for Pacemaker and Debezium Server:

| Protocol | Port           | Description                                                                                                        |
| -------- | -------------- | ------------------------------------------------------------------------------------------------------------------ |
| TCP      | 2224           | pcsd Web UI and node-to-node communication                                                                         |
| UDP      | 5404-5412      | Required on [corosync](http://corosync.github.io/corosync/) nodes to facilitate communication between nodes        |
| TCP      | 8088           | Query Debezium Server status                                                                                       |
| TCP      | _\<Redis DI>_  | The port of the Redis DI database, as specified in [_application.properties_]({{<relref "/rdi/reference/debezium">}})    |
| TCP      | _\<Source DB>_ | The port used by the source database, as specified in [_application properties_]({{<relref "/rdi/reference/debezium">}}) |

For further details on Pacemaker, please refer to the Red Hat documentation [here](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/8/html/configuring_and_managing_high_availability_clusters/assembly_creating-high-availability-cluster-configuring-and-managing-high-availability-clusters#proc_enabling-ports-for-high-availability-creating-high-availability-cluster).



## Install and run Pacemaker

- [CentOS 7/RHEL 7](#centos-7rhel-7)
- [CentOS Stream 8/RHEL 8/Rocky 8](#centos-stream-8-rhel-8--rocky-linux-8)
- [Ubuntu 18.04](#ubuntu-1804)
- [Ubuntu 20.04](#ubuntu-2004)

### CentOS 7/RHEL 7

Repeat the following steps on each virtual machine that will run Debezium Server:

#### Configure the repository in RHEL 7

This step is not needed for CentOS 7

Pacemaker is only available through CentOS 7 repositories, so the first step is to configure a suitable repository:

```bash
sudo bash -c 'cat << EOF > /etc/yum.repos.d/centos.repo
[centos-7-base]
name=CentOS-7 - Base
mirrorlist=http://mirrorlist.centos.org/?release=7&arch=x86_64&repo=os
enabled=1
gpgkey=http://mirror.centos.org/centos/7/os/x86_64/RPM-GPG-KEY-CentOS-7
EOF'
```

#### Install Pacemaker and dependencies

```bash
sudo yum install -y pacemaker pcs resource-agents
```

#### Start the Pacemaker service

```bash
sudo systemctl start pcsd.service
sudo systemctl enable pcsd.service
```

#### Set a password for the cluster

Replace `<PASSWORD>` with a password of your choice.

```bash
sudo bash -c 'echo <PASSWORD> | passwd --stdin hacluster'
```

#### Run the Pacemaker cluster

Replace `<PASSWORD>` with the value from the previous step and `<HOST1>`, `<HOST2>` and `<HOST3>` with the hostnames/IP addresses of the respective VMs forming the cluster.

```bash
sudo pcs cluster auth <HOST1> <HOST2> <HOST3> -u hacluster -p <PASSWORD>
sudo pcs cluster setup --force --name cluster_dbz <HOST1> <HOST2> <HOST3>
sudo pcs cluster start --all
sudo pcs property set stonith-enabled=false
```

To learn how to verify that the cluster is working correctly, see [Validating the installation](#validating-the-installation).

### CentOS Stream 8, RHEL 8, & Rocky Linux 8

Repeat the following steps on each of the VMs that will run the Debezium Server:

#### Configure RHEL & Rocky Linux repository

if you're using RHEL or Rocky Linux, add the high availability repository to your package manager to obtain Pacemaker binaries:

```bash
sudo dnf config-manager --set-enabled ha
```

This step is not needed for CentOS Stream 8

#### Install Pacemaker and dependencies

```bash
sudo dnf -y install pcs pacemaker
```

#### Start the Pacemaker service

```bash
sudo systemctl start pcsd.service
sudo systemctl enable pcsd.service
```

#### Set a password for the cluster

Replace `<PASSWORD>` with a password of your choice.

```bash
sudo bash -c 'echo <PASSWORD> | passwd --stdin hacluster'
```

#### Run the Pacemaker cluster

Replace `<PASSWORD>` with the value from the previous step and `<HOST1>`, `<HOST2>` and `<HOST3>` with the hostnames/IP addresses of the respective VMs forming the cluster.

```bash
sudo pcs host auth <HOST1> <HOST2> <HOST3>
sudo pcs cluster setup cluster_dbz <HOST1> <HOST2> <HOST3>
sudo pcs cluster start --all
sudo pcs property set stonith-enabled=false
```

See [here](#validating-the-installation) how to check that the cluster is running correctly.

### Ubuntu 18.04

Repeat the following steps on each of the VMs that will run the Debezium Server:

#### Install Pacemaker and dependencies

```bash
sudo apt update
sudo apt install -y pacemaker pcs
```

#### Start the Pacemaker service

```bash
sudo systemctl start pcsd.service
sudo systemctl enable pcsd.service
```

#### Set a password for the cluster

Replace `<PASSWORD>` with a password of your choice.

```bash
sudo bash -c 'echo "hacluster:<PASSWORD>" | chpasswd'
```

#### Run the Pacemaker cluster

Replace `<PASSWORD>` with the value from the previous step and `<HOST1>`, `<HOST2>` and `<HOST3>` with the hostnames/IP addresses of the respective VMs forming the cluster.

```bash
sudo pcs cluster auth <HOST1> <HOST2> <HOST3> -u hacluster -p <PASSWORD>
sudo pcs cluster setup --force --name cluster_dbz <HOST1> <HOST2> <HOST3>
sudo pcs cluster start --all
sudo pcs property set stonith-enabled=false
```

To learn how to verify that the cluster is working correctly, see [Validating the installation](#validating-the-installation).

### Ubuntu 20.04

Repeat the following steps on each of the VMs that will run the Debezium Server:

#### Install Pacemaker and dependencies

```bash
sudo apt update
sudo apt install -y pcs pacemaker
```

#### Start the Pacemaker service

```bash
sudo systemctl start pcsd.service
sudo systemctl enable pcsd.service
```

#### Set a password for the cluster

Replace `<PASSWORD>` with a password of your choice.

```bash
sudo bash -c 'echo "hacluster:<PASSWORD>" | chpasswd'
```

#### Run the Pacemaker cluster

Replace `<PASSWORD>` with the value from the previous step and `<HOST1>`, `<HOST2>` and `<HOST3>` with the hostnames/IP addresses of the respective VMs forming the cluster.

```bash
sudo pcs cluster destroy
sudo pcs host auth <HOST1> <HOST2> <HOST3>
sudo pcs cluster setup cluster_dbz <HOST1> <HOST2> <HOST3>
sudo pcs cluster start --all
sudo pcs property set stonith-enabled=false
```

To learn how to verify that the cluster is working correctly, see [Validating the installation](#validating-the-installation).

### Validate the installation

Check the status of the cluster:

```bash
sudo pcs status cluster
```

For a correctly configured cluster, this will result in output similar to the following:

```shell
Cluster Status:
 Stack: corosync
 Current DC: gvb-deb7-1 (version 1.1.23-1.el7-9acf116022) - partition WITHOUT quorum
 Last updated: Thu Dec 29 14:11:26 2022
 Last change: Thu Dec 29 14:11:22 2022 by hacluster via crmd on gvb-deb7-1
 3 nodes configured
 0 resource instances configured

PCSD Status:
  gvb-deb7-3: Online
  gvb-deb7-2: Online
  gvb-deb7-1: Online
```

## Set up Pacemaker to manage Debezium Server

The cluster is now ready to run Debezium Server.

You can launch Debezium Server with Pacemaker in one of the following ways:

- [Run Debezium Server as a Container](#run-debezium-server-as-a-container) using the Heartbeat resource agent for [Podman](https://podman.io/) or [Docker](https://www.docker.com/).
- [Run Debezium Server as a Standalone Java Process](#run-debezium-server-as-a-standalone-java-process) using the `systemd` resource agent.

### Run Debezium Server as a container

Repeat the following steps on each of the Pacemaker cluster nodes.

#### Provide the Debezium configuration file

Debezium Server expects to find file _application.properties_ in a specified directory. Use the file created through command `redis-di scaffold` (edited for your source database) and upload it to the home directory of each virtual machine in your cluster:

```bash
mkdir conf
mv application.properties conf
```

#### Create the Pacemaker resource for the Debezium container

> This guide uses Podman to illustrate the required commands. If you are using the Docker resource agent, simply replace `podman` with `docker` below.

> The Podman resource agent is not available for CentOS 7/RHEL 7/Ubuntu 18.04 - use the Docker resource agent, instead.

To launch the Debezium Server container use the podman service of Pacemaker (Podman needs to be installed as a pre-requisite):

```bash
sudo pcs resource create dbz_server ocf:heartbeat:podman image=docker.io/debezium/server allow_pull=yes run_opts="--network=host --privileged -v $PWD/conf:/debezium/conf"
```

> Note the `run_opts` parameter that accepts any parameter you would normally provide to the Debezium Server container, namely the mapping of the `application.properties` configuration file.

#### Oracle

If you are using Oracle as your source DB, please note that Debezium Server does not include the Oracle JDBC driver. As a result, it will fail with an error. You should follow these steps to add the JDBC driver:

- Download the driver (for Oracle 19c):

  ```bash
  wget -P lib https://repo1.maven.org/maven2/com/oracle/database/jdbc/ojdbc10/19.17.0.0/ojdbc10-19.17.0.0.jar
  ```

- Adjust the pcs resource creation command:

  ```bash
  sudo pcs resource create dbz_server ocf:heartbeat:podman image=docker.io/debezium/server allow_pull=yes run_opts="--network=host --privileged -v $PWD/conf:/debezium/conf -v $PWD/lib/ojdbc10-19.17.0.0.jar:/debezium/lib/ojdbc10-19.17.0.0.jar"
  ```

#### Verify status

```bash
sudo pcs status
```

This should result in output similar to this:

```shell
Cluster name: cluster_dbz
Stack: corosync
Current DC: gvb-deb7-1 (version 1.1.23-1.el7-9acf116022) - partition with quorum
Last updated: Thu Dec  8 16:02:53 2022
Last change: Thu Dec  8 16:02:48 2022 by root via cibadmin on gvb-deb7-1

3 nodes configured
1 resource instance configured

Online: [ gvb-deb7-1 gvb-deb7-2 gvb-deb7-3 ]

Full list of resources:

 dbz_server	(ocf::heartbeat:podman):	Started gvb-deb7-1

Daemon Status:
  corosync: active/disabled
  pacemaker: active/disabled
  pcsd: active/enabled
```

#### Troubleshoot

Check the container is running with `sudo podman ps -a`. This should result in output similar to this:

```shell
CONTAINER ID   IMAGE             COMMAND              CREATED          STATUS          PORTS     NAMES
2aecd4e9fa31   debezium/server   "/debezium/run.sh"   2 minutes ago    Up 2 minutes              dbz_server
```

If the Debezium Server fails to start or doesn’t behave as expected, you can check the log file:

```bash
sudo podman logs dbz_server
```

A healthy log ends like this:

```shell
2022-12-08 16:02:57,284 INFO  [io.deb.con.mys.MySqlStreamingChangeEventSource] (blc-gvb-mysql:3306) Connected to MySQL binlog at gvb-mysql:3306, starting at MySqlOffsetContext [sourceInfoSchema=Schema{io.debezium.connector.mysql.Source:STRUCT}, sourceInfo=SourceInfo [currentGtid=null, currentBinlogFilename=mysql-bin.000006, currentBinlogPosition=154, currentRowNumber=0, serverId=0, sourceTime=null, threadId=-1, currentQuery=null, tableIds=[], databaseName=null], snapshotCompleted=false, transactionContext=TransactionContext [currentTransactionId=null, perTableEventCount={}, totalEventCount=0], restartGtidSet=null, currentGtidSet=null, restartBinlogFilename=mysql-bin.000006, restartBinlogPosition=154, restartRowsToSkip=0, restartEventsToSkip=0, currentEventLengthInBytes=0, inTransaction=false, transactionId=null, incrementalSnapshotContext =IncrementalSnapshotContext [windowOpened=false, chunkEndPosition=null, dataCollectionsToSnapshot=[], lastEventKeySent=null, maximumKey=null]]
2022-12-08 16:02:57,285 INFO  [io.deb.uti.Threads] (blc-gvb-mysql:3306) Creating thread debezium-mysqlconnector-chinook-binlog-client
2022-12-08 16:02:57,285 INFO  [io.deb.con.mys.MySqlStreamingChangeEventSource] (debezium-mysqlconnector-chinook-change-event-source-coordinator) Waiting for keepalive thread to start
2022-12-08 16:02:57,390 INFO  [io.deb.con.mys.MySqlStreamingChangeEventSource] (debezium-mysqlconnector-chinook-change-event-source-coordinator) Keepalive thread is running
```

#### Test

The following tests can be manually performed to verify the expected behavior:

- Kill the container:

  ```bash
  sudo podman rm -f dbz_server
  ```

- Pacemaker will restart the container.
  You can verify with:

  ```bash
  sudo pcs status
  ```

- Kill the VM:

  Pacemaker will start the Debezium container on the standby node.
  You can verify with:

  ```bash
  sudo pcs status
  ```

### Run Debezium Server as a standalone Java process

Debezium Server is a Java application, so Java 17 ([OpenJDK 17](https://openjdk.org/projects/jdk/17/)) must be installed as a prerequisite.

Repeat the following steps on each of the Pacemaker cluster nodes.

#### Download and install Debezium Server

```bash
wget https://repo1.maven.org/maven2/io/debezium/debezium-server-dist/{{ site.debezium_server_version }}.Final/debezium-server-dist-{{ site.debezium_server_version }}.Final.tar.gz
sudo tar xvfz debezium-server-dist-2.1.1.Final.tar.gz -C /opt
```

This will install Debezium Server in the directory _/opt/debezium-server_.

#### Provide the Debezium configuration File

Debezium expects to find file _application.properties_ in a specified directory. Use the file created through command `redis-di scaffold` (edited for your source database) and upload it to the home directory of each VM, then move it to the Debezium Server conf directory:

```bash
sudo mv application.properties /opt/debezium-server/conf
```

#### Create a systemd unit file for Debezium Server

- Create file _debezium.service_ with the following content:

```ini
[Unit]
Description=systemd service unit file for Debezium Server

[Service]
ExecStart=/bin/bash -c 'cd /opt/debezium-server && ./run.sh'
```

- Copy to _/usr/lib/systemd/system_:

```bash
sudo cp debezium.service /usr/lib/systemd/system
```

> Note: For Ubuntu, the target directory is _/lib/systemd/system_

- Reload the systemd daemon:

```bash
sudo systemctl daemon-reload
```

> Important: Make sure that the Debezium systemd service to be controlled by the cluster is **_not_** enabled to start at boot.

#### Oracle

If you are using Oracle as your source DB, please note that Debezium Server does not include the Oracle JDBC driver. As a result, it will fail with an error. You should download the JDBC driver to _/opt/debezium-server/lib_ before creating the Pacemaker resource:

```bash
sudo wget -O /opt/debezium-server/lib/ojdbc10-19.17.0.0.jar \
https://repo1.maven.org/maven2/com/oracle/database/jdbc/ojdbc10/19.17.0.0/ojdbc10-19.17.0.0.jar
```

#### Set Up Pacemaker to launch and control Debezium

To launch Debezium Server use the systemd resource agent of Pacemaker:

```bash
sudo pcs resource create dbz_server systemd:debezium
```

#### Verify status

```bash
sudo pcs status
```

This should result in output similar to this:

```shell
Cluster name: cluster_dbz
Status of pacemakerd: 'Pacemaker is running' (last updated 2022-12-09 12:21:55Z)
Cluster Summary:
  * Stack: corosync
  * Current DC: gvb-deb3 (version 2.1.5-2.el8-631339ca5aa) - partition with quorum
  * Last updated: Fri Dec  9 12:21:55 2022
  * Last change:  Fri Dec  9 12:10:24 2022 by root via cibadmin on gvb-deb1
  * 3 nodes configured
  * 1 resource instance configured

Node List:
  * Online: [ gvb-deb1 gvb-deb2 gvb-deb3 ]

Full List of Resources:
  * dbz_server	(systemd:debezium):	 Started gvb-deb1

Daemon Status:
  corosync: active/disabled
  pacemaker: active/disabled
  pcsd: active/enabled
```

#### Troubleshoot

If the Debezium Server fails to start or doesn’t behave as expected, you can check the status with `systemctl`:

```bash
sudo systemctl status debezium
```

A healthy status looks like this:

```shell
● debezium.service - Cluster Controlled debezium
   Loaded: loaded (/usr/lib/systemd/system/debezium.service; static; vendor preset: disabled)
  Drop-In: /run/systemd/system/debezium.service.d
           └─50-pacemaker.conf
   Active: active (running) since Fri 2022-12-09 12:10:24 UTC; 12min ago
 Main PID: 64740 (java)
    Tasks: 23 (limit: 22203)
   Memory: 179.6M
   CGroup: /system.slice/debezium.service
           └─64740 java -cp debezium-server-dist-2.0.1.Final-runner.jar:conf:lib/* io.debezium.server.Main

…
2022-12-08 16:02:57,285 INFO  [io.deb.uti.Threads] (blc-gvb-mysql:3306) Creating thread debezium-mysqlconnector-chinook-binlog-client
2022-12-08 16:02:57,285 INFO  [io.deb.con.mys.MySqlStreamingChangeEventSource] (debezium-mysqlconnector-chinook-change-event-source-coordinator) Waiting for keepalive thread to start
2022-12-08 16:02:57,390 INFO  [io.deb.con.mys.MySqlStreamingChangeEventSource] (debezium-mysqlconnector-chinook-change-event-source-coordinator) Keepalive thread is running
```

#### Test

The following tests can be manually performed to verify the expected behavior:

- Kill the Debezium Server process:

  ```bash
  ps -elf | grep debezium
  ```

  This will show the running Debezium Server, for example:

  ```shell
  root     19297     1 79 14:50 ?        00:00:08 java -cp debezium-server-dist-2.0.1.Final-runner.jar:conf:lib/* io.debezium.server.Main
  ```

  Kill the process - in this example:

  ```bash
  sudo kill 19297
  ```

- Pacemaker will restart the service.
  You can verify with:

  ```bash
  sudo pcs status
  ```

- Kill the VM:

  Pacemaker will start Debezium Server on the standby node.
  You can verify with:

  ```bash
  sudo pcs status
  ```