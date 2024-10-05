# Unpacking NSO

There are 3 packages that we have to unpack. We are going to seperate the packages in different folder as below:

```bash
sysadmin@nso01:~$ tree 
.
├── backup
│   ├── ncs-6.3-cisco-ios-6.106.8-freetrial.signed.bin
│   ├── ncs-6.3-cisco-iosxr-7.55.7-freetrial.signed.bin
│   └── nso-6.3-freetrial.linux.x86_64.signed.bin
├── nso_installer
│   └── nso-6.3-freetrial.linux.x86_64.signed.bin
├── nso_ned-ios-xe
│   └── ncs-6.3-cisco-ios-6.106.8-freetrial.signed.bin
└── nso_ned-ios-xr
    └── ncs-6.3-cisco-iosxr-7.55.7-freetrial.signed.bin

4 directories, 6 files
```
The backup folder just containing the same original packages as backup. If you noticed that there are 2 types of packages, the installer and ned. Installer `nso-6.3-freetrial.linux.x86_64.signed.bin` is the actual NSO software and `ncs-6.3-cisco-*` are the neds which going to be use by NSO to manage specific device type

## Unpack NSO

Let's unpack the NSO software as below:

```bash
sysadmin@nso01:~/nso_installer$ ls
nso-6.3-freetrial.linux.x86_64.signed.bin
sysadmin@nso01:~/nso_installer$ sh nso-6.3-freetrial.linux.x86_64.signed.bin --skip-verification
Unpacking...
sysadmin@nso01:~/nso_installer$ ls -l
total 475596
-rw------- 1 sysadmin sysadmin      2757 Apr 25 14:27 README.signature
-rwxr-xr-x 1 sysadmin sysadmin     16782 Nov 29  2022 cisco_x509_verify_release.py3
-rw------- 1 sysadmin sysadmin    357874 Apr 25 14:27 ncs-6.3-observability-exporter-1.3.0.tar.gz
-rw------- 1 sysadmin sysadmin       512 Apr 25 14:27 ncs-6.3-observability-exporter-1.3.0.tar.gz.signature
-rw------- 1 sysadmin sysadmin     58917 Apr 25 14:27 ncs-6.3-phased-provisioning-1.2.0.tar.gz
-rw------- 1 sysadmin sysadmin       512 Apr 25 14:27 ncs-6.3-phased-provisioning-1.2.0.tar.gz.signature
-rw------- 1 sysadmin sysadmin   3509196 Apr 25 14:27 ncs-6.3-resource-manager-project-4.2.6.tar.gz
-rw------- 1 sysadmin sysadmin       512 Apr 25 14:27 ncs-6.3-resource-manager-project-4.2.6.tar.gz.signature
-rw------- 1 sysadmin sysadmin    161321 Apr 25 14:27 ncs-6.3-tailf-hcc-project-6.0.2.tar.gz
-rw------- 1 sysadmin sysadmin       512 Apr 25 14:27 ncs-6.3-tailf-hcc-project-6.0.2.tar.gz.signature
-rw-r--r-- 1 sysadmin sysadmin 243483374 Sep 29 14:15 nso-6.3-freetrial.linux.x86_64.signed.bin
-rwx------ 1 sysadmin sysadmin 239372542 Apr 25 14:27 nso-6.3.linux.x86_64.installer.bin
-rw------- 1 sysadmin sysadmin       512 Apr 25 14:27 nso-6.3.linux.x86_64.installer.bin.signature
-rw-r--r-- 1 sysadmin sysadmin      1732 Oct 31  2022 tailf.cer
```
More details on the installation and files can be obtain from devnet [Installation](https://developer.cisco.com/docs/nso/getting-and-installing-nso/#performing-a-local-installation) The NSO installer is `nso-6.3.linux.x86_64.installer.bin` 

## Installing NSO

There are 2 types of installation, in this case we are going to use local install

```bash
sysadmin@nso01:~/nso_installer$ sh nso-6.3.linux.x86_64.installer.bin --local-install ~/nso-6.3
INFO  Using temporary directory /tmp/ncs_installer.4653 to stage NCS installation bundle
INFO  Unpacked ncs-6.3 in /home/sysadmin/nso-6.3
INFO  Found and unpacked corresponding DOCUMENTATION_PACKAGE
INFO  Found and unpacked corresponding EXAMPLE_PACKAGE
INFO  Found and unpacked corresponding JAVA_PACKAGE
INFO  Generating default SSH hostkey (this may take some time)
INFO  SSH hostkey generated
INFO  Generating self-signed certificates for HTTPS
INFO  Environment set-up generated in /home/sysadmin/nso-6.3/ncsrc
INFO  NSO installation script finished
INFO  Found and unpacked corresponding NETSIM_PACKAGE
INFO  NCS installation complete

sysadmin@nso01:~/nso_installer$ ls ~/nso-6.3/
CHANGES  README     VERSION  doc     etc           include  lib  ncsrc       netsim    scripts  support
LICENSE  SBOM.spdx  bin      erlang  examples.ncs  java     man  ncsrc.tcsh  packages  src      var
```
You can see that the proceed unpack into `nso-6.3` directory, you can validate some folder. The important part is the ned

```bash
sysadmin@nso01:~/nso_installer$ cd ~/nso-6.3/ 
sysadmin@nso01:~/nso-6.3$ ll packages/neds/
total 48
drwxr-xr-x 12 sysadmin sysadmin 4096 Apr 24 20:22 ./
drwxr-xr-x  7 sysadmin sysadmin 4096 Apr 24 20:22 ../
drwxr-xr-x  8 sysadmin sysadmin 4096 Apr 24 20:22 a10-acos-cli-3.0/
drwxr-xr-x  7 sysadmin sysadmin 4096 Apr 24 20:22 alu-sr-cli-3.4/
drwxr-xr-x  8 sysadmin sysadmin 4096 Apr 24 20:22 cisco-asa-cli-6.6/
drwxr-xr-x  7 sysadmin sysadmin 4096 Apr 24 20:22 cisco-ios-cli-3.0/
drwxr-xr-x  7 sysadmin sysadmin 4096 Apr 24 20:22 cisco-ios-cli-3.8/
drwxr-xr-x  8 sysadmin sysadmin 4096 Apr 24 20:22 cisco-iosxr-cli-3.0/
drwxr-xr-x  8 sysadmin sysadmin 4096 Apr 24 20:22 cisco-iosxr-cli-3.5/
drwxr-xr-x  8 sysadmin sysadmin 4096 Apr 24 20:22 cisco-nx-cli-3.0/
drwxr-xr-x  8 sysadmin sysadmin 4096 Apr 24 20:22 dell-ftos-cli-3.0/
drwxr-xr-x  5 sysadmin sysadmin 4096 Apr 24 20:22 juniper-junos-nc-3.0/
```

We have download the latest ned so let's proceed to unpack ned

## Unpack NED

Let's start with IOS-XE

```bash
sysadmin@nso01:~/nso-6.3$ cd ~/nso_ned-ios-xe/
sysadmin@nso01:~/nso_ned-ios-xe$ sh ncs-6.3-cisco-ios-6.106.8-freetrial.signed.bin 
Unpacking...
Verifying signature...
Retrieving CA certificate from http://www.cisco.com/security/pki/certs/crcam2.cer ...
Successfully retrieved and verified crcam2.cer.
Retrieving SubCA certificate from http://www.cisco.com/security/pki/certs/innerspace.cer ...
Successfully retrieved and verified innerspace.cer.
Successfully verified root, subca and end-entity certificate chain.
Successfully fetched a public key from tailf.cer.
Successfully verified the signature of ncs-6.3-cisco-ios-6.106.8.tar.gz using tailf.cer
```

Continue with IOS-XR

```bash
sysadmin@nso01:~/nso_ned-ios-xe$ cd ~/nso_ned-ios-xr/
sysadmin@nso01:~/nso_ned-ios-xr$ sh ncs-6.3-cisco-iosxr-7.55.7-freetrial.signed.bin 
Unpacking...
Verifying signature...
Retrieving CA certificate from http://www.cisco.com/security/pki/certs/crcam2.cer ...
Successfully retrieved and verified crcam2.cer.
Retrieving SubCA certificate from http://www.cisco.com/security/pki/certs/innerspace.cer ...
Successfully retrieved and verified innerspace.cer.
Successfully verified root, subca and end-entity certificate chain.
Successfully fetched a public key from tailf.cer.
Successfully verified the signature of ncs-6.3-cisco-iosxr-7.55.7.tar.gz using tailf.cer
```

You will see archive and compress file after completed unpacked.

```bash
sysadmin@nso01:~/nso_ned-ios-xr$ ll | grep .tar
-rw-r--r-- 1 sysadmin sysadmin 49496048 Apr 27 10:29 ncs-6.3-cisco-iosxr-7.55.7.tar.gz
-rw-r--r-- 1 sysadmin sysadmin      512 Apr 27 10:29 ncs-6.3-cisco-iosxr-7.55.7.tar.gz.signature
```

These are the files that we are going to use and add into the NSO as below:

```bash
sysadmin@nso01:~/nso_ned-ios-xr$ cd ~/nso-6.3/
sysadmin@nso01:~/nso-6.3$ cd packages/neds/
sysadmin@nso01:~/nso-6.3/packages/neds$ tar -zxf ~/nso_ned-ios-xe/ncs-6.3-cisco-ios-6.106.8.tar.gz
sysadmin@nso01:~/nso-6.3/packages/neds$ tar -zxf ~/nso_ned-ios-xr/ncs-6.3-cisco-iosxr-7.55.7.tar.gz
sysadmin@nso01:~/nso-6.3/packages/neds$ ll
total 56
drwxr-xr-x 14 sysadmin sysadmin 4096 Oct  5 02:08 ./
drwxr-xr-x  7 sysadmin sysadmin 4096 Apr 24 20:22 ../
drwxr-xr-x  8 sysadmin sysadmin 4096 Apr 24 20:22 a10-acos-cli-3.0/
drwxr-xr-x  7 sysadmin sysadmin 4096 Apr 24 20:22 alu-sr-cli-3.4/
drwxr-xr-x  8 sysadmin sysadmin 4096 Apr 24 20:22 cisco-asa-cli-6.6/
drwxr-xr-x  7 sysadmin sysadmin 4096 Apr 24 20:22 cisco-ios-cli-3.0/
drwxr-xr-x  7 sysadmin sysadmin 4096 Apr 24 20:22 cisco-ios-cli-3.8/
drwxr-xr-x  8 sysadmin sysadmin 4096 May  2 09:08 cisco-ios-cli-6.106/
drwxr-xr-x  8 sysadmin sysadmin 4096 Apr 24 20:22 cisco-iosxr-cli-3.0/
drwxr-xr-x  8 sysadmin sysadmin 4096 Apr 24 20:22 cisco-iosxr-cli-3.5/
drwxr-xr-x  9 sysadmin sysadmin 4096 Apr 27 04:51 cisco-iosxr-cli-7.55/
drwxr-xr-x  8 sysadmin sysadmin 4096 Apr 24 20:22 cisco-nx-cli-3.0/
drwxr-xr-x  8 sysadmin sysadmin 4096 Apr 24 20:22 dell-ftos-cli-3.0/
drwxr-xr-x  5 sysadmin sysadmin 4096 Apr 24 20:22 juniper-junos-nc-3.0/
```
You can see that there are 2 new directories that created with the latest ned `cisco-ios-cli-6.106` and `cisco-iosxr-cli-7.55` We are done with unpacking all the necessary packages and shall continue to boot up NSO
