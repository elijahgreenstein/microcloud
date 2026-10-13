(tutorial-instances)=
# Work with instances

Now that your MicroCloud cluster is ready to use, you can start working with instances.

## Launch some instances

Run these commands on one of the machines, such as `micro1`.
If you are using virtual machines, run this command to shell into one of the machines:

```bash
lxc exec micro1 -- su -ubuntu
```

1. Launch an Ubuntu container with the default settings:

   ```bash
   lxc launch ubuntu:24.04 u1
   ```

1. Launch another Ubuntu container, but use local storage instead of the default remote storage:

   ```bash
   lxc launch ubuntu:24.04 u2 --storage local
   ```

1. Launch an Ubuntu VM:

   ```bash
   lxc launch ubuntu:24.04 u3 --vm
   ```

1. Check the list of instances.
   Note that the instances are running on different cluster members.

   ```{terminal}
   :user: root
   :host: micro1
   :scroll:
   lxc list

   +------+---------+---------------------+----------------------------------------------+-----------------+-----------+----------+
   | NAME |  STATE  |        IPV4         |                     IPV6                     |      TYPE       | SNAPSHOTS | LOCATION |
   +------+---------+---------------------+----------------------------------------------+-----------------+-----------+----------+
   | u1   | RUNNING | 198.51.100.2 (eth0) | 2001:db8:d960:91cf:216:3eff:fe4e:9642 (eth0) | CONTAINER       | 0         | micro1   |
   +------+---------+---------------------+----------------------------------------------+-----------------+-----------+----------+
   | u2   | RUNNING | 198.51.100.3 (eth0) | 2001:db8:d960:91cf:216:3eff:fe79:6765 (eth0) | CONTAINER       | 0         | micro3   |
   +------+---------+---------------------+----------------------------------------------+-----------------+-----------+----------+
   | u3   | RUNNING | 198.51.100.4 (eth0) | 2001:db8:d960:91cf:216:3eff:fe66:f24b (eth0) | VIRTUAL-MACHINE | 0         | micro2   |
   +------+---------+---------------------+----------------------------------------------+-----------------+-----------+----------+
   ```

1. Check the storage.
   Note that the instance volumes are located on the specified storage pools.

   ```{terminal}
   :user: root
   :host: micro1
   :scroll:
   lxc storage volume list remote

   +-----------------+------------------------------------------------------------------+-------------+--------------+---------+----------+
   | TYPE | NAME | DESCRIPTION | CONTENT-TYPE | USED BY | LOCATION |
   +-----------------+------------------------------------------------------------------+-------------+--------------+---------+----------+
   | container       | u1                                                               |             | filesystem   | 1       |          |
   +-----------------+------------------------------------------------------------------+-------------+--------------+---------+----------+
   | image           | 17fbc145125c659b7ef926b2de5e5304370083e28846f084a0d514c7a96777bc |             | block        | 1       |          |
   +-----------------+------------------------------------------------------------------+-------------+--------------+---------+----------+
   | image           | 45613e262f8a5fc9467330f679862147c289516f045e3edc313e07ebcb0aab4a |             | filesystem   | 1       |          |
   +-----------------+------------------------------------------------------------------+-------------+--------------+---------+----------+
   | virtual-machine | u3                                                               |             | block        | 1       |          |
   +-----------------+------------------------------------------------------------------+-------------+--------------+---------+----------+
   ```

   ```{terminal}
   :user: root
   :host: micro1
   :scroll:
   lxc storage volume list local

   +-----------+------------------------------------------------------------------+-------------+--------------+---------+----------+
   |   TYPE    |                               NAME                               | DESCRIPTION | CONTENT-TYPE | USED BY | LOCATION |
   +-----------+------------------------------------------------------------------+-------------+--------------+---------+----------+
   | container | u2                                                               |             | filesystem   | 1       | micro3   |
   +-----------+------------------------------------------------------------------+-------------+--------------+---------+----------+
   | custom    | backups                                                          |             | filesystem   | 1       | micro2   |
   +-----------+------------------------------------------------------------------+-------------+--------------+---------+----------+
   | custom    | backups                                                          |             | filesystem   | 1       | micro3   |
   +-----------+------------------------------------------------------------------+-------------+--------------+---------+----------+
   | custom    | backups                                                          |             | filesystem   | 1       | micro4   |
   +-----------+------------------------------------------------------------------+-------------+--------------+---------+----------+
   | custom    | backups                                                          |             | filesystem   | 1       | micro1   |
   +-----------+------------------------------------------------------------------+-------------+--------------+---------+----------+
   | custom    | images                                                           |             | filesystem   | 1       | micro2   |
   +-----------+------------------------------------------------------------------+-------------+--------------+---------+----------+
   | custom    | images                                                           |             | filesystem   | 1       | micro3   |
   +-----------+------------------------------------------------------------------+-------------+--------------+---------+----------+
   | custom    | images                                                           |             | filesystem   | 1       | micro4   |
   +-----------+------------------------------------------------------------------+-------------+--------------+---------+----------+
   | custom    | images                                                           |             | filesystem   | 1       | micro1   |
   +-----------+------------------------------------------------------------------+-------------+--------------+---------+----------+
   | image     | 45613e262f8a5fc9467330f679862147c289516f045e3edc313e07ebcb0aab4a |             | filesystem   | 1       | micro3   |
   +-----------+------------------------------------------------------------------+-------------+--------------+---------+----------+
   ```

