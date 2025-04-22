# Unpacking NSO

There are 3 packages that we have to unpack. We are going to seperate the packages in different folder as below:

```bash
sysadmin@nso01:~$ tree
.
├── ned-installer
│   ├── ncs-6.4-cisco-ios-6.107.2-freetrial.signed.bin
│   └── ncs-6.4-cisco-iosxr-7.61-freetrial.signed.bin
└── nso-installer
    └── nso-6.4-freetrial.linux.x86_64.signed.bin

3 directories, 3 files
```
The backup folder just containing the same original packages as backup. If you noticed that there are 2 types of packages, the installer and ned. Installer `nso-6.4-freetrial.linux.x86_64.signed.bin` is the actual NSO software and `ncs-6.4-cisco-*` are the neds which going to be use by NSO to manage specific device type

## Unpack NSO

Let's unpack the NSO software as below:

```bash
sysadmin@nso01:~/nso-installer$ ls
nso-6.4-freetrial.linux.x86_64.signed.bin
sysadmin@nso01:~/nso-installer$ sh nso-6.4-freetrial.linux.x86_64.signed.bin --skip-verification
Unpacking...
sysadmin@nso01:~/nso-installer$ ls -l
total 511328
-rw------- 1 sysadmin sysadmin      2757 Nov 15 12:16 README.signature
-rwxr-xr-x 1 sysadmin sysadmin     16782 Nov 29  2022 cisco_x509_verify_release.py3
-rw------- 1 sysadmin sysadmin    361328 Nov 15 12:16 ncs-6.4-observability-exporter-1.4.0.tar.gz
-rw------- 1 sysadmin sysadmin       512 Nov 15 12:16 ncs-6.4-observability-exporter-1.4.0.tar.gz.signature
-rw------- 1 sysadmin sysadmin     59115 Nov 15 12:16 ncs-6.4-phased-provisioning-1.2.0.tar.gz
-rw------- 1 sysadmin sysadmin       512 Nov 15 12:16 ncs-6.4-phased-provisioning-1.2.0.tar.gz.signature
-rw------- 1 sysadmin sysadmin   3593049 Nov 15 12:16 ncs-6.4-resource-manager-project-4.2.8.tar.gz
-rw------- 1 sysadmin sysadmin       512 Nov 15 12:16 ncs-6.4-resource-manager-project-4.2.8.tar.gz.signature
-rw------- 1 sysadmin sysadmin    126534 Nov 15 12:16 ncs-6.4-tailf-hcc-project-6.0.5.tar.gz
-rw------- 1 sysadmin sysadmin       512 Nov 15 12:16 ncs-6.4-tailf-hcc-project-6.0.5.tar.gz.signature
-rw-rw-r-- 1 sysadmin sysadmin 261777981 Apr 22 13:23 nso-6.4-freetrial.linux.x86_64.signed.bin
-rw------- 1 sysadmin sysadmin 257613943 Nov 15 12:16 nso-6.4.linux.x86_64.installer.bin
-rw------- 1 sysadmin sysadmin       512 Nov 15 12:16 nso-6.4.linux.x86_64.installer.bin.signature
-rw-r--r-- 1 sysadmin sysadmin      1732 Oct 31  2022 tailf.cer
```
More details on the installation and files can be obtain from devnet [Installation](https://developer.cisco.com/docs/nso/getting-and-installing-nso/#performing-a-local-installation) The NSO installer is `nso-6.4.linux.x86_64.installer.bin` 

## Installing NSO

There are 2 types of installation, in this case we are going to use local install

```bash
sysadmin@nso01:~/nso-installer$ sh nso-6.4.linux.x86_64.installer.bin --local-install ~/nso-6.4
INFO  Using temporary directory /tmp/ncs_installer.3207 to stage NCS installation bundle
INFO  Unpacked ncs-6.4 in /home/sysadmin/nso-6.4
INFO  Found and unpacked corresponding DOCUMENTATION_PACKAGE
INFO  Found and unpacked corresponding EXAMPLE_PACKAGE
INFO  Found and unpacked corresponding JAVA_PACKAGE
INFO  Generating default SSH hostkey (this may take some time)
INFO  SSH hostkey generated
INFO  Generating self-signed certificates for HTTPS
INFO  Environment set-up generated in /home/sysadmin/nso-6.4/ncsrc
INFO  NSO installation script finished
INFO  Found and unpacked corresponding NETSIM_PACKAGE
INFO  NCS installation complete

sysadmin@nso01:~/nso-installer$ ls ~/nso-6.4/
CHANGES  README     VERSION  doc     etc           include  lib  ncsrc       netsim    scripts  support
LICENSE  SBOM.spdx  bin      erlang  examples.ncs  java     man  ncsrc.tcsh  packages  src      var
```
You can see that the proceed unpack into `nso-6.4` directory, you can validate some folder. The important part is the ned

```bash
sysadmin@nso01:~/nso-installer$ cd ~/nso-6.4/
sysadmin@nso01:~/nso-6.4$ ll packages/neds/
total 48
drwxr-xr-x 12 sysadmin sysadmin 4096 Nov 14 12:47 ./
drwxr-xr-x  7 sysadmin sysadmin 4096 Nov 14 12:47 ../
drwxr-xr-x  8 sysadmin sysadmin 4096 Nov 14 12:47 a10-acos-cli-3.0/
drwxr-xr-x  7 sysadmin sysadmin 4096 Nov 14 12:47 alu-sr-cli-3.4/
drwxr-xr-x  8 sysadmin sysadmin 4096 Nov 14 12:47 cisco-asa-cli-6.6/
drwxr-xr-x  7 sysadmin sysadmin 4096 Nov 14 12:47 cisco-ios-cli-3.0/
drwxr-xr-x  7 sysadmin sysadmin 4096 Nov 14 12:47 cisco-ios-cli-3.8/
drwxr-xr-x  8 sysadmin sysadmin 4096 Nov 14 12:47 cisco-iosxr-cli-3.0/
drwxr-xr-x  8 sysadmin sysadmin 4096 Nov 14 12:47 cisco-iosxr-cli-3.5/
drwxr-xr-x  8 sysadmin sysadmin 4096 Nov 14 12:47 cisco-nx-cli-3.0/
drwxr-xr-x  8 sysadmin sysadmin 4096 Nov 14 12:47 dell-ftos-cli-3.0/
drwxr-xr-x  5 sysadmin sysadmin 4096 Nov 14 12:47 juniper-junos-nc-3.0/
```

We have download the latest ned so let's proceed to unpack ned

## Unpack NED

Let's start with IOS-XE

```bash
sysadmin@nso01:~/nso-6.4$ cd ~/ned-installer/
sysadmin@nso01:~/ned-installer$ ls
ncs-6.4-cisco-ios-6.107.2-freetrial.signed.bin  ncs-6.4-cisco-iosxr-7.61-freetrial.signed.bin
sysadmin@nso01:~/ned-installer$ sh ncs-6.4-cisco-ios-6.107.2-freetrial.signed.bin
Unpacking...
Verifying signature...
Retrieving CA certificate from http://www.cisco.com/security/pki/certs/crcam2.cer ...
Successfully retrieved and verified crcam2.cer.
Retrieving SubCA certificate from http://www.cisco.com/security/pki/certs/innerspace.cer ...
Successfully retrieved and verified innerspace.cer.
Successfully verified root, subca and end-entity certificate chain.
Successfully fetched a public key from tailf.cer.
Successfully verified the signature of ncs-6.4-cisco-ios-6.107.2.tar.gz using tailf.cer
sysadmin@nso01:~/ned-installer$ ls
README.signature               ncs-6.4-cisco-ios-6.107.2-freetrial.signed.bin  ncs-6.4-cisco-ios-6.107.2.tar.gz.signature     tailf.cer
cisco_x509_verify_release.py3  ncs-6.4-cisco-ios-6.107.2.tar.gz                ncs-6.4-cisco-iosxr-7.61-freetrial.signed.bin
```

Continue with IOS-XR

```bash
sysadmin@nso01:~/ned-installer$ sh ncs-6.4-cisco-iosxr-7.61-freetrial.signed.bin
Unpacking...
Verifying signature...
Retrieving CA certificate from http://www.cisco.com/security/pki/certs/crcam2.cer ...
Successfully retrieved and verified crcam2.cer.
Retrieving SubCA certificate from http://www.cisco.com/security/pki/certs/innerspace.cer ...
Successfully retrieved and verified innerspace.cer.
Successfully verified root, subca and end-entity certificate chain.
Successfully fetched a public key from tailf.cer.
Successfully verified the signature of ncs-6.4-cisco-iosxr-7.61.tar.gz using tailf.cer
sysadmin@nso01:~/ned-installer$ ls
README.signature                                ncs-6.4-cisco-ios-6.107.2.tar.gz               ncs-6.4-cisco-iosxr-7.61.tar.gz
cisco_x509_verify_release.py3                   ncs-6.4-cisco-ios-6.107.2.tar.gz.signature     ncs-6.4-cisco-iosxr-7.61.tar.gz.signature
ncs-6.4-cisco-ios-6.107.2-freetrial.signed.bin  ncs-6.4-cisco-iosxr-7.61-freetrial.signed.bin  tailf.cer
```

You will see archive and compress files after completed unpacked.

```bash
sysadmin@nso01:~/ned-installer$ ll | grep .tar
-rw-r--r-- 1 sysadmin sysadmin 65703166 Nov 19 16:49 ncs-6.4-cisco-ios-6.107.2.tar.gz
-rw-r--r-- 1 sysadmin sysadmin      512 Nov 19 16:49 ncs-6.4-cisco-ios-6.107.2.tar.gz.signature
-rw-r--r-- 1 sysadmin sysadmin 50621786 Nov 20 08:29 ncs-6.4-cisco-iosxr-7.61.tar.gz
-rw-r--r-- 1 sysadmin sysadmin      512 Nov 20 08:29 ncs-6.4-cisco-iosxr-7.61.tar.gz.signature
```

These are the files that we are going to use and add into the NSO as below:

```bash
sysadmin@nso01:~/ned-installer$ cd ~/nso-6.4/
sysadmin@nso01:~/nso-6.4$ cd packages/neds/
sysadmin@nso01:~/nso-6.4/packages/neds$ tar -zxf ~/ned-installer/ncs-6.4-cisco-ios-6.107.2.tar.gz
sysadmin@nso01:~/nso-6.4/packages/neds$ tar -zxf ~/ned-installer/ncs-6.4-cisco-iosxr-7.61.tar.gz
sysadmin@nso01:~/nso-6.4/packages/neds$ ll
total 56
drwxr-xr-x 14 sysadmin sysadmin 4096 Apr 22 13:47 ./
drwxr-xr-x  7 sysadmin sysadmin 4096 Nov 14 12:47 ../
drwxr-xr-x  8 sysadmin sysadmin 4096 Nov 14 12:47 a10-acos-cli-3.0/
drwxr-xr-x  7 sysadmin sysadmin 4096 Nov 14 12:47 alu-sr-cli-3.4/
drwxr-xr-x  8 sysadmin sysadmin 4096 Nov 14 12:47 cisco-asa-cli-6.6/
drwxr-xr-x  7 sysadmin sysadmin 4096 Nov 14 12:47 cisco-ios-cli-3.0/
drwxr-xr-x  7 sysadmin sysadmin 4096 Nov 14 12:47 cisco-ios-cli-3.8/
drwxr-xr-x  8 sysadmin sysadmin 4096 Nov 19 15:14 cisco-ios-cli-6.107/
drwxr-xr-x  8 sysadmin sysadmin 4096 Nov 14 12:47 cisco-iosxr-cli-3.0/
drwxr-xr-x  8 sysadmin sysadmin 4096 Nov 14 12:47 cisco-iosxr-cli-3.5/
drwxr-xr-x  9 sysadmin sysadmin 4096 Nov 20 07:38 cisco-iosxr-cli-7.61/
drwxr-xr-x  8 sysadmin sysadmin 4096 Nov 14 12:47 cisco-nx-cli-3.0/
drwxr-xr-x  8 sysadmin sysadmin 4096 Nov 14 12:47 dell-ftos-cli-3.0/
drwxr-xr-x  5 sysadmin sysadmin 4096 Nov 14 12:47 juniper-junos-nc-3.0/
```
You can see that there are 2 new directories that created with the latest ned `cisco-ios-cli-6.107` and `cisco-iosxr-cli-7.61` We are done with unpacking all the necessary packages and shall continue to boot up NSO
