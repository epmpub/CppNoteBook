# High-Speed Packet Transmission in Go: From net.Dial to AF_XDP

Pushing limits in Go: from net.Dial to syscalls, AF_PACKET, and lightning-fast AF_XDP. Benchmarking packet sending performance..

Recently, I developed a [Go program that sends ICMP ping messages](https://github.com/atoonk/ping-aws-ips) to millions of IP addresses. Obviously I wanted this to be done as fast and efficiently as possible. So this prompted me to look into the various methods of interfacing with the network stack and sending packets, fast! It was a fun journey, so in this article, I’ll share some of my learnings and document them for my future self :)  You'll see how we get to 18.8Mpps with just 8 cores. There’s also [this Github repo that has the example code](https://github.com/atoonk/go-pktgen), making it easy to follow along.

## The use case

Let’s start with a quick background of the problem statement. I want to be able to send as many packets per second from a Linux machine. There are a few use cases, for example, the Ping example I mentioned earlier, but also maybe something more generic like dpdk-pktgen or even something Iperf. I guess you could summarize it as a packet generator.

I’m using the Go programming language to explore the various options. In general, the explored methods could be used in any programming language since these are mostly Go-specific interfaces around what the Linux Kernel provides. However,  you may be limited by the libraries or support that exist in your favorite programming language.

Let’s start our adventure and explore the various ways to generate network packets in Go. I’ll go over the options, and we’ll end with a benchmark, showing us which method is the best for our use case. I’ve included examples of the various methods in a Go package; you can [find the code here](https://github.com/atoonk/go-pktgen/). We’ll use the same code to run a benchmark and see how the various methods compare.

## The net.Dial method

The net.Dial method is the most likely candidate for working with network connections in Go. It’s a high-level abstraction provided by the standard library's net package, designed to establish network connections in an easy-to-use and straightforward manner.  You would use this for bi-directional communication where you can simply read and write to a Net.Conn (socket) without having to worry about the details.

In our case, we’re primarily interested in sending traffic, using the net.Dial method that looks like this:

```Golang
conn, err := net.Dial("udp", fmt.Sprintf("%s:%d", s.dstIP, s.dstPort))
if err != nil {
	return fmt.Errorf("failed to dial UDP: %w", err)
}
defer conn.Close()
```

After that, you can simply write bytes to your conn like this

```go
conn.Write(payload)
```

You can find our code for this in the file [af_inet.go](https://github.com/atoonk/go-pktgen/blob/main/pktgen/af_inet.go)

That’s it! Pretty simple, right? As we’ll see, however, when we get to the benchmark, this is the slowest method and not the best for sending packets quickly. Using this method, we can get to about 697,277 pps

## Raw Socket

Moving deeper into the network stack, I decided to use raw sockets to send packets in Go. Unlike the more abstract net.Dial method, raw sockets provide a lower-level interface with the network stack, offering granular control over packet headers and content. This method allows us to craft entire packets, including the IP header, manually.

To create a raw socket, we’ll have to make our own syscall, give it the correct parameters, and provide the type of traffic we’re going to send. We’ll then get back a file descriptor. We can then read and write to this file descriptor. This is what it looks like at the high level; see rawsocket.go for the complete code.

```Go
fd, err := syscall.Socket(syscall.AF_INET, syscall.SOCK_RAW, syscall.IPPROTO_RAW)
if err != nil {
	log.Fatalf("Failed to create raw socket: %v", err)
}
defer syscall.Close(fd)

// Set options: here, we enable IP_HDRINCL to manually include the IP header
if err := syscall.SetsockoptInt(fd, syscall.IPPROTO_IP, syscall.IP_HDRINCL, 1); err != nil {
	log.Fatalf("Failed to set IP_HDRINCL: %v", err)
}
```


That’s it, and now we can read and write our raw packet to file descriptor like this

```Go
err := syscall.Sendto(fd, packet, 0, dstAddr)
```

Since I’m using IPPROTO_RAW, we’re bypassing the transport layer of the kernel's network stack, and the kernel expects us to provide a complete IP packet. We do that using the [BuildPacket function](https://github.com/atoonk/go-pktgen/blob/main/pktgen/rawsocket.go#L66C17-L66C28). It’s slightly more work, but the neat thing about raw sockets is that you can construct whatever packet you want.

We’re telling the kernel just to take our packet, it has to do less work, and thus, this process is faster. All we’re really asking from the network stack is to take this IP packet, add the ethernet headers, and hand it to the network card for sending. It comes as no surprise, then, that this option is indeed faster than the Net.Dial option. Using this method, we can reach about 793,781 pps, about 100k PPS more than the net.Dial method.

## The AF_INET Syscall Method

Now that we’re used to using syscalls directly, we have another option. In this [example](https://github.com/atoonk/go-pktgen/blob/main/pktgen/af_inet_syscall.go), we create a UDP socket directly like below

```Go
fd, err := syscall.Socket(syscall.AF_INET, syscall.SOCK_DGRAM, syscall.IPPROTO_UDP)
```

After that we can simply write our payload to it using the Sendto method like before.

```Go
err = syscall.Sendto(fd, payload, 0, dstAddr)
```

It looks similar to the raw socket example, but a few differences exist. The key difference is that in this case we’ve created a socket of type UDP, which means we don’t need to construct the complete packet (IP and UDP header) like before.  When using this method, the kernel manages the construction of the UDP header based on the destination IP and port we specify and handles the encapsulation process into an IP packet.

In this case, the payload is just the UDP payload. In fact, this method is similar to the Net.Dial method before, but with fewer abstractions.

Compared to the raw socket method before, I’m now seeing 861,372 pps—that’s a 70k jump. We’re getting faster each step of the way. I’m guessing we get the benefit of some UDP optimizations in the kernel.

## The Pcap Method

It may be surprising to see Pcap here for sending packets. Most folks know pcap from things like tcpdump or Wireshark to capture packets. But it’s also a fairly common way to send packets. In fact, if you look at many of the Go-packet or Python Scappy examples, this is typically the method listed to send custom packets. So, I figured I should include it and see its performance. I was skeptical, but was pleasantly surprised when I saw the pps numbers!

First, let’s take a look at what this looks like in Go; again, for the complete example, see my implementation in [pcap.go here](https://github.com/atoonk/go-pktgen/blob/main/pktgen/pcap.go)

We start by creating a Pcap handle like this:

```Golang
handle, err := pcap.OpenLive(s.iface, 1500, false, pcap.BlockForever)
if err != nil {
	return fmt.Errorf("could not open device: %w", err)
}
defer handle.Close()
```

Then we [create the packet manually,](https://github.com/atoonk/go-pktgen/blob/main/pktgen/pcap.go#L51-L64) similar to the Raw socket method earlier, but in this case, we include the Ethernet headers.
After that, we can write the packet to the pcap handle, and we’re done!

```Golang
err := handle.WritePacketData(packet)
```

To my surprise, this method resulted in quite a performance win. We surpassed the one million packets per second mark by quite a margin: 1,354,087 pps—almost a 500k pps jump!

Note that, towards the end of this article, we’ll look at  a caveat, but good to know that this method stops working well when sending multiple streams (go routines).

## The af_packet method

As we explore the layers of network packet crafting and transmission in Go, we next find the AF_PACKET method. This method is popular for IDS systems on Linux, and for good reasons!

It gives us direct access to the network device layer, allowing for the transmission of packets at the link layer. This means we can craft packets, including the Ethernet header, and send them directly to the network interface, bypassing the higher networking layers. We can create a socket of type AF_PACKET using a syscall. In Go this will look like this:

```Golang
fd, err := syscall.Socket(syscall.AF_PACKET, syscall.SOCK_RAW, int(htons(syscall.ETH_P_IP)))
```

This line of code creates a raw socket that can send packets at the Ethernet layer. With AF_PACKET, we specify SOCK_RAW to indicate that we are interested in raw network protocol access. By setting the protocol to ETH_P_IP, we tell the kernel that we’ll be dealing with IP packets.

After obtaining a socket descriptor, we must bind it to a network interface. This step ensures that our crafted packets are sent out through the correct network device:

```Golang
addr := &syscall.SockaddrLinklayer{
	Protocol: htons(syscall.ETH_P_IP),
	Ifindex:  ifi.Index,
}
```

Crafting packets with AF_PACKET involves manually creating the Ethernet frame. This includes setting both source and destination MAC addresses and the EtherType to indicate what type of payload the frame is carrying (in our case, IP). We’re using the same [BuildPacket function](https://github.com/atoonk/go-pktgen/blob/main/pktgen/af_packet.go#L56-L68) as we used for the Pcap method earlier.

The packet is then ready to be sent directly onto the wire:

```Golang
syscall.Sendto(fd, packet, 0, addr)
```

The performance of the AF_PACKET method turns out to be almost identical to that achieved with the pcap method earlier. A quick Google, shows that libpcap, the library underlying tools like tcpdump and the Go pcap bindings, uses AF_PACKET for packet capture and injection on Linux platforms. So that explains the performance similarities.

## Using the AF_XDP Socket

We have one more option to try. AF_XDP is a relatively recent development and promises impressive numbers! It is designed to dramatically increase the speed at which applications can send and receive packets directly from and to the network interface card (NIC) by utilizing a fast path through the traditional Linux network stack. Also see [my earlier blog on XDP here](https://toonk.io/building-an-xdp-express-data-path-based-bgp-peering-router/index.html).

AF_XDP leverages the XDP (eXpress Data Path) framework. This capability not only provides minimal latency by avoiding kernel overhead but also maximizes throughput by enabling packet processing at the earliest possible point in the software stack.

The Go standard library doesn’t natively support AF_XDP sockets,  and I was only able to find one library to help with this. So it’s all relatively new still.

I’m using this library [github.com/asavie/xdp](http://github.com/asavie/xdp) and this is how you can initiate an AF_XDP socket.

```Golang
xsk, err := xdp.NewSocket(link.Attrs().Index, s.queueID, nil)
```

Note that we need to provide a NIC queue; this is a clear indicator that we’re working at a lower level than before. The complete code is a bit more complicated than the other options, partially because we need to work with a user-space memory buffer (UMEM) for packet data. This method reduces the kernel's involvement in packet processing, cutting down the time packets spend traversing system layers. By crafting and injecting packets directly at the driver level. So instead of pasting the code, please look at [my code here](https://github.com/atoonk/go-pktgen/blob/main/pktgen/af_xdp.go#L40-L97).

The results look great; using this method, I’m now able to generate 2,647,936 pps. That’s double the performance we saw with AF_PACKET! Whoohoo!

## Wrap-up and some takeaways

First off, this was fun to do and learn! We looked at the various options to generate packets from the traditional *net.Dial* method, to *raw sockets*, *pcap*, *AF_PACKET* and finally *AF_XDP*. The graph below shows the numbers per method (all using one CPU and one NIC queue). AF_XDP is the big winner!

![img](https://lh7-us.googleusercontent.com/xLiXH5evY6KAyktPqflagrmU8w6FVOpqkSEQKVELG8CzpU4PG-iMy8qN55Lx6vyAvK1Y43Qe2tclpwk1o2yykmIBfF0m415N1p6lrzvhQlNuZG94dFpg8UgshDmQgnTTHlqqQrtWSL-A-NcVTJUsd5M)The various ways to send network traffic in Go


If interested, you can run the benchmarks yourself on a Linux system like below:

```bash
./go-pktgen --dstip 192.168.64.2 --method benchmark \
 --duration 5 --payloadsize 64 --iface veth0

+-------------+-----------+------+
|   Method    | Packets/s | Mb/s |
+-------------+-----------+------+
| af_xdp      |   2647936 | 1355 |
| af_packet   |   1368070 |  700 |
| af_pcap     |   1354087 |  693 |
| udp_syscall |    861372 |  441 |
| raw_socket  |    793781 |  406 |
| net_conn    |    697277 |  357 |
+-------------+-----------+------+
```

The important number to look at is packets per second as that is the limitation on software network stacks. The Mb/s number is simply the packet size x the PPS number you can generate. It’s interesting to see the easy 2x jump from the traditional net.Dial approach to using AF_PACKET. And then another 2x jump when using AF_XDP. Certainly good to know if you’re interested in sending packets fast!

The benchmark tool above uses one CPU and, thus, one NIC queue by default. The user can, however, elect to use more CPUs, which will start multiple go routines to do the same tests in parallel. The screenshot below shows the tool running with 8 streams (and 8 CPUs) using AF_XDP,  generating 186Gb/s with 1200 byte packets (18.8Mpps)! That’s really quite impressive for a Linux box (and not using DPDK). Faster than what you can do with Iperf3 for example.

![img](https://lh7-us.googleusercontent.com/yHN-VEhY_ykNpfk7nbdBTXxrfdy8fs_Qd5LZ3m970jhRhnvt7nI17W_lq_NuUTJ-iJiCLTCBv77r5jCDmPLE79LXndzyFFHjUyiZz4CJ9MOSoinspjlLt31YEieDY15vaNTK7ECNbtp-sZn4vkiFWx0)

## Some caveats and things I’d like to look at in the future 

Running multiple streams (go routines) using the PCAP method doesn’t work well. The performance degrades significantly. The comparable AF_PACKET method, on the other hand, works well with multiple streams and go routines.

The AF_XDP library I’m using doesn’t seem to work well on most hardware NICs. I opened a [GitHub issue](https://github.com/asavie/xdp/issues/31) for this and hope it will be resolved. It would be great to see this be more reliable as it kind of limits more real-world AF_XDP Go applications. I did most of my testing using veth interfaces; i’d love to see how it works on a physical NIC and a driver with XDP support.

It turns out that for AF_PACKET, there's a zero-copy mode facilitated by the use of memory-mapped (mmap) ring buffers. This feature allows user-space applications to directly access packet data in kernel space without the need for copying data between the kernel and user space, effectively reducing CPU usage and increasing packet processing speed. This means that, in theory, the performance of AF_PACKET and AF_XDP could be very similar.  However, it appears the Go implementations of AF_PACKET do not support zero-copy mode or [only for RX](https://twitter.com/jtollet/status/1762616103883227490) and not TX. So I wasn’t able to use that. I found [this patch](https://github.com/csulrong/gopacket/pull/1/files) but unfortunately couldn’t get it to work within an hour or so, so I moved on. If this works, this will likely be the preferred approach as you don’t have to rely on AF_XDP support.

Finally, I’d love to include DPDK support in this [pktgen library](https://github.com/atoonk/go-pktgen/tree/main). It’s the last one missing. But that’s a whole beast on its own, and I need to rely on good Go DPDK libraries. Perhaps in the future!

That’s it; you made it to the end! Thanks for reading!

Cheers
-Andree