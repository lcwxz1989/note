# _

## Basic Structure

                     ----------------------------
                     |    network applications  |
                     |                          |
                     |...  \ | /  ..  \ | /  ...|
                     |     -----      -----     |
                     |     |TCP|      |UDP|     |
                     |     -----      -----     |
                     |         \      /         |
                     |         --------         |
                     |         |  IP  |         |
                     |  -----  -*------         |
                     |  |ARP|   |               |
                     |  -----   |               |
                     |      \   |               |
                     |      ------              |
                     |      |ENET|              |
                     |      ---@--              |
                     ----------|-----------------
                               |
         ----------------------o---------
             Ethernet Cable

                  Figure 1.  Basic TCP/IP Network Node
The boxes represent processing of the data as it passes through the computer, and the lines connecting boxes show the path of data.  The horizontal line at the bottom represents the Ethernet cable; the "o" is the transceiver.  The "*" is the IP address and the "@" is the Ethernet address.

## Terminology

The name of a unit of data that flows through an internet is dependent upon where it exists in the protocol stack.  In summary:

1. if it is on an Ethernet it is called an `Ethernet frame`;
2. if it is between the Ethernet driver and the IP module it is called a `IP packet`;
3. if it is between the IP module and the UDP module it is called a `UDP datagram`;
4. if it is between the IP module and the TCP module it is called a `TCP segment` (more generally, a transport message);
5. if it is in a network application it is called a `application message`

## TCP

### STATE MESHINE

                                  +---------+ ---------\      active OPEN
                                  |  CLOSED |            \    -----------
                                  +---------+<---------\   \   create TCB
                                    |     ^              \   \  snd SYN
                       passive OPEN |     |   CLOSE        \   \
                       ------------ |     | ----------       \   \
                        create TCB  |     | delete TCB         \   \
                                    V     |                      \   \
                                  +---------+            CLOSE    |    \
                                  |  LISTEN |          ---------- |     |
                                  +---------+          delete TCB |     |
                       rcv SYN      |     |     SEND              |     |
                      -----------   |     |    -------            |     V
     +---------+      snd SYN,ACK  /       \   snd SYN          +---------+
     |         |<-----------------           ------------------>|         |
     |   SYN   |                    rcv SYN                     |   SYN   |
     |   RCVD  |<-----------------------------------------------|   SENT  |
     |         |                    snd ACK                     |         |
     |         |------------------           -------------------|         |
     +---------+   rcv ACK of SYN  \       /  rcv SYN,ACK       +---------+
       |           --------------   |     |   -----------
       |                  x         |     |     snd ACK
       |                            V     V
       |  CLOSE                   +---------+
       | -------                  |  ESTAB  |
       | snd FIN                  +---------+
       |                   CLOSE    |     |    rcv FIN
       V                  -------   |     |    -------
     +---------+          snd FIN  /       \   snd ACK          +---------+
     |  FIN    |<-----------------           ------------------>|  CLOSE  |
     | WAIT-1  |------------------                              |   WAIT  |
     +---------+          rcv FIN  \                            +---------+
       | rcv ACK of FIN   -------   |                            CLOSE  |
       | --------------   snd ACK   |                           ------- |
       V        x                   V                           snd FIN V
     +---------+                  +---------+                   +---------+
     |FINWAIT-2|                  | CLOSING |                   | LAST-ACK|
     +---------+                  +---------+                   +---------+
       |                rcv ACK of FIN |                 rcv ACK of FIN |
       |  rcv FIN       -------------- |    Timeout=2MSL -------------- |
       |  -------              x       V    ------------        x       V
        \ snd ACK                 +---------+delete TCB         +---------+
         ------------------------>|TIME WAIT|------------------>| CLOSED  |
                                  +---------+                   +---------+

                          TCP Connection State Diagram
                                   Figure 6.

### Three-way Handshake

Build full duplex in three steps
Build one side half duplex in first and second steps
Build another side half duplex in second and third steps

      TCP A                                                 TCP B

      CLOSED                                                LISTEN

      SYN-SENT    --> <SEQ=100><CTL=SYN>                --> SYN-RECEIVED

      ESTABLISHED <-- <SEQ=300><ACK=101><CTL=SYN,ACK>   <-- SYN-RECEIVED

      ESTABLISHED --> <SEQ=101><ACK=301><CTL=ACK>       --> ESTABLISHED

      ESTABLISHED --> <SEQ=101><ACK=301><CTL=ACK><DATA> --> ESTABLISHED

          Basic 3-Way Handshake for Connection Synchronization

                                Figure 7.

### Four-way Handshake

