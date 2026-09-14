
C2950 Boot Loader (C2950-HBOOT-M) Version 12.1(11r)EA1, RELEASE SOFTWARE (fc1)

Compiled Mon 22-Jul-02 18:57 by miwang

Cisco WS-C2950T-24 (RC32300) processor (revision C0) with 21039K bytes of memory.

2950T-24 starting...

Base ethernet MAC Address: 0001.427A.96AC

Xmodem file system is available.

Initializing Flash...

flashfs[0]: 1 files, 0 directories

flashfs[0]: 0 orphaned files, 0 orphaned directories

flashfs[0]: Total bytes: 64016384

flashfs[0]: Bytes used: 3058048

flashfs[0]: Bytes available: 60958336

flashfs[0]: flashfs fsck took 1 seconds.

...done Initializing Flash.

  

Boot Sector Filesystem (bs:) installed, fsid: 3

Parameter Block Filesystem (pb:) installed, fsid: 4

  

  

Loading "flash:/c2950-i6q4l2-mz.121-22.EA4.bin"...

########################################################################## [OK]

Restricted Rights Legend

  

Use, duplication, or disclosure by the Government is

subject to restrictions as set forth in subparagraph

(c) of the Commercial Computer Software - Restricted

Rights clause at FAR sec. 52.227-19 and subparagraph

(c) (1) (ii) of the Rights in Technical Data and Computer

Software clause at DFARS sec. 252.227-7013.

  

cisco Systems, Inc.

170 West Tasman Drive

San Jose, California 95134-1706

  

  

  

Cisco Internetwork Operating System Software

IOS (tm) C2950 Software (C2950-I6Q4L2-M), Version 12.1(22)EA4, RELEASE SOFTWARE(fc1)

Copyright (c) 1986-2005 by cisco Systems, Inc.

Compiled Wed 18-May-05 22:31 by jharirba

  

Cisco WS-C2950T-24 (RC32300) processor (revision C0) with 21039K bytes of memory.

Processor board ID FHK0610Z0WC

Running Standard Image

24 FastEthernet/IEEE 802.3 interface(s)

2 Gigabit Ethernet/IEEE 802.3 interface(s)

  

63488K bytes of flash-simulated non-volatile configuration memory.

Base ethernet MAC Address: 0001.427A.96AC

Motherboard assembly number: 73-5781-09

Power supply part number: 34-0965-01

Motherboard serial number: FOC061004SZ

Power supply serial number: DAB0609127D

Model revision number: C0

Motherboard revision number: A0

Model number: WS-C2950T-24

System serial number: FHK0610Z0WC

  

Cisco Internetwork Operating System Software

IOS (tm) C2950 Software (C2950-I6Q4L2-M), Version 12.1(22)EA4, RELEASE SOFTWARE(fc1)

Copyright (c) 1986-2005 by cisco Systems, Inc.

Compiled Wed 18-May-05 22:31 by jharirba

  

Press RETURN to get started!

  

  

%LINK-5-CHANGED: Interface FastEthernet0/2, changed state to up

  

%LINEPROTO-5-UPDOWN: Line protocol on Interface FastEthernet0/2, changed state to up

  

%LINK-3-UPDOWN: Interface FastEthernet0/2, changed state to down

  

%LINEPROTO-5-UPDOWN: Line protocol on Interface FastEthernet0/2, changed state to down

  

%LINK-5-CHANGED: Interface FastEthernet0/2, changed state to up

  

%LINEPROTO-5-UPDOWN: Line protocol on Interface FastEthernet0/2, changed state to up

  

%LINK-3-UPDOWN: Interface FastEthernet0/2, changed state to down

  

%LINEPROTO-5-UPDOWN: Line protocol on Interface FastEthernet0/2, changed state to down

  

%LINK-5-CHANGED: Interface FastEthernet0/2, changed state to up

  

%LINEPROTO-5-UPDOWN: Line protocol on Interface FastEthernet0/2, changed state to up

  

%LINK-5-CHANGED: Interface FastEthernet0/3, changed state to up

  

%LINEPROTO-5-UPDOWN: Line protocol on Interface FastEthernet0/3, changed state to up

  

