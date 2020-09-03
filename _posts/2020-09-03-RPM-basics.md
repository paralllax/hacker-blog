---
title: Getting to know RPM
ipublished: true
---

* * *

This will be a brief, no frills, overview of RPM and what it is. There will be a few examples, followed by scenarios with issues you may come across frequently. This should be a sufficient introduction to get you comfortable with rpm. Note this will not include installing and uninstalling packages, as there are a million guides on that. Rather we will touch on some of the other basics uses and features that are, at times, overlooked.

## Index

1. [RPM](#rpm)
2. [RPM Keys](#rpm_keys)
3. [Useful Commands](#commands)
4. [Common Issues](#issue)

* * *

### RPM<a name="rpm"><a/>

RPM - (Redhat Package Manager) A tool developed by redhat for rpm based systems. The tool is not available for debian based such as ubuntu. The main issue with rpm, despite it's many uses, is dependency hell. RPM does not have an efficient way of tracking dependencies like yum or apt do.

* * *

### RPM Keys<a name="rpm_keys"><a/>

These are ‘fake packages’ created to store and manage keys. If you erase/remove them, you will be unable to update/upgrade any package from that repo.

List all installed gpg-keys for packages.

```
[root@tartarus ~]# rpm -q gpg-pubkey --qf '%{NAME}-%{VERSION}-%{RELEASE}\t%{SUMMARY}\n'
gpg-pubkey-ef8d349f-57b6233e    gpg(Puppet, Inc. Release Key (Puppet, Inc. Release Key) <release@puppet.com>)
gpg-pubkey-352c64e5-52ae6884    gpg(Fedora EPEL (7) <epel@fedoraproject.org>)
gpg-pubkey-f4a80eb5-53a7ff4b    gpg(CentOS-7 Key (CentOS 7 Official Signing Key) <security@centos.or
```

Remove a key, in this case, we removed the one for puppet.

```
[root@tartarus ~]# rpm -e gpg-pubkey-ef8d349f-57b6233e
[root@tartarus ~]# rpm -q gpg-pubkey --qf '%{NAME}-%{VERSION}-%{RELEASE}\t%{SUMMARY}\n'
gpg-pubkey-352c64e5-52ae6884    gpg(Fedora EPEL (7) <epel@fedoraproject.org>)
gpg-pubkey-f4a80eb5-53a7ff4b    gpg(CentOS-7 Key (CentOS 7 Official Signing Key) <security@centos.org>)
```

Add a key,  in this case, we re-added the key for puppet.

```
[root@tartarus ~]# rpm --import /etc/pki/rpm-gpg/RPM-GPG-KEY-puppet5-release
[root@tartarus ~]# rpm -q gpg-pubkey --qf '%{NAME}-%{VERSION}-%{RELEASE}\t%{SUMMARY}\n'
gpg-pubkey-352c64e5-52ae6884    gpg(Fedora EPEL (7) <epel@fedoraproject.org>)
gpg-pubkey-f4a80eb5-53a7ff4b    gpg(CentOS-7 Key (CentOS 7 Official Signing Key) <security@centos.org>)
gpg-pubkey-ef8d349f-57b6233e    gpg(Puppet, Inc. Release Key (Puppet, Inc. Release Key) <release@puppet.com>)
```

* * *

### Useful Commands <a name="commands"><a/>

Query all packages. This will return a list of all packages on the system. This can be pared with a package name to query all packages with that name. If you pair it with a package the `a` flag is not needed.

```
[root@tartarus ~]# rpm -qa kernel
kernel-3.10.0-862.11.6.el7.x86_64
kernel-3.10.0-862.14.4.el7.x86_64
```

Find when a package was last updated.

```
[root@tartarus ~]# rpm -q --last kernel
kernel-3.10.0-862.14.4.el7.x86_64             Tue 23 Oct 2018 01:33:20 AM UTC
kernel-3.10.0-862.11.6.el7.x86_64             Sat 15 Sep 2018 01:30:42 PM UT
```

Find out which package owns a particular file.

```
[root@tartarus ~]# rpm -qf /etc/plymouth/plymouthd.conf
plymouth-0.8.9-0.31.20140113.el7.centos.x86_64
```

View the changelog of a package. This is also helpful in finding out if a previous CVE was addressed.

```
[root@tartarus ~]# rpm -q --changelog kernel
* Tue Aug 14 2018 CentOS Sources <bugs@centos.org> - 3.10.0-862.11.6.el7
- Apply debranding changes
 
* Fri Aug 10 2018 Jan Stancek <jstancek@redhat.com> [3.10.0-862.11.6.el7]
- [kernel] cpu/hotplug: Fix 'online' sysfs entry with 'nosmt' (Josh Poimboeuf) [1593383 1593384] {CVE-2018-3620}
 
* Thu Aug 09 2018 Frantisek Hrbata <fhrbata@hrbata.com> [3.10.0-862.11.5.el7]
- [kernel] cpu/hotplug: Enable 'nosmt' as late as possible (Josh Poimboeuf) [1593383 1593384] {CVE-2018-3620}
[...]
```

Find the documentation for a package.

```
[root@tartarus ~]# rpm -q --docfiles openssh
/usr/share/doc/openssh-7.4p1/CREDITS
/usr/share/doc/openssh-7.4p1/ChangeLog
/usr/share/doc/openssh-7.4p1/INSTALL
/usr/share/doc/openssh-7.4p1/OVERVIEW
/usr/share/doc/openssh-7.4p1/PROTOCOL
/usr/share/doc/openssh-7.4p1/PROTOCOL.agent
/usr/share/doc/openssh-7.4p1/PROTOCOL.certkeys
/usr/share/doc/openssh-7.4p1/PROTOCOL.chacha20poly1305
/usr/share/doc/openssh-7.4p1/PROTOCOL.key
/usr/share/doc/openssh-7.4p1/PROTOCOL.krl
/usr/share/doc/openssh-7.4p1/PROTOCOL.mux
/usr/share/doc/openssh-7.4p1/README
[...]
```

Find what scripts run when the package is installed.

```
[root@tartarus ~]# rpm -q --scripts openssh
preinstall scriptlet (using /bin/sh):
getent group ssh_keys >/dev/null || groupadd -r ssh_keys || :
```


* * *

### Common Issues<a name="issue"><a/>

Here is how you would fix a corrupted rpm database. This is probably the most common issue you will run into.

```
[root@tartarus rpm]# rpm -qa
^Cerror: db5 error(11) from dbenv->open: Resource temporarily unavailable
error: cannot open Packages index using db5 - Resource temporarily unavailable (11)
error: cannot open Packages database in /var/lib/rpm
^C^C^Cerror: db5 error(11) from dbenv->open: Resource temporarily unavailable
error: cannot open Packages database in /var/lib/rpm
[root@tartarus rpm]# rsync -av /var/lib/rpm rpmbackup
[root@tartarus rpm]# rm -f /var/lib/rpm/__db.*
[root@tartarus rpm]# rpm --rebuilddb
[root@tartarus rpm]# /usr/lib/rpm/rpmdb_verify /var/lib/rpm/Packages
BDB5105 Verification of Packages succeeded.
```

Another scenario. You have tried the above steps, but instead of verification succeeding, `Packages` is still broken, and absolutely nothing else you know of is working.  'Packages' is the master package metadata file. You will need to take a backup, rebuild the rpmdb from scratch, recover, reimport, and then verify. Then CHECK if rpm/yum are OK. If not, see below.
 
```
[root@tartarus ~]# /usr/lib/rpm/rpmdb_verify /var/lib/rpm/Packages
rpmdb_verify: BDB0521 Page 0: Incomplete metadata page
rpmdb_verify: /var/lib/rpm/Packages: BDB0090 DB_VERIFY_BAD: Database verification failed
BDB5105 Verification of /var/lib/rpm/Packages failed.
[root@tartarus ~]# rsync -av /var/lib/rpm rpmbackup
sending incremental file list
rpm/
rpm/.dbenv.lock
rpm/.rpm.lock
rpm/Basenames
rpm/Conflictname
rpm/Dirnames
rpm/Group
rpm/Installtid
rpm/Name
rpm/Obsoletename
rpm/Packages
rpm/Providename
rpm/Requirename
rpm/Sha1header
rpm/Sigmd5
rpm/Triggername
 
sent 5,847,427 bytes  received 305 bytes  11,695,464.00 bytes/sec
total size is 5,844,998  speedup is 1.00
[root@tartarus ~]# rm -fr /var/lib/rpm/*
[root@tartarus ~]# rpm --initdb
[root@tartarus ~]# mv /var/lib/rpm/Packages /var/lib/rpm/Packages.original
[root@tartarus ~]# db_recover -h /var/lib/rpm
[root@tartarus ~]# db_dump /var/lib/rpm/Packages.original | db_load Packages
[root@tartarus ~]# /usr/lib/rpm/rpmdb_verify /var/lib/rpm/Packages
BDB5105 Verification of /var/lib/rpm/Packages succeeded.
```

If it is still broken (rpm -qa returns nothing / yum is broken) Only then proceed - Even then, you may want to consider rebuilding the server and file a bug report with rpm as the next steps may leave your server in an inconsistent state. Note this is for rhel 7.X, your mileage will vary on other releases. Additional note - I don't know if this is an "accepted" solution. This simply worked for me. If this is rhel 4,5,6 - [try this redhat solution first.](https://access.redhat.com/solutions/23743)

re-import all of the gpg-keys and reset the environment variables for yum.

```
[root@tartarus rpmdb-indexes]# rpm --import /etc/pki/rpm-gpg/RPM-GPG-KEY-*
[root@tartarus rpmdb-indexes]# RELEASEVER=7
[root@tartarus rpmdb-indexes]# ARCH=x86_64
[root@tartarus rpmdb-indexes]# echo $RELEASEVER > /etc/yum/vars/releasever
[root@tartarus rpmdb-indexes]# echo $ARCH > /etc/yum/vars/arch
```

If yum is still broken - try setting the following directive in the given file, note the releaseserver number would probably change depending on your platform.


`file: /usr/lib/python2.X/site-packages/yum/config.py`
 
`startupconf.releasever = releaserver`
 
OR

`startupconf.releasever = 6`

If yum works, then you'll query the cached packages and attempt to reinstall them all. This will take a very long time.

```
[root@tartarus rpmdb-indexes]# screen
[root@tartarus rpmdb-indexes]# for i in $(find /var/lib/yum/yumdb/ -maxdepth 2 -type d| cut -d '-' -f 2- | grep -v yumdb |cut -d '.' -f1 | sed 's/-[0-9]$//g'| tr '\n' ' '); do yum install -y ${i}; done | tee reinstall.log
```

The previous will likely miss a few packages, check with the below.

```
[root@tartarus rpmdb-indexes]# find /var/lib/yum/yumdb/ -maxdepth 2 -type d| cut -d '-' -f 2- | grep -v yumdb > packages.list
[root@tartarus rpmdb-indexes]# for i in $(grep '^No ' reinstall.log | awk '{print $3}'); do grep $i packages.list ; done
```

At the end, the rpm database should be repopulated. Verify the packages as well. Please note however, there are some likely untracked packages. If they were installed outside of yum or weren't in the cache from earlier, they'll be on the system, but not in the rpm database.

```
[root@tartarus rpmdb-indexes]# rpm -qa | wc -l
580
[root@tartarus rpmdb-indexes]# rpm -Va
```

* * *

That concludes this article. If you've read this far, thank you!
