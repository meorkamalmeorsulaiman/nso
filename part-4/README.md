# Syncing Devices to NSO

This part we will connect all the devices to NSO and sync all the configuration. We device this into 2 part:
- Connecting to NSO
- Sync device configuration

## To Connect Devices to NSO

There are several authentication method to use for NSO to connect to the remote devices. In this lab we will be using simple SSH. First we have setup SSH on the remote devices. We are not covering that in this part and Let's proceed to configure NSO and adding all the devices by entering the CLI

```bash
sysadmin@nso01:~/nso-lab$ ncs_cli

sysadmin connected from 192.168.1.15 using ssh on nso01
sysadmin@ncs> switch cli
sysadmin@ncs#
```

Then configure the authentication group, we going to split the authentication for IOS-XE and IOS-XR

```
sysadmin@ncs# config t
Entering configuration mode terminal
sysadmin@ncs(config)# devices authgroups group NSO-IOSXE 
sysadmin@ncs(config-group-NSO-IOSXE)# default-map remote-name nso
sysadmin@ncs(config-group-NSO-IOSXE)# default-map remote-password [USERNAME PASSWORD]
sysadmin@ncs(config-group-NSO-IOSXE)# default-map remote-secondary-password [ENABLE PASSWORD]
sysadmin@ncs(config-group-NSO-IOSXE)# top 
sysadmin@ncs(config)# devices authgroups group NSO-IOSXR
sysadmin@ncs(config-group-NSO-IOSXR)# default-map remote-name xrv
sysadmin@ncs(config-group-NSO-IOSXR)# default-map remote-password [USERNAME PASSWORD]
```
The `remote-secondary-password` is actually the enable secret, furthermore in IOS-XR we are not enabling secret therefore there is no need to add that in the auth group. Let's proceed to add devices

```
sysadmin@ncs(config-group-NSO-IOSXR)# top
sysadmin@ncs(config)# devices device R9
sysadmin@ncs(config-device-R9)# address 192.168.101.109
sysadmin@ncs(config-device-R9)# ssh host-key-verification none 
sysadmin@ncs(config-device-R9)# authgroup NSO-IOSXR
sysadmin@ncs(config-device-R9)# ssh host-key-verification none 
sysadmin@ncs(config-device-R9)# device-type cli ned-id cisco-iosxr-cli-7.55 
sysadmin@ncs(config-device-R9)# device-type cli protocol ssh 
sysadmin@ncs(config-device-R9)# state admin-state unlocked 
sysadmin@ncs(config-device-R9)# top
sysadmin@ncs(config)# devices device R7
sysadmin@ncs(config-device-R7)# address 192.168.101.107
sysadmin@ncs(config-device-R7)# authgroup NSO-IOSXE 
sysadmin@ncs(config-device-R7)# device-type cli ned-id cisco-ios-cli-6.106 
sysadmin@ncs(config-device-R7)# device-type cli protocol ssh
sysadmin@ncs(config-device-R7)# ssh host-key-verification none 
sysadmin@ncs(config-device-R7)# state admin-state unlocked
sysadmin@ncs(config-device-R7)# commit
Commit complete.
```
That's all for adding the devices and now let's test the connection toward the remote devices

```
sysadmin@ncs# devices connect 
connect-result {
    device R7
    result false
    info Failed to authenticate towards device R7: SSH key exchange failed
}
connect-result {
    device R9
    result false
    info Failed to authenticate towards device R9: SSH key exchange failed
}
sysadmin@ncs#
```

We have problem with RSA algo, we need to configure   

```
sysadmin@ncs(config)# devices global-settings ssh-algorithms public-key ssh-rsa
sysadmin@ncs(config)# commit
Commit complete.
sysadmin@ncs(config)# end
sysadmin@ncs# devices connect    
connect-result {
    device R7
    result true
    info (sysadmin) Connected to R7 - 192.168.101.107:22
}
connect-result {
    device R9
    result true
    info (sysadmin) Connected to R9 - 192.168.101.109:22
}
```
We have succesfully connected and we can add the rest of devices and look at the list as below

