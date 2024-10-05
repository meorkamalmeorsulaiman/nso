# Initialize NSO

In order to bring up the NSO, we have to initialize it into an instance. First let's set the environment variable as below:

```bash
sysadmin@nso01:~/nso-6.3/packages/neds$ source $HOME/nso-6.3/ncsrc
sysadmin@nso01:~/nso-6.3/packages/neds$ ncs --help
Usage: ncs [--conf ConfFile] [--addloadpath Dir] [--nolog]
           [--smp Nr] [--dns-resolvers Nr]
           [--foreground [-v | --verbose ] [--stop-on-eof ]]
           [--with-package-reload] [--with-package-reload-force]
           [--ignore-initial-validation] [--full-upgrade-validation]
           [--start-phase0] [--epoll {true|false}] [--cd Dir]
           [--disable-compaction-on-start]

Usage: ncs {--wait-phase0 [ TryTime]  | --start-phase1 | --start-phase2 |
            --wait-started [ TryTime]  |  --reload |
            --areload | --status | --check-callbacks [ Namespace | Path ] |
            --loadfile File |
            --rollback Nr | --debug-dump File | --cli-j-dump File |
            --loadxmlfiles File... |
            --mergexmlfiles File... | --stop}
           [--timeout MaxTime]

Usage: ncs {--version |
            --cdb-debug-dump Directory |
            --cdb-compact Directory |
            --cdb-validate Directory}

 -c, --conf ConfFile
     ConfFile is the absolute path to a ncs.conf file;
     default is /home/sysadmin/nso-6.3/etc/ncs/ncs.conf

 --nolog
     Don't log inital startup messages to syslog.

 --smp Nr
     Number of threads to run for Symmetric Multiprocessing
     (SMP). The default is to use as many threads as the system
     has logical processors.

 --dns-resolvers Nr
     Number of threads to run hostname lookups on. The default
     is to use as many threads as the system has logical processors.

 --cd Dir
     Change working directory

 --with-package-reload
     Reload all packages, i.e. copy packages from the load-path to
     the NCS daemon's private directory tree, as part of the startup
     procedure.

 --with-package-reload-force
     Reload all packages in an unsafe manner and override warnings if any.
     Refer to Loading Packages section in NSO Administration Guide for more
     information.

 --ignore-initial-validation
     Do not invoke validation callpoints in the (CDB initiated)
     init and upgrade transactions.

 --full-upgrade-validation
     Perform a full validation of the entire database if the data models
     have been upgraded.  This is useful in order to trigger external
     validation to run even if the database content has not been modified.

 --disable-compaction-on-start
     Do not compact CDB files when starting the NCS daemon.

 --start-phase0
     Start the daemon, but only bring up CDB (used to control
     the NCS daemon when upgrading database layout).

 --start-phase1
     Finish start phase0, do not start the subsystems that
     listens to the management ip address.

 --start-phase2
     Must be called after the management interface has been
     brought up, if --start-phase1 has been used. Starts
     the subsystems that listens to the management ip address.

 --foreground [ -v | --verbose ] [ --stop-on-eof ]
     Do not daemonize. Can be used to start ncsd from a
     process manager. In combination with -v or --verbose,
     all log messages are printed to stdout. Useful during
     development. In combination with --stop-on-eof, ncsd
     will stop if it receives EOF (ctrl-d) on standard input.
     Note that to stop ncsd when run in foreground, send EOF
     (if --stop-on-eof was used) or use ncs --stop. Do not
     terminate with ctrl-c, since ncsd in that case won't
     have the chance to close the database files.

 --addloadpath Dir
     Add Dir to the ncsd load-path. Convenient way to make
     ncsd load some additional fxs/ccl files

 --epoll {true|false}
     Determines whether ncsd should use an enhanced poll()
     function (e.g. Linux epoll(7)). The default is true.

 --wait-phase0 [ TryTime ]
     This call hangs until ncsd has initialized start phase0.
     After this call has returned, it is safe to register
     validation callbacks, upgrade CDB etc.
     This function is useful when ncsd has been started with
     --foreground and --start-phase0.
     It will keep trying the initial connection to ncsd
     for at most TryTime seconds (default 5)

 --wait-started [ TryTime ]
     This call hangs until ncsd is completely started.
     This function is useful when ncsd has been started with
     --foreground.
     It will keep trying the initial connection to ncsd
     for at most TryTime seconds (default 5)

 --status
     Print status about the NCS daemon.

 --debug-dump File
     Write internal ncsd information to File for troubleshooting purposes.
     Client can extend collect timeout by using --collect-timeout option and
     then provides the timeout in second (default timeout is 10 seconds).
     Client can compress debug dump by using --compress option, the debug dump
     will be compressed to File.gz.

 --cli-j-dump File
     Write CLI structure of Juniper style CLI on XML-format to File.

 --stop
     Stop the NCS daemon.

 --reload
     Reload the NCS daemon configuration. All log files are
     closed and reopened, which means that ncs --reload can
     be used from e.g. logrotate(8). Note: If you update a .fxs
     file it is not enough to do a reload; the "packages reload"
     action must be invoked, or the dameon must be restarted
     with the --with-package-reload option.

 --areload
     Asynchronously reload the NCS daemon configuration.

 --loadfile File
     Load configuration from File in curly bracket format.

 --rollback Nr
     Load saved configuration from rollback file Nr.

 --loadxmlfiles File ...
     Replace configuration with data from Files in XML format.

 --mergexmlfiles File ...
     Merge configuration with data from Files in XML format.

 --check-callbacks [ Namespace | Path ]
     Walks through the entire data tree (config and stat), or only the
     Namespace or Path, and verifies that all read-callbacks are
     implemented for all elements, and verifies their return values.

 --timeout MaxTime
     Specify the maximum time to wait for the NCS daemon to
     complete the command, in seconds. If this option is not given,
     no timeout is used.

 --cdb-debug-dump Directory
     Dumps information about CDB files in Directory as text to stdout
     (runs standalone and does not require a running NCS).

 --cdb-compact Directory
     Compacts the CDB files in Directory
     (runs standalone and does not require a running NCS).
     It is not recommended to use this command while NCS is running.
     This is controlled by the lockfile compact.lock in Directory.
     If the command fails when no running NCS is using the CDB files,
     please remove the lockfile and run the command again.
     For manual compaction while NCS is running, please see
     cdb_initiate_journal_compaction() in confd_lib_cdb.

 --cdb-validate Directory [ --log-file File ] [ --log-level Level ] [ --validate Item ]...
     Validate the content of CDB files in Directory
     (runs standalone and does not require a running NCS).
     If provided --log-file, all logs are written to corresponding File.
     The --log-level can be set to 'error', 'warning', 'info' or 'debug' to
     control the size of produced log. It is set to 'info' by default.
     The --validate option can be set to one of 'utf8', 'when-expr', or
     'must-expr' to select which type of validation that user wants to have.
     By default, validation are done for all existing items.

 --version
     Report version without communicating with the NCS daemon

 --help
     Print help text
```

