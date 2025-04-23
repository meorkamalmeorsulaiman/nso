# API Paths

The idea is to get the API paths. 


## Display as RESTCONF

We can display the output from NSO cli to get the API path. Below is how we can get the config API path for a particular interface in IOS-XE 

```
sysadmin@ncs# show running-config devices device R1 config interface GigabitEthernet 2 | display restconf
/restconf/data/tailf-ncs:devices/device=R1/config/tailf-ned-cisco-ios:interface/GigabitEthernet=2/description R2
/restconf/data/tailf-ncs:devices/device=R1/config/tailf-ned-cisco-ios:interface/GigabitEthernet=2/negotiation/auto true
/restconf/data/tailf-ncs:devices/device=R1/config/tailf-ned-cisco-ios:interface/GigabitEthernet=2/ip/address/primary/address 10.1.2.1
/restconf/data/tailf-ncs:devices/device=R1/config/tailf-ned-cisco-ios:interface/GigabitEthernet=2/ip/address/primary/mask 255.255.255.0
/restconf/data/tailf-ncs:devices/device=R1/config/tailf-ned-cisco-ios:interface/GigabitEthernet=2/ip/ospf/process-id=100/area 0
/restconf/data/tailf-ncs:devices/device=R1/config/tailf-ned-cisco-ios:interface/GigabitEthernet=2/ip/ospf/network [ point-to-point ]
```

Now we can take the root path and apply in our script as below

```
import requests

username = input("Username:")
password = input("password:")

url = "http://IP-ADDRESS:8080/restconf/data/tailf-ncs:devices/device=R1/config/tailf-ned-cisco-ios:interface/GigabitEthernet=2"

payload = {}
headers = {
  'Accept': 'application/yang-data+json',
}

response = requests.request("GET", url, headers=headers, auth=(username, password), data=payload)

print(response.text)

```

Then we can see the interface configuration using the API call

```
[kamal@desktop01 xe]$ python3 test.py
{
  "tailf-ned-cisco-ios:GigabitEthernet": [
    {
      "name": "2",
      "description": "R2",
      "negotiation": {
        "auto": true
      },
      "ip": {
        "address": {
          "primary": {
            "address": "10.1.2.1",
            "mask": "255.255.255.0"
          }
        },
        "ospf": {
          "process-id": [
            {
              "id": 100,
              "area": 0
            }
          ],
          "network": ["point-to-point"]
        }
      }
    }
  ]
}
```

## Operational Data

The API calls the we use previously are mainly to get the configuration. We can also get the operational data. This data available under the live-status resource. Below sample to get operational data using cli:

```
sysadmin@ncs# show devices device R1 live-status interfaces-state interface GigabitEthernet2
live-status interfaces-state interface GigabitEthernet2
 admin-status up
 phys-address 50:00:00:01:00:01
 speed        1000000000
 statistics in-octets 0
 statistics in-unicast-pkts 0
 statistics in-broadcast-pkts 0
 statistics in-multicast-pkts 0
 statistics in-errors 0
 statistics out-octets 53450
 statistics out-unicast-pkts 518
 statistics out-errors 0
 ipv4 forwarding true
 ipv4 mtu   1500
 ipv4 address 10.1.2.1
  netmask 255.255.255.0
  origin  other
 ipv6 forwarding true
 ipv6 mtu   1500
```

Again we can parse it into RESTCONF so that we can see the API path

