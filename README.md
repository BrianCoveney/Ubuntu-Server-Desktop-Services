# Internet & Network Services

## Ubuntu Server & Desktop Services

### Report by: Brian Coveney 

### YouTube Demo [Video][1]  

---

##Table of Contents
1. OpenSSH Server
1. <a href="#20-ruby-on-rails">Ruby On Rails</a>

---


### 1. OpenSSH Server:

---
#### 1.1 Installation:
`sudo apt-get install openssh-server`

We will edit the configuration file, but first we will make a copy of it and secure that copy:

`sudo cp /etc/ssh/sshd_config /etc/ssh/sshd_config.origina` <br>
`sudo chmod a-w /etc/ssh/sshd_config.original`

---
#### 1.2 Disable Password authentication
---

Next we will disable password authentication for logging into the system. This is because password authentication is not secure enough -- it can be too easily hacked, by methods such as a brute force attack

Open the `shhd_config` file in the [nano][2] text editor:

`sudo nano /etc/ssh/sshd_config`

Disabling password authentication is done by changing the following line:

`PasswordAuthentication yes` <- from this <br> 
`PasswordAuthentication no`  <- to this


For this to alteration to take effect, we must restart SSH: 

`sudo /etc/init.d/ssh restart`

---
#### 1.3 Key-based authentication
---

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


---
#### 1.4 Troubleshooting SSH
---

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

---
#### 1.5 SSH with Bridged Mode
---

So far we have only been able to communicate between the two machines by configuring the VMware NIC settings to Host only. Also having the client and server set up with *static* IP addresses. We then have no internet connection. For us to get connection, so we can pull down packages, we need to configure the VMware settings back to NAT and the network interfaces to *dhcp*. Thus, losing the connection between client and host.      
To configure our environment so that we have both, connection between the VMs on our network and connection  
to the internet. We change the VM settings of the client and server to *Bridged Mode* and their network 
interfaces from *static* to *dhcp*.   
 
Then, on the server type `ifconfig`, which will show us our IP address: 
`inet 192.168.1.8` 
 
Back on our client, we open the terminal and SHH into the server using this IP address: 
`ssh rubyadmin@192.18.1.8`

---

### 2.0 Ruby On Rails 

---
#### 2.1 Ruby with Rbenv and Bundler:
---

Now that we have access on the client to the server and the internet, the following steps will be performed on the client. 
 
Install Ruby dependencies:

```
    sudo apt-get update 
    sudo apt-get install git-core curl zlib1g-dev build-essential libssl-dev libreadline-dev libyaml-dev libsqlite3-dev sqlite3 libxml2-dev libxslt1-dev libcurl4-openssl-dev python-software-properties libffi-de
```

Next, we will install Ruby using rbenv – Ruby version management tool. The following lines will clone the correct Ruby version from GitHub and set up the environment variables: 

```
    git clone https://github.com/rbenv/rbenv.git ~/.rbenv 
    echo 'export PATH="$HOME/.rbenv/bin:$PATH"' >> ~/.bashrc 
    echo 'eval "$(rbenv init -)"' >> ~/.bashrc 
    exec $SHELL 
    git clone https://github.com/rbenv/ruby-build.git ~/.rbenv/plugins/ruby-build 
    echo 'export PATH="$HOME/.rbenv/plugins/ruby-build/bin:$PATH"' >> ~/.bashrc 
    exec $SHELL 
    git clone https://github.com/rbenv/rbenv-gem-rehash.git ~/.rbenv/plugins/rbenv-gem-rehash 
```


```
    rbenv install 2.3.0 
    rbenv global 2.3.0 
    ruby -v
```

Then we need to get Bundler, so we can install our project dependencies: 
```
    gem install bundler 
    rbenv rehash
```


---
#### 2.2 Nginx and Passenger:
---
Next, we will set up our production server. We will pull down an official Ubuntu package that includes Nginx and Phusion Passenger:

```
    sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv-keys 561F9B9CAC40B2F7 
    sudo apt-get install -y apt-transport-https ca-certificates 
     
    sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv-keys 561F9B9CAC40B2F7 
    sudo apt-get install -y apt-transport-https ca-certificates 
     
    # Add Passenger APT repository 
    sudo sh -c 'echo deb https://oss-binaries.phusionpassenger.com/apt/passenger trusty main > /etc/apt/sources.list.d/passenger.list' 
    sudo apt-get update 
     
    # Install Passenger & Nginx 
    sudo apt-get install -y nginx-extras passenge

```

