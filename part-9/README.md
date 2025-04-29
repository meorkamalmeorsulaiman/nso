# Service Template

## Initialize Directory

```
sysadmin@nso01:~/nso-lab/packages$ ncs-make-package --service-skeleton python dns-config-xe
sysadmin@nso01:~/nso-lab/packages$ ls
cisco-ios-cli-6.107  cisco-iosxr-cli-7.61  dns-config-xe  vpn-service-xe  vpn-service-xr
sysadmin@nso01:~/nso-lab/packages$ tree dns-config-xe/
dns-config-xe/
├── README
├── package-meta-data.xml
├── python
│   └── dns_config_xe
│       ├── __init__.py
│       └── main.py
├── src
│   ├── Makefile
│   └── yang
│       └── dns-config-xe.yang
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
Generate files

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
        template_vars.add('dns-ip', '192.0.2.1')
        template = ncs.template.Template(service)
        template.apply('dns-config-xe', template_vars)
```

Update yang variable

```
sysadmin@nso01:~/nso-lab/packages/dns-config-xe$ cat src/yang/dns-config-xe.yang
module dns-config-xe {

  namespace "http://example.com/dns-config-xe";
  prefix dns-config-xe;

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

  list dns-config-xe {
    description "This is an RFS skeleton service";

    key name;
    leaf name {
      tailf:info "Unique service id";
      tailf:cli-allow-range;
      type string;
    }

    uses ncs:service-data;
    ncs:servicepoint dns-config-xe-servicepoint;

    // may replace this with other ways of refering to the devices.
    leaf-list device {
      type leafref {
        path "/ncs:devices/ncs:device/ncs:name";
      }
    }

    // replace with your own stuff here
    leaf dns-ip {
      type inet:ipv4-address;
    }
  }
}
```

Create templates
```
<config xmlns="http://tail-f.com/ns/config/1.0">
  <devices xmlns="http://tail-f.com/ns/ncs">
    <device>
      <name>{/device}</name>
      <config>
        <sys xmlns="http://example.com/router">
          <dns>
            <server>
              <address>{$dns-ip}</address>
            </server>
          </dns>
        </sys>
      </config>
    </device>
  </devices>
</config>
```

Reload packages
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
    package dns-config-xe
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
sysadmin@ncs#
System message at 2025-04-29 13:55:24...
    Subsystem stopped: ncs-dp-2-cisco-ios-cli-6.107:IOSDp
sysadmin@ncs#
System message at 2025-04-29 13:55:24...
    Subsystem started: ncs-dp-3-cisco-ios-cli-6.107:IOSDp
```

Parse template to file

```
sysadmin@ncs# show running-config devices device R1 config ip name-server | display xml | save nso-lab/packages/dns-config-xe/templates/dns-config-x
```

Edit template

```
sysadmin@nso01:~/nso-lab/packages/dns-config-xe/templates$ cat dns-config-xe.xml
<config xmlns="http://tail-f.com/ns/config/1.0">
  <devices xmlns="http://tail-f.com/ns/ncs">
    <device>
      <name>{/device}</name>
      <config>
        <ip xmlns="urn:ios">
          <name-server>
            <name-server-list>
               <address>{$dns-ip}</address>
            </name-server-list>
          </name-server>
        </ip>
      </config>
    </device>
  </devices>
</config>
```

Reload packages and reapply config

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

