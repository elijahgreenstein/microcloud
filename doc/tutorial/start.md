(tutorial-start)=

# Install and initialize MicroCloud

After you have {ref}`set up physical or virtual machines <tutorial-prereqs-setup>`, you can install and initialize MicroCloud on each machine.

In this tutorial, the three machines are referred to as `micro1`, `micro2`, and `micro3`.
If you are using virtual machines (VMs), these correspond to the names used in the setup process.
If you are using physical machines, then you should consistently run machines for `micro1` on one machine, and those for `micro2` and `micro3` on the other two machines.

To complete this part of the tutorial, you will need the IP address of each machine.
If you are completing this tutorial with VMs, you can find the IPv4 addresses assigned to the VMs you set up in the {ref}`setup stage of the tutorial <tutorial-setup-vms>` by running `lxc list`.

## Install MicroCloud

Before you can create the MicroCloud cluster, you must install the required snaps.
Run the following commands on each machine:

```bash
sudo snap install lxd --channel=6/stable --cohort="+"
sudo snap install microceph --channel=tentacle/stable --cohort="+"
sudo snap install microovn --channel=26.03/stable --cohort="+"
sudo snap install microcloud --channel=3/stable --cohort="+"
```

The `--cohort="+"` flag in the command ensures that the same version of the snap is installed on all machines.
See {ref}`howto-update-sync` for more information.

## Initialize MicroCloud

You can initialize MicroCloud either interactively or non-interactively.
The steps below detail how to use the interactive initialization process; for the non-interactive process, see {ref}`howto-initialize-preseed`.

1. Access the shell in `micro1` and start the initialization process:

   ```bash
   microcloud init
   ```

1. Answer the questions:

   1. Select `yes` to select more than one cluster member.
   1. As the address for MicroCloud's internal traffic, select the listed IPv4 address.
   1. Copy the session passphrase.

1. Start the join process for the `micro2` machine:

   ```bash
   microcloud join
   ```

   Select the listed IPv4 address for MicroCloud's internal traffic.

   When prompted, enter the session passphrase.

1. Repeat the above step for the `micro3` machine.

1. Return to `micro1` to continue the initialization process:

   1. Select all listed systems to join the cluster. These should be `micro2` and `micro3`.
   1. Select `yes` to set up local storage.
   1. Select the listed local disks.
      If you set up virtual machines, these should be `local1`, `local2`, and `local3`.

      ```{tip}
      The table is filtered by the characters that you type.
      ```

   1. Press {kbd}`Enter` without selecting any disks to wipe.
   1. Select `yes` to set up distributed storage.
   1. Select `yes` to confirm that there are fewer disks available than machines.
   1. Select all listed disks.
      If you set up virtual machines, these should be `remote1` and  `remote2`.
   1. Press {kbd}`Enter` without selecting any disks to wipe.
   1. You don't need to encrypt any disks to get started.
   1. Select `yes` to optionally configure the CephFS distributed file system.
   1. Press {kbd}`Enter` to accept the default option for the IPv4 or IPv6 CIDR subnet address used for the Ceph internal network.
   1. Press {kbd}`Enter` to accept the default option for the IPv4 or IPv6 CIDR subnet address used for the Ceph public network.
   1. Select `yes` to configure distributed networking.
   1. Select all listed network interfaces.
      If you set up virtual machines, these should be `enp6s0` on all machines.
   1. Specify the IPv4 address that {ref}`you noted down<tutorial-note-ips>` for your `microbr0` network's IPv4 gateway.
   1. Specify an IPv4 address in the address range as the first IPv4 address.
      For example, if your IPv4 gateway is `192.0.2.1/24`, the first address could be `192.0.2.100`.
   1. Specify a higher IPv4 address in the range as the last IPv4 address.
      As we're setting up three machines only, the range must contain a minimum of three addresses, but setting up a bigger range is more fail-safe.
      For example, if your IPv4 gateway is `192.0.2.1/24`, the last address could be `192.0.2.254`.
   1. Specify the IPv6 address that {ref}`you noted down <tutorial-note-ips>` for your `microbr0` network as the IPv6 gateway.
   1. Press {kbd}`Enter` to accept the default option for the DNS addresses for the distributed network.
   1. Press {kbd}`Enter` to accept the default option for configuring an underlay network for OVN.

