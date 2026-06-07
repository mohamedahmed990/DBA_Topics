
# Oracle Clusterware Software Concepts and Requirements

Oracle Clusterware uses voting files to provide fencing and cluster node membership determination. Oracle Cluster Registry (OCR) provides cluster configuration information. Collectively, voting files and OCR are referred to as **Oracle Clusterware files**.

Oracle Clusterware files must be stored on Oracle ASM. If the underlying storage for the Oracle ASM disks is not hardware protected, such as RAID, then Oracle recommends that you configure multiple locations for OCR and voting files. The voting files and OCR are described as follows:

---

## Voting Files

Oracle Clusterware uses voting files to determine which nodes are members of a cluster. You can configure voting files on Oracle ASM, or you can configure voting files on shared storage.

* **Configuring on Oracle ASM:** If you configure voting files on Oracle ASM, then you do not need to manually configure the voting files. Depending on the redundancy of your disk group, an appropriate number of voting files are created.
* **Configuring on Shared Storage (Non-ASM):** If you do not configure voting files on Oracle ASM, then for high availability, Oracle recommends that you have a minimum of three voting files on physically separate storage. This avoids having a single point of failure. If you configure a single voting file, then you must use external mirroring to provide redundancy.
* **File Count Recommendations:** Oracle recommends that you do not use more than five voting files, even though Oracle supports a maximum number of 15 voting files.

---

## Oracle Cluster Registry

Oracle Clusterware uses the Oracle Cluster Registry (OCR) to store and manage information about the components that Oracle Clusterware controls, such as Oracle RAC databases, listeners, virtual IP addresses (VIPs), and services and any applications.

OCR stores configuration information in a series of key-value pairs in a tree structure. To ensure cluster high availability, Oracle recommends that you define multiple OCR locations. In addition:

* You can have up to five OCR locations.
* Each OCR location must reside on shared storage that is accessible by all of the nodes in the cluster.
* You can replace a failed OCR location online if it is not the only OCR location.
* You must update OCR through supported utilities such as:
* Oracle Enterprise Manager
* The Oracle Clusterware Control Utility (`CRSCTL`)
* The Server Control Utility (`SRVCTL`)
* The OCR configuration utility (`OCRCONFIG`)
* The Database Configuration Assistant (`DBCA`)