```
sysadmin@ncs# devices connect
connect-result {
    device R10
    result true
    info (sysadmin) Connected to R10 - 192.168.101.110:22
}
connect-result {
    device R11
    result true
    info (sysadmin) Connected to R11 - 192.168.101.111:22
}
connect-result {
    device R12
    result true
    info (sysadmin) Connected to R12 - 192.168.101.112:22
}
connect-result {
    device R13
    result true
    info (sysadmin) Connected to R13 - 192.168.101.113:22
}
connect-result {
    device R14
    result true
    info (sysadmin) Connected to R14 - 192.168.101.114:22
}
connect-result {
    device R15
    result true
    info (sysadmin) Connected to R15 - 192.168.101.115:22
}
connect-result {
    device R16
    result true
    info (sysadmin) Connected to R16 - 192.168.101.116:22
}
connect-result {
    device R7
    result true
    info (sysadmin) Connected to R7 - 192.168.101.107:22
}
connect-result {
    device R8
    result true
    info (sysadmin) Connected to R8 - 192.168.101.108:22
}
connect-result {
    device R9
    result true
    info (sysadmin) Connected to R9 - 192.168.101.109:22
}
sysadmin@ncs# show devices list 
NAME  ADDRESS          DESCRIPTION  NED ID                ADMIN STATE  
---------------------------------------------------------------------
R10   192.168.101.110  -            cisco-iosxr-cli-7.55  unlocked     
R11   192.168.101.111  -            cisco-iosxr-cli-7.55  unlocked     
R12   192.168.101.112  -            cisco-iosxr-cli-7.55  unlocked     
R13   192.168.101.113  -            cisco-iosxr-cli-7.55  unlocked     
R14   192.168.101.114  -            cisco-iosxr-cli-7.55  unlocked     
R15   192.168.101.115  -            cisco-iosxr-cli-7.55  unlocked     
R16   192.168.101.116  -            cisco-iosxr-cli-7.55  unlocked     
R7    192.168.101.107  -            cisco-ios-cli-6.106   unlocked     
R8    192.168.101.108  -            cisco-ios-cli-6.106   unlocked     
R9    192.168.101.109  -            cisco-iosxr-cli-7.55  unlocked
```
Let's proceed to sync the device configuration


## Syncing devices configuration to NSO

There are several ways to sync the configuration, since this is a brownfield deployment. We are going to sync from the device to NSO by first, check the sync state

```
sysadmin@ncs# devices check-sync 
sync-result {
    device R10
    result unknown
}
sync-result {
    device R11
    result unknown
}
sync-result {
    device R12
    result unknown
}
sync-result {
    device R13
    result unknown
}
sync-result {
    device R14
    result unknown
}
sync-result {
    device R15
    result unknown
}
sync-result {
    device R16
    result unknown
}
sync-result {
    device R7
    result unknown
}
sync-result {
    device R8
    result unknown
}
sync-result {
    device R9
    result unknown
}
```
NSO doesn't now what is the devices configuration, let's sync the state now

```
sysadmin@ncs# devices device R10 sync-? 
Possible completions:
  sync-from - Synchronize the config by pulling from the device
  sync-to   - Synchronize the config by pushing to the device
sysadmin@ncs# devices check-sync device R10
sync-result {
    device R10
    result in-sync
}
```

Now we can see that R10 have synced it configuration. You notice that there 2 ways to sync the configuration explained in the output above. Let's sync the device all at once

```
sysadmin@ncs# devices sync-from 
sync-result {
    device R10
    result true
}
sync-result {
    device R11
    result true
}
sync-result {
    device R12
    result true
}
sync-result {
    device R13
    result true
}
sync-result {
    device R14
    result true
}
sync-result {
    device R15
    result true
}
sync-result {
    device R16
    result true
}
sync-result {
    device R7
    result true
}
sync-result {
    device R8
    result true
}
sync-result {
    device R9
    result true
}
```
Let's check the VRF configuration on R10

```
sysadmin@ncs# show running-config devices device R10 config vrf 
devices device R10
 config
  vrf green
   description GREEN-SITE-1
   address-family ipv4 unicast
    import route-target
     64500:101
    exit
    export route-target
     65400:101
    exit
   exit
   address-family ipv6 unicast
    import route-target
     64500:101
    exit
    export route-target
     65400:101
    exit
   exit
  exit
 !
!
```

We can already see the configuration from NSO. That's all and we are done for this part let continue on the next part.






