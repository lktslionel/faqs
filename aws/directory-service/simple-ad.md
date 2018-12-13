###### AWS > DS
# Simple AD


## How to connect a linux machine to an AD directory




### Create a simple AD directory

### Create A linux instance

### Configure the security group to match 


Type              | Proto | ports           | Source                             | Description
------------------|-----|---------------|-----------------------------------------|----------
Custom TCP Rule   | TCP | 873           | 10.3.0.0/16                             | Rsync 
DNS (UDP)         | UDP | 53            | 10.3.0.0/16                             | 
LDAP              | TCP | 389           | 10.3.0.0/16                             | LDAP 
Custom UDP Rule   | UDP | 123           | 10.3.0.0/16                             | NTP 
RDP               | TCP | 3389          | 0.0.0.0/0                               | All RDP traffic 
SMB               | TCP | 445           | 10.3.0.0/16                             | SAMBA 
Custom TCP Rule   | TCP | 3268          | 10.3.0.0/16                             | Global Catalog 
Custom TCP Rule   | TCP | 1024 - 65535  | 10.3.0.0/16                             | Ephemeral ports fo... 
Custom TCP Rule   | TCP | 88            | 10.3.0.0/16                             | Kbr5 
All traffic       | All | All           | sg-0a80908760263511b (launch-wizard-10) | Self 
SSH               | TCP | 22            | 195.190.27.9/32                         | OS DSI 
Custom UDP Rule   | UDP | 137 - 138     | 10.3.0.0/16                             | Netlogon 
Custom TCP Rule   | TCP | 139           | 10.3.0.0/16                             | Netlogon 
Custom TCP Rule   | TCP | 135           | 10.3.0.0/16                             | RPC 
Custom TCP Rule   | TCP | 636           | 10.3.0.0/16                             | LDAPS 
DNS (TCP)         | TCP | 53            | 10.3.0.0/16                             | VPC 3 
Custom UDP Rule   | UDP | 88            | 10.3.0.0/16                             | Kerberos authentic



### join the AD

#### Connnect to instance

```
$ ssh-agent bash
$ ssh-add <PATH_TO_SSH_KEY>
$ ssh -A ec2-user@<PUBLIC_INSTANCE_IP>
```
#### Install required softwares

```
$ sudo yum update
$ sudo yum install -y \
    realmd\
    sssd\
    kbr5-workstation\
    oddjob\
    oddjob-mkhomedir\
    samba-common-tools
```

#### Join the AD using the admin account from simple AD directory

```
$ sudo realm join -U Administrator@ad.<DOMAIN_NAME> ad.<DOMAIN_NAME> --verbose  # Eg: <DOMAIN_NAME = example.com
```

### Now we will try to connect to the instance using a user from the AD

#### Prepare the instance for password authentication


First let's allow password authentication via ssh. Uncomment the following line in the `/etc/ssh/sshd_config` file.

```
# To disable tunneled clear text passwords, change to no here!
PasswordAuthentication yes
```

And then restart the ssd agent : 

```
sudo service sshd restart 
```

Change the root password : 

```
sudo passwd root
```

And reboot the instance. (from the AWS console)


#### Now lets add our ad administrators to the sudoers

Run the command `visudo` and edit the sudoers files by adding the following line:

```
%Domain\ Admins@ad.<DOMAIN_NAME> ALL = (ALL:ALL) ALL
%Groups\ Admins@ad.<DOMAIN_NAME> ALL = (ALL:ALL) ALL
```
The lines below allow groups `%Domain\ Admins@ad.<DOMAIN_NAME>`and `%Groups\ Admins@ad.<DOMAIN_NAME>` on host `ALL` to run `ALL` commands as user:group `ALL:ALL`.