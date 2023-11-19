# 2.5.3 Virtual network bridge equipment Linux Bridge

We use Veth to implement the point -to -point communication between two Network Namespace, but if it is multiple network namespace?In the physical network, if multiple hosts need to be connected, we will use the network bridge (can also be understood as a switch) device to form a small local area network.In the Linux network virtualization system, the network bridge virtual implementation Linux Bridge is also provided.

Linux Bridge is the second layer of reposting tools provided by the Linux Kernel 2.2 version, which is consistent with the physical switch mechanism. It can access any two -layer network device (whether it is real physical device, such as ETH0 or virtual devices, such as VETH, TAP, etc.) However, Linux Bridge is different from ordinary physical switches. Ordinary switches will only make two layers of reposting, but Linux Bridge can also send the data packets sent to it to the host's three -layer protocol stack.

When we deploy Docker or Kubernetes, CNI0 and Docker0 in the host are the virtual Bridge devices they create.

<div  align="center">
    <img src="../assets/linux-bridge.svg" width = "500"  align=center />
    <p>图 2-24 conntrack 示例</p>
</div>

## 1. Linux Bridge Operating practice

笔者在这里通过实践操作，创建多个 Network namespace 并通过 Bridge 实现通信。该实践的网络拓扑如图 2-2 所示。
The author creates multiple Network namespace and communicates through Bridge through practical operations here.The network topology of this practice is shown in Figure 2-2.

Create two Network Namespace.

```plain
$ ip netns add ns1
$ ip netns add ns2
```

Create a Linux Bridge and start the device.

```plain
$ brctl addbr bridge0
$ ip link set bridge0 up
```

Create VETH, add one end of VETH to network namespace, and add the other end to the network bridge.

```plain
$ ip link add veth1 type veth peer name veth1-peer

$ ip link set veth1 netns ns1
$ brctl addif bridge0 veth1-peer
$ ip -n ns1 link veth1 up
$ ip link set veth1-peer up

$ ip link add veth2 type veth peer name veth2-peer

$ ip link set veth2 netns ns2
$ brctl addif bridge0 veth2-peer
$ ip -n ns2 link veth2 up
$ ip link set veth1-peer up
```

Configure IP information for Network Namespace, which is located in the same subnet 172.16.0.0/24.

```plain
$ ip -n ns1 addr add local 172.16.0.1/24 dev veth1-peer
$ ip -n ns2 addr add local 172.16.0.2/24 dev veth2-peer
```

Through the above operations, we completed the connection between namespace and Linux Bridge. We check whether the communication is normal.

```plain
$ ip netns exec ns1 ping 172.16.0.2

PING 172.16.1.2 (172.16.0.2) 56(84) bytes of data.
64 bytes from 172.16.0.2: icmp_seq=1 ttl=64 time=0.153 ms
64 bytes from 172.16.0.2: icmp_seq=2 ttl=64 time=0.148 ms
...
```

Through the above practice, we use Linux Bridge to connect multiple Network Namespace to the same two -layer network and implement communication between them.