MicroCloud will now initialize the cluster.
See {ref}`explanation-initialization` for more information.

(initialization-process)=

### Example initialization process

See the full process here for the initiating side; the addresses and names may vary depending on your setup.

```{terminal}
:user: root
:host: micro1
:scroll:
microcloud init

Do you want to set up more than one cluster member? (yes/no) [default=yes]: yes
Select an address for MicroCloud's internal traffic:
Space to select; enter to confirm; type to filter results.
Up/down to move; right to select all; left to select none.
       +----------------------+--------+
       |       ADDRESS        | IFACE  |
       +----------------------+--------+
> [X]  | 10.1.123.10          | enp5s0 |
  [ ]  | fd42:1:1234:1234::10 | enp5s0 |
       +----------------------+--------+

 Using address "10.1.123.10" for MicroCloud

Use the following command on systems that you want to join the cluster:

 microcloud join

When requested enter the passphrase:

 koala absorbing update dorsal

Verify the fingerprint "5d0808de679d" is displayed on joining systems.
Waiting to detect systems ...
Systems will appear in the table as they are detected. Select those that should join the cluster:
Space to select; enter to confirm; type to filter results.
Up/down to move; right to select all; left to select none.
       +---------+-------------+--------------+
       |  NAME   |   ADDRESS   | FINGERPRINT  |
       +---------+-------------+--------------+
> [x]  | micro3  | 10.1.123.30 | 4e80954d6a64 |
  [x]  | micro2  | 10.1.123.20 | 84e0b50e13b3 |
       +---------+-------------+--------------+

 Selected "micro1" at "10.1.123.10"
 Selected "micro3" at "10.1.123.30"
 Selected "micro2" at "10.1.123.20"

Would you like to set up local storage? (yes/no) [default=yes]: yes
Select exactly one disk from each cluster member:
Space to select; enter to confirm; type to filter results.
Up/down to move; right to select all; left to select none.
       +----------+---------------+----------+------+------------------------------------------------------+
       | LOCATION |     MODEL     | CAPACITY | TYPE |                         PATH                         |
       +----------+---------------+----------+------+------------------------------------------------------+
  [x]  | micro1   | QEMU HARDDISK | 10.00GiB | scsi | /dev/disk/by-id/scsi-0QEMU_QEMU_HARDDISK_lxd_local1  |
  [ ]  | micro1   | QEMU HARDDISK | 20.00GiB | scsi | /dev/disk/by-id/scsi-0QEMU_QEMU_HARDDISK_lxd_remote1 |
  [x]  | micro2   | QEMU HARDDISK | 10.00GiB | scsi | /dev/disk/by-id/scsi-0QEMU_QEMU_HARDDISK_lxd_local2  |
  [ ]  | micro2   | QEMU HARDDISK | 20.00GiB | scsi | /dev/disk/by-id/scsi-0QEMU_QEMU_HARDDISK_lxd_remote2 |
  [x]  | micro3   | QEMU HARDDISK | 10.00GiB | scsi | /dev/disk/by-id/scsi-0QEMU_QEMU_HARDDISK_lxd_local3  |
  [ ]  | micro3   | QEMU HARDDISK | 20.00GiB | scsi | /dev/disk/by-id/scsi-0QEMU_QEMU_HARDDISK_lxd_remote3 |
       +----------+---------------+----------+------+------------------------------------------------------+

Select which disks to wipe:
Space to select; enter to confirm; type to filter results.
Up/down to move; right to select all; left to select none.
       +----------+---------------+----------+------+------------------------------------------------------+
       | LOCATION |     MODEL     | CAPACITY | TYPE |                         PATH                         |
       +----------+---------------+----------+------+------------------------------------------------------+
> [ ]  | micro1   | QEMU HARDDISK | 10.00GiB | scsi | /dev/disk/by-id/scsi-0QEMU_QEMU_HARDDISK_lxd_local1  |
  [ ]  | micro2   | QEMU HARDDISK | 10.00GiB | scsi | /dev/disk/by-id/scsi-0QEMU_QEMU_HARDDISK_lxd_local2  |
  [ ]  | micro3   | QEMU HARDDISK | 10.00GiB | scsi | /dev/disk/by-id/scsi-0QEMU_QEMU_HARDDISK_lxd_local3  |
       +----------+---------------+----------+------+------------------------------------------------------+

 Using "/dev/disk/by-id/scsi-0QEMU_QEMU_HARDDISK_lxd_local3" on "micro3" for local storage pool
 Using "/dev/disk/by-id/scsi-0QEMU_QEMU_HARDDISK_lxd_local1" on "micro1" for local storage pool
 Using "/dev/disk/by-id/scsi-0QEMU_QEMU_HARDDISK_lxd_local2" on "micro2" for local storage pool

Would you like to set up distributed storage? (yes/no) [default=yes]: yes
Unable to find disks on some systems. Continue anyway? (yes/no) [default=yes]: yes
Select from the available unpartitioned disks:
Space to select; enter to confirm; type to filter results.
Up/down to move; right to select all; left to select none.
       +----------+---------------+----------+------+------------------------------------------------------+
       | LOCATION |     MODEL     | CAPACITY | TYPE |                         PATH                         |
       +----------+---------------+----------+------+------------------------------------------------------+
> [x]  | micro1   | QEMU HARDDISK | 20.00GiB | scsi | /dev/disk/by-id/scsi-0QEMU_QEMU_HARDDISK_lxd_remote1 |
  [x]  | micro2   | QEMU HARDDISK | 20.00GiB | scsi | /dev/disk/by-id/scsi-0QEMU_QEMU_HARDDISK_lxd_remote2 |
  [x]  | micro3   | QEMU HARDDISK | 20.00GiB | scsi | /dev/disk/by-id/scsi-0QEMU_QEMU_HARDDISK_lxd_remote3 |
       +----------+---------------+----------+------+------------------------------------------------------+

Select which disks to wipe:
Space to select; enter to confirm; type to filter results.
Up/down to move; right to select all; left to select none.
       +----------+---------------+----------+------+------------------------------------------------------+
       | LOCATION |     MODEL     | CAPACITY | TYPE |                         PATH                         |
       +----------+---------------+----------+------+------------------------------------------------------+
> [ ]  | micro1   | QEMU HARDDISK | 20.00GiB | scsi | /dev/disk/by-id/scsi-0QEMU_QEMU_HARDDISK_lxd_remote1 |
  [ ]  | micro2   | QEMU HARDDISK | 20.00GiB | scsi | /dev/disk/by-id/scsi-0QEMU_QEMU_HARDDISK_lxd_remote2 |
  [ ]  | micro3   | QEMU HARDDISK | 20.00GiB | scsi | /dev/disk/by-id/scsi-0QEMU_QEMU_HARDDISK_lxd_remote3 |
       +----------+---------------+----------+------+------------------------------------------------------+

 Using 1 disk(s) on "micro1" for remote storage pool
 Using 1 disk(s) on "micro2" for remote storage pool
 Using 1 disk(s) on "micro3" for remote storage pool

Do you want to encrypt the selected disks? (yes/no) [default=no]: no
Would you like to set up CephFS remote storage? (yes/no) [default=yes]:  yes
What subnet (either IPv4 or IPv6 CIDR notation) would you like your Ceph internal traffic on? [default: 10.1.123.0/24]:
What subnet (either IPv4 or IPv6 CIDR notation) would you like your Ceph public traffic on? [default: 10.1.123.0/24]:
Configure distributed networking? (yes/no) [default=yes]:  yes
Select an available interface per system to provide external connectivity for distributed network(s):
Space to select; enter to confirm; type to filter results.
Up/down to move; right to select all; left to select none.
       +----------+--------+----------+
       | LOCATION | IFACE  |   TYPE   |
       +----------+--------+----------+
> [x]  | micro2   | enp6s0 | physical |
  [x]  | micro3   | enp6s0 | physical |
  [x]  | micro1   | enp6s0 | physical |
       +----------+--------+----------+

 Using "enp6s0" on "micro3" for OVN uplink
 Using "enp6s0" on "micro1" for OVN uplink
 Using "enp6s0" on "micro2" for OVN uplink

Specify the IPv4 gateway (CIDR) on the uplink network (empty to skip IPv4): 192.0.2.1/24
Specify the first IPv4 address in the range to use on the uplink network: 192.0.2.100
Specify the last IPv4 address in the range to use on the uplink network: 192.0.2.254
Specify the IPv6 gateway (CIDR) on the uplink network (empty to skip IPv6): 2001:db8:d:200::1/64
Specify the DNS addresses (comma-separated IPv4 / IPv6 addresses) for the distributed network (default: 192.0.2.1,2001:db8:d:200::1):
Configure dedicated underlay networking? (yes/no) [default=no]:

Initializing new services
 Local MicroCloud is ready
 Local LXD is ready
 Local MicroOVN is ready
 Local MicroCeph is ready
Awaiting cluster formation ...
 Peer "micro2" has joined the cluster
 Peer "micro3" has joined the cluster
Configuring cluster-wide devices ...
MicroCloud is ready
```

