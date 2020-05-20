# CentOS 8 as an Active Directory Domain Services (AD DS) Member

> Assumption is made that this is a newly installed base system
>
> Replace *DOMAIN* with AD DS NetBIOS Name and *domain.example.com* with AD DS FQDN

### DNF

Install EPEL and PowerTools:

```bash
$ sudo dnf install https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm
$ sudo dnf config-manager --enable PowerTools
$ sudo dnf update
```

### Cockpit

Remove old version of Cockpit if version is less than 201 and install latest Cockpit Preview

```bash
$ sudo dnf remove cockpit\*
$ sudo dnf config-manager --add-repo https://copr.fedorainfracloud.org/coprs/g/cockpit/cockpit-preview/repo/epel-8/group_cockpit-cockpit-preview-epel-8.repo

$ sudo dnf install cockpit cockpit-storaged setroubleshoot-server
```

Enable Cockpit:

```bash
$ sudo systemctl enable --now cockpit.socket
```

Create firewall rules for Cockpit:

```bash
$ sudo firewall-cmd --permanent --zone=public --add-service=cockpit
$ sudo firewall-cmd --reload
```

Install Cockpit ZFS Manager

```bash
$ git clone https://github.com/optimans/cockpit-zfs-manager.git
$ sudo cp -r cockpit-zfs-manager/zfs /usr/share/cockpit
```

### ZFS

Install ZFS as per own requirements from ZFS on Linux: [https://github.com/openzfs/zfs/wiki/RHEL-and-CentOS](https://github.com/openzfs/zfs/wiki/RHEL-and-CentOS)

### Samba

Install Samba

```bash
$ sudo dnf install -y realmd oddjob-mkhomedir oddjob samba-winbind-clients samba-winbind samba-common-tools
$ sudo dnf install -y samba
$ sudo dnf install -y samba-winbind-krb5-locator krb5-workstation samba-client

$ sudo rm /etc/samba/smb.conf
```

Join AD DS:

```bash
$ sudo realm discover -vvv domain.example.com
$ sudo realm join --client-software=winbind domain.example.com -U Administrator
```

Start Samba

```bash
$ sudo systemctl start smb
```

Verify information is retrieved from AD DS:

```
$ sudo getent passwd "DOMAIN\Administrator"
$ sudo getent group "DOMAIN\Domain Users"
$ sudo wbinfo -g
$ sudo wbinfo -u
```

Edit Samba configuration file and set the AD DS schema mode, ACLs and Previous Versions properties:

```bash
$ sudo nano /etc/samba/smb.conf
```

Append to [global] section

```
idmap config DOMAIN : schema_mode = rfc2307

vfs objects = acl_xattr shadow_copy2
store dos attributes = yes
map acl inherit = yes
inherit acls = yes
inherit permissions = yes
				
shadow: snapdir = .zfs/snapshot
shadow: sort = desc
shadow: format = %Y.%m.%d-%H.%M.%S
shadow: localtime = yes

admin users = @"DOMAIN\Domain Admins"
```

Reload Samba configuration:

```bash
$ sudo smbcontrol all reload-config
```

Grant Disk Operator Privileges:

```bash
$ sudo net rpc rights grant "DOMAIN\Domain Admins" SeDiskOperatorPrivilege -U "DOMAIN\Administrator"
$ sudo net rpc rights grant "DOMAIN\Enterprise Admins" SeDiskOperatorPrivilege -U "DOMAIN\Administrator"
```
Enable SELinux booleans:

```bash
$ sudo setsebool -P samba_export_all_ro=1 samba_export_all_rw=1
$ sudo getsebool -a | grep samba_export
```
Create firewall rules for Samba:

```bash
$ sudo firewall-cmd --permanent --add-service=samba
$ sudo firewall-cmd --reload
```

Restart and Enable Samba service:

```bash
$ sudo systemctl restart smb
$ sudo systemctl enable smb
```

### Local Authorisation

Edit Kerberos configuration file to enable domain users to authenticate to local services:

```bash
$ sudo nano /etc/krb5.conf
```

Append to end of file

```
[plugins]
    localauth = {
        module = winbind:/usr/lib64/samba/krb5/winbind_krb5_localauth.so
        enable_only = winbind
    }
```

Create sudoers configuration file to allow sudo access to domain groups

```bash
$ sudo nano /etc/sudoers.d/DOMAIN
```

Add to file

```
DOMAIN\\Domain\ Admins ALL=(ALL) ALL
DOMAIN\\Enterprise\ Admins ALL=(ALL) ALL
```

### Red Hat Enterprise Linux 8 Documentation

 * [Chapter 2. Using Samba as a Server](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/8/html/deploying_different_types_of_servers/assembly_using-samba-as-a-server_deploying-different-types-of-servers)
