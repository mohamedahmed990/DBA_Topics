# Oracle ASM Mirroring and Disk Group Redundancy

---

## Mirroring, Redundancy, and Failure Group Options

If you specify mirroring for a file, then Oracle ASM automatically stores redundant copies of the file extents in separate failure groups. Failure groups apply to normal, high, flex, and extended redundancy disk groups. You can define the failure groups for each disk group when you create or alter the disk group.

There are multiple types of disk groups based on the Oracle ASM redundancy level. **Table 4-2** lists the types with their supported and default mirroring levels. The default mirroring levels indicate the mirroring level with which each file is created unless a different mirroring level is designated.

### Table 4-2: Mirroring options for Oracle ASM disk group types

| Disk Group Type | Supported Mirroring Levels | Default Mirroring Level |
| --- | --- | --- |
| **EXTERNAL redundancy** | Unprotected (none) | Unprotected |
| **NORMAL redundancy** | Two-way, three-way, unprotected (none) | Two-way |
| **HIGH redundancy** | Three-way | Three-way |
| **FLEX redundancy** | Two-way, three-way, unprotected (none) | Two-way (newly-created) |
| **EXTENDED redundancy** | Two-way, three-way, unprotected (none) | Two-way |

---

### Failure Tolerance by Disk Group Type

* **NORMAL and HIGH Redundancy:** The redundancy level controls how many disk failures are tolerated without dismounting the disk group or losing data. Each file is allocated based on its own redundancy, but the default comes from the disk group.
* **FLEX Redundancy:** The number of failures tolerated before dismount depends on the number of failure groups.
* For **five or more** failure groups, **two** disk failures are tolerated.
* For **three or four** failure groups, **one** disk failure is tolerated.


* **EXTENDED Redundancy:** Each site is similar to a flex disk group.
* If a site has **five or more** failure groups, **two** disk failures within that site can be tolerated before the site becomes compromised.
* If a site has **three or four** failure groups, it can tolerate **one** disk failure.
* When **two sites** are compromised, the disk group dismounts. An extended disk group requires a minimum of three failure groups for each data site.



> **Note on FLEX and EXTENDED groups:** Mirroring describes the availability of the files within a disk group, not the disk group itself. For example, if a file is unprotected in a flex disk group that has five failure groups, then after one failure the disk group remains mounted, but that specific file becomes unavailable.

---

### Detailed Redundancy Levels

* **EXTERNAL Redundancy:** Oracle ASM does not provide mirroring redundancy and relies on the storage system to provide RAID functionality. Any write error causes a forced dismount of the disk group. All disks must be located to successfully mount the disk group.
* **NORMAL Redundancy:** Oracle ASM provides two-way mirroring by default, meaning that all files are mirrored so that there are two copies of every extent. A loss of one Oracle ASM disk is tolerated. You can optionally choose three-way or unprotected mirroring.
* *Note:* A file specified with `HIGH` redundancy (three-way mirroring) in a `NORMAL` redundancy disk group provides additional protection from a bad disk sector in one disk plus the failure of another disk. However, this scenario does not protect against the simultaneous failure of two entire disks.


* **HIGH Redundancy:** Oracle ASM provides three-way (triple) mirroring by default. A loss of two Oracle ASM disks in different failure groups is tolerated.
* **FLEX Redundancy:** Oracle ASM provides two-way mirroring by default for newly-created flex disk groups. For migrated flex disk groups, the default values are obtained from the template values in the normal or high redundancy disk groups before migration.
* **EXTENDED Redundancy:** Oracle ASM provides two-way mirroring by default. The redundancy setting describes redundancy *within* a data site. For example, if there is a two-way mirrored file in a two-data-site extended disk group, there will be four copies of the file (two in each data site).

### General Rules