See the full process here for one of the joining sides (`micro2`):

```{terminal}
:user: root
:host: micro2
:scroll:
microcloud join

Select an address for MicroCloud's internal traffic:
Space to select; enter to confirm; type to filter results.
Up/down to move; right to select all; left to select none.
       +----------------------+--------+
       |        ADDRESS       | IFACE  |
       +----------------------+--------+
> [ ]  | 10.1.123.20          | enp5s0 |
  [ ]  | fd42:1:1234:1234::20 | enp5s0 |
       +----------------------+--------+

Using address "10.1.123.20" for MicroCloud

Verify the fingerprint "84e0b50e13b3" is displayed on the other system.
Specify the passphrase for joining the system: koala absorbing update dorsal
Searching for an eligible system ...

 Found system "micro1" at "10.1.123.10" using fingerprint "5d0808de679d"

Select "micro2" on "micro1" to let it join the cluster

 Received confirmation from system "micro1"

Do not exit out to keep the session alive.
Complete the remaining configuration on "micro1" ...
Successfully joined the MicroCloud cluster and closing the session.
Commencing cluster join of the remaining services (LXD, MicroCeph, MicroOVN)
```

## Inspect your MicroCloud setup

You can now inspect your cluster setup.

The examples below use `micro1`, but you can run these commands on any of the cluster members.
The output of these commands may vary depending on your setup, and whether or not you are using physical or virtual machines.

