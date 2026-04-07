# UBUNTU_0 by Blága Máté

This is a common guide for ubuntu configurations

## Network Settings

### Via Netplan

The Config file is in ``/etc/netplan/01-netcfg.yaml``

Set Static IP
```yaml
network:
  version: 2
  ethernets:
    <INTERFACE NAME>:
      dhcp4: false
      addresses:
        - <IP Address>/<SM>
        - <IPv6 Address>/<prefix>
      routes:
        - to: default
          via: <Default Gatway>
        - to: default
          via: <IPv6 Address>
      nameservers:
        addresses: [8.8.8.8,8.8.4.4]
```

Set DHCP
```yaml
network:
  version: 2
  ethernets:
    <INTERFACE NAME>:
      dhcp4: true
      dhcp6: true
```

Validate Netplan Configuration
```sh
 sudo netplan generate
```
Apply Netplan config
```sh
sudo netplan apply
```

## User Management

### Add User with home directory
```sh
sudo useradd -m <USERNAME>
```

### Add Group
```sh
sudo groupadd <GROUPNAME>
```

### Add User to Group
```sh
sudo usermod -aG <GROUPNAME> <USERNAME>
```

### Remove User from Group
```sh
sudo gpasswd -d <USERNAME> <GROUPNAME>
```
### Delete User
```sh
sudo deluser <USERNAME>
```
### Delete Group
```sh
sudo groupdel <GROUPNAME>
```
### Change User Password
```sh
sudo passwd <USERNAME>
```
### Add User to Sudo Group
```sh
sudo usermod -aG sudo <USERNAME>
sudo visudo # Add the following line at the end of the file
<USERNAME> ALL=(ALL) NOPASSWD:ALL
```

## File Permissions


### Permission Representation
| Num | Per | Bin |
|-----|-----|-----|
| 7   | rwx | 111 |
| 6   | rw- | 110 |
| 5   | r-x | 101 |
| 4   | r-- | 100 |
| 3   | -wx | 011 |
| 2   | -w- | 010 |
| 1   | --x | 001 |
| 0   | --- | 000 |

### Change File Owner
```sh
sudo chown <USERNAME>:<GROUPNAME> <FILE>
```
### Change File Permissions
```sh
sudo chmod <PERMISSIONS> <FILE/DIRECTORY>
```
### Change File Permissions Recursively
```sh
sudo chmod -R <PERMISSIONS> <DIRECTORY>
```

## ACL Permissions

0. Step is to install ACL package
```sh
sudo apt install acl
```


Set ACL permissions for a user/group
```sh
sudo setfacl -m <u/g>:<USERNAME>:<PERMISSIONS> <FILE/DIRECTORY>
```

### View ACL permissions for a file/directory
```sh
getfacl <FILE/DIRECTORY>
```

### Remove ACL permissions for a user/group
```sh
sudo setfacl -x <u/g>:<USERNAME> <FILE/DIRECTORY>
```

## Set quota
1. Install quota package
```sh
sudo apt install quota
```
2. Edit ``/etc/fstab`` and add usrquota and grpquota options to the desired partition
```
/dev/sda1 / ext4 defaults,usrquota,grpquota 0 1
```
3. Remount the partition
```sh
systemctl daemon-reload
sudo mount -o remount /
```

4. Create quota files
```sh
sudo quotacheck -ugm /
```

5. Enable quota
```sh
sudo quotaon -v /
```
6. Set quota for a user
```sh
sudo setquota -u <USERNAME> <SOFT LIMIT> <HARD LIMIT> 0 0 /
```
7. View quota for a user
```sh
sudo quota -u <USERNAME>
```
8. Remove quota for a user
```sh
sudo setquota -u <USERNAME> 0 0 0 0 /
```
## TAR Command
### Create a tar archive
```sh
tar -cvf <ARCHIVE_NAME>.tar <FILE/DIRECTORY>
```
### Extract a tar archive
```sh
tar -xvf <ARCHIVE_NAME>.tar -C <DESTINATION_DIRECTORY>
```
### Compression methods
| Option  | Compression Method | File Extension |
|---------|--------------------|----------------|
| -z      | gzip               | .tar.gz        |
| -j      | bzip2              | .tar.bz2       |
| -J      | xz                 | .tar.xz        |
| --zstd  | zstd               | .tar.zst       |

