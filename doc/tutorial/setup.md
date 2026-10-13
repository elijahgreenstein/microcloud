---
myst:
  html_meta:
    description: Step-by-step tutorial to learn the basics of how to install, initialize, and use MicroCloud using a physical machine as a single cluster member.
---

(tutorial-prereqs-setup)=
# Prerequisites and setup

A production MicroCloud should use at least three physical machines as cluster members.

To complete this tutorial, however, you can use either physical or virtual machines as cluster numbers.
The minimum requirements to complete the tutorial with physical machines are detailed below, along with setup instructions.
If you only have a single machine available, then you can set up virtual machines for the tutorial by following the steps in {ref}`tutorial-setup-vms`.

(tutorial-physical-requirements)=
## Physical machine requirements

To follow the tutorial with physical machines, each machine must meet the requirements below.
If you cannot meet these requirements, or if you only have one machine available, follow the steps in {ref}`tutorial-setup-vms` to set up a confined environment in which you can practice setting up a MicroCloud.

```{warning}
The requirements below are suitable for the tutorial but are insufficient for a production MicroCloud.
For details about requirements and recommendations for production, see the {ref}`reference-requirements` reference.
```

(tutorial-physical-requirements-general)=
### General

The tutorial requires three physical machines that meet the following requirements:

- 2 GiB of RAM available on each machine
- Ensure that the machines meets the requirements listed in the **General** tab in the {ref}`pre-deployment-requirements`.
- If Docker is installed, uninstall or disable it. Docker can cause networking issues when installed alongside LXD. If you must leave Docker enabled, visit this page for other options: {ref}`network-lxd-docker`.
- MicroCloud cannot be initialized on a machine that already has LXD initialized. If you have an initialized installation of LXD, you must remove it completely. If LXD was installed using its snap, use this command to purge it from your system:

  ```bash
  sudo snap remove --purge lxd
  ```

- If LXD is installed but not initialized, you do not need to remove it. However, ensure that it is running the most recent LTS version. Refer to {ref}`lxd:howto-snap` in the LXD documentation for details.
- If MicroCeph or MicroOVN are already installed, ensure that they are also running their most recent LTS versions. To ensure that the versions are compatible for MicroCloud, refer to {ref}`ref-releases-matrix`.
- Due to variation in physical machine setups, it is beyond the scope of this tutorial to instruct you on how to set up your network interfaces and storage disks. Thus, this tutorial requires a higher level of knowledge of server management on your part. If you require step-by-step instructions, {ref}`set up virtual machines <tutorial-setup-vms>` instead so that you can still follow the tutorial in a confined environment.

(tutorial-physical-requirements-storage)=
### Storage requirements

MicroCloud supports both local and remote storage. For remote storage (also called distributed storage), you need one additional physical disk attached to each machine, such as an external SSD. These disks must be free of partitions and file systems, and able to be wiped. At least 10 GiB of storage space is recommended for the tutorial.

To configure local storage as well, you'll need a second disk attached to each machine that meets the same requirements. If you only have one additional disk to use, you must use it for remote storage.

(tutorial-single-requirements-network)=
### Network requirements

Two network interfaces must be configured on each machine, such as with a dual-port NIC:

- One interface is used for an uplink network that provides external connectivity to cluster members. The network interface for the uplink network must support both broadcast and multicast, and it must not have any IP addresses bound directly to it.
- The other interface is used for internal (or intra-cluster) communication, meaning communication between MicroCloud cluster members. It must have assigned IPs.

During the MicroCloud initialization process, you'll be asked for the IPv4 and IPv6 gateway addresses for the uplink network. Be prepared to provide at least one; you can optionally provide both. If you provide the IPv4 gateway address, you'll need to provide the IPv4 subnet ranges as well.

(tutorial-setup-vms)=
## Set up virtual machines for the tutorial

```{warning}
Use physical machines in a production environment.
Only use virtual machines as cluster members in testing or development environments, such as this tutorial.
```

If you do not have enough physical machines to set up a three-member MicroCloud, you can follow these steps to set up virtual machines instead.

### Prerequisites

- Ubuntu, or another Linux distribution that supports LXD
- Enough storage to support four virtual machines; at least 40GiB of available storage is recommended

### 1. Install and initialize LXD

MicroCloud requires LXD version 5.21 or later.
These instructions provide the steps for installing the LXD snap.
Visit {ref}`lxd:installing` in the LXD documentation for additional installation options.


1. Install `snapd`:

   1. Run {command}`snap version` to find out if snap is installed on your system:

      ```{terminal}
      :user: ubuntu
      :host: lxd-host
      snap version

      snap    2.59.4
      snapd   2.59.4
      series  16
      ubuntu  24.04
      kernel  5.15.0-73-generic
      ```

      If you see a table of version numbers, snap is installed.
      If the version for `snapd` is 2.59 or later, you are all set and can continue with the next step of installing LXD.

   1. If the version for `snapd` is earlier than 2.59, or if the {command}`snap version` command returns an error, run the following commands to install the latest version of `snapd`:

          sudo apt update
          sudo apt install snapd

1. If LXD is already installed, enter the following command to update it to the most recent LTS version (LXD 6):

       sudo snap refresh lxd --channel=6/stable

   Otherwise, enter the following command to install LXD:

       sudo snap install lxd

1. Add the current user to the `lxd` group

   ```{caution}
   Local access to LXD through the Unix socket always grants full access to LXD.
   This includes the ability to attach file system paths or devices to any instance as well as tweak the security features on any instance.

   Therefore, you should only give such access to users who you'd trust with root access to your system.

   For more information, see {ref}`lxd:security-daemon-access`.
   ```

   Installing LXD through its snap should automatically create a `lxd` group on your system.
   The user you are logged in as must be in this group to interact with LXD.

   Enter the following command to add your user to the `lxd` group:

   ```bash
   sudo usermod -aG lxd "$USER"
   ```

   Then restart your machine to apply the change.

