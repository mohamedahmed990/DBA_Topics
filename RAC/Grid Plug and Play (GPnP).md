
---

# Grid Plug and Play (GPnP) – Oracle 11.2 Feature

## Overview

Grid Plug and Play (GPnP) simplifies cluster node administration by centralizing bootstrap configuration in a profile rather than relying on node-specific configuration files.

---

## GPnP Profile (`profile.xml`)

### Location & Management

* **Path:** `$GRID_HOME/gpnp/<hostname>/profiles/peer/profile.xml`
* **Storage:** Stored in local OCR and cluster OCR.
* **Wallet Path:** `/u01/app/11203/grid/gpnp/<hostname>/wallets/`
* **Daemon:** If errors occur, the GPnP Daemon (`gpnpd`) recreates the profile.
* ⚠️ **Warning:** **Never change `profile.xml` directly.** Use administration tools such as:
* `asmcmd`
* `oifcfg`
* `ASMCA`
* OUI (Oracle Universal Installer)



### Characteristics & Attributes

The GPnP profile contains bootstrap information used to establish the global personality of a node. It contains no node-specific information and is maintained across all nodes in the GPnP cache.

Key attributes included in the profile:

* Cluster name
* Network classifications (Public / Private / Interconnect)
* Storage locations for CSS
* Storage parameters for ASM (SPFILE location, `ASM_DISKSTRING`, etc.)
* Digital signature information

---

## Profile XML Sample Output

When formatted using `gpnptool get`:

```xml
ProfileSequence="4"
ClusterUId="2ae3c3415014ef2abf2ff662c5bf8512"
ClusterName="GRACE2"

<gpnp:Network-Profile>
  <gpnp:HostNetwork id="gen" HostName="*">
    <gpnp:Network id="net1" IP="192.168.1.0" Adapter="eth0" Use="public"/>
    <gpnp:Network id="net2" IP="192.168.2.0" Adapter="eth1" Use="cluster_interconnect"/>
  </gpnp:HostNetwork>
</gpnp:Network-Profile>

<orcl:CSS-Profile id="css" DiscoveryString="+asm" LeaseDuration="400"/>
<orcl:ASM-Profile id="asm" DiscoveryString="/dev/oracleasm/disks/*" SPFile="+DATA/grace2/asmparameterfile/registry.253.821039237"/>
<ds:Signature xmlns:ds="http://www.w3.org/2000/09/xmldsig#">
  ...
</ds:Signature>

```

---

## Operations & Cluster Startup Flow

### When the Profile Updates

The profile is updated whenever changes are made via tools like:

* `oifcfg` (Change network settings)
* `crsctl` (Change Voting Disk location)
* `asmcmd` (Change `ASM_DISKSTRING` or SPFILE location)

### Startup Sequence

1. **Clusterware Boot:** Accesses Voting Disks using information read from `<orcl:CSS-Profile DiscoveryString="...">` via `kfed` (even if ASM is down).
2. **Node Join:** Clusterware verifies profile consistency across all nodes.
* *Existing Node:* Reads local cached `profile.xml`.
* *New Node:* Uses mDNS multicast to locate an existing GPnP agent and retrieve the profile.


3. **CRSD & ASM SPfile Location Search Order:**
1. GPnP profile
2. `$ORACLE_HOME/dbs/spfile<sid>.ora`
3. `$ORACLE_HOME/dbs/init<sid>.ora`



---

## `gpnptool` Command Reference

### Basic Utility Commands

* **Read formatted GPnP profile:**
```bash
$GRID_HOME/bin/gpnptool get

```


* **Check if local GPnP daemon is running:**
```bash
$GRID_HOME/bin/gpnptool lfind

```


* **Find ASM SPfile location when ASM is down:**
```bash
$GRID_HOME/bin/gpnptool getpval -asm_spf -p=$GRID_HOME/gpnp/grac1/profiles/peer/profile.xml

```


* **Validate profile configuration:**
```bash
$GRID_HOME/bin/gpnptool check -p=$GRID_HOME/gpnp/grac1/profiles/peer/profile.xml

```


* **Verify profile digital signature:**
```bash
$GRID_HOME/bin/gpnptool verify -p=$GRID_HOME/gpnp/grac1/profiles/peer/profile.xml \
  -w="file://$GRID_HOME/gpnp/grac1/wallets/peer" -wu=peer

```


* **Check responsiveness of a specific host daemon:**
```bash
$GRID_HOME/bin/gpnptool find -h=grac2

```


* **Verify all cluster peers are responding:**
```bash
$GRID_HOME/bin/gpnptool find -c=GRACE2

```



---

## Useful Shell One-Liners & Scripts

### Extract Profile Sequence & Cluster Name

```bash
$GRID_HOME/bin/gpnptool get 2>/dev/null | xmllint --format - | awk '/ProfileSequence/ { printf("%s %s\n", $9,$11); }'

```

### Extract Network, CSS, and ASM Data

```bash
$GRID_HOME/bin/gpnptool get 2>/dev/null | xmllint --format - | egrep 'CSS-Profile|ASM-Profile|Network id'

```

### Script: `get_profile.sh`

Compare profile parameters across multiple cluster nodes via SSH:

```bash
#!/bin/bash
host1=grac41
host2=grac42
host3=grac43

echo "*** GPnP Info - Verify profile.xml on all nodes"

# Print Profile Sequence & Cluster Name across nodes
ssh $host1 /bin/hostname; $GRID_HOME/bin/gpnptool get 2>/dev/null | xmllint --format - | awk '/ProfileSequence/ { printf("%s %s\n", $9,$11); }'
ssh $host2 /bin/hostname; $GRID_HOME/bin/gpnptool get 2>/dev/null | xmllint --format - | awk '/ProfileSequence/ { printf("%s %s\n", $9,$11); }'
ssh $host3 /bin/hostname; $GRID_HOME/bin/gpnptool get 2>/dev/null | xmllint --format - | awk '/ProfileSequence/ { printf("%s %s\n", $9,$11); }'

# Print CSS, ASM, and Network details across nodes
ssh $host1 /bin/hostname; $GRID_HOME/bin/gpnptool get 2>/dev/null | xmllint --format - | egrep 'CSS-Profile|ASM-Profile|Network id'
ssh $host2 /bin/hostname; $GRID_HOME/bin/gpnptool get 2>/dev/null | xmllint --format - | egrep 'CSS-Profile|ASM-Profile|Network id'
ssh $host3 /bin/hostname; $GRID_HOME/bin/gpnptool get 2>/dev/null | xmllint --format - | egrep 'CSS-Profile|ASM-Profile|Network id'

```

---

## Updating `profile.xml` Manually

To manually modify `profile.xml` in emergency/recovery scenarios, follow the sequence:

1. `gpnptool unsign`
2. `gpnptool edit`
3. `gpnptool sign`
4. `gpnptool put`