1. Inspect the cluster setup:

   ```{terminal}
   :user: root
   :host: micro1
   :scroll:
   lxc cluster list

   +--------+--------------------------+------------------+--------------+----------------+-------------+--------+-------------------+
   |  NAME  |            URL           |      ROLES       | ARCHITECTURE | FAILURE DOMAIN | DESCRIPTION | STATE  |      MESSAGE      |
   +--------+--------------------------+------------------+--------------+----------------+-------------+--------+-------------------+
   | micro1 | https://10.1.123.10:8443 | database-leader  | x86_64       | default        |             | ONLINE | Fully operational |
   |        |                          | database         |              |                |             |        |                   |
   +--------+--------------------------+------------------+--------------+----------------+-------------+--------+-------------------+
   | micro2 | https://10.1.123.20:8443 | database         | x86_64       | default        |             | ONLINE | Fully operational |
   +--------+--------------------------+------------------+--------------+----------------+-------------+--------+-------------------+
   | micro3 | https://10.1.123.30:8443 | database         | x86_64       | default        |             | ONLINE | Fully operational |
   +--------+--------------------------+------------------+--------------+----------------+-------------+--------+-------------------+
   ```

   ```{terminal}
   :user: root
   :host: micro1
   :scroll:
   microcloud cluster list

   +--------+------------------+----------+------------------------------------------------------------------+--------+
   |  NAME  |      ADDRESS     | ROLE     |                           FINGERPRINT                            | STATUS |
   +--------+------------------+----------+------------------------------------------------------------------+--------+
   | micro1 | 10.1.123.10:9443 | voter    | 47a74cb2ed8b844544ce71f45e96acb2c8021d4c1ffc2f1f449cdbf2f6898fd8 | ONLINE |
   +--------+------------------+----------+------------------------------------------------------------------+--------+
   | micro2 | 10.1.123.20:9443 | voter    | 56bee3adbd5e1de2186dd22788baffd5e1358e408ec3d9b713ed930741a339f2 | ONLINE |
   +--------+------------------+----------+------------------------------------------------------------------+--------+
   | micro3 | 10.1.123.30:9443 | voter    | aabdd5f64d4c2796a50d6ce9d91939f248bfeb27195426158dff05d660f93f86 | ONLINE |
   +--------+------------------+----------+------------------------------------------------------------------+--------+
   ```

   ```{terminal}
   :user: root
   :host: micro1
   :scroll:
   microceph cluster list

   +--------+------------------+----------+------------------------------------------------------------------+--------+
   |  NAME  |      ADDRESS     | ROLE     |                           FINGERPRINT                            | STATUS |
   +--------+------------------+----------+------------------------------------------------------------------+--------+
   | micro1 | 10.1.123.10:7443 | voter    | a2b370cce1deb02437b583aa73be5e5c519aed75f02f4b98f6df150fd62c648a | ONLINE |
   +--------+------------------+----------+------------------------------------------------------------------+--------+
   | micro2 | 10.1.123.20:7443 | voter    | e37ea1acd14b984152cac4cb861cbe35ac438151233b9d0ee606c44c2e27d759 | ONLINE |
   +--------+------------------+----------+------------------------------------------------------------------+--------+
   | micro3 | 10.1.123.30:7443 | voter    | 152ccf372ecc93faffa8a6801cedd5eca49d977eea72e3f2239245cc22965399 | ONLINE |
   +--------+------------------+----------+------------------------------------------------------------------+--------+
   ```

   ```{terminal}
   :user: root
   :host: micro1
   :scroll:
   microovn cluster list

   +--------+------------------+----------+------------------------------------------------------------------+--------+
   |  NAME  |      ADDRESS     | ROLE     |                           FINGERPRINT                            | STATUS |
   +--------+------------------+----------+------------------------------------------------------------------+--------+
   | micro1 | 10.1.123.10:6443 | voter    | a552d316c159a50a4e11253c36a1cd25a3902bee50e24ed1e073ee7728be0410 | ONLINE |
   +--------+------------------+----------+------------------------------------------------------------------+--------+
   | micro2 | 10.1.123.20:6443 | voter    | 2c779eb10409576a33fa01a29cede39abea61f7cd6a07837c369858b515ed02a | ONLINE |
   +--------+------------------+----------+------------------------------------------------------------------+--------+
   | micro3 | 10.1.123.30:6443 | voter    | 7f76cddfdbbe3d768c343b1a5f402842565c25d0e4e3ebbc8514263fc14ea28b | ONLINE |
   +--------+------------------+----------+------------------------------------------------------------------+--------+
   ```