```
sysadmin@ncs# show devices device R1 live-status interfaces-state interface GigabitEthernet2 | display restconf
/restconf/data/tailf-ncs:devices/device=R1/live-status/ietf-interfaces:interfaces-state/interface=GigabitEthernet2/admin-status up
/restconf/data/tailf-ncs:devices/device=R1/live-status/ietf-interfaces:interfaces-state/interface=GigabitEthernet2/phys-address 50:00:00:01:00:01
/restconf/data/tailf-ncs:devices/device=R1/live-status/ietf-interfaces:interfaces-state/interface=GigabitEthernet2/speed 1000000000
/restconf/data/tailf-ncs:devices/device=R1/live-status/ietf-interfaces:interfaces-state/interface=GigabitEthernet2/statistics/in-octets 0
/restconf/data/tailf-ncs:devices/device=R1/live-status/ietf-interfaces:interfaces-state/interface=GigabitEthernet2/statistics/in-unicast-pkts 0
/restconf/data/tailf-ncs:devices/device=R1/live-status/ietf-interfaces:interfaces-state/interface=GigabitEthernet2/statistics/in-broadcast-pkts 0
/restconf/data/tailf-ncs:devices/device=R1/live-status/ietf-interfaces:interfaces-state/interface=GigabitEthernet2/statistics/in-multicast-pkts 0
/restconf/data/tailf-ncs:devices/device=R1/live-status/ietf-interfaces:interfaces-state/interface=GigabitEthernet2/statistics/in-errors 0
/restconf/data/tailf-ncs:devices/device=R1/live-status/ietf-interfaces:interfaces-state/interface=GigabitEthernet2/statistics/out-octets 54220
/restconf/data/tailf-ncs:devices/device=R1/live-status/ietf-interfaces:interfaces-state/interface=GigabitEthernet2/statistics/out-unicast-pkts 525
/restconf/data/tailf-ncs:devices/device=R1/live-status/ietf-interfaces:interfaces-state/interface=GigabitEthernet2/statistics/out-errors 0
set /restconf/data/tailf-ncs:devices/device=R1/live-status/ietf-interfaces:interfaces-state/interface=GigabitEthernet2/ietf-ip:ipv4
/restconf/data/tailf-ncs:devices/device=R1/live-status/ietf-interfaces:interfaces-state/interface=GigabitEthernet2/ietf-ip:ipv4/forwarding true
/restconf/data/tailf-ncs:devices/device=R1/live-status/ietf-interfaces:interfaces-state/interface=GigabitEthernet2/ietf-ip:ipv4/mtu 1500
/restconf/data/tailf-ncs:devices/device=R1/live-status/ietf-interfaces:interfaces-state/interface=GigabitEthernet2/ietf-ip:ipv4/address=10.1.2.1/netmask 255.255.255.0
/restconf/data/tailf-ncs:devices/device=R1/live-status/ietf-interfaces:interfaces-state/interface=GigabitEthernet2/ietf-ip:ipv4/address=10.1.2.1/origin other
set /restconf/data/tailf-ncs:devices/device=R1/live-status/ietf-interfaces:interfaces-state/interface=GigabitEthernet2/ietf-ip:ipv6
/restconf/data/tailf-ncs:devices/device=R1/live-status/ietf-interfaces:interfaces-state/interface=GigabitEthernet2/ietf-ip:ipv6/forwarding true
/restconf/data/tailf-ncs:devices/device=R1/live-status/ietf-interfaces:interfaces-state/interface=GigabitEthernet2/ietf-ip:ipv6/mtu 1500
```

Take the root path in the API call

```
import requests

username = input("Username:")
password = input("password:")

url = "http://IP-ADDRESS:8080/restconf/data/tailf-ncs:devices/device=R1/live-status/ietf-interfaces:interfaces-state/interface=GigabitEthernet2"

payload = {}
headers = {
  'Accept': 'application/yang-data+json',
}

response = requests.request("GET", url, headers=headers, auth=(username, password), data=payload)

print(response.text)
```

Below are the output

```
[kamal@desktop01 xe]$ python3 test.py
{
  "ietf-interfaces:interface": [
    {
      "name": "GigabitEthernet2",
      "admin-status": "up",
      "phys-address": "50:00:00:01:00:01",
      "speed": "1000000000",
      "statistics": {
        "in-octets": "0",
        "in-unicast-pkts": "0",
        "in-broadcast-pkts": "0",
        "in-multicast-pkts": "0",
        "in-errors": 0,
        "out-octets": "55100",
        "out-unicast-pkts": "533",
        "out-errors": 0
      },
      "ietf-ip:ipv4": {
        "forwarding": true,
        "mtu": 1500,
        "address": [
          {
            "ip": "10.1.2.1",
            "netmask": "255.255.255.0",
            "origin": "other"
          }
        ]
      },
      "ietf-ip:ipv6": {
        "forwarding": true,
        "mtu": 1500
      }
    }
  ]
}
```

The are many operational data available that we can use as below

```
sysadmin@ncs# show devices device R1 live-status ?
Description: Status data fetched from the device
Possible completions:
  access-tunnel            - show access tunnel
  arp                      - show arp
  bgp                      - BGP information
  cdp                      - show cdp
  config-register          - config-register information from show bootvar
  crypto                   -
  device-tracking-database - show device-tracking database
  if:interfaces            -
  interfaces-state         -
  inventory                - show inventory
  ios-stats:interfaces     - show interfaces
  ip                       -
  lisp                     - show lisp
  lldp                     - show lldp
  modules-state            -
  running-config           - Read only 'config' still shown in running-config on device
  test                     -
  version                  - show version
  voice                    - Global voice stats
  vrf                      - show vrf
  yang-library             -
  |                        - Output modifiers
  <cr>
```
