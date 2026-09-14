System Bootstrap, Version 12.1(3r)T2, RELEASE SOFTWARE (fc1)

Copyright (c) 2000 by cisco Systems, Inc.

Cisco 2620 (MPC860) processor (revision 0x200) with 253952K/8192K bytes of memory

  

Readonly ROMMON initialized

  

Self decompressing the image :

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

IOS (tm) C2600 Software (C2600-I-M), Version 12.2(28), RELEASE SOFTWARE (fc5)

Technical Support: http://www.cisco.com/techsupport

Copyright (c) 1986-2005 by cisco Systems, Inc.

Compiled Wed 27-Apr-04 19:01 by miwang

  

Cisco 2620 (MPC860) processor (revision 0x200) with 253952K/8192K bytes of memory

.

Processor board ID JAD05190MTZ (4292891495)

M860 processor: part number 0, mask 49

Bridging software.

X.25 software, Version 3.0.0.

1 FastEthernet/IEEE 802.3 interface(s)

32K bytes of non-volatile configuration memory.

63488K bytes of ATA CompactFlash (Read/Write)

  

  

--- System Configuration Dialog ---

  

Would you like to enter the initial configuration dialog? [yes/no]:

% Please answer 'yes' or 'no'.

Would you like to enter the initial configuration dialog? [yes/no]:

% Please answer 'yes' or 'no'.

Would you like to enter the initial configuration dialog? [yes/no]: no

  

  

Press RETURN to get started!

  

  

  

Router>

Router>en

Router>enable

Router#conf

Router#configure ter

Router#configure terminal

Enter configuration commands, one per line. End with CNTL/Z.

Router(config)#hos

Router(config)#hostname R

Router(config)#hostname Router0

Router0(config)#interfa

Router0(config)#interface fas

Router0(config)#interface fastEthernet 0/0

Router0(config-if)#no i

Router0(config-if)#no ip ad

Router0(config-if)#no ip address

Router0(config-if)#no sh

Router0(config-if)#no shutdown

  

Router0(config-if)#

%LINK-5-CHANGED: Interface FastEthernet0/0, changed state to up

  

%LINEPROTO-5-UPDOWN: Line protocol on Interface FastEthernet0/0, changed state to up

  

Router0(config-if)#exit

Router0(config)#inter

Router0(config)#interface fas

Router0(config)#interface fastEthernet 0/0.1

Router0(config-subif)#

%LINK-3-UPDOWN: Interface FastEthernet0/0.1, changed state to down

  

%LINEPROTO-5-UPDOWN: Line protocol on Interface FastEthernet0/0.1, changed state to up

  

Router0(config-subif)#encap

Router0(config-subif)#encapsulation dot

Router0(config-subif)#encapsulation dot1Q 1 native

Router0(config-subif)#ip ad

Router0(config-subif)#ip address 1.0.0.1 255.0.0.0

Router0(config-subif)#exit

Router0(config)#inter

Router0(config)#interface fas

Router0(config)#interface fastEthernet 0/0.2

Router0(config-subif)#

%LINK-3-UPDOWN: Interface FastEthernet0/0.2, changed state to down

  

%LINEPROTO-5-UPDOWN: Line protocol on Interface FastEthernet0/0.2, changed state to up

  

Router0(config-subif)#encap

Router0(config-subif)#encapsulation dot

Router0(config-subif)#encapsulation dot1Q 10

Router0(config-subif)#ip ad

Router0(config-subif)#ip address 10.0.0.1 255.0.0.0

Router0(config-subif)#exit

Router0(config)#inter

Router0(config)#interface fast

Router0(config)#interface fastEthernet 0/0.3

Router0(config-subif)#

%LINK-3-UPDOWN: Interface FastEthernet0/0.3, changed state to down

  

%LINEPROTO-5-UPDOWN: Line protocol on Interface FastEthernet0/0.3, changed state to up

  

Router0(config-subif)#enca

Router0(config-subif)#encapsulation do

Router0(config-subif)#encapsulation dot1Q 20

Router0(config-subif)#ip ad

Router0(config-subif)#ip address 20.0.0.1 255.0.0.0

Router0(config-subif)#exit

Router0(config)#exit

Router0#

%SYS-5-CONFIG_I: Configured from console by console

  

Router0#copy runn

Router0#copy running-config star

Router0#copy running-config startup-config

Destination filename [startup-config]?

Building configuration...

[OK]

Router0#