## DHCP Server Configuration
1. Install DHCP server package
```sh
sudo apt install isc-dhcp-server
```
2. Edit DHCP server configuration file
```sh
sudo nano /etc/dhcp/dhcpd.conf
```
3. Add the following lines to the configuration file
```
subnet <SUBNET> netmask <NETMASK> {
  range <START_IP> <END_IP>;
  option routers <DEFAULT_GATEWAY>;
  option domain-name-servers <DNS_SERVER>;
}
```
4. Restart DHCP server
```sh
sudo systemctl restart isc-dhcp-server
```

## DNS Server Configuration
1. Install DNS server package
```sh
sudo apt install dnsmasq
```
2. Edit DNS server configuration file
```sh
sudo nano /etc/dnsmasq.conf
```
3. Change the following lines in the configuration file
```interface=<INTERFACE_NAME>
listen-address=<IP_ADDRESS>
```
4. Stop systemd-resolved service and change the dns server
```sh
sudo systemctl stop systemd-resolved
sudo systemctl disable systemd-resolved
echo "nameserver <IP_ADDRESS>" | sudo tee /etc/resolv.conf
```
5. Restart DNS server
```sh
sudo systemctl restart dnsmasq
```

6. Add custom DNS records
```sh
sudo nano /etc/hosts
```
Add the following lines to the hosts file
```
<IP_ADDRESS> <HOSTNAME>
```

## DHCP and DNS server configuration (dnsmasq)

```sh 
sudo apt install dnsmasq
```

Modify the config file at ``/etc/dnsmasq.conf``

### DHCP
```
interface=enp1s0

# start address, end address, lease time
dhcp-range=192.168.1.100,192.168.1.150,12h

# default gateway
dhcp-option=3,192.168.1.1

# static ip to client
dhcp-host=aa:bb:cc:dd:ee:ff,192.168.1.101
```

### DNS
```
# Don't read resolv.conf
no-resolv

# Forward queries
server=8.8.8.8
server=8.8.4.4

# Set custom records
address=/router/192.168.1.1
local=/example.home/
domain=exam.local
```

## Enable ipv4 forwarding
1. Edit sysctl configuration file
```sh
sudo nano /etc/sysctl.conf
```
2. Add the following line to the configuration file
```
net.ipv4.ip_forward=1
```
3. Apply the changes
```sh
sudo sysctl -p
```
## Apache Web Server Configuration
1. Install Apache web server package
```sh
sudo apt install apache2
```
2. Create a new virtual host configuration file
```sh
sudo nano /etc/apache2/sites-available/<DOMAIN_NAME>.conf
```
3. Add the following lines to the configuration file
```
<VirtualHost *:80>
    ServerName <DOMAIN_NAME>
    DocumentRoot /var/www/<DOMAIN_NAME>
    <Directory /var/www/<DOMAIN_NAME>>
        Options Indexes FollowSymLinks
        AllowOverride All
        Require all granted
    </Directory>
</VirtualHost>
```
4. Enable the new virtual host
```sh
sudo a2ensite <DOMAIN_NAME>.conf
```
5. Disable the default virtual host
```sh
sudo a2dissite 000-default.conf
```

6. Restart Apache web server
```sh
sudo systemctl restart apache2
```

### PHP Configuration
1. Install PHP package
```sh
sudo apt install php libapache2-mod-php
```
2. Create a new PHP file in the document root
```sh
sudo nano /var/www/<DOMAIN_NAME>/index.php
```
3. Add the following lines to the PHP file
```php
<?php
phpinfo();
?>
```
4. Restart Apache web server
```sh
sudo systemctl restart apache2
```

## Basic HTML
```html
<!DOCTYPE html><html><body><h1>Hello World!</h1></body></html>
```

## MySQL Database Server Configuration
1. Install MySQL database server package
```sh
sudo apt install mysql-server
```
2. Secure MySQL installation
```sh
sudo mysql_secure_installation
```
3. Log in to MySQL shell
```sh
sudo mysql -u root -p
```
4. Create a new database
```sql
CREATE DATABASE <DATABASE_NAME>;
```
5. Create a new user and grant privileges
```sql
CREATE USER '<USERNAME>'@'localhost' IDENTIFIED BY '<PASSWORD>';
GRANT ALL PRIVILEGES ON <DATABASE_NAME>.* TO '<USERNAME>'@'localhost';
FLUSH PRIVILEGES;
```
6. Exit MySQL shell
```sql
EXIT;
```
7. Restart MySQL database server
```sh
sudo systemctl restart mysql
```