1. Inspect the storage setup:

   ```{terminal}
   :user: root
   :host: micro1
   :scroll:
   lxc storage list

   +-----------+--------+--------------------------------------------+---------+---------+
   |  NAME     | DRIVER |         DESCRIPTION                        | USED BY |  STATE  |
   +-----------+--------+--------------------------------------------+---------+---------+
   | local     | zfs    | Local storage on ZFS                       | 8       | CREATED |
   +-----------+--------+--------------------------------------------+---------+---------+
   | remote    | ceph   | Distributed storage on Ceph                | 1       | CREATED |
   +-----------+--------+--------------------------------------------+---------+---------+
   | remote-fs | cephfs | Distributed file-system storage using Ceph | 1       | CREATED |
   +-----------+--------+--------------------------------------------+---------+---------+
   ```

   ```{terminal}
   :user: root
   :host: micro1
   :scroll:
   lxc storage info local

   info:
     description: Local storage on ZFS
     driver: zfs
     name: local
     space used: 747.00KiB
     total space: 9.20GiB
   used by:
     volumes:
     - backups (location "micro1")
     - backups (location "micro2")
     - backups (location "micro3")
     - images (location "micro1")
     - images (location "micro2")
     - images (location "micro3")
   ```

   ```{terminal}
   :user: root
   :host: micro1
   :scroll:
   lxc storage info remote

   info:
     description: Distributed storage on Ceph
     driver: ceph
     name: remote
     space used: 25.41KiB
     total space: 29.67GiB
   used by:
     profiles:
     - default
   ```

   ```{terminal}
   :user: root
   :host: micro1
   :scroll:
   lxc storage info remote-fs

   info:
     description: Distributed file-system storage using CephFS
     driver: cephfs
     name: remote-fs
     space used: 0B
     total space: 29.67GiB
   used by: {}
   ```

