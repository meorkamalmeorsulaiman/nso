# Provision NSO

## Requirement

To deploy NSO, you will need a server or container that act as the controller. It can run on a server or as a container. In this lab, we are going to use Ubuntu as below:

```bash
sysadmin@nso01:~$ cat /etc/os-release
PRETTY_NAME="Ubuntu 24.04.2 LTS"
NAME="Ubuntu"
VERSION_ID="24.04"
VERSION="24.04.2 LTS (Noble Numbat)"
VERSION_CODENAME=noble
ID=ubuntu
ID_LIKE=debian
HOME_URL="https://www.ubuntu.com/"
SUPPORT_URL="https://help.ubuntu.com/"
BUG_REPORT_URL="https://bugs.launchpad.net/ubuntu/"
PRIVACY_POLICY_URL="https://www.ubuntu.com/legal/terms-and-policies/privacy-policy"
UBUNTU_CODENAME=noble
LOGO=ubuntu-logo
```

NSO packages also needed and evaluation version are available thru Cisco website. In this setup, we are running IOS-XE(RR) and IOS-XR(PE) several packages downloaded into the NSO VM as below:

```bash
sysadmin@nso01:~$ ls -l nso-installer/
total 255648
-rw-rw-r-- 1 sysadmin sysadmin 261777981 Apr 22 13:23 nso-6.4-freetrial.linux.x86_64.signed.bin
sysadmin@nso01:~$ ls -l ned-installer/
total 113608
-rw-rw-r-- 1 sysadmin sysadmin 65707583 Apr 22 13:24 ncs-6.4-cisco-ios-6.107.2-freetrial.signed.bin
-rw-rw-r-- 1 sysadmin sysadmin 50618625 Apr 22 13:24 ncs-6.4-cisco-iosxr-7.61-freetrial.signed.bin
```
More details about all these packages could be find in devnet [Installation](https://developer.cisco.com/docs/nso/guides/installation/#li.download.the.installer)

## Linux Packages Installation

- There are several packages required before unpacking the NSO packages. These packages are available with many Linux distro. Here the links from devnet that listed all the packages [Linux Packages](https://developer.cisco.com/docs/nso/guides/installation/#li.fulfill.installation.reqs)
- You can install all these packages as below:

```bash
apt-get install openjdk-17-jdk openjdk-17-jre ant python3 python3-pip python3-setuptools libxml2-utils tree xsltproc
```
Once the setup is complete we can proceed with unpacking the NSO packages. We will continue on part-2
