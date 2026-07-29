

# Understanding Voting Disks in Oracle RAC

A voting disk is a file that manages information about node membership. It can be located on a shared cluster system, a shared raw device file, or inside Oracle ASM (from 11gR2 onwards).

Its primary purpose is to help in scenarios where private network communication fails. The **CSS (Cluster Synchronization Service)** determines which nodes in the cluster are available by communicating through a dedicated private network, using the voting disk as a secondary communication mechanism. The CSS service sends heartbeat messages through both the network and the voting disk.

### The Role of Voting Disks during Network Failures

* **Node Tracking:** When a private network failure occurs, nodes cannot synchronize I/O to the shared disks, causing some nodes to go offline. At this time, the voting disk is used to communicate node and trace information to determine which node is offline. Without the voting disk, it is difficult to know whether nodes are experiencing network problems or if they are no longer available.
* **Split-Brain Prevention:** If a voting disk is not used, a network failure leaves nodes unable to communicate with each other or synchronize with the database. This situation is called a cluster **SPLIT-BRAIN** problem. When any node is unable to send a heartbeat to the voting disk, it will reboot itself.
* **Redundancy:** For high availability, Oracle recommends having a minimum of three voting disks. If you configure a single voting disk, you should use external mirroring to provide redundancy.

---

## Important Points About Voting Disks in Oracle RAC

These files can be stored either in ASM or in shared storage.

1. **When Stored in ASM:** There is no need to configure them manually, as the files will be created automatically depending on the redundancy configured in ASM.
2. **When Stored in Shared Storage:** You must manually configure these files with a redundancy setup for high availability.
* **Odd Number Required:** You must have an odd number of disks.
* **Quantity Limits:** Oracle recommends a minimum of 3 and a maximum of 5. In 10g, Oracle Clusterware can support up to 32 voting disks, but in 11gR2, it supports up to 15 voting disks.
* **Quorum Rules:** A node must be able to access more than half of the voting disks at any time. For example, if you have 5 voting disks, a node must be able to access at least 3. If it cannot access the minimum number of voting disks, it is evicted/removed from the cluster.
* **Heartbeat Registration:** All nodes in the RAC cluster register their heartbeat information in the voting disks/files. The RAC heartbeat is the polling mechanism sent over the cluster interconnect to ensure all RAC nodes are available.



---

## Network vs. Disk Heartbeats

All nodes in the RAC cluster register their heartbeat information in the voting disks/files to ensure they are available. Voting disks/files act like attendance registers where nodes mark their presence (heartbeats).

The CSSD process (Cluster Services Synchronization Daemon) is a multithreaded process that uses various threads to monitor these two distinct heartbeats:

### 1. Network Heartbeat

The CSSD process on every node makes entries in the voting disk to ascertain the membership of the node. While marking their own presence, all nodes also register information about their ability to communicate with other nodes via the cluster interconnect in the voting disk.

### 2. Disk Heartbeat

The CSSD process in each RAC node maintains the heartbeat in a block (of size 1 OS block) within the "hot block" of the voting disk at a specific offset.

* The written block features a header area containing the node name.
* The heartbeat counter increments every second on every write call. Thus, the heartbeats of various nodes are recorded at different offsets in the voting disk.
* In addition to maintaining its own disk block, CSSD processes monitor the disk blocks maintained by the CSSD processes of other nodes in the cluster.
* Healthy nodes will have continuous network and disk heartbeats exchanged between them. A break in these heartbeats indicates a possible error scenario.

### The Kill Block Mechanism

If the disk is not updated within a short timeout period, the node is considered unhealthy and may be rebooted to protect the database. In this case, a message is written into the **KILL BLOCK** of that specific node. Each node reads its own KILL BLOCK once per second; if the kill block message is not overwritten, the node commits suicide (reboots).

---

## How Voting Disks Resolve Error Scenarios

CSSD processes monitor the health of RAC nodes employing Network and Disk heartbeats. When heartbeats break, a few different error scenarios are possible:

1. Network heartbeat is successful, but disk heartbeat is missed.
2. Disk heartbeat is successful, but network heartbeat is missed.
3. Both heartbeats failed.

With numerous nodes, other complex scenarios can occur:

* **Split Sets:** Nodes split into *N* sets of nodes, communicating within their own set but unable to communicate with members of another set.
* **Single Node Failure:** Just one node is unhealthy. In this case, nodes with quorum will maintain active membership of the cluster, and the unhealthy node(s) will be fenced/rebooted.

---

## What is Stored in a Voting Disk?

Voting disks contain both static and dynamic data:

* **Static Data:** Information about the nodes currently in the cluster.
* **Dynamic Data:** Disk heartbeat logging.

Overall, it maintains and consists of crucial details regarding cluster node membership, such as:

* Which node is part of the cluster.
* Which node is joining the cluster.
* Which node is leaving the cluster.

---

## Important Operations & Commands for Voting Disks

### A. List Currently Configured Voting Disks

```bash
$ORA_CRS_HOME/bin/crsctl query css votedisk

```

### B. Adding or Deleting Voting Disks

To add a voting disk:

```bash
crsctl add css votedisk

```

Run the following command as the `root` user to remove a voting disk:

```bash
crsctl delete css votedisk

```

### C. Replacing a Voting Disk

```bash
crsctl replace votedisk

```

### D. Backup Voting Disks

Run the following `dd` command to back up a voting disk. Perform this operation on every voting disk as needed (where `voting_disk_name` is the name of the active voting disk and `backup_file_name` is the target backup file):

```bash
dd if=voting_disk_name of=backup_file_name

```

**Example:**

```bash
$ dd if=[votedisk1] of=/home/oracle/vote/vote.dmp bs=4k

```

*(Result output: 1675289+1 records in, 1675289+1 records out. In this case, `vote.dmp` is the backup file).*

### E. Recovering Voting Disks

Run the following command to restore/recover a voting disk from a backup file:

```bash
dd if=backup_file_name of=voting_disk_name

```

**Example:**

```bash
[oracle@rac1 bin]$ dd if=/home/oracle/vote/vote.dmp of=[votedisk1] bs=4k

```

*(Result output: 1675289+1 records in, 1675289+1 records out).*

---

### Reference

* **Source URL:** [https://techgoeasy.com/what-is-voting-disk-in-oracle/](https://techgoeasy.com/what-is-voting-disk-in-oracle/)