%LINK-3-UPDOWN: Interface FastEthernet0/3, changed state to down

  

%LINEPROTO-5-UPDOWN: Line protocol on Interface FastEthernet0/3, changed state to down

  

%LINK-5-CHANGED: Interface FastEthernet0/4, changed state to up

  

%LINEPROTO-5-UPDOWN: Line protocol on Interface FastEthernet0/4, changed state to up

  

%LINK-3-UPDOWN: Interface FastEthernet0/4, changed state to down

  

%LINEPROTO-5-UPDOWN: Line protocol on Interface FastEthernet0/4, changed state to down

  

%LINK-5-CHANGED: Interface FastEthernet0/4, changed state to up

  

%LINEPROTO-5-UPDOWN: Line protocol on Interface FastEthernet0/4, changed state to up

  

  

Switch>

Switch>enable

Switch#CONFI

Switch#CONFIgure ter

Switch#CONFIgure terminal

Enter configuration commands, one per line. End with CNTL/Z.

Switch(config)#host

Switch(config)#hostname sw1

sw1(config)#line vty 0 4

sw1(config-line)#password cisco

sw1(config-line)#login

sw1(config-line)#exit

sw1(config)#pas

sw1(config)#pass

sw1(config)#enable pass

sw1(config)#enable password cis

sw1(config)#enable password cisco

sw1(config)#interface vlan 1

sw1(config-if)#ip addre

sw1(config-if)#ip address 1.0.0.2 255.0.0.0

sw1(config-if)#no shut

sw1(config-if)#no shutdown

  

sw1(config-if)#

%LINK-3-UPDOWN: Interface Vlan1, changed state to down

  

%LINEPROTO-5-UPDOWN: Line protocol on Interface Vlan1, changed state to up

  

sw1(config-if)#exit

sw1(config)#ip defa

sw1(config)#ip default-gateway 1.0.0.1

sw1(config)#exit

sw1#

%SYS-5-CONFIG_I: Configured from console by console

  

sw1#confi

sw1#configure ter

sw1#configure terminal

Enter configuration commands, one per line. End with CNTL/Z.

sw1(config)#vlan 10

sw1(config-vlan)#name IN

sw1(config-vlan)#name INGENIERIA

sw1(config-vlan)#vlan 20

sw1(config-vlan)#EXIT

sw1(config)#vlan 20

sw1(config-vlan)#name VENTAS

sw1(config-vlan)#exit

sw1(config)#inter

sw1(config)#interface fas

sw1(config)#interface fastEthernet 0/2

sw1(config-if)#swit

sw1(config-if)#switchport mod

sw1(config-if)#switchport mode acc

sw1(config-if)#switchport mode access

sw1(config-if)#sw

sw1(config-if)#switchport acc

sw1(config-if)#switchport access vlan 10

sw1(config-if)#exit

sw1(config)#inter

sw1(config)#interface fa

sw1(config)#interface fastEthernet 0/4

sw1(config-if)#sw

sw1(config-if)#switchport mode acces

sw1(config-if)#switchport mode access

sw1(config-if)#sw

sw1(config-if)#switchport acc

sw1(config-if)#switchport access vlan 20

sw1(config-if)#

%LINEPROTO-5-UPDOWN: Line protocol on Interface Vlan1, changed state to down

  

sw1(config-if)#exit

sw1(config)#interface fa

sw1(config)#interface fastEthernet 0/1

sw1(config-if)#sw

sw1(config-if)#switchport mode tr

sw1(config-if)#switchport mode trunk

sw1(config-if)#exit

sw1(config)#exit

sw1#

%SYS-5-CONFIG_I: Configured from console by console

run

sw1#copy ru

sw1#copy running-config st

sw1#copy running-config startup-config

Destination filename [startup-config]?

Building configuration...

[OK]

sw1#

%LINK-5-CHANGED: Interface FastEthernet0/1, changed state to up

  

%LINEPROTO-5-UPDOWN: Line protocol on Interface FastEthernet0/1, changed state to up

  

%LINEPROTO-5-UPDOWN: Line protocol on Interface Vlan1, changed state to up