## Inspect your networking

The instances that you have launched are all on the same subnet.
You can, however, create a different network to isolate some instances from others.

1. Check the list of running instances:

   ```{terminal}
   :user: root
   :host: micro1
   :scroll:
   lxc list

   +------+---------+---------------------+----------------------------------------------+-----------------+-----------+----------+
   | NAME |  STATE  |        IPV4         |                    IPV6                      |      TYPE       | SNAPSHOTS | LOCATION |
   +------+---------+---------------------+----------------------------------------------+-----------------+-----------+----------+
   | u1   | RUNNING | 198.51.100.2 (eth0) | 2001:db8:d960:91cf:216:3eff:fe4e:9642 (eth0) | CONTAINER       | 0         | micro1   |
   +------+---------+---------------------+----------------------------------------------+-----------------+-----------+----------+
   | u2   | RUNNING | 198.51.100.3 (eth0) | 2001:db8:d960:91cf:216:3eff:fe79:6765 (eth0) | CONTAINER       | 0         | micro3   |
   +------+---------+---------------------+----------------------------------------------+-----------------+-----------+----------+
   | u3   | RUNNING | 198.51.100.4 (eth0) | 2001:db8:d960:91cf:216:3eff:fe66:f24b (eth0) | VIRTUAL-MACHINE | 0         | micro2   |
   +------+---------+---------------------+----------------------------------------------+-----------------+-----------+----------+
   ```

1. Access the shell in `u1`:

       lxc exec u1 -- bash

1. Ping the IPv4 address of `u2`:

   ```{terminal}
   :user: root
   :host: u1
   :scroll:
   ping 198.51.100.3

   PING 198.51.100.3 (198.51.100.3) 56(84) bytes of data.
   64 bytes from 198.51.100.3: icmp_seq=1 ttl=64 time=1.33 ms
   64 bytes from 198.51.100.3: icmp_seq=2 ttl=64 time=1.74 ms
   64 bytes from 198.51.100.3: icmp_seq=3 ttl=64 time=0.985 ms
   ^C
   --- 198.51.100.3 ping statistics ---
   3 packets transmitted, 3 received, 0% packet loss, time 2004ms
   rtt min/avg/max/mdev = 0.985/1.352/1.739/0.308 ms
   ```

1. Ping the IPv6 address of `u3`:

   ```{terminal}
   :user: root
   :host: u1
   :scroll:
   ping6 -n 2001:db8:d960:91cf:216:3eff:fe66:f24b

   PING 2001:db8:d960:91cf:216:3eff:fe66:f24b(2001:db8:d960:91cf:216:3eff:fe66:f24b) 56 data bytes
   64 bytes from 2001:db8:d960:91cf:216:3eff:fe66:f24b: icmp_seq=1 ttl=64 time=16.8 ms
   64 bytes from 2001:db8:d960:91cf:216:3eff:fe66:f24b: icmp_seq=2 ttl=64 time=3.41 ms
   64 bytes from 2001:db8:d960:91cf:216:3eff:fe66:f24b: icmp_seq=3 ttl=64 time=3.86 ms
   ^C
   --- 2001:db8:d960:91cf:216:3eff:fe66:f24b ping statistics ---
   3 packets transmitted, 3 received, 0% packet loss, time 2004ms
   rtt min/avg/max/mdev = 3.407/8.012/16.774/6.197 ms
   ```

