# Cockpit ZFS Manager

**An interactive ZFS on Linux admin package for Cockpit.**

### WARNING!

Cockpit ZFS Manager is currently pre-release software.

#### Use at own risk!

Not recommended for use on production systems.\
Ensure all critical data is adequately backed up before use.

## Requirements

 * Cockpit: 201+
 * Samba: 4+
 * ZFS: 0.8+

#### Tested Distributions:

* CentOS Linux 8.0
* Debian 10.0
* Oracle Linux Server 8.0
* Red Hat Enterprise Linux 8.0
* Ubuntu 19.10

## Installation

Copy zfs folder to cockpit

```bash
$ sudo cp -r zfs /usr/share/cockpit
```

#### Samba

Auto generated snapshot names are created in YYYY.MM.DD-HH.MM.SS format.

It is recommended to add the following properties to the Samba configuration file to allow access to Previous Versions in Windows Explorer:

```bash
$ sudo nano /etc/samba/smb.conf
```

Append to [global] section or individual share sections

```
[global]
~
shadow: snapdir = .zfs/snapshot
shadow: sort = desc
shadow: format = %Y.%m.%d-%H.%M.%S
shadow: localtime = yes	
vfs objects = acl_xattr shadow_copy2
```

### SELinux

If using SELinux in enforcing mode, it is recommended to enable the boolean states for Samba:

```bash
$ sudo setsebool -P samba_export_all_ro=1 samba_export_all_rw=1
```

## Using Cockpit ZFS Manager

Login to Cockpit as a privileged user and click ZFS from the navigation list.

A Welcome to Cockpit ZFS Manager modal will display and allow you to configure initial settings.

Note: Inline help is currently available in modals. Documentation will be created at a later date.



## Caveats

#### Storage Pools

New storage pools are created with the following properties set (not visible in Create Storage Pool modal):

 * aclinherit=passthrough
 * acltype=posixacl
 * casesensitivity=sensitive
 * sharenfs=off
 * sharesmb=off
 * xattr=sa

#### File Systems

New file systems are created with the following properties set (not visible in Create File System modal):

 * normalization=formD
 * utf8only=on

Passphrase is currently supported for encrypted file systems.

If SELinux contexts for Samba is selected, the following properties are set:

 * context=system_u:object_r:samba_share_t:s0
 * fscontext=system_u:object_r:samba_share_t:s0
 * defcontext=system_u:object_r:samba_share_t:s0
 * rootcontext=system_u:object_r:samba_share_t:s0

#### Samba

ZFS always creates shares in /var/lib/samba/usershares folder when ShareSMB property is enabled. This is also the case even if Cockpit ZFS Manager is managing the shares. To avoid duplicate shares of the same file system, it is recommended to configure a different usershares folder path if required or to disable usershares in the Samba configuration file.

If enabled, Cockpit ZFS Manager manages shares for the file systems only. Samba global configuration will need to be configured externally.
## More Information

* [Roadmap](ROADMAP.md)
* [ServeTheHome and ServeThe.Biz Forums](https://forums.servethehome.com/index.php?threads/25668/)

## Guides

 * [Installation Example for CentOS 8.0 as an Active Directory Domain Services (AD DS) Member](guides/CentOS-8.0.md)