On my server, I already have *Drupal installed and listening on port 80*, so when I go to 192.168.1.7, is displays the Drual home page  
 
*I configured Nginx to listen on port 81* 
 
`sudo nano /etc/nginx/sites-enabled/default`


```
Server{ 
    Listen: 81 
     
sudo service nginx restart
```


Now that Nginx and Passenger is installed, we can manage the Nginx webserver by typing: 
`sudo service nginx start ` 
 
Make sure that `Nginx is running` by typing into the browser - **the servers IP address**, followed by the port number:  
```
    [your_ipaddress]:[port_no]
    192.168.1.7:81
```

Next, we need to make alterations to Nginx configuration file, so that it points Passenger to the version of Ruby we are using: 
`sudo nano /etc/nginx/nginx.conf`


Find the following lines and uncomment them: 
```
 passenger_root /usr/lib/ruby/vendor_ruby/phusion_passenger/locations.ini; 
 passenger_ruby /home/deploy/.rbenv/shims/ruby; 
```
 
Now we will need to restart Nginx with the new Passenger configuration: 

`sudo service nginx restart`


---
#### 2.3 Apache:
---

I started with Nginx, because, to be honest, I was following the steps in a tutorial. I realize that the assignment  
specification, requires Apache, therefore I am adding it in here. 
 
Apache vs Nginx or Both Side by Side: 
 
Apache and Nginx are both webservers. After some research I found out that Apache is more widely used and has  
more features. While Nginx benefits from being less resource dependent, thus it servers content more efficiently.  
It is also possible to run Apache and Nginx together, where Nginx shines by serving static content very fast,  
allowing Apache to concentrate on dynamic requests.   
 
Apache Install & Configuration:  
 
Install Apache2: 

`sudo apt-get install apache2 ` 
 
Copy the file 000-default.conf to a configuration file with our domain name: 

`sudo cp /etc/apache2/sites-available/000-default.conf /etc/apache2/sites-available/khufunet.com.conf `
 
Change its privileges to everyone can read write and execute:  

`sudo chmod 777 /etc/apache2/sites-available/khufunet.com.conf `
 
Edit this file, replacing all instance of example.com with our domain name 
*khufunet.com*

`sudo nano /etc/apache2/sites-available/khufunet.com.conf `
 
Make a directory for our site: 

`sudo mkdir /var/www/html/khufunet `
 
Enable the site and restart Apache: 

`sudo a2ensite /etc/apache2/sites-available/khufunet.com.conf 
sudo service apache2 restart`

---
#### 2.4 PostgreSQL:
---

For my first assignment I used MySQL, this time I will be using PostgreSQL. The following lines will pull down and install Postgres: 
 
    sudo sh -c "echo 'deb http://apt.postgresql.org/pub/repos/apt/ precise-pgdg main' > /etc/apt/sources.list.d/pgdg.list" 
    wget --quiet -O - http://apt.postgresql.org/pub/repos/apt/ACCC4CF8.asc | sudo apt-key add - 
    sudo apt-get update 
    sudo apt-get install postgresql-common 
    sudo apt-get install postgresql-9.5 libpq-dev 

 
Next we will need to create a user: 

`sudo –u postgres createuser rubyadmin –s `
 
Move into the application directory: 

`cd myapp` 

(Note that you will need to ‘cd’ into this directory each time you fire up the terminal) 
 
Then edit the `config/database.yml` to include the user name and password you set up when installing Postgres:

`sudo nano config/database.yml` 
 
Then create the database: `rake db:create` 

Start the rails server: `bin/rails server –b 0.0.0.0` 

We can now launch our browser and type: `192.168.1.7:3000`

---
### 3.0 Change the colour of the bash prompt
---

To distinguish the client from the server, we can customize the color of the prompts:

`nano ~/.bashrc` 
 
un-comment this line: 
`#force_color_prompt=yes `

Here is a [color chart][4]

Note the numbers on the lhs, e.g. 1;32m is green

---
### 4.0 DNS Setup

---
#### 4.1 Set the Hostname on the Name Server
---

