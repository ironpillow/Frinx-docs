[Documentation main page](https://frinxio.github.io/Frinx-docs/)
[Carbon Release Notes main page](https://frinxio.github.io/Frinx-docs/FRINX_ODL_Distribution/Carbon/release_notes.html)

This document describes the latest changes, additions, known issues, and fixes for the Frinx ODL Distribution.<!--more-->

**Note that FRINX ODL distribution 3.1.5 requires Java 8 (Openjdk 1.8.0-171 or newer)**
To install Java:  
Ubuntu: In a terminal type

    sudo apt-get install openjdk-8-jre

CentOS: In a terminal type

    sudo yum install java-1.8.0-openjdk

#### New Features, Improvements
* CLI and UNITOPO units (Device features)
    - LLDP unit added for IOS XR over CLI (version 4, 5, 6) - Enables (operational) neighbor information to be pulled from an XR device
    - Initial (plaintext *execute-and-read* RPC) support for Cisco NEXUS 9k switches - Enables basic (plaintext) management of Cisco Nexus switches
    - Bugfixing
* CLI topology
    - Scaling improvements - CLI topology can now quickly and easily mount & manage thousands of devices
    - Last-fail-cause added to operational cli topology - This field shows the reason, why a connect attempt failed when mounting a device over cli e.g. no route to host
* Unified topology
    - Optimize direct-unit proxy - Do not use binding aware APIs in the direct-unit proxy. Instead directly delegate calls to southbound layer.
    - It is no longer necessary to add generated classes for new openconfig models to the direct unit.
* Uniconfig
    - Enable parallel execution in uniconfig-node-manager - In case multiple tasks are submitted to uniconfig-node-manager, it will execute them in parallel as long as they do not overlap in terms of devices accessed.
    - Number of threads to use for parallel execution can be configured in uniconfig-node-manager’s blueprint file.
    - If there is no free thread to execute a task, the task will be enqueued.
* LLDP Topology app (new component)
    - New application on top of unified layer, which builds a network topology view from operational LLDP data.
    - Current support for Cisco IOS, IOS XE and IOS XR devices.
    - LLDP topology app video: https://www.youtube.com/watch?v=bN3t7toJrCs.

#### Known Issues
1. odl-netconf-clustered-topology:
    - Contains critical bugs and is not intended for production use, so odl-netconf-topology was modified by FRINX so that it can work in cluster. FRINX recommends using odl-netconf-topology in production environments.
2. restconf/operational/entity-owners:
    - entity-owners contains no data as entity ownership service was rewritten. Entity owners are assigned to the same node that hosts shard leaders.
3. CLI telnet connectivity with reverse telnet on Cisco devices is not supported in this release.
4. L2/3VPN service modules are supported on single node ODL.
5. Readers returning default data for non-existent instances.
    - When a specific query is issued for a child readers e.g. AreaReader in OSPF for XR, it will return default data back instead of a 404 response.
6.  Update in CLI translation units does not work properly - it invokes delete and create operations by default

#### Opendaylight Carbon Release Notes
The Frinx controller 3.1.5 is based on OpenDaylight Carbon.

<https://wiki.opendaylight.org/view/Simultaneous_Release/Carbon/Release_Notes>
<https://wiki.opendaylight.org/view/Simultaneous_Release:Carbon_Release_Plan>
<https://wiki.opendaylight.org/view/BGP_LS_PCEP:Carbon_Release_Notes>
