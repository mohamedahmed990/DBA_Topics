Here is the content from the document structured clearly with headings, bullet points, and consistent formatting, while keeping the information, commands, and phrasing intact.

---

# Voting Disks in Oracle RAC

## Description

A voting disk is a file that manages information about node membership. It is located on the shared cluster system or a shared raw device file or Oracle ASM(11gr2 onwards) . Its primary purpose is to help in a situation where the private network communication fails.

CSS (Cluster Synchronization Service) is the service that determines which nodes in the cluster are available via communication through a dedicated private network and with a voting disk used as a secondary communication mechanism. CSS service is sending heartbeat messages through the network and voting disk.

In a situation, when due to private network failure, nodes are not able to synchronize I/O to the shared disks, Therefore some of the nodes will be going to the offline stage. At this time VOTING DISK is used to communicate the node and trace information, which node is in the offline stage. Without the voting disk, it's difficult to know, whether nodes are facing network problems or nodes are no longer available.

If we are not using a voting disk, then due to network failure, nodes are neither able to communicate with each other nor synchronize with the database, this situation is called a cluster SPLIT-BRAIN problem. When any node is not able to send a heartbeat to the voting disk, then it will reboot itself.

For high availability, Oracle recommends that you have a minimum of three voting disks. If you configure a single voting disk, then you should use external mirroring to provide redundancy.

---

## Important points about Voting disks in Oracle RAC

These files can be stored either in ASM or in shared storage.

1. **If it is stored in ASM:** no need to configure manually as the files will be created depending on the redundancy in ASM.
2. **In a shared storage system:** we need to manually configure these files with a redundancy setup for high availability.
* **a)** We must have an odd number of disks.
* **b)** Oracle recommends a minimum of 3 and a maximum of 5. In 10g, Oracle Clusterware can support 32 voting disks but in 11g R 2 supports 15 voting disks.
* **c)** A node must be able to access more than half of the voting disks at any time. For eg, if you have 5 voting disks, a node must be able to access at least 3 of the voting disks. If it cannot access the minimum of voting disks, then it is evicted/removed from the cluster.
* **d)** All nodes in the RAC cluster register their heartbeat information in the voting disks/files. RAC heartbeat is the polling mechanism that is sent over the cluster interconnect to ensure all RAC nodes are available.



---

## What is NETWORK and DISK HEARTBEAT and how does it register in VOTING DISKS/FILES?

* **1)** All nodes in the RAC cluster register their heartbeat information in the voting disks/files. RAC heartbeat is the polling mechanism that is sent over the cluster interconnect to ensure all RAC:
   + **a.** nodes are available.
   + **b.** Voting disks/files are just like attendance registers where you have nodes mark their attendance (heartbeats).
* **2)** CSSD process on every node makes entries in the voting disk to ascertain the membership of the node. While marking their own presence, all the nodes also register the information about their communicability with other nodes in the voting disk. This is called NETWORK HEARTBEAT.
* **3)** CSSD process in each RAC maintains the heartbeat in a block of size 1 OS block in the hot block of the voting disk at a specific offset. The written block has a header area with the node name. The heartbeat counter increments every second on every write call. Thus heartbeat of various nodes is recorded at different offsets in the voting disk. This process is called DISK HEARTBEAT. In addition to maintaining its own disk block, CSSD processes also monitor the disk block maintained by the CSSD processes of other nodes in the cluster. Healthy nodes will have continuous network & disk heartbeats exchanged between the nodes. Break-in heartbeats indicate a possible error scenario.
* **4)** If the disk is not updated in a short timeout period, the node is considered unhealthy and maybe rebooted to protect the database. In this case, a message to this effect is written in the KILL BLOCK of the node. Each node reads its KILL BLOCK once per second, if the kill block is not overwritten, the node commits suicide.

---

## How Voting disks in RAC help

CSSD processes (Cluster Services Synchronization Daemon) monitor the health of RAC nodes employing two distinct heartbeats: Network heartbeat and Disk heartbeat. Healthy nodes will have a continuous network and disk heartbeats exchanged between the nodes. Break-in heartbeat indicates a possible error scenario.

### Missing Heartbeats Scenarios:

There are a few different scenarios possible with missing heartbeats:

1. Network heartbeat is successful, but disk heartbeat is missed.
2. Disk heartbeat is successful, but network heartbeat is missed.
3. Both heartbeats failed.

In addition, with numerous nodes, there are other possible scenarios too. Few possible scenarios:

1. Nodes have split into N sets of nodes, communicating within the set, but not with members in another set.
2. Just one node is unhealthy. Nodes with quorum will maintain active membership of the cluster and other nodes (s) will be fenced/rebooted.

CSSD is a mutithreaded process.So it use various thread to monitor the heart beat

---

## What is stored in voting disks in RAC?

Voting disks contain static and dynamic data.

* **Static data:** Info about nodes in the cluster
* **Dynamic data:** Disk heartbeat logging

It maintains and consists of important details about the cluster nodes membership, such as:

* – which node is part of the cluster,
* – who (node) is joining the cluster, and
* – who (node) is leaving the cluster.

---

## Important Operation for Voting disks in RAC

### a) To list currently configured voting disk

```bash
$ORA_CRS_HOME/bin/crsctl query css votedisk

```

### b) Adding or deleting the Voting disks

```bash
crsctl add css votedisk

```

Run the following command as the root user to remove a voting disk:

```bash
crsctl delete css votedisk

```

### c) replacing Voting disk

```bash
crsctl replace votedisk

```

### Backup voting disks in RAC:

Run the following command to back up a voting disk. Perform this operation on every voting disk as needed where voting_disk_name is the name of the active voting disk and backup_file_name is the name of the file to which you want to back up the voting disk contents:

```bash
dd if=voting_disk_name of=backup_file_name

```

```bash
$ dd if=[votedisk1] of=/home/oracle/vote/vote.dmp bs=4k

```

`1675289+1 records in`
`1675289+1records out`
( vote.dmp is the name of backup file of voting disk)

### Recovering Voting Disks in RAC

Run the following command to recover a voting disk where backup_file_name is the name of the voting disk backup file and voting_disk_name is the name of the active voting disk:

```bash
dd if=backup_file_name of=voting_disk_name

```

```bash
[oracle@rac1 bin]$ dd if=/home/oracle/vote/vote.dmp of=[votedisk1] bs=4k

```

`1675289+1 records in`
`1675289+1records out`

---

**Reference URL:** [https://techgoeasy.com/what-is-voting-disk-in-oracle/](https://techgoeasy.com/what-is-voting-disk-in-oracle/)