Before we configure our nameserver, we ensure that our hostname is configured correctly on our DNS server. We configure this so that it identifies the hostname and the FQDN (fully qualified domain name). 
Open the `/etc/hosts` file: 

`sudo nano /etc/hosts` 
 
Modify the second line so it points our Servers IP address, to the host and domain combination. We will also add an alias at the end. This is what the second line should be:  

`192.168.1.12    ns1.khufunet.com    ns1 `
 
We also add a record of our unqualified hostname ns1 to /etc/hostname  

`sudo nano /etc/hostname` 
<br>`ns1`

We then need to read the value into the currently running system: 

`sudo hostname –F /etc/hostname`


---
#### 4.2 Install Bind on the Name Server
---

Next we will install Bind, which is available within Ubuntu default repositories. We just need to update our local package index. The install will also include the documentation and common utilities: 

`sudo apt-get update 

sudo apt-get install bind9 bind9utils bind9-doc `


---
#### 4.3 Adding a Zone File to the DNS Server
---

Now we will add entries to the zone file. All zone files start with db. Here, we will look at where the zone file maps domain names to IP addresses. I am using khufunet.com as the domain name. 

In my environment, I have  
*ns1.khufunet.com which maps to 192.168.1.12 (address of the nameserver) <br>
www.khufunet.com which maps to 192.168.1.14 (address of the webserver) <br>
gw.khufunet.comwhich maps to 192.168.1.1 (address of the gateway)*
 
 
Next we will create our own zone file. To do this we will copy the <br>`/etc/bind/db.local to /etc/bind/db.khufunet.com 
cp /etc/bind/db.local /etc/bind/db.khufunet.com `
 
Then open the file: 

`sudo nano /etc/bind/db.khufunet.com`


First we will edit the SOA (start of authority) record. This begins with the first @ symbol and ends at the closing parenthesis. 
 
We replace `localhost`. with the FQDN of this machine `ns1.khufunet.com`.  
On the same line, we replace `root.localhost`. with the email of the domains administer `admin.khufunet.com`. 
 
`@INAns1.khufunet.com   admin.khufunet.com. (` 
 
Next, we will look at the serial number. This needs to be revised each time an alteration is made to the file. Common practice is to use this date format: YYYYMMDD followed by the revision number 
 
`@INAns1.khufunet.com.   admin.khufunet.com. ( ` <br>
`2016042701   ;   Serial `
 
To map the nameserver to the IP address of the server, we delete that last three lines in the file and add: 
 
`ns1IN A192.168.1.12` <br>
`ns1INA 192.168.0.200` <= This is our slave DNS server. 
 
Now we enter additional A (address records). We will point requests for the general domain (in this case khufunet.com) to the webserver: 
 

`@INA192.168.1.14 `

`wwwINA192.168.1.14 ` 

`gwIN A192.168.1.1` 

Then we will add our mailserver. We need to add MX 10 mail on the line below the SOA’s closing parenthesis. If we had a second mail server the number (in our case 10), would signify it’s priority. The lower the number, the higher the priority.  
 
To map the mailserver to its IP address, add the following to the bottom of the file: 

`mail IN A192.168.0.202`

 
Now that the zone file has been created, we need to add a reference to it in named.conf.default.zones. 

`sudo nano /etc/bind/named.conf.default.zones `
 
To do this we will configure the forward zone of our khufunet.com domain, by pointing Bind, to the zone file 

`sudo nano /etc/bind/db.khufunet.com.` Add the following to the bottom of the file: 
 

Zone khufunet.com { <br>
type master; <br>
file /etc/bind/db.khufunet.com; <br>
}; 
 
We also want this zone to be transferred to our slaveserver, if our nameserver goes down.  
 
Zone khufunet.com { <br>
type master; <br>
file /etc/bind/db.khufunet.com;<br> 
allow-transfer {192.168.0.200;}; <br>
}; 
 
The final step is to reload Bind: 

`sudo service bind9 reload` 
 
We should now be able to query the nameserver, using dig and nslookup.
 
 
---
### 5.0 Print Server Setup 
--- 
Ubuntu’s primary printing system is the Common UNIX Printing System (CUPS).  
CUPS manages print batch jobs and provides network printing using the standard Internet Printing Protocol (IPP), while supporting a large range of printers. CUPS also includes a simple we-bases configuration and administration tool.  
 