Let's proceed to initialize the instance, we can think of instance as a project. We also have to include which ned that we are going to use

```bash
sysadmin@nso01:~$ ncs-setup --package ~/nso-6.3/packages/neds/cisco-ios-cli-6.106 --package ~/nso-6.3/packages/neds/cisco-iosxr-cli-7.55 --dest nso-lab
sysadmin@nso01:~$ ls -l nso-lab/
total 36
-rw-rw-r-- 1 sysadmin sysadmin   735 Oct  5 02:33 README.ncs
drwxrwxr-x 2 sysadmin sysadmin  4096 Oct  5 02:33 logs
drwxrwxr-x 2 sysadmin sysadmin  4096 Oct  5 02:33 ncs-cdb
-rw-rw-r-- 1 sysadmin sysadmin 11520 Oct  5 02:33 ncs.conf
drwxrwxr-x 2 sysadmin sysadmin  4096 Oct  5 02:33 packages
drwxrwxr-x 4 sysadmin sysadmin  4096 Oct  5 02:33 scripts
drwxrwxr-x 2 sysadmin sysadmin  4096 Oct  5 02:33 state
sysadmin@nso01:~$ ll nso-lab/packages/
total 8
drwxrwxr-x 2 sysadmin sysadmin 4096 Oct  5 02:33 ./
drwxrwxr-x 7 sysadmin sysadmin 4096 Oct  5 02:33 ../
lrwxrwxrwx 1 sysadmin sysadmin   56 Oct  5 02:33 cisco-ios-cli-6.106 -> /home/sysadmin/nso-6.3/packages/neds/cisco-ios-cli-6.106/
lrwxrwxrwx 1 sysadmin sysadmin   57 Oct  5 02:33 cisco-iosxr-cli-7.55 -> /home/sysadmin/nso-6.3/packages/neds/cisco-iosxr-cli-7.55/
```
We can see that the ned is referencing the source that we have unpack earlier let's proceed to start the NSO

```bash
sysadmin@nso01:~$ cd nso-lab/
sysadmin@nso01:~/nso-lab$ ncs
sysadmin@nso01:~/nso-lab$ ncs --status | grep status
status: started
        db=running id=33 priority=1 path=/ncs:devices/device/live-status-protocol/device-type
sysadmin@nso01:~/nso-lab$
```

Once it started, you can have access to it's web portal using `http://192.168.101.253:8080/` with username and password `admin` as below


![NSO Web Portal](https://github.com/meorkamalmeorsulaiman/nso/blob/main/images/portal.jpg)

That's all for this part, we shall proceed to connect all the devices
