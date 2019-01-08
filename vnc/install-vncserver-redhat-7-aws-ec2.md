###### VNC
# How to connect to AWS EC2 Linux Centos7 server via VNC ? 

In this guide, we are going to setup a vnc server on an aws ec2 isntance (Redhat 7) and connect to the instance via a VNC client.

<br>

## Pre-requisites

To follow this guide we need : 

* 1 EC2 Instance launch on AWS
* A VPN client. I use [VNC viewer](https://www.realvnc.com/en/connect/download/viewer/) for Mac. But it is available for any platform.

And to have basic knowledges of : 

* `YUM package manager`
* `Firewalld`
* AWS EC2 Service

<br>

## Steps 

### 1. Create a Redhat 7 - EC2 Instance

Go to the console or use `awscli`and launch and EC2 instance. I will not go into more detail about how to do it 'cause the guide expect you to have the basic knowledge of the EC2 service.

When your instance is created and ready, go to the **step 2**.

### 2. Install required packages

First, connect to the instance and update packages on our server with the command below : 

```bash
sudo yum -y update
```

Then, install the gnome GUI components using the following command : 

```bash
sudo yum groupinstall -y "Server with GUI"
```

Make sure the systemd, at during boot, starts with the graphical interface

```bash 
sudo systemctl set-default graphical.target
sudo systemctl default
```

Now, install the VNC service

```bash
sudo yum install -y tigervnc-server
```

### 2. Configure the VNC service

First, let's change the password of the `ec2-user`and `root` users; this will be used when you login to your EC2 Desktop : 

```bash 
# Change ec2-user password
sudo passwd ec2-user

# Change root password
sudo su 
passwd
```

Then, let the VNC service being managed as a systemd service

```bash
sudo cp /lib/systemd/system/vncserver\@.service /etc/systemd/system/vncserver\@:1.service
```

you have notice that we added a `:1` after the `@` sign; that's because `:1` means the first display/screen.
And the vnc server will be running on port `5901`. 

> We are configuring our first display. If you have a display numbered `<X>`. You will have a config file at `/etc/systemd/system/vncserver\@:<X>.service` and the vnc service are going to be started on `590<X>`. So, just stick to this convention :-p.

<br>

Next, make sure the service is configure to run with the user `ec2-user`, by executing the command : 

```bash 
sudo sed -i 's/<USER>/ec2-user/g'  /etc/systemd/system/vncserver\@:1.service
```

Reload systemd configuration and ensure that the service is started at boot : 

```bash
sudo systemctl daemon-reload
systemctl enable vncserver@:1.service
```


### 3. Disable SE-Linux

Run the following command to check the status of SE-Linux : 

```bash
sudo getenforce
#=> Enforcing
```

`Enforcing` means that it is activated. Run the following commands to change the state of SE Linux from `Enforcing`to `permissive` : 

```bash
sed -i 's/SELINUX=enforcing/SELINUX=permissive/' /etc/selinux/config
setenforce 0
```


### 4. Start the VNC service

First, initialize the vnc server for the user `ec2-user`.

```bash
sudo su - ec2-user

# And run
vncserver

# You will require a password to access your desktops.
# 
# Password:
# Verify:
# Would you like to enter a view-only password (y/n)? n
# A view-only password is not used
# xauth:  file /home/ec2-user/.Xauthority does not exist
# 
# New 'XXXXXXXX.eu-west-1.compute.internal:1 (ec2-user)' desktop is XXXXXXXX.eu-west-1.compute.internal:1
# 
# Creating default startup script /home/ec2-user/.vnc/xstartup
# Creating default config /home/ec2-user/.vnc/config
# Starting applications specified in /home/ec2-user/.vnc/xstartup
# Log file is /home/ec2-user/.vnc/XXXXXXXX.eu-west-1.compute.internal:1.log
```

And then start the vnc service as root with the command : 

```bash
systemctl start vncserver@:1.service
```


### 5. Connecft to the instance using a VNC Cient

Open you VNC Client and try to connect to the following url : 

```
<PUBLIC_IP_FO_EC2_INSTANCE>:5901
```

This will probably fails; not said for sure :-). You will get the following error : 

```
Timed out waiting for a response from the computer
```

<br>

Go the the section `Known issues` to find out more about the errors; and the retry, everything should be fine.


<br>

## Alternatives

None

<br>

## Known issues 

### Security groups misconfigured

You probably got an error when trying to initiate a VNC connection between your client and the EC2 instance previously configured. One of the reason why it doesn't work is because you have yet allowed the traffic to flow into the instance on port 5901. So, go to the security group attached to your instance and add the following entry : 

Type  | Protocol | Port Range | Source | Description
------|----------|------------|--------|-------------
Custom TCP Rule|TCP|5901|(your IP or CIDR from which you want to connect)|VNC client


<br>

### Traffic denied by firewalld on the instance

The connect error could be due to `firewalld` which denied the inbound traffic for the service `svncserver`.
You fix that, run the following command: 

```bash 
# Check the status of firewalld
sudo firewall-cmd --state
# => running

# Add service 'vnc-server' to the 'public' zone 
sudo firewall-cmd --zone=public --add-service=vnc-server --permanent

# Check if the service is present
sudo firewall-cmd --list-all

# public (active)
...
#   services: ssh dhcpv6-client vnc-server
...
```

<br>

### The resolution doesn't match the one on my screen

When you are connect to the instance, launch the `Terminal` app and list all resolution supported : 

```bash
# List all resolutions supported
xrandr

# Screen 0: minimum 32 x 32, current 1024 x 768, maximum 32768 x 32768
# VNC-0 connected primary 1024x768+0+0 0mm x 0mm
#    1024x768      60.00*+
#    1920x1200     60.00  
#    1920x1080     60.00  
#    1600x1200     60.00  
#    1680x1050     60.00  
#    1400x1050     60.00  
#    1360x768      60.00  
#    1280x1024     60.00  

```

Then choose the one you want but executing the command : 

```bash
# For example I choosed '1920x1080' :-p
xrandr -s 1920x1080

```



<br>

## References 

* [Ghist | How to Install VNC on an Amazon EC2 Centos 7.2 AMI](https://gist.github.com/mikaelMortensenADI/d820435eaee92f518e12f3d92e3a0ce4)
* [Youtube | How to configure vncserver in redhat 7 or centos 7](https://www.youtube.com/watch?v=-3C4XDmF1fY)
* [tecmint.com | How to Install and Configure VNC Server in CentOS 7](https://www.tecmint.com/install-and-configure-vnc-server-in-centos-7/)
* [digitalocean.com | How To Set Up a Firewall Using FirewallD on CentOS 7](https://www.digitalocean.com/community/tutorials/how-to-set-up-a-firewall-using-firewalld-on-centos-7)