This line will install CUPS on our server: 

`sudo apt-get install cups `
 
Next we will make some configuration changes. First we will make a copy of the etc/cups/cupsd.conf file, which will serve as a reference. We, will also lock down the file by protecting it from writing: 

`sudo cp /etc/cups/cupsd.conf /etc/cups/cupsd.conf.original` 

`sudo chmod a-w /etc/cups/cupsd.conf.original` 
 
Now we configure the `/etc/cups/cupsd.conf` file: 

`sudo nano /etc//etc/cups/cupsd.conf `
 
We need to add a listening directive to the CUPS server. This is done by adding our IP address and the *Port No. 631*. Then we will have the CUPS server accessible to all machines on our network. 
 
`Listen 127.0.0.1:631`           # existing loopback Listen <br>
`Listen /var/run/cups/cups.sock` # existing socket Listen <br>
`Listen 192.168.1.12:631`      # Listen on the LAN interface, Port 631 (IPP) 
 
Lastly, we need to restart CUPS for our changes to take effect: 

`sudo service cups restart`

Note: The web interface we mentioned earlier, resides at http:/localhost:631/admin


---
### 6.0 Security  
--- 
####6.1 Sudo
---

Sudo is a program that allows users root access to a machine. It originally stood for “superuser do”, but as  
the time progressed, it also added support for running commands as other (restricted) users. Now it is commonly  
known as “substitute user do”.  
 
We do not want all users in an origination full root access. This is done by disabling the use of sudo for superuser  
privileges. Implementing the security *“principal of least privilege”*, we will only grant root access to specific  
areas which a user or group is responsible for.  
 
For example, a new employee, Brian, has been give the responsibility of looking after only the CUPS server. 
He will only be able to use cupsctl (to reload CUPS after alterations). 
 
First we add a new user: 

`sudo adduser peter `
 
Open the `/etc/sudoers` file with sudo visudo. The use of visudo will ensure that there are no errors in our  
sudoers file, which could potentially lock us out.  

`sudo visudo  `

Below the line root  `ALL=(ALL:ALL) ALL`, we add our new user and restrict him access to CUPS, by typing: 

`peterALL=(ALL) NOPASSWD: /usr/sbin/service cups start, /usr/sbin/service cups stop, /usr/sbin/service cups restart,` 
 
Use the command *fg* to regain access to Visudo, if you get the message /etc/sudoers busy try again later 
 
Then, close the SSH connection to our server and reconnect by typing: 

`ssh peter@192.168.1.12 `
 
Lastly, we will test that the change has taken effect. 

First try to `sudo apt-get update` which should fail. 

Then attempt to restart cups with `sudo service cups restart`, which should pass.


As the sudoers file could potentially get very large and difficult to administer, there is the option of making life  
easier by using **Sudo Alias**. This allows us to assign specific privileges to groups of users, instead of assigning  
privileges to each user one by one.   
 
Under the line # User alias specification add the following line, to add users to our User_Alias group: 

`User_Alias WEB_ADMIN = mary,david` 
 
Under the line # Cmnd alias specification add the following lines (note ‘\’ allows us a new line): 

Cmnd_Alias WEB_COMMANDS = /usr/sbin/apache2ctl, \ <br>
/usr/sbin/a2enmod, \ <br>
/usr/sbin/a2dismod, \ <br>
/usr/sbin/a2ensite, \ <br>
/usr/sbin/apache2 start, \ <br>
/usr/sbin/apache2 stop, \ <br>
/usr/sbin/apache2 restart <br>
 
Under the line # User privilege specification add the following line: <br>
`WEB_ADMIN ALL=(ALL) WEBCOMMANDS` 

*Any host on the network:  All= as any target users:  (ALL) *

---
#### 6.2 Rootkit Scanner & Security Auditing Tool 
---

As servers are generally connected to the internet, they are at risk of attacks and scans. We will install two tools  
which will allow us to check if an attacker got in. With these tools, we can then scan our system for viruses,  
malware and rootkits. In a production server, they would be run every night as a cron  job and the results emailed  
to the administrator.  
 
First we will need to be logged in as root: 

`sudo su` 
 
Next, we will install the scanner - Chkrootkit: 

`apt-get install chkrootkit` 
 
Then check our server, by typing: 