## SSH Server Configuration
1. Install SSH server package
```sh
sudo apt install openssh-server
```
2. Edit SSH server configuration file
```sh
sudo nano /etc/ssh/sshd_config
```
3. Change the following lines in the configuration file
```
Port <PORT_NUMBER>
PermitRootLogin no
PasswordAuthentication no
```
4. Restart SSH server
```sh
sudo systemctl restart ssh
```

## Fail2Ban Configuration
1. Install Fail2Ban package
```sh
sudo apt install fail2ban
```
2. Create a new jail configuration file
```sh
sudo nano /etc/fail2ban/jail.local
```
3. Add the following lines to the configuration file
```
[sshd]
enabled = true
port = <PORT_NUMBER>
filter = sshd
logpath = /var/log/auth.log
maxretry = 5
```
4. Restart Fail2Ban service
```sh
sudo systemctl restart fail2ban
```

## Useful Ports

| Service       | Port Number | Protocol |
|---------------|-------------|----------|
| SSH           | 22          | TCP      |
| HTTP          | 80          | TCP      |
| HTTPS         | 443         | TCP      |
| HTTP Proxy    | 8080        | TCP      |
| HTTPS Proxy   | 8443        | TCP      |
| MySQL         | 3306        | TCP      |
| PostgreSQL    | 5432        | TCP      |
| DNS           | 53          | TCP/UDP  |
| DHCP          | 67          | UDP      |
| NTP           | 123         | UDP      |
| SNMP          | 161         | UDP      |
| SNMP Trap     | 162         | UDP      |
| Samba         | 445         | TCP      |
| FTP           | 21          | TCP      |
| FTP (Data)    | 20          | TCP      |
| SFTP/SSH      | 22          | TCP      |
| TFTP          | 69          | UDP      |
| SMTP          | 25          | TCP      |
| SMTP (SSL)    | 465         | TCP      |
| POP3          | 110         | TCP      |
| POP3 (SSL)    | 995         | TCP      |
| IMAP          | 143         | TCP      |
| IMAP (SSL)    | 993         | TCP      |


## IPTables Configuration
### Install IPTables package
```sh
sudo apt install iptables
```
### Check current IPTables rules
```sh
sudo iptables -L -v
```
### Allow incoming port
```sh
sudo iptables -A INPUT -p <tcp/udp> --dport <PORT_NUMBER> -j ACCEPT
```
### Drop all other incoming connections
```sh
sudo iptables -A INPUT -j DROP
```
### Save IPTables rules
```sh
sudo iptables-save | sudo tee /etc/iptables/rules.v4
```
### Load IPTables rules on boot
```sh
sudo nano /etc/rc.local
```
Add the following lines to the rc.local file
```sh
#!/bin/bash
iptables-restore < /etc/iptables/rules.v4
exit 0
```
Dont forget to make rc.local file executable
```sh
sudo chmod +x /etc/rc.local
```
## UFW Configuration
### Install UFW package
```sh
sudo apt install ufw
```
### Allow a specific port
```sh
sudo ufw allow <PORT_NUMBER>/<tcp/udp>
```
### Allow a specific service
```sh
sudo ufw allow <SERVICE_NAME>
```
### Enable UFW
```sh
sudo ufw enable
```
### Check UFW status
```sh
sudo ufw status verbose
```
### Restart UFW
```sh
sudo ufw reload
```

## SAMBA Server Configuration
1. Install SAMBA package
```sh
sudo apt install samba
```
2. Edit SAMBA configuration file
```sh
sudo nano /etc/samba/smb.conf
```
3. Add the following lines to the configuration file
```
[SHARE_NAME]
   path = /path/to/share
   available = yes
   valid users = <USERNAME>
   read only = no
   browsable = yes
   public = yes
   writable = yes
```

4. Create a new SAMBA user
```sh
sudo smbpasswd -a <USERNAME>
```
5. Restart SAMBA service
```sh
sudo systemctl restart smbd
```

## Sambe Client Configuration

### Install SAMBA client package
```sh
sudo apt install samba-client
```
### Mount a SAMBA share
```sh
sudo mount -t cifs //<SERVER_IP>/<SHARE_NAME> /mnt/samba -o username=<USERNAME>,password=<PASSWORD>
```

