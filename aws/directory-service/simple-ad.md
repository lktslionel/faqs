###### AWS > DS
# Simple AD


### How to connect a linux machine to an AD directory



#### Create a simple AD directory
#### Create A linux instance
#### join the AD

##### Connnect to instance

```
$ ssh-agent bash
$ ssh-add <PATH_TO_SSH_KEY>
$ ssh -A ec2-user@<PUBLIC_INSTANCE_IP>
```
##### Install required softwares

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

##### Join the AD using the admin account from simple AD directory

```
$ sudo realm join -U Administrator@ad.<DOMAIN_NAME> ad.<DOMAIN_NAME> --verbose  # Eg: <DOMAIN_NAME = example.com
```