Close full duplex in four steps
Close one side half duplex in first and second steps, and `fin_wait_2` statement for final data received
Passive close side change statement to `close_wait` for sending all the final data
When all the final data have send yet, close another side half duplex in third and fourth steps

      TCP A                                                TCP B

      ESTABLISHED                                          ESTABLISHED

      (Close)
      FIN-WAIT-1  --> <SEQ=100><ACK=300><CTL=FIN,ACK>  --> CLOSE-WAIT

      FIN-WAIT-2  <-- <SEQ=300><ACK=101><CTL=ACK>      <-- CLOSE-WAIT

                                                           (Close)
      TIME-WAIT   <-- <SEQ=300><ACK=101><CTL=FIN,ACK>  <-- LAST-ACK

      TIME-WAIT   --> <SEQ=101><ACK=301><CTL=ACK>      --> CLOSED

      (2 MSL)
      CLOSED

                         Normal Close Sequence

                               Figure 13.

### Sliding Window

### Header Format

        0                   1                   2                   3
        0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
       +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
       |          Source Port          |       Destination Port        |
       +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
       |                        Sequence Number                        |
       +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
       |                    Acknowledgment Number                      |
       +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
       |  Data |           |U|A|P|R|S|F|                               |
       | Offset| Reserved  |R|C|S|S|Y|I|            Window             |
       |       |           |G|K|H|T|N|N|                               |
       +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
       |           Checksum            |         Urgent Pointer        |
       +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
       |                    Options                    |    Padding    |
       +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
       |                             data                              |
       +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+

                                TCP Header Format

              Note that one tick mark represents one bit position.

                                   Figure 3.

### Terminology_

#### Sequence Number

  Every byte has a sequence number. Seq number is the first byte of the data content of the segment.
  Ex: Segment A, the seq is 100, and the content data include 100 byte. So the next segment sequence is 200(1 * 100 + 100)

#### ACK

#### Resend Mechanism

#### Send and Receive cache

#### One message match one ACK

#### Sticking bag and unpacking

#### Congestion control

![congestion control](images/congestion_control.png)

### UDP

This protocol  provides  a procedure  for application  programs  to send
messages  to other programs  with a minimum  of protocol mechanism.  The
protocol  is transaction oriented, and delivery and duplicate protection
are not guaranteed

#### Format

                  0      7 8     15 16    23 24    31
                 +--------+--------+--------+--------+
                 |     Source      |   Destination   |
                 |      Port       |      Port       |
                 +--------+--------+--------+--------+
                 |                 |                 |
                 |     Length      |    Checksum     |
                 +--------+--------+--------+--------+
                 |
                 |          data octets ...
                 +---------------- ...

                      User Datagram Header Format

## IP

This protocol  provides  a procedure  for application  programs  to send messages  to other programs  with a minimum  of protocol mechanism.  The protocol  is transaction oriented, and delivery and duplicate protection are not guaranteed.
The IP module is central to the success of internet technology. Builds a single logical network from multiple physical networks. This interconnection of physical networks is the source of the name: internet

### Routing

#### Direct Routing

Net Topology

                          A      B      C
                          |      |      |
                        --o------o------o--
                        Ethernet 1
                        IP network "development"

                       Figure 6.  One IP Network

Ethernet Fram Schema

                ----------------------------------------
                |address            source  destination|
                ----------------------------------------
                |IP header          A       B          |
                |Ethernet header    A       B          |
                ----------------------------------------
       TABLE 5.  Addresses in an Ethernet frame for an IP packet
                              from A to B

#### Indirect Routing

Net Topology

          A      B      C      ----D----      E      F      G
          |      |      |      |   |   |      |      |      |
        --o------o------o------o-  |  -o------o------o------o--
        Ethernet 1                 |  Ethernet 2
        IP network "development"   |  IP network "accounting"
                                   |
                                   |
                                   |     H      I      J
                                   |     |      |      |
                                 --o-----o------o------o--
                                  Ethernet 3
                                  IP network "factory"

               Figure 7.  Three IP Networks; One internet

Ethernet Fram Schema

                ----------------------------------------
                |address            source  destination|
                ----------------------------------------
                |IP header          A       E          |
                |Ethernet header    A       D          |
                ----------------------------------------
       TABLE 6.  Addresses in an Ethernet frame for an IP packet
                         from A to E (before D)

#### IP Module Routing Rules

IP must decide whether to send the IP packet directly or indirectly, and IP must choose a lower network interface.  These choices are made by consulting the `route table`.
These decisions are made before the IP packet is handed to the lower. If the IP packet is being forwarded, it is treated as an outgoing IP packet. When an incoming IP packet arrives it is never forwarded back out through the same network interface.
These decisions are made before the IP packet is handed to the lower interface and before the ARP table is consulted.

##### IP Address

One part of a 4-byte IP address is the `IP network number`, the other part is the `IP computer number` (or host number)

##### Names

