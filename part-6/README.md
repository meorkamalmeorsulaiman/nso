# Working with Northbound API

The idea that we use northbound API to interact with NSO. Legacy way for northbound interfaces are cli and web interface. More details on supported APIs from DevNet [Northbound API](https://developer.cisco.com/docs/nso-guides-6.3/northbound-apis-introduction/#introduction)

## Get interface configuration on R1 

Now we can use python to get interface attributes on R1 Gig2. We can create a script as below

```
import requests

url = "http://IP-ADDRESS:8080/restconf/data/tailf-ncs:devices/device=R1/config/tailf-ned-cisco-ios:interface/GigabitEthernet=2/"

payload = {}
headers = {
  'Accept': 'application/yang-data+json',
}

response = requests.request("GET", url, headers=headers, auth=('USERNAME', 'PASSWORD'), data=payload)

print(response.text)
```
Below are the return output

```
[kamal@desktop01 xe]$ python3 getIntGi2.py
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
      },
      "shutdown": [null]
    }
  ]
}
```

In general, the root path for configuration is `/restconf/data/tailf-ncs:devices/device=R1/config/` For XR is different as it using different NED. Below is the script for XR 

```
import requests

url = "http://IP-ADDRESS:8080/restconf/data/tailf-ncs:devices/device=R2/config/tailf-ned-cisco-ios-xr:interface/GigabitEthernet=0%2F0%2F0%2F0"

payload = {}
headers = {
  'Accept': 'application/yang-data+json',
}

response = requests.request("GET", url, headers=headers, auth=('USERNAME', 'PASSWORD'), data=payload)

print(response.text)
```

Then the output quite similar
```
kamal@desktop01 xr]$ python3 getIntGi2.py
{
  "tailf-ned-cisco-ios-xr:GigabitEthernet": [
    {
      "id": "0/0/0/0",
      "description": "R1",
      "ipv4": {
        "address": {
          "ip": "10.1.2.2",
          "mask": "/24"
        }
      }
    }
  ]
}
```