1. Inspect the OVN network setup:

   ```{terminal}
   :user: root
   :host: micro1
   :scroll:
   lxc network list

   +---------+----------+---------+-----------------+--------------------------+-------------+---------+---------+
   |  NAME   |   TYPE   | MANAGED |      IPV4       |           IPV6           | DESCRIPTION | USED BY |  STATE  |
   +---------+----------+---------+-----------------+--------------------------+-------------+---------+---------+
   | UPLINK  | physical | YES     |                 |                          |             | 1       | CREATED |
   +---------+----------+---------+-----------------+--------------------------+-------------+---------+---------+
   | br-int  | bridge   | NO      |                 |                          |             | 0       |         |
   +---------+----------+---------+-----------------+--------------------------+-------------+---------+---------+
   | default | ovn      | YES     | 198.51.100.1/24 | 2001:db8:d960:91cf::1/64 |             | 1       | CREATED |
   +---------+----------+---------+-----------------+--------------------------+-------------+---------+---------+
   | enp5s0  | physical | NO      |                 |                          |             | 0       |         |
   +---------+----------+---------+-----------------+--------------------------+-------------+---------+---------+
   | enp6s0  | physical | NO      |                 |                          |             | 1       |         |
   +---------+----------+---------+-----------------+--------------------------+-------------+---------+---------+
   | lxdovn1 | bridge   | NO      |                 |                          |             | 0       |         |
   +---------+----------+---------+-----------------+--------------------------+-------------+---------+---------+
   ```

   ```{terminal}
   :user: root
   :host: micro1
   :scroll:
   lxc network show default

   config:
     bridge.mtu: "1442"
     ipv4.address: 198.51.100.1/24
     ipv4.nat: "true"
     ipv6.address: 2001:db8:d960:91cf::1/64
     ipv6.nat: "true"
     network: UPLINK
     volatile.network.ipv4.address: 192.0.2.100
     volatile.network.ipv6.address: 2001:db8:e647:610d:216:3eff:fe96:ed5c
   description: ""
   name: default
   type: ovn
   used_by:
   - /1.0/profiles/default
   managed: true
   status: Created
   locations:
   - micro1
   - micro3
   - micro2
   ```

1. Make sure that you can ping the virtual router within OVN.

   1. Within the output of the previous command (`lxc network show default`), find the value for `volatile.network.ipv4.address`. This is the virtual router's IPv4 address.

   1. Ping that IPv4 address.
   
   ```{terminal}
   :user: root
   :host: micro1
   :scroll:
   ping 192.0.2.100

   PING 192.0.2.100 (192.0.2.100) 56(84) bytes of data.
   64 bytes from 192.0.2.100: icmp_seq=1 ttl=253 time=2.05 ms
   64 bytes from 192.0.2.100: icmp_seq=2 ttl=253 time=2.01 ms
   64 bytes from 192.0.2.100: icmp_seq=3 ttl=253 time=1.78 ms
   ^C
   --- 192.0.2.100 ping statistics ---
   4 packets transmitted, 3 received, 25% packet loss, time 3005ms
   rtt min/avg/max/mdev = 1.777/1.945/2.052/0.120 ms
   ```

1. Inspect the default profile:

   ```{terminal}
   :user: root
   :host: micro1
   :scroll:
   lxc profile show default

   config: {}
   description: ""
   devices:
     eth0:
       name: eth0
       network: default
       type: nic
     root:
       path: /
       pool: remote
       type: disk
   name: default
   used_by: []
   ```

## Next steps

Now that you have set up your MicroCloud cluster, you can also {ref}`set up the MicroCloud UI <tutorial-ui>`.