For larger networks, this translation data file is stored on a server and accessed across the network when needed

IP computer number are given names like this:

    223.1.2.1     alpha

IP networks(subnets) are also given names like this:

    223.1.2     development

##### IP Route Table

This table if for searching next Ethernet Address

Primary Column

     ----------------------------------------------------------------------
     |network      direct/indirect flag  router           interface number|
     ----------------------------------------------------------------------
     |223.1.2      direct                <blank>          1               |
     |223.1.3      direct                <blank>          3               |
     |223.1.4      direct                <blank>          2               |
     ----------------------------------------------------------------------
                  TABLE 13.  Delta's Route Table with Numbers

##### Routing Workflow

**Context**:

1. Topology

          ---------           ---------           ---------
          | alpha |           | delta |           |epsilon|
          |    1  |           |1  2  3|           |   1   |
          ---------           ---------           ---------
               |               |  |  |                |
       --------o---------------o- | -o----------------o--------
        Ethernet 1                |     Ethernet 2
        IP network "Development"  |     IP network "accounting"
                                  |
                                  |     --------
                                  |     | iota |
                                  |     |  1   |
                                  |     --------
                                  |        |
                                --o--------o--------
                                    Ethernet 3
                                    IP network "factory"

             Figure 9.  Close-up View of Three IP Networks
2. Alpha IP is `223.1.2.1`
3. Epsilon IP is `223.1.3.2`
4. Alpha Route Table with Numbers

        --------------------------------------------------------------------
        |network      direct/indirect flag  router         interface number|
        --------------------------------------------------------------------
        |223.1.2      direct                <blank>        1               |
        |223.1.3      indirect              223.1.2.4      1               |
        |223.1.4      indirect              223.1.2.4      1               |
        --------------------------------------------------------------------
                     TABLE 11.  Alpha Route Table with Numbers

5. Delta's Route Table

        ----------------------------------------------------------------------
        |network      direct/indirect flag  router           interface number|
        ----------------------------------------------------------------------
        |223.1.2      direct                <blank>          1               |
        |223.1.3      direct                <blank>          3               |
        |223.1.4      direct                <blank>          2               |
        ----------------------------------------------------------------------
                     TABLE 13.  Delta's Route Table with Numbers

```sequence
Note right of Alpha IP Model: extracts the network portion (223.1.3)
Note right of Alpha IP Model: search in Route Table
Note right of Alpha IP Model: find router `223.1.2.4`
Note right of Alpha IP Model: find Ethernet Address of `223.1.2.4` in ARP table
Alpha IP Model -> Delta IP Model: send packet to Epsilon by Ethernet Address throw iface 1
Note left of Delta IP Model: destination IP address is examined
Note left of Delta IP Model: not match & forward the IP packet
Note left of Delta IP Model: extracts the network portion (223.1.3)
Note left of Delta IP Model: find Route Table and direct
Note left of Delta IP Model: IP packet contains the IP destination address of epsilon
Note left of Delta IP Model: Ethernet destination address of epsilon by ARP Table
Delta IP Model -> Epsilon IP Model: send packet to Epsilon by Ethernet Address throw iface 3
Note left of Epsilon IP Model: IP Address match & pass the IP packet to upper layer
```

##### Routing Summary

   When a IP packet travels through a large internet it may go through
   many IP-routers before it reaches its destination.  The path it takes
   is not determined by a central source but is a result of consulting
   each of the routing tables used in the journey

## Ethernet

### Terminology__

**frame:** An Ethernet frame contains the destination address, source address, type field, and data.
**address:** An Ethernet address is 6 bytes. "FF-FF-FF-FF-FF-FF" (in hexadecimal), called a "broadcast" address.
**CSMA/CD:** Ethernet uses CSMA/CD (Carrier Sense and Multiple Access with Collision Detection). CSMA/CD means that all devices communicate on a single medium, that only one can transmit at a time, and that they can all receive simultaneously.  If 2 devices try to transmit at the same instant, the transmit collision is detected, and both devices wait a random (but short) period before trying to transmit again.
**ARP:** Address Resolution Protocol is used to translate IP addresses to Ethernet addresses.  The translation is done only for outgoing IP packets, because this is when the IP header and the Ethernet header are created

### ARP

#### ARP Table

The ARP table is used to look-up the destination Ethernet address:

            ------------------------------------
            |IP address       Ethernet address |
            ------------------------------------
            |223.1.2.1        08-00-39-00-2F-C3|
            |223.1.2.3        08-00-5A-21-A7-22|
            |223.1.2.4        08-00-10-99-AC-54|
            ------------------------------------
                TABLE 1.  Example ARP Table

Two things happen when the ARP table can not be used to translate an address:

