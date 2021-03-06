# Troubleshooting Cisco IOS Customer Gateway without Border Gateway Protocol Connectivity<a name="Cisco_Troubleshooting_NoBGP"></a>


|  | 
| --- |
| This guide \(the Network Administrator Guide\) has been merged into the AWS Site\-to\-Site VPN User Guide and is no longer maintained\. For more information about configuring your customer gateway device, see the [AWS Site\-to\-Site VPN User Guide](https://docs.aws.amazon.com/vpn/latest/s2svpn/your-cgw.html)\. | 

When you troubleshoot the connectivity of a Cisco customer gateway, consider three things: IKE, IPsec, and tunnel\. You can troubleshoot these areas in any order, but we recommend that you start with IKE \(at the bottom of the network stack\) and move up\.

## IKE<a name="IOS_NoBGP_IKE"></a>

Use the following command\. The response shows a customer gateway with IKE configured correctly\.

```
router# show crypto isakmp sa
```

```
IPv4 Crypto ISAKMP SA
dst             src             state          conn-id slot status
174.78.144.73 205.251.233.121 QM_IDLE           2001    0 ACTIVE
174.78.144.73 205.251.233.122 QM_IDLE           2002    0 ACTIVE
```

You should see one or more lines containing an `src` value for the remote gateway specified in the tunnels\. The `state` should be `QM_IDLE` and `status` should be `ACTIVE`\. The absence of an entry, or any entry in another state, indicates that IKE is not configured properly\.

For further troubleshooting, run the following commands to enable log messages that provide diagnostic information\.

```
router# term mon
router# debug crypto isakmp
```

To disable debugging, use the following command:

```
router# no debug crypto isakmp
```

## IPsec<a name="IOS_NoBGP_IPsec"></a>

Use the following command\. The response shows a customer gateway with IPsec configured correctly\.

```
router# show crypto ipsec sa
```

```
interface: Tunnel1
    Crypto map tag: Tunnel1-head-0, local addr 174.78.144.73

    protected vrf: (none)
    local  ident (addr/mask/prot/port): (0.0.0.0/0.0.0.0/0/0)
    remote ident (addr/mask/prot/port): (0.0.0.0/0.0.0.0/0/0)
    current_peer 72.21.209.225 port 500
     PERMIT, flags={origin_is_acl,}
     #pkts encaps: 149, #pkts encrypt: 149, #pkts digest: 149
     #pkts decaps: 146, #pkts decrypt: 146, #pkts verify: 146
     #pkts compressed: 0, #pkts decompressed: 0
     #pkts not compressed: 0, #pkts compr. failed: 0
     #pkts not decompressed: 0, #pkts decompress failed: 0
     #send errors 0, #recv errors 0

     local crypto endpt.: 174.78.144.73, remote crypto endpt.:205.251.233.121
     path mtu 1500, ip mtu 1500, ip mtu idb FastEthernet0
     current outbound spi: 0xB8357C22(3090512930)

     inbound esp sas:
      spi: 0x6ADB173(112046451)
       transform: esp-aes esp-sha-hmac ,
       in use settings ={Tunnel, }
       conn id: 1, flow_id: Motorola SEC 2.0:1, crypto map: Tunnel1-head-0
       sa timing: remaining key lifetime (k/sec): (4467148/3189)
       IV size: 16 bytes
       replay detection support: Y  replay window size: 128
       Status: ACTIVE

     inbound ah sas:

     inbound pcp sas:

     outbound esp sas:
      spi: 0xB8357C22(3090512930)
       transform: esp-aes esp-sha-hmac ,
       in use settings ={Tunnel, }
       conn id: 2, flow_id: Motorola SEC 2.0:2, crypto map: Tunnel1-head-0
       sa timing: remaining key lifetime (k/sec): (4467148/3189)
       IV size: 16 bytes
       replay detection support: Y  replay window size: 128
       Status: ACTIVE

     outbound ah sas:

     outbound pcp sas:

interface: Tunnel2
     Crypto map tag: Tunnel2-head-0, local addr 205.251.233.122

     protected vrf: (none)
     local  ident (addr/mask/prot/port): (0.0.0.0/0.0.0.0/0/0)
     remote ident (addr/mask/prot/port): (0.0.0.0/0.0.0.0/0/0)
     current_peer 72.21.209.193 port 500
      PERMIT, flags={origin_is_acl,}
     #pkts encaps: 26, #pkts encrypt: 26, #pkts digest: 26
     #pkts decaps: 24, #pkts decrypt: 24, #pkts verify: 24
     #pkts compressed: 0, #pkts decompressed: 0
     #pkts not compressed: 0, #pkts compr. failed: 0
     #pkts not decompressed: 0, #pkts decompress failed: 0
     #send errors 0, #recv errors 0

     local crypto endpt.: 174.78.144.73, remote crypto endpt.:205.251.233.122
     path mtu 1500, ip mtu 1500, ip mtu idb FastEthernet0
     current outbound spi: 0xF59A3FF6(4120526838)

     inbound esp sas:
      spi: 0xB6720137(3060924727)
       transform: esp-aes esp-sha-hmac ,
       in use settings ={Tunnel, }
       conn id: 3, flow_id: Motorola SEC 2.0:3, crypto map: Tunnel2-head-0
       sa timing: remaining key lifetime (k/sec): (4387273/3492)
       IV size: 16 bytes
       replay detection support: Y  replay window size: 128
       Status: ACTIVE

     inbound ah sas:

     inbound pcp sas:

     outbound esp sas:
      spi: 0xF59A3FF6(4120526838)
       transform: esp-aes esp-sha-hmac ,
       in use settings ={Tunnel, }
       conn id: 4, flow_id: Motorola SEC 2.0:4, crypto map: Tunnel2-head-0
       sa timing: remaining key lifetime (k/sec): (4387273/3492)
       IV size: 16 bytes
       replay detection support: Y  replay window size: 128
       Status: ACTIVE

     outbound ah sas:

     outbound pcp sas:
```

For each tunnel interface, you should see both an inbound `esp sas` and outbound `esp sas`\. This assumes that an SA is listed \(for example, `spi: 0x48B456A6`\), the status is `ACTIVE`, and IPsec is configured correctly\.

For further troubleshooting, use the following command to enable debugging\.

```
router# debug crypto ipsec
```

To disable debugging, use the following command:

```
router# no debug crypto ipsec
```

## Tunnel<a name="IOS_NoBGP_tunnel"></a>

First, check that you have the necessary firewall rules in place\. For more information, see [Configuring a Firewall Between the Internet and Your Customer Gateway Device](Introduction.md#FirewallRules)\.

If your firewall rules are set up correctly, then continue troubleshooting with the following command:

```
router# show interfaces tun1
```

```
Tunnel1 is up, line protocol is up 
  Hardware is Tunnel
  Internet address is 169.254.249.18/30
  MTU 17867 bytes, BW 100 Kbit/sec, DLY 50000 usec, 
    reliability 255/255, txload 2/255, rxload 1/255
  Encapsulation TUNNEL, loopback not set
  Keepalive not set
  Tunnel source 174.78.144.73, destination 205.251.233.121
  Tunnel protocol/transport IPSEC/IP
  Tunnel TTL 255
  Tunnel transport MTU 1427 bytes
  Tunnel transmit bandwidth 8000 (kbps)
  Tunnel receive bandwidth 8000 (kbps)
  Tunnel protection via IPSec (profile "ipsec-vpn-92df3bfb-0")
  Last input never, output never, output hang never
  Last clearing of "show interface" counters never
  Input queue: 0/75/0/0 (size/max/drops/flushes); Total output drops: 0
  Queueing strategy: fifo
  Output queue: 0/0 (size/max)
  5 minute input rate 0 bits/sec, 1 packets/sec
  5 minute output rate 1000 bits/sec, 1 packets/sec
    407 packets input, 30010 bytes, 0 no buffer
    Received 0 broadcasts, 0 runts, 0 giants, 0 throttles
```

Make sure that the line protocol is up\. Check that the tunnel source IP address, source interface, and destination respectively match the tunnel configuration for the customer gateway outside IP address, interface, and virtual private gateway outside IP address\. Make sure that `Tunnel protection through IPSec` is present\. Make sure to run the command on both tunnel interfaces\. To resolve any problems, review the configuration and check the physical connections to your customer gateway\.

You can also use the following command, replacing `169.254.249.18` with the inside IP address of your virtual private gateway\.

```
router# ping 169.254.249.18 df-bit size 1410
```

```
Type escape sequence to abort.
Sending 5, 1410-byte ICMP Echos to 169.254.249.18, timeout is 2 seconds:
Packet sent with the DF bit set
!!!!!
```

You should see five exclamation points\.

### Routing<a name="IOS_NoBGP_routing"></a>

To see your static route table, use the following command:

```
router# sh ip route static
```

```
     1.0.0.0/8 is variably subnetted
S       10.0.0.0/16 is directly connected, Tunnel1
is directly connected, Tunnel2
```

You should see that the static route for the VPC CIDR through both tunnels exists\. If it does not exist, add the static routes as shown here:

```
router# ip route 10.0.0.0 255.255.0.0 Tunnel1 track 100 
router# ip route 10.0.0.0 255.255.0.0 Tunnel2 track 200
```

### Checking the SLA Monitor<a name="IOS_NoBGP_sla"></a>

```
router# show ip sla statistics 100
```

```
IPSLAs Latest Operation Statistics

IPSLA operation id: 100
        Latest RTT: 128 milliseconds
Latest operation start time: *18:08:02.155 UTC Wed Jul  15 2012
Latest operation return code: OK
Number of successes: 3
Number of failures: 0
Operation time to live: Forever
```

```
router# show ip sla statistics 200
```

```
IPSLAs Latest Operation Statistics

IPSLA operation id: 200
        Latest RTT: 128 milliseconds
Latest operation start time: *18:08:02.155 UTC Wed Jul  15 2012
Latest operation return code: OK
Number of successes: 3
Number of failures: 0
Operation time to live: Forever
```

The value of "Number of successes" indicates whether the SLA monitor has been set up successfully\.

For further troubleshooting, review the configuration\.

## Virtual Private Gateway Attachment<a name="IOS_NoBGP_VPG"></a>

Verify that your virtual private gateway is attached to your VPC\. Your integration team does this with the AWS Management Console\.

If you have questions or need further assistance, please use the [Amazon VPC forum](https://forums.aws.amazon.com/forum.jspa?forumID=58)\.