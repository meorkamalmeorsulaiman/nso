# Making Configuration Changes

The idea is to apply configuration to the router using NSO

## Adding IP address on R2 

We can see that R1 p2p interface has been configured

```
sysadmin@ncs# show running-config devices device R1 config interface GigabitEthernet 2
devices device R1
 config
  interface GigabitEthernet2
   description R2
   no switchport
   negotiation auto
   ip address 10.1.2.1 255.255.255.0
   ip ospf 100 area 0
   ip ospf network point-to-point
   shutdown
  exit
 !
!
```

Now we want to apply the configuration on R2 p2p interface

```
sysadmin@ncs# config t
Entering configuration mode terminal
sysadmin@ncs(config)# devices device R2 config interface GigabitEthernet 0/0/0/0
sysadmin@ncs(config-if)# ipv4 address 10.1.2.2 /24
sysadmin@ncs(config-if)# no shutdown
sysadmin@ncs(config-if)# description R1
```

Before we apply the config we can check the conifguration

```
sysadmin@ncs(config)# commit dry-run outformat native
native {
    device {
        name R2
        data interface GigabitEthernet 0/0/0/0
              description R1
              ipv4 address 10.1.2.2/24
              no shutdown
             exit
    }
}
```
Above shows that the config will be added on the interface, now we commit the changes with a detail proceed running

```
sysadmin@ncs(config)# commit | details
applying transaction for running datastore usid=47 tid=361 trace-id=6fde4b5c6c28d1e1e9d81585421d8231
 2025-04-22T14:42:28.528 waiting to apply... ok (0.000 s)
entering validate phase
 2025-04-22T14:42:28.528 creating rollback checkpoint... ok (0.000 s)
 2025-04-22T14:42:28.528 creating rollback file... ok (0.001 s)
 2025-04-22T14:42:28.530 creating pre-transform checkpoint... ok (0.000 s)
 2025-04-22T14:42:28.530 creating transform checkpoint... ok (0.000 s)
 2025-04-22T14:42:28.530 run transforms and transaction hooks... ok (0.000 s)
 2025-04-22T14:42:28.531 creating validation checkpoint... ok (0.000 s)
 2025-04-22T14:42:28.531 mark inactive... ok (0.000 s)
 2025-04-22T14:42:28.531 pre validate... ok (0.000 s)
 2025-04-22T14:42:28.531 run validation over the changeset... ok (0.000 s)
 2025-04-22T14:42:28.532 run dependency-triggered validation... ok (0.000 s)
 2025-04-22T14:42:28.532 check configuration policies... ok (0.000 s)
 2025-04-22T14:42:28.532 check for read-write conflicts... ok (0.000 s)
 2025-04-22T14:42:28.532 taking transaction lock... ok (0.000 s)
 2025-04-22T14:42:28.532 holding transaction lock...
 2025-04-22T14:42:28.532 check for read-write conflicts... ok (0.000 s)
leaving validate phase (0.004 s)
entering write-start phase
 2025-04-22T14:42:28.532 cdb: write-start
 2025-04-22T14:42:28.533 service-manager: write-start
 2025-04-22T14:42:28.533 device-manager: write-start
 2025-04-22T14:42:28.533 cdb: match subscribers... ok (0.000 s)
 2025-04-22T14:42:28.533 cdb: create pre commit running... ok (0.000 s)
 2025-04-22T14:42:28.533 cdb: write changeset... ok (0.000 s)
 2025-04-22T14:42:28.534 check data kickers... ok (0.000 s)
leaving write-start phase (0.001 s)
entering prepare phase
 2025-04-22T14:42:28.534 cdb: prepare
 2025-04-22T14:42:28.535 device-manager: prepare
 2025-04-22T14:42:29.784 device R2: push configuration...
leaving prepare phase (6.355 s)
entering commit phase
 2025-04-22T14:42:34.889 cdb: commit
 2025-04-22T14:42:34.889 cdb: switch to new running... ok (0.002 s)
 2025-04-22T14:42:34.893 device-manager: commit
 2025-04-22T14:42:34.893 device R2: push configuration: ok (5.108 s)
 2025-04-22T14:42:35.611 holding transaction lock: ok (7.079 s)
leaving commit phase (0.722 s)
applying transaction for running datastore usid=47 tid=361 trace-id=6fde4b5c6c28d1e1e9d81585421d8231 (7.083 s)
Commit complete.
```

## Test

Now we can see the configuration applied
```
RP/0/0/CPU0:R2#show ip int br | ex down
Tue Apr 22 14:42:51.208 UTC

Interface                      IP-Address      Status          Protocol Vrf-Name
Loopback0                      10.1.1.2        Up              Up       default
MgmtEth0/0/CPU0/0              192.168.101.102 Up              Up       default
GigabitEthernet0/0/0/0         10.1.2.2        Up              Up       default
```

The p2p now working as expected
```
RP/0/0/CPU0:R2#ping 10.1.2.1
Tue Apr 22 14:46:47.822 UTC
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.1.2.1, timeout is 2 seconds:
.!!!!
Success rate is 80 percent (4/5), round-trip min/avg/max = 1/1/1 ms
```
