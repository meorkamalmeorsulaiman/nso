# Service Template

All variable supply thru python, normal template require input variables by administrator. On the other hand, service template us variable supplied from python code. This help work repetitive work, for example creating VLANs across multiple switches or VRF on multiple PE.

## Creating Service Package

This will create package codes and service name as below:

```
sysadmin@nso01:~/nso-lab/packages$ ncs-make-package --service-skeleton python vrf-config-xe
sysadmin@nso01:~/nso-lab/packages$ ls
cisco-ios-cli-6.107  cisco-iosxr-cli-7.61  vpn-service-xe  vpn-service-xr  vrf-config-xe
sysadmin@nso01:~/nso-lab/packages$ tree vrf-config-xe/
vrf-config-xe/
├── README
├── package-meta-data.xml
├── python
│   └── vrf_config_xe
│       ├── __init__.py
│       └── main.py
├── src
│   ├── Makefile
│   └── yang
│       └── vrf-config-xe.yang
├── templates
└── test
    ├── Makefile
    └── internal
        ├── Makefile
        └── lux
            ├── Makefile
            └── service
                ├── Makefile
                ├── dummy-device.xml
                ├── dummy-service.xml
                ├── pyvm.xml
                └── run.lux

10 directories, 14 files
```

Now, we have the package files and let's create the YANG variables

```
sysadmin@nso01:~/nso-lab/packages/vrf-config-xe$ cat src/yang/vrf-config-xe.yang
module vrf-config-xe {

  namespace "http://example.com/vrf-config-xe";
  prefix vrf-config-xe;

  import ietf-inet-types {
    prefix inet;
  }
  import tailf-common {
    prefix tailf;
  }
  import tailf-ncs {
    prefix ncs;
  }

  description
    "Bla bla...";

  revision 2016-01-01 {
    description
      "Initial revision.";
  }

  list vrf-config-xe {
    description "This is an RFS skeleton service";

    key name;
    leaf name {
      tailf:info "Unique service id";
      tailf:cli-allow-range;
      type string;
    }

    uses ncs:service-data;
    ncs:servicepoint vrf-config-xe-servicepoint;

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

    leaf import-rt {
      type string;
    }

    leaf export-rt {
      type string;
    }

  }
}
```

We also need the VRF configuration template we can copy from NSO to the directory 
```
sysadmin@ncs# show running-config devices device R1 config vrf definition vrf-2 | display xml | save packages/vrf-config-xe/templates/vrf-config-xe.yml
```

Then we edit to match the variable

```
sysadmin@nso01:~/nso-lab/packages/vrf-config-xe$ cat templates/vrf-config-xe-template.yml
<config xmlns="http://tail-f.com/ns/config/1.0">
  <devices xmlns="http://tail-f.com/ns/ncs">
    <device>
      <name>{/device}</name>
      <config>
        <vrf xmlns="urn:ios">
          <definition>
            <name>{$vrf-name}</name>
            <rd>{$rd}</rd>
            <route-target>
              <export>
                <asn-ip>{$export-rt}</asn-ip>
              </export>
              <import>
                <asn-ip>{$import-rt}</asn-ip>
              </import>
            </route-target>
          </definition>
        </vrf>
      </config>
    </device>
  </devices>
</config>
```

At this point we have completed the necessary files for service.  Now we can proceed to compile the updated YANG

```
sysadmin@nso01:~/nso-lab/packages/vrf-config-xe/src$ make
mkdir -p ../load-dir
/home/sysadmin/nso-6.4/bin/ncsc  `ls vrf-config-xe-ann.yang  > /dev/null 2>&1 && echo "-a vrf-config-xe-ann.yang"` \
        --fail-on-warnings \
         \
        -c -o ../load-dir/vrf-config-xe.fxs yang/vrf-config-xe.yang
```

## Template Codes

Template variable can get it's input from different source. For service template we can using programming language. Add below section under `cb_create` function

```
sysadmin@nso01:~/nso-lab/packages/dns-config-xe$ vim python/dns_config_xe/main.py
#Codes Snipped
class ServiceCallbacks(Service):

    # The create() callback is invoked inside NCS FASTMAP and
    # must always exist.
    @Service.create
    def cb_create(self, tctx, root, service, proplist):
        self.log.info('Service create(service=', service._path, ')')
        template_vars = ncs.template.Variables()
        template_vars.add('vrf-name', 'vrf-4')
        template_vars.add('rd', '1:4')
        template_vars.add('export-rt', '1:4')
        template_vars.add('import-rt', '1:4')
        template = ncs.template.Template(service)
        template.apply('vrf-config-xe', template_vars)
#Code Snipped
```
Above, we can invoke multiple `.add()` function to apply multiple variable as we define in YANG template.

## Load Package and Apply Service Template

Reload package to load the service template

```
sysadmin@ncs# packages reload

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
reload-result {
    package vpn-service-xr
    result true
}
reload-result {
    package vrf-config-xe
    result true
}
sysadmin@ncs#
System message at 2025-05-04 03:52:59...
    Subsystem stopped: ncs-dp-1-cisco-ios-cli-6.107:IOSDp
sysadmin@ncs#
System message at 2025-05-04 03:52:59...
    Subsystem started: ncs-dp-2-cisco-ios-cli-6.107:IOSDp
```

Now, the package has been loaded. Let's proceed to apply the service template on the router.

```
sysadmin@ncs(config-dns-config-xe-dns-r1)# commit dry-run
cli {
    local-node {
        data  devices {
                  device R1 {
                      config {
                          ip {
                              name-server {
             +                    # first
             +                    name-server-list 192.0.2.1;
                              }
                          }
                      }
                  }
              }
             +dns-config-xe dns-r1 {
             +    device [ R1 ];
             +}
    }
}
```

Validate device configured correctly

```
sysadmin@ncs# show running-config devices device R1 config ip name-server
devices device R1
 config
  ip name-server 192.0.2.1
 !
!
```