* Oracle ASM file groups in a flex or extended disk group can have different redundancy levels.
* If there are not enough online failure groups to satisfy the file mirroring specified in the disk group file type template, Oracle ASM allocates as many mirror copies as possible and subsequently allocates the remaining mirrors when sufficient online failure groups become available.
* System reliability can diminish if your environment has an insufficient number of failure groups.

---

## Oracle ASM Failure Groups

Failure groups are used to store mirror copies of data. When Oracle ASM allocates an extent for a normal redundancy file, it allocates a primary copy and a secondary copy. Oracle ASM chooses the disk for the secondary copy so that it resides in a different failure group than the primary copy, ensuring that the simultaneous failure of all disks in a single failure group does not result in data loss.

### Characteristics & Recommendations

* **Definition:** A failure group is a subset of disks in a disk group that could fail at the same time because they share hardware (e.g., four drives in a single removable JBOD tray). Drives in the same cabinet can be split into multiple failure groups if the cabinet has redundant power and cooling.
* **Implicit Creation:** If you do not specify a failure group for a disk, Oracle automatically creates a new failure group containing just that single disk (except for disks on Oracle Exadata cells).
* **Minimum Requirements:** A `NORMAL` redundancy disk group must contain at least **two** failure groups. A `HIGH` redundancy disk group must contain at least **three** failure groups.
* **Oracle Recommendations:** * Oracle recommends a minimum of **three** failure groups for `NORMAL` redundancy and **five** failure groups for `HIGH` redundancy disk groups to maintain the necessary number of copies of the Partner Status Table (PST).
* In a system failure, three failure groups in a `NORMAL` redundancy disk group allow a comparison among three PSTs to accurately determine the most up-to-date and correct version, which cannot be achieved with only two.


* **Quorum Failure Groups:** If configuring an extra failure group presents a storage capacity management issue, a quorum failure group can be used to store a copy of the PST. A quorum failure group does not require the same capacity as regular failure groups.

---

## How Oracle ASM Manages Disk Failures

Depending on the redundancy level of a disk group and your failure group definitions, the failure of one or more disks results in one of the following:

1. **Disks Taken Offline and Dropped:** The disks are first taken offline and then automatically dropped. The disk group remains mounted and serviceable. Because of mirroring, all data remains accessible. After the disk drop, Oracle ASM performs a rebalance to restore full redundancy.
2. **Disk Group Dismounted:** The entire disk group is automatically dismounted, causing a total loss of data accessibility.

---

## Guidelines for Using Failure Groups

* Each disk in a disk group can belong to only one failure group.
* Failure groups should all be of the **same size**. Failure groups of different sizes may lead to reduced storage availability.
* Oracle ASM requires at least **two** failure groups for a `NORMAL` redundancy disk group and at least **three** failure groups for a `HIGH` redundancy disk group.

---

## Failure Group Frequently Asked Questions

### How Many Failure Groups Should I Create?

Choosing the number of failure groups depends on the types of failures you must tolerate without data loss.

* For small numbers of disks (fewer than 20), it is usually best to use the default failure group creation that puts every disk in its own failure group.
* This is also applicable for large numbers of disks where your main concern is individual disk failure. However, if you are using modular disk arrays and want to survive the failure of an entire module, a failure group should consist of all disks in that specific module.

### How are Multiple Failure Groups Recovered after Simultaneous Failures?

A simultaneous failure can occur if hardware shared by multiple failure groups fails. This type of failure usually forces a dismount of the disk group if all disks become unavailable.

### When Should External, Normal, or High Redundancy Be Used?

Oracle ASM mirroring runs on the database server. Oracle recommends offloading this processing to the storage hardware RAID controller by using **EXTERNAL** redundancy.

You should use **NORMAL** (or HIGH) redundancy in the following scenarios:

* The storage system does not have a hardware RAID controller.
* You need to mirror across completely separate storage arrays.
* You are deploying Extended Cluster configurations.

> In general, Oracle ASM mirroring is the Oracle alternative to third-party logical volume managers, eliminating additional layers of software complexity in your Oracle Database environment.