1. An ARP request packet with a broadcast Ethernet address is sent out on the network to every computer.
2. The outgoing IP packet is queued.

                ---------------------------------------
                |Sender IP Address   223.1.2.1        |
                |Sender Enet Address 08-00-39-00-2F-C3|
                ---------------------------------------
                |Target IP Address   223.1.2.2        |
                |Target Enet Address <blank>          |
                ---------------------------------------
                     TABLE 2.  Example ARP Request

Every computer's Ethernet interface receives the broadcast Ethernet frame. Each ARP module examines the IP address and if the Target IP address matches its own IP address, it sends a response directly to the source Ethernet address.

                ---------------------------------------
                |Sender IP Address   223.1.2.2        |
                |Sender Enet Address 08-00-28-00-38-A9|
                ---------------------------------------
                |Target IP Address   223.1.2.1        |
                |Target Enet Address 08-00-39-00-2F-C3|
                ---------------------------------------
                     TABLE 3.  Example ARP Response

## Q&A

### Why don't use MAC address instead of IP for having “Internet” or doing communication

1. The reason for that is the routing. If you have an IP, for example 104.103.84.161 for the www.microsoft.com, every router on the Internet knows, where to send packets to this IP address. They are organized into a tree-like structure in the Internet, where the IP networks of the organizations have a global registry. In the case of the MAC addresses, there is no such thing. From a MAC address you can only see which company manufactured the chip, but no more.

2. MAC addresses are normally fixed in the NIC firmware, while IP addresses can be freely set everywhere.

3. There is also a deeper reason: in the cases of the NIC, there is no guarantee that you want to make IP traffic on them, it is only a quasi-common standard. Earlier existed different protocols (for example, IPX, or the microsoft win2000 servers also had one) which didn't use IP, but another protocol over the NIC cards.

4. IP is the standard of the Internet, while the MAC is the standard of the Ethernet. It is not obligatory to use IP from Ethernet, for example in the ancient times there were ARCNET cards which used a quite different protocol (as I know, their "MAC" were much shorter, too). Or there are VPN protocols which also traffic only IP packets without an Ethernet frame. So, IP is the common of the Internet, and you can connect into it with everything. Most people connects with Ethernet packets but it is not needed.

5. Btw, having an "Internet" which uses MAC addresses instead of IP, would be possible, although it would require to develop the complex protocols and shared databases on every protocol layer to use them. They are developed for IP in the times where the Ethernet didn't even existed. So, the final answer to your question is the social inertia.

6. IP hides the underlying network hardware from the network applications.  If you invent a new physical network, you can put it into service by implementing a new driver that connects to the internet underneath IP.  Thus, the network applications remain intact and are not vulnerable to changes in hardware technology.

### TCP / HTTP Listening On Ports: How Can Many Users Share the Same Port

1. On a server, a process is listening on a port. Once it gets a connection, it hands it off to another thread. The communication never hogs the listening port.
2. Connections are uniquely identified by the OS by the following 5-tuple: (local-IP, local-port, remote-IP, remote-port, protocol). If any element in the tuple is different, then this is a completely independent connection.
3. When a client connects to a server, it picks a random, unused high-order source port. This way, a single client can have up to ~64k connections to the server for the same destination port.

So, this is really what gets created when a client connects to a server:

| Local Computer   | Remote Computer           | Role |
|-|-|-|
|0.0.0.0:80       | \<none>                 | LISTENING |
|127.0.0.1:80     | 10.1.2.3:<random_port>  | ESTABLISHED |

here is the output of running netstat again:

    Proto Recv-Q Send-Q Local Address           Foreign Address         State  
    tcp        0      0 0.0.0.0:500             0.0.0.0:*               LISTEN

So now there is one process that is actively listening (State: LISTEN) on port 500. The local address is 0.0.0.0, which is code for "listening for all ip addresses". An easy mistake to make is to only listen on port 127.0.0.1, which will only accept connections from the current computer. So this is not a connection, this just means that a process requested to bind() to port IP, and that process is responsible for handling all connections to that port. This hints to the limitation that there can only be one process per computer listening on a port (there are ways to get around that using multiplexing, but this is a much more complicated topic). If a web-server is listening on port 80, it cannot share that port with other web-servers.

## Reference

1. [A TCP/IP Tutorail](https://datatracker.ietf.org/doc/html/rfc1180)
2. [Why don't we use MAC address instead of IP for having “Internet” or doing communication?](https://networkengineering.stackexchange.com/questions/32358/why-dont-we-use-mac-address-instead-of-ip-for-having-internet-or-doing-commun)
3. [TRANSMISSOIN CONTROL PROTOCOL](https://www.rfc-editor.org/rfc/rfc793.html)
4. [How can user share listening port of one service](https://stackoverflow.com/questions/11129212/tcp-can-two-different-sockets-share-a-port)
