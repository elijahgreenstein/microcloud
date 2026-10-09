---
myst:
  html_meta:
    description: Step-by-step tutorial to learn the basics of how to install, initialize, and use MicroCloud using a physical machine as a single cluster member.
---

(tutorial-single-requirements)=
## Requirements

If you cannot meet the requirements below, try the {ref}`multi-member cluster tutorial<tutorial-multi>`, which provides step-by-step instructions within a confined environment with fewer prerequisites.

(tutorial-single-requirements-general)=
### General

- Use a physical machine for the tutorial with at least 2 GiB of RAM available. This is less than the recommended hardware requirement, but it should be sufficient for this tutorial.
- Ensure that the machine meets the requirements listed in the **General** tab in the {ref}`pre-deployment-requirements`.
- If Docker is installed, uninstall or disable it. Docker can cause networking issues when installed alongside LXD. If you must leave Docker enabled, visit this page for other options: {ref}`network-lxd-docker`.
- MicroCloud cannot be initialized on a machine that already has LXD initialized. If you have an initialized installation of LXD, you must remove it completely. If LXD was installed using its snap, use this command to purge it from your system:

  ```bash
  sudo snap remove --purge lxd
  ```

- If LXD is installed but not initialized, you do not need to remove it. However, ensure that it is running the most recent LTS version. Refer to {ref}`lxd:howto-snap` in the LXD documentation for details.
- If MicroCeph or MicroOVN are already installed, ensure that they are also running their most recent LTS versions. To ensure that the versions are compatible for MicroCloud, refer to {ref}`ref-releases-matrix`.
- Due to variation in physical machine setups, it is beyond the scope of this tutorial to instruct you on how to set up your network interfaces and storage disks. Thus, this tutorial requires a higher level of knowledge of server management on your part. If you require step-by-step instructions, follow the {ref}`multi-member cluster tutorial<tutorial-multi>` instead.

(tutorial-single-requirements-storage)=
### Storage requirements

MicroCloud supports both local and remote storage. For remote storage (also called distributed storage), you need one additional physical disk attached to the machine, such as an external SSD. This disk must be free of partitions and file systems, and able to be wiped. At least 10 GiB of storage space is recommended.

To configure local storage as well, you'll need a second disk attached to that machine that meets the same requirements. If you only have one additional disk to use, you must use it for remote storage.

(tutorial-single-requirements-network)=
### Network requirements

Two network interfaces must be configured on your machine, such as with a dual-port NIC:

- One interface is used for an uplink network that provides external connectivity to cluster members. The network interface for the uplink network must support both broadcast and multicast, and it must not have any IP addresses bound directly to it.
- The other interface is used for internal (or intra-cluster) communication, meaning communication between MicroCloud cluster members. It must have assigned IPs.

  Even though we are setting up a cluster with a single member, this network is still required by MicroCloud. This is because it uses an address on this network to bind its services, as well as those of its components.

During the MicroCloud initialization process, you'll be asked for the IPv4 and IPv6 gateway addresses for the uplink network. Be prepared to provide at least one; you can optionally provide both. If you provide the IPv4 gateway address, you'll need to provide the IPv4 subnet ranges as well.

