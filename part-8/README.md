# Template

Written in XML with YANG schema

## Getting the example

You can use the cli to generate the sample template on XE

```
sysadmin@ncs# show running-config devices device R1 config vrf definition vrf-1 | display xml
<config xmlns="http://tail-f.com/ns/config/1.0">
  <devices xmlns="http://tail-f.com/ns/ncs">
    <device>
      <name>R1</name>
      <config>
        <vrf xmlns="urn:ios">
          <definition>
            <name>vrf-1</name>
            <rd>1:1</rd>
            <route-target>
              <export>
                <asn-ip>1:1</asn-ip>
              </export>
              <import>
                <asn-ip>1:1</asn-ip>
              </import>
            </route-target>
          </definition>
        </vrf>
      </config>
    </device>
  </devices>
</config>
```
## Creating template

First we generate the necessary files

```
sysadmin@nso01:~/nso-lab/packages$ ncs-make-package --service-skeleton template vpn-service-xe
```

Once created, you will see few folders and file. We first going to create the template

```
sysadmin@nso01:~/nso-lab/packages/vpn-service-xe$ ls
package-meta-data.xml  src  templates  test
sysadmin@nso01:~/nso-lab/packages/vpn-service-xe$ cd templates/
sysadmin@nso01:~/nso-lab/packages/vpn-service-xe/templates$ ls
vpn-service-xe-template.xml
sysadmin@nso01:~/nso-lab/packages/vpn-service-xe/templates$ vim vpn-service-xe-template.xml
```
Edit template as below

```
<config-template xmlns="http://tail-f.com/ns/config/1.0"
                 servicepoint="vpn-service-xe">
  <devices xmlns="http://tail-f.com/ns/ncs">
    <device>
      <!--
          Select the devices from some data structure in the service
          model. In this skeleton the devices are specified in a leaf-list.
          Select all devices in that leaf-list:
      -->
      <name>{/device}</name>
      <config>
        <vrf xmlns="urn:ios">
          <definition>
            <name>{/vrf-name}</name>
            <rd>{/rd}</rd>
            <route-target>
              <export>
                <asn-ip>{/export-rt}</asn-ip>
              </export>
              <import>
                <asn-ip>{/import-rt}</asn-ip>
              </import>
            </route-target>
          </definition>
        </vrf>
      </config>
    </device>
  </devices>
</config-template>
```
Above, we see few variable within the cruly brackets. Now we have to set the roll of those variables by editing the YANG file within below directory

```
sysadmin@nso01:~/nso-lab/packages/vpn-service-xe/templates$ cd ../src/yang/
sysadmin@nso01:~/nso-lab/packages/vpn-service-xe/src/yang$ ls
vpn-service-xe.yang
sysadmin@nso01:~/nso-lab/packages/vpn-service-xe/src/yang$ vim vpn-service-xe.yang
```

We define the variables as below
```
sysadmin@nso01:~/nso-lab/packages/vpn-service-xe$ cat src/yang/vpn-service-xe.yang
module vpn-service-xe {
  namespace "http://com/example/vpnservicexe";
  prefix vpn-service-xe;

  import ietf-inet-types {
    prefix inet;
  }
  import tailf-ncs {
    prefix ncs;
  }

  list vpn-service-xe {
    key name;

    uses ncs:service-data;
    ncs:servicepoint "vpn-service-xe";

    leaf name {
      type string;
    }

    // may replace this with other ways of refering to the devices.
    leaf-list device {
      type leafref {
        path "/ncs:devices/ncs:device/ncs:name";
      }
    }

    // replace with your own stuff here
    leaf vrf-name {
      type string;
    }
    leaf rd {
      type string;
    }
    leaf export-rt {
      type string;
    }
    leaf import-rt {
      type string;
    }
  }
}
```

## Generate 

Once the template and variable has been define. Now we generate it
```
sysadmin@nso01:~/nso-lab/packages/vpn-service-xe$ cd src/
sysadmin@nso01:~/nso-lab/packages/vpn-service-xe/src$ make
mkdir -p ../load-dir
/home/sysadmin/nso-6.4/bin/ncsc `ls vpn-service-xe-ann.yang  > /dev/null 2>&1 && echo "-a vpn-service-xe-ann.yang"` \
        --fail-on-warnings \
         \
        -c -o ../load-dir/vpn-service-xe.fxs yang/vpn-service-xe.yang
```

## Load teamplate in NSO

After generate, now we can access nso and load the template

```
sysadmin@ncs# packages reload force

>>> System upgrade is starting.
>>> Sessions in configure mode must exit to operational mode.
>>> No configuration changes can be performed until upgrade has completed.
>>> System upgrade has completed successfully.
reload-result {
    package cisco-ios-cli-6.107
    result true
}
reload-result {
    package cisco-iosxr-cli-7.61
    result true
}
reload-result {
    package vpn-service-xe
    result true
}
sysadmin@ncs#
System message at 2025-04-25 15:02:22...
    Subsystem stopped: ncs-dp-9-cisco-ios-cli-6.107:IOSDp
sysadmin@ncs#
System message at 2025-04-25 15:02:22...
    Subsystem started: ncs-dp-10-cisco-ios-cli-6.107:IOSDp
```