`sudo chkrootkit`

---

Next, we will install Lynis. This is a security auditing tool, which scans through our system, checking for vulnerabilities in our configuration.  


    cd /tmp 
    wget https://cisofy.com/files/lynis-2.1.1.tar.gz 
    tar xvfz lynis-2.1.1.tar.gz 
    mv lynis /usr/local/ 
    ln -s /usr/local/lynis/lynis /usr/local/bin/lynis 

 
Then, we will run Lynis: 

`lynis audit system `
 
Lynis will scan our system and display the results.

Lynis will have saved a report into /var/log/lynis.log 
 
To display the Lynis ‘Suggestions’, type:    

`sudo grep Suggestions /var/log/lynis.log `  


---
#### 6.3 Firewall  
---
 
Lastly we will configure and enable the Uncomplicated Firewall (ufw) which is available by default. This provides  
an ‘uncomplicated’ frontend for iptables. It allows users set up a firewall without knowing complicated iptables  
commands. 
 
Next we will write a cron job that disables ufw every 15min. This will ensure that we are not accidentally locked out, e.g. by not having a SSH rule in place.  
 
Basic Usage – allow ssh access, enable logging, enable firewall and check the status: 
 
    sudo ufw allow ssh / tcp  
    sudo ufw logging on 
    sudo ufw enable 
    sudo ufw status  
 
Here are some common firewall rules that I have configured: 
 
    sudo ufw allow domain <= DNS 
    sudo ufw allow www<= Web 
    sudo ufw allow mysql<= MySQL 
    sudo ufw allow 5432<= Postgres 
    sudo ufw allow 631<= Cups


---
### 7. Final Report - Summary and Conclusions: 
---
 
This was a large project that Terrence and myself were tasked with, as part of the CIT module – Internet & 
Network Services. The project was broken down into four main sections each. I focused on getting each section,  
or service, up and running, before moving on to the next one. These were the four services I was responsible for: 

1. Ruby on Rails on an Apache Web Server 
1. DNS Server (Master)  
1. SSH Server  
1. Network Printing (CUPS) 

I started by setting up Ruby on Rails by following this tutorial by Chris Oliver. Initially, it seems to be a very good  
tutorial with the author not only explaining what steps to take but the reasons why such steps are a good idea.  
I then noticed, after installing Ruby and Nginx, that this tutorial might leading me in a different direction than  
what was laid out in the assignment specification.  
 
I went on the search again for a more suitable tutorial and funnily enough I found one written by the same author.   
I finished settings things up by installing PostgreSQL. I then created the Rails app, but ran into problems with 
the final step -- starting the rails server, with the command rails server. I found out that the tutorial is now out  
of date as Ruby on Rails 4.2 release notes, state that when using a virtual machine, please start the server with  
(minus symbol)b 0.0.0.0. Using this, I was able to get Ruby on Rails up and running. I enjoyed setting Ruby up, as with each  
service, if I got stuck at some point -- I was able to find a solution online.  
 
Next up was installing SSH. Everything seemed to work fine, but I then noticed, I was still being prompted for a  
Password, each time I was SSHing in from the client to server. Again, I found online what I need to do to fix this  -- copy the public key into a new directory and point the ssh_config file at that directory.  
While SSHing into the server is obviously the ideal way to connect remotely with servers, having my VMs set to  
host only, with static IPs addresses - access to the internet was not possible. I could not find a  
solution online, so I started tinkering with settings. Happily, I noticed when I set my VMs to bridged mode and  
the machines IPs to dhcp, I could get this functionality.   
 
Cups wat the easiest service to set up, I did not find any issues with this.  
 
Finally, there were no headaches configuring the Uncomplicated Firewall. Ive read that this is by design  
-- to allow those unfamiliar with complicated iptables commands, easier access to control of the firewall. Also,  
setting up Chkrootkit and Lynis was very interesting. I especially liked how Lynis, which runs on the host, runs  
through our system checking for security vulnerabilities.



[1]: https://www.youtube.com/watch?v=MPosff0AJGM&feature=youtu.be
[2]: http://www.nano-editor.org/dist/v2.2/nano.html
[3]: https://help.ubuntu.com/community/SSH/OpenSSH/Keys
[4]: http://abload.de/img/bash-color-chartmxjbp.png