1. Enter the following command to initialize LXD:

       lxd init

   Accept the default values except for the following questions:

   - `Size in GiB of the new loop device (1GiB minimum)`

     Enter `40`.

   - `Would you like the LXD server to be available over the network? (yes/no)`

     Enter `yes`.

1. Modify the default network so we can later define specific IPv6 addresses for the VMs:

       lxc network set lxdbr0 ipv6.dhcp.stateful true

### 2. Provide storage disks

MicroCloud supports both local and remote storage.
For local storage, you need one disk per cluster member.
For remote storage with high availability (HA), you need at least three disks that are located across three different cluster members.

In this tutorial, we will set up each of the three cluster members with local and remote storage.

Complete the following steps to create the required disks in a LXD storage pool:

1. Create a ZFS storage pool called `disks`:

       lxc storage create disks zfs size=100GiB

1. Configure the default volume size for the `disks` pool:

       lxc storage set disks volume.size 10GiB

1. Create three disks to use for local storage:

       lxc storage volume create disks local1 --type block
       lxc storage volume create disks local2 --type block
       lxc storage volume create disks local3 --type block

1. Create three disks to use for remote storage:

       lxc storage volume create disks remote1 --type block size=20GiB
       lxc storage volume create disks remote2 --type block size=20GiB
       lxc storage volume create disks remote3 --type block size=20GiB

1. Check that the disks have been created correctly:

   ```{terminal}
   :user: ubuntu
   :host: lxd-host
   lxc storage volume list disks

   +--------+---------+-------------+--------------+---------+
   |  TYPE  |  NAME   | DESCRIPTION | CONTENT-TYPE | USED BY |
   +--------+---------+-------------+--------------+---------+
   | custom | local1  |             | block        | 0       |
   +--------+---------+-------------+--------------+---------+
   | custom | local2  |             | block        | 0       |
   +--------+---------+-------------+--------------+---------+
   | custom | local3  |             | block        | 0       |
   +--------+---------+-------------+--------------+---------+
   | custom | remote1 |             | block        | 0       |
   +--------+---------+-------------+--------------+---------+
   | custom | remote2 |             | block        | 0       |
   +--------+---------+-------------+--------------+---------+
   | custom | remote3 |             | block        | 0       |
   +--------+---------+-------------+--------------+---------+
   ```

### 3. Create a network

MicroCloud requires an uplink network that the cluster members can use for external connectivity.
See {ref}`exp-networking-ovn-architecture` for more information.

Complete the following steps to set up this network:

1. Create a bridge network without any parameters:

       lxc network create microbr0

      (tutorial-note-ips)=
1. Enter the following commands to find out the assigned IPv4 and IPv6 addresses for the network, and note them down:

       lxc network get microbr0 ipv4.address
       lxc network get microbr0 ipv6.address

### 4. Create and configure your VMs

Next, we'll create the VMs that will serve as the MicroCloud cluster members.

Complete the following steps:

1. Create the VMs, but don't start them yet:

       lxc init ubuntu:24.04 micro1 --vm --config limits.cpu=2 --config limits.memory=2GiB
       lxc init ubuntu:24.04 micro2 --vm --config limits.cpu=2 --config limits.memory=2GiB
       lxc init ubuntu:24.04 micro3 --vm --config limits.cpu=2 --config limits.memory=2GiB

   ```{tip}
   Run these commands in sequence, not in parallel.

   LXD downloads the image the first time you use it to initialize a VM.
   For subsequent runs, LXD uses the cached image.
   Therefore, the {command}`init` command will take longer to complete on the first run.
   ```

1. Attach the disks to the VMs:

       lxc storage volume attach disks local1 micro1
       lxc storage volume attach disks local2 micro2
       lxc storage volume attach disks local3 micro3
       lxc storage volume attach disks remote1 micro1
       lxc storage volume attach disks remote2 micro2
       lxc storage volume attach disks remote3 micro3

1. Create and add network interfaces that use the dedicated MicroCloud network to each VM:

       lxc config device add micro1 eth1 nic network=microbr0
       lxc config device add micro2 eth1 nic network=microbr0
       lxc config device add micro3 eth1 nic network=microbr0

1. Start the VMs:

       lxc start micro1
       lxc start micro2
       lxc start micro3

### 5. Access virtual machines

Once your virtual machines are started, you can use the following command to gain shell access to a VM, with a prompt that should match the prompts used in this tutorial (such as `ubuntu@micro1:~$`):

```bash
lxc exec <vm-name> -- su - ubuntu
```

You can also open three different terminals, one for each VM, so that you can run commands on each machine in parallel.

### 6. Configure network interfaces

MicroCloud requires a network interface that doesn't have an IP address assigned.
Thus, you must configure the network interface connected to `microbr0` to refuse any IP addresses.

Run the following command in each virtual machine:

```bash
cat << EOF > /etc/netplan/99-microcloud.yaml
# MicroCloud requires a network interface that doesn't have an IP address
network:
    version: 2
    ethernets:
        enp6s0:
            accept-ra: false
            dhcp4: false
            link-local: []
EOF
chmod 0600 /etc/netplan/99-microcloud.yaml
```

`enp6s0` is the name that the VM assigns to the network interface that we previously added as `eth1`.

Now bring the network interface up:

```bash
netplan apply
```

## Next steps

Once your physical or virtual machines are set up, you're ready to set up your MicroCloud.
Proceed to {ref}`tutorial-start`.