Above we can see that it is successfully loaded. Validate that we can see the package loaded 

```
sysadmin@ncs# show packages package package-version
packages package cisco-ios-cli-6.107
 package-version 6.107.2
packages package cisco-iosxr-cli-7.61
 package-version 7.61
packages package vpn-service-xe
 package-version 1.0
```

## Using template

Now the template is ready and we can then use it to apply our vrf configuration

```
sysadmin@ncs(config)# vpn-service-xe vrf-3
sysadmin@ncs(config-vpn-service-xe-vrf-3)# device R1
sysadmin@ncs(config-vpn-service-xe-vrf-3)# vrf-name vrf-3
sysadmin@ncs(config-vpn-service-xe-vrf-3)# rd 1:3
sysadmin@ncs(config-vpn-service-xe-vrf-3)# export-rt 3:3
sysadmin@ncs(config-vpn-service-xe-vrf-3)# import-rt 3:3
sysadmin@ncs(config-vpn-service-xe-vrf-3)# top
sysadmin@ncs(config)# show configuration
vpn-service-xe vrf-3
 device    [ R1 ]
 vrf-name  vrf-3
 rd        1:3
 export-rt 3:3
 import-rt 3:3
sysadmin@ncs(config)# commit
Commit complete.
```

Now we validate the config applied

```
sysadmin@ncs# show running-config devices device R1 config vrf definition vrf-3
devices device R1
 config
  vrf definition vrf-3
   rd 1:3
   route-target export 3:3
   route-target import 3:3
  !
 !
!
```

Now we can see on the actual device

```
R1#show vrf vrf-3
  Name                             Default RD            Protocols   Interfaces
  vrf-3                            1:3
R1#show run | sec definition
vrf definition vrf-1
 rd 1:1
 route-target export 1:1
 route-target import 1:1
vrf definition vrf-2
 rd 1:2
 route-target export 2:2
 route-target import 2:2
vrf definition vrf-3
 rd 1:3
 route-target export 3:3
 route-target import 3:3
```

## Rollback template

We can also remove the template applied

```
sysadmin@ncs(config)# vpn-service-xe vrf-3 un-deploy
sysadmin@ncs(config)#
System message at 2025-04-25 15:07:20...
Commit performed by sysadmin via ssh using cli.
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
```

## XR Template

Same procedure, just that the template would be different as below

```
sysadmin@nso01:~/nso-lab/packages/vpn-service-xr/src$ cat ../templates/vpn-service-xr-template.xml
<config-template xmlns="http://tail-f.com/ns/config/1.0"
                 servicepoint="vpn-service-xr">
  <devices xmlns="http://tail-f.com/ns/ncs">
    <device>
      <!--
          Select the devices from some data structure in the service
          model. In this skeleton the devices are specified in a leaf-list.
          Select all devices in that leaf-list:
      -->
      <name>{/device}</name>
      <config>
        <vrf xmlns="http://tail-f.com/ned/cisco-ios-xr">
          <vrf-list>
            <name>{/vrf-name}</name>
            <rd>{/rd}</rd>
            <address-family>
              <ipv4>
                <unicast>
                  <import>
                    <route-target>
                      <address-list>
                        <name>{/import-rt}</name>
                      </address-list>
                    </route-target>
                  </import>
                  <export>
                    <route-target>
                      <address-list>
                        <name>{/export-rt}</name>
                      </address-list>
                    </route-target>
                  </export>
                </unicast>
              </ipv4>
            </address-family>
          </vrf-list>
        </vrf>
      </config>
    </device>
  </devices>
</config-template>
```

Once reloaded, we can applied the template

```
sysadmin@ncs(config)# vpn-service-xr vrf-3
sysadmin@ncs(config-vpn-service-xr-vrf-3)# device R2
sysadmin@ncs(config-vpn-service-xr-vrf-3)# vrf-name vrf-3
sysadmin@ncs(config-vpn-service-xr-vrf-3)# rd 2:3
sysadmin@ncs(config-vpn-service-xr-vrf-3)# export-rt 3:3
sysadmin@ncs(config-vpn-service-xr-vrf-3)# import-rt 3:3
sysadmin@ncs(config-vpn-service-xr-vrf-3)# top
sysadmin@ncs(config)# show configuration
vpn-service-xr vrf-3
 device    [ R2 ]
 vrf-name  vrf-3
 rd        2:3
 import-rt 3:3
 export-rt 3:3
```

The config applied to the device

```
RP/0/0/CPU0:R2#show vrf vrf-3
Fri Apr 25 15:13:44.200 UTC
VRF                  RD                  RT                         AFI   SAFI
vrf-3                2:3
                                         import  3:3                 IPV4  Unicast
                                         export  3:3                 IPV4  Unicast
```





