# Initialize NSO

In order to bring up the NSO, we have to initialize it into an instance. First let's set the environment variable as below:

```bash           
sysadmin@nso01:~$ source $HOME/nso-6.4/ncsrc
sysadmin@nso01:~$ ncs --version
6.4
```

Let's proceed to initialize the instance, we can think of instance as a project. We also have to include which ned that we are going to use

```bash
sysadmin@nso01:~$ ncs-setup --package ~/nso-6.4/packages/neds/cisco-ios-cli-6.107 --package ~/nso-6.4/packages/neds/cisco-iosxr-cli-7.61 --dest nso-lab
sysadmin@nso01:~$ ls -l nso-lab/
total 36
-rw-rw-r-- 1 sysadmin sysadmin   735 Apr 22 14:00 README.ncs
drwxrwxr-x 2 sysadmin sysadmin  4096 Apr 22 14:00 logs
drwxrwxr-x 2 sysadmin sysadmin  4096 Apr 22 14:00 ncs-cdb
-rw-rw-r-- 1 sysadmin sysadmin 11573 Apr 22 14:00 ncs.conf
drwxrwxr-x 2 sysadmin sysadmin  4096 Apr 22 14:00 packages
drwxrwxr-x 4 sysadmin sysadmin  4096 Apr 22 14:00 scripts
drwxrwxr-x 2 sysadmin sysadmin  4096 Apr 22 14:00 state
sysadmin@nso01:~$ ls -l nso-lab/packages/
total 0
lrwxrwxrwx 1 sysadmin sysadmin 56 Apr 22 14:00 cisco-ios-cli-6.107 -> /home/sysadmin/nso-6.4/packages/neds/cisco-ios-cli-6.107
lrwxrwxrwx 1 sysadmin sysadmin 57 Apr 22 14:00 cisco-iosxr-cli-7.61 -> /home/sysadmin/nso-6.4/packages/neds/cisco-iosxr-cli-7.61
```
We can see that the ned is referencing the source that we have unpack earlier let's proceed to start the NSO

```bash
sysadmin@nso01:~/nso-lab$ ncs
sysadmin@nso01:~/nso-lab$ ncs --status | grep status
status: started
        db=running id=33 priority=1 path=/ncs:devices/device/live-status-protocol/device-type
```

Once it started, you can have access to it's web portal using `http://IP ADDRESS:8080/` with username and password `admin` as below


![NSO Web Portal](https://github.com/meorkamalmeorsulaiman/nso/blob/main/images/portal.jpg)

That's all for this part, we shall proceed to connect all the devices
