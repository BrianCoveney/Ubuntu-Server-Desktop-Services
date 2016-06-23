
# Internet & Network Services

## Ubuntu Server & Desktop Services

### Report by: Brian Coveney 

### YouTube Demo [Video][1]  

### brian.coveney@mycit.ie R00105727

---


### 1. OpenSSH Server:

#### 1.1 Installation:
`sudo apt-get install openssh-server`

We will edit the configuration file, but first we will make a copy of it and secure that copy:

`sudo cp /etc/ssh/sshd_config /etc/ssh/sshd_config.origina` <br>
`sudo chmod a-w /etc/ssh/sshd_config.original`


#### 1.2 Disable Password authentication

Next we will disable password authentication for logging into the system. This is because password authentication is not secure enough -- it can be too easily hacked, by methods such as a brute force attack

Open the `shhd_config` file in the [nano][2] text editor:

`sudo nano /etc/ssh/sshd_config`

Disabling password authentication is done by changing the following line:

`PasswordAuthentication yes` <- from this <br> 
`PasswordAuthentication no`  <- to this


For this to alteration to take effect, we must restart SSH: 

`sudo /etc/init.d/ssh restart`


#### 1.3 Key-based authentication

We will now set-up Key-based authentication. This is far more secure than using the plain password login method. Key-based authentication, uses [Public and Private Keys][3]. The private key is stored and kept secret on the machine you log in on. It decrypts (unlocks) the public key which is stored in the `.ssh/authorized_keys` file on the machine you want to log in to. The public key does not need to be secure and can be placed on any server.

On the client, we will create the key pairs: 

``` bash
    mkdir ~/.ssh 
    chmod 700 ~/.ssh 
    ssh-keygen -t rsa
```

A prompt will appear, requesting a location to save the keys and their passphrase. The passphrase is a second form of security, used to secure your key on the hard drive.

Your public key is now available as `.ssh/id_rsa.pub` in your home folder. You can view it using this command:

`ssh-keygen lf ~/.ssh/id_rsa.pub`

Next, transfer the client key to the host: `ssh-copy-id <username>@<host>`  

Then log in: `ssh <username>@<host>`  


#### 1.4 Troubleshooting SSH

When I SSHing into the server, I was still being prompted for my password. To ensure that I am logging in with the keys, I needed to do the following:  
 
Create a new folder in the ssh directory: `/etc/ssh/brian`  

Change permissions to read, write & execute:  `sudo chmod 755 /etc/ssh/brian` 

Copy in the public key: `cp /etc/ssh/ssh_host_rsa_key.pub /etc/ssh/brian`

Edit the ssh_config file where at the line beginning with ‘AuthorizedKeysFile’, so that it points at the new location of our public key:

``` bash
    sudo nano /etc/ssh_config 
    AuthorizedKeysFile/etc/ssh/%u/authorized_keys
```

Then restart SSH: `sudo services ssh restart`

#### Complete the following steps on the Server:

Create the user account: `sudo adduser rubyadmin` 

Grant root privileges to the user: `sudo adduser rubyadmin` 

log in the user: `su rubyadmin`

#### Complete the following steps on the Client: 

copy key for the user: `ssh-copy-id rubyadmin@192.168.0.100` 
 
SSH in as the user: `ssh rubyadmin@192.168.0.100`


#### 1.5 SSH with Bridged Mode

So far we have only been able to communicate between the two machines by configuring the VMware NIC settings to Host only. Also having the client and server set up with *static* IP addresses. We then have no internet connection. For us to get connection, so we can pull down packages, we need to configure the VMware settings back to NAT and the network interfaces to *dhcp*. Thus, losing the connection between client and host.      
To configure our environment so that we have both, connection between the VMs on our network and connection  
to the internet. We change the VM settings of the client and server to *Bridged Mode* and their network 
interfaces from *static* to *dhcp*.   
 
Then, on the server type `ifconfig`, which will show us our IP address: 
`inet 192.168.1.8` 
 
Back on our client, we open the terminal and SHH into the server using this IP address: 
`ssh rubyadmin@192.18.1.8`







[1]: https://www.youtube.com/watch?v=MPosff0AJGM&feature=youtu.be
[2]: http://www.nano-editor.org/dist/v2.2/nano.html
[3]: https://help.ubuntu.com/community/SSH/OpenSSH/Keys