1. Confirm that the instance has connectivity to the outside world:

   ```{terminal}
   :user: root
   :host: u1
   :scroll:
   ping www.example.com

   PING www.example.com (93.184.216.34) 56(84) bytes of data.
   64 bytes from 93.184.216.34 (93.184.216.34): icmp_seq=1 ttl=49 time=111 ms
   64 bytes from 93.184.216.34 (93.184.216.34): icmp_seq=2 ttl=49 time=95.2 ms
   64 bytes from 93.184.216.34 (93.184.216.34): icmp_seq=3 ttl=49 time=96.2 ms
   ^C
   --- www.example.com ping statistics ---
   3 packets transmitted, 3 received, 0% packet loss, time 2018ms
   rtt min/avg/max/mdev = 95.233/100.870/111.165/7.290 ms
   ```

1. Log out of the `u1` shell:

       exit

1. Create an OVN network with the default settings:

       lxc network create isolated --type=ovn

   There is only one `UPLINK` network, so the new network uses this one as its parent.

1. Show information about the new network:

   ```{terminal}
   :user: root
   :host: micro1
   :scroll:
   lxc network show isolated

   config:
     bridge.mtu: "1442"
     ipv4.address: 198.51.100.201/24
     ipv4.nat: "true"
     ipv6.address: 2001:db8:452a:32b2::1/64
     ipv6.nat: "true"
     network: UPLINK
     volatile.network.ipv4.address: 192.0.2.101
     volatile.network.ipv6.address: 2001:db8:e647:610d:216:3eff:feef:6361
   description: ""
   name: isolated
   type: ovn
   used_by: []
   managed: true
   status: Created
   locations:
   - micro1
   - micro3
   - micro2
   - micro4
   ```

1. Check that you can ping the `volatile.network.ipv4.address`:

   ```{terminal}
   :user: root
   :host: micro1
   :scroll:
   ping 192.0.2.101

   PING 192.0.2.101 (192.0.2.101) 56(84) bytes of data.
   64 bytes from 192.0.2.101: icmp_seq=1 ttl=253 time=1.25 ms
   64 bytes from 192.0.2.101: icmp_seq=2 ttl=253 time=1.04 ms
   64 bytes from 192.0.2.101: icmp_seq=3 ttl=253 time=1.68 ms
   ^C
   --- 192.0.2.101 ping statistics ---
   3 packets transmitted, 3 received, 0% packet loss, time 2002ms
   rtt min/avg/max/mdev = 1.042/1.321/1.676/0.264 ms
   ```

1. Launch an Ubuntu container that uses the new network:

       lxc launch ubuntu:24.04 u4 --network isolated

1. Access the shell in `u4`:

       lxc exec u4 -- bash

1. Confirm that the instance has connectivity to the outside world:

   ```{terminal}
   :user: root
   :host: u4
   :scroll:
   ping www.example.com

   PING www.example.com (93.184.216.34) 56(84) bytes of data.
   64 bytes from 93.184.216.34 (93.184.216.34): icmp_seq=1 ttl=49 time=95.6 ms
   64 bytes from 93.184.216.34 (93.184.216.34): icmp_seq=2 ttl=49 time=118 ms
   64 bytes from 93.184.216.34 (93.184.216.34): icmp_seq=3 ttl=49 time=94.6 ms
   ^C
   --- www.example.com ping statistics ---
   3 packets transmitted, 3 received, 0% packet loss, time 2004ms
   rtt min/avg/max/mdev = 94.573/102.587/117.633/10.646 ms
   ```

1. Ping the IPv4 address of `u2`:

   ```{terminal}
   :user: root
   :host: u4
   :scroll:
   ping 198.51.100.3

   PING 198.51.100.3 (198.51.100.3) 56(84) bytes of data.
   ^C
   --- 198.51.100.3 ping statistics ---
   14 packets transmitted, 0 received, 100% packet loss, time 13301ms
   ```

   The ping fails; `u2` is not reachable because it is on a different OVN subnet.

