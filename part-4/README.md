# Syncing Devices to NSO

This part we will connect all the devices to NSO and sync all the configuration. We device this into 2 part:
- Connecting to NSO
- Sync device configuration

## To Connect Devices to NSO

There are several authentication method to use for NSO to connect to the remote devices. In this lab we will be using simple SSH. First we have setup SSH on the remote devices. We are not covering that in this part and Let's proceed to configure NSO and adding all the devices by entering the CLI

```bash
sysadmin@nso01:~/nso-lab$ ncs_cli

sysadmin connected from 192.168.1.10 using ssh on nso01
sysadmin@ncs> switch cli
sysadmin@ncs#
```

Then configure the authentication group, we going to split the authentication for IOS-XE and IOS-XR

```
sysadmin@ncs# config t
Entering configuration mode terminal
sysadmin@ncs(config)# devices authgroups group ios-xe
sysadmin@ncs(config-group-ios-xe)# default-map remote-name nso
sysadmin@ncs(config-group-ios-xe)# default-map remote-password [USER PASSWORD]
sysadmin@ncs(config-group-ios-xe)# default-map remote-secondary-password [ENABLE PASSWORD]
sysadmin@ncs(config-group-ios-xe)# top
sysadmin@ncs(config)# devices authgroups group ios-xr
sysadmin@ncs(config-group-ios-xr)# default-map remote-name nso
sysadmin@ncs(config-group-ios-xr)# default-map remote-password [USER PASSWORD]
```
The `remote-secondary-password` is actually the enable secret, furthermore in IOS-XR we are not enabling secret therefore there is no need to add that in the auth group. Let's proceed to add devices

```
sysadmin@ncs(config)# devices device R1
sysadmin@ncs(config-device-R1)# address 192.168.101.101
sysadmin@ncs(config-device-R1)# ssh host-key-verification none 
sysadmin@ncs(config-device-R1)# authgroup ios-xe
sysadmin@ncs(config-device-R1)# device-type cli ned-id cisco-ios-cli-6.107
sysadmin@ncs(config-device-R1)# device-type cli protocol ssh 
sysadmin@ncs(config-device-R1)# state admin-state unlocked 
sysadmin@ncs(config-device-R1)# top
sysadmin@ncs(config)# devices device R2
sysadmin@ncs(config-device-R2)# address 192.168.101.102
sysadmin@ncs(config-device-R2)# authgroup ios-xr
sysadmin@ncs(config-device-R2)# device-type cli ned-id cisco-iosxr-cli-7.61
sysadmin@ncs(config-device-R2)# device-type cli protocol ssh
sysadmin@ncs(config-device-R2)# ssh host-key-verification none
sysadmin@ncs(config-device-R2)# state admin-state unlocked
sysadmin@ncs(config-device-R2)# commit
Commit complete.
```
That's all for adding the devices and now let's test the connection toward the remote devices

```
sysadmin@ncs# devices connect device R1
connect-result {
    device R1
    result true
    info (sysadmin) Connected to R1 - 192.168.101.101:22
}
sysadmin@ncs# devices connect device R2
connect-result {
    device R2
    result false
    info Failed to connect to device R2: connection refused: Failed to connect: NEDCOM CONNECT: Unable to reach a settlement of HostKeyAlgorithms: [ssh-ed25519, ecdsa-sha2-nistp256, ecdsa-sha2-nistp384, ecdsa-sha2-nistp521, rsa-sha2-512, rsa-sha2-256] and [ssh-rsa] in new state
}
sysadmin@ncs# *** ALARM connection-failure: Failed to connect to device R2: connection refused: Failed to connect: NEDCOM CONNECT: Unable to reach a settlement of HostKeyAlgorithms: [ssh-ed25519, ecdsa-sha2-nistp256, ecdsa-sha2-nistp384, ecdsa-sha2-nistp521, rsa-sha2-512, rsa-sha2-256] and [ssh-rsa] in new state
```

We have problem with RSA algo, we need to configure   

```
sysadmin@ncs(config)# devices global-settings ssh-algorithms public-key ssh-rsa
sysadmin@ncs(config)# commit
Commit complete.
sysadmin@ncs(config)# end
sysadmin@ncs# devices connect
connect-result {
    device R1
    result true
    info (sysadmin) Connected to R1 - 192.168.101.101:22
}
connect-result {
    device R2
    result true
    info (sysadmin) Connected to R2 - 192.168.101.102:22
}
```
We have succesfully connected and we can add the rest of devices and look at the list as below

```
sysadmin@ncs# show devices list
NAME  ADDRESS          DESCRIPTION  NED ID                ADMIN STATE
---------------------------------------------------------------------
R1    192.168.101.101  -            cisco-ios-cli-6.107   unlocked
R2    192.168.101.102  -            cisco-iosxr-cli-7.61  unlocked
```
Let's proceed to sync the device configuration


## Syncing devices configuration to NSO

There are several ways to sync the configuration, since this is a brownfield deployment. We are going to sync from the device to NSO by first, check the sync state

```
sysadmin@ncs# devices check-sync
sync-result {
    device R1
    result unknown
}
sync-result {
    device R2
    result unknown
}
```
NSO doesn't now what are the devices configuration, let's sync the state now

```
sysadmin@ncs# devices device R1 sync-?
Possible completions:
  sync-from - Synchronize the config by pulling from the device
  sync-to   - Synchronize the config by pushing to the device
sysadmin@ncs# devices device R1 sync-from
result true
sysadmin@ncs# devices check-sync device R1
sync-result {
    device R1
    result in-sync
}
```

Now we can see that R1 have synced it configuration. You notice that there 2 ways to sync the configuration explained in the output above. Let's sync the device all at once

```
sysadmin@ncs# devices sync-from
sync-result {
    device R1
    result true
}
sync-result {
    device R2
    result true
}
```
Let's check the VRF configuration on R1 and R2

```
sysadmin@ncs# show running-config devices device R1 config vrf
devices device R1
 config
  vrf definition vrf-1
   rd 1:1
   route-target export 1:1
   route-target import 1:1
  !
  vrf definition vrf-2
   rd 1:2
   route-target export 2:2
   route-target import 2:2
  !
 !
!
sysadmin@ncs# show running-config devices device R2 config vrf
devices device R2
 config
  vrf vrf-1
   rd 2:1
   address-family ipv4 unicast
    import route-target
     1:1
    exit
    export route-target
     1:1
    exit
   exit
  exit
  vrf vrf-2
   rd 2:2
   address-family ipv4 unicast
    import route-target
     2:2
    exit
    export route-target
     2:2
    exit
   exit
  exit
 !
!
```

We can already see the configuration from NSO. That's all and we are done for this part let continue on the next part.