## CRON Job Configuration
### Install CRON package
```sh
sudo apt install cron
```
### Edit CRON jobs
```sh
sudo crontab -e
```
### Add a new CRON job
```
* * * * * /path/to/script.sh
```
### Restart CRON service
```sh
sudo systemctl restart cron
```
### Cron syntax
| Field        | Allowed Values           | Special Characters |
|--------------|--------------------------|--------------------|
| Minute       | 0-59                     | * / , -            |
| Hour         | 0-23                     | * / , -            |
| Day of Month | 1-31                     | * / , -            |
| Month        | 1-12 or Jan-Dec          | * / , -            |
| Day of Week  | 0-7 or Sun-Sat (0 and 7 are Sunday) | * / , -            |

### Example CRON job to run a script every 5 minutes
```
*/5 * * * * /path/to/script.sh
```

## Rsyslog Configuration
### Install Rsyslog package
```sh
sudo apt install rsyslog
```
### Check Rsyslog

```sh
sudo cat /var/log/syslog
```

### Edit Rsyslog configuration file
```sh
echo "*.* /var/log/all.log" | sudo tee -a /etc/rsyslog.conf
```

### Restart Rsyslog service
```sh
sudo systemctl restart rsyslog
```


## Fdisk Command - Disk Partitioning

### List all disks and partitions
```sh
sudo fdisk -l
```
### Create a new partition
```sh
sudo fdisk /dev/sdX
```
Follow the interactive prompts to create a new partition. After creating the partition, you need to format it and mount it to use it.

### Format the new partition for ext4 filesystem
```sh
sudo mkfs.ext4 -L <Lable Name> /dev/sdX1
```
### Create a mount point and mount the new partition
```sh
sudo mkdir /mnt/new_partition
sudo mount /dev/sdX1 /mnt/new_partition
```
### Add the new partition to /etc/fstab for automatic mounting on boot by Label
```sh
echo "LABEL=<Lable Name> /mnt/new_partition ext4 defaults 0 2" | sudo tee -a /etc/fstab
```
### Remount by Fstab settings
```sh
systemctl daemon-reload 
sudo mount -a
```
### Delete a partition
```sh
sudo fdisk /dev/sdX
```
Follow the interactive prompts to delete the desired partition. Be careful when deleting partitions, as it will result in data loss.

### Resize a partition
```sh
sudo fdisk /dev/sdX
```
Follow the interactive prompts to delete the existing partition and create a new partition with the desired size. After resizing the partition, you need to resize the filesystem to use the new size.

## Rsnapshot Configuration - Backup Tool

```sh
sudo apt install rsnapshot
```

Basic config file. NOTE: Config options and values need to be separated by tabs
```sh
config_version	1.2

# Destination of the snapshots
snapshot_root	/var/cache/rsnapshot/

# External commands
cmd_cp		/bin/cp
cmd_rm		/bin/rm
cmd_rsync	/usr/bin/rsync
cmd_logger	/usr/bin/logger

# How many snapshots to retain with aliases
retain   daily 7
retain   weekly 4
retain   monthly 12
# The alias can be anything
# retain	mysnapshot	6

### BACKUP POINTS / SCRIPTS
# The destination path is relative to the backup root
# backup /path/to/source path/to/destination
backup	/home/		localhost/
backup	/etc/		localhost/
backup	/usr/local/	localhost/
```

Backups can be run with
```sh
rsnapshot <snapshot_alias>
```

Example cronfile for snapshotting 
```
# m   h   dom mon dow       command
  0   4   *   *   *         /usr/bin/rsnapshot daily   # 4AM every day
  0   3   *   *   1         /usr/bin/rsnapshot weekly  # Sunday 3AM
  0   2   1   *   *         /usr/bin/rsnapshot monthly # 2AM on the 1st
```

## SCRIPTING

### Set Variables
```sh
VARIABLE_NAME="Value"
```
### Use Variables
```sh
echo "The value of the variable is: $VARIABLE_NAME"
```
### Unset Variables
```sh
unset VARIABLE_NAME
```

### Read User Input
```sh
read -p "Enter your name: " NAME
echo "Hello, $NAME!"
```

### IF statement
```sh
if [ <CONDITION> ]; then
    # Commands to execute if condition is true
elif [ <ANOTHER_CONDITION> ]; then
    # Commands to execute if another condition is true
else
    # Commands to execute if all conditions are false
fi
```

### FOREACH loop
```sh
for <VARIABLE> in <LIST>; do
    # Commands to execute for each item in the list
done
```
### FOR loop
```sh
for (( i=0; i < 10; i+=1 )); do
    # Commands to execute for each iteration
done
```
### WHILE loop
```sh
while [ <CONDITION> ]; do
    # Commands to execute while condition is true
done
```
