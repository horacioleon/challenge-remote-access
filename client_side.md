# Client side

## Considerations
1. The authentication over the ssh server use a RSA key instead of password to increases the security.
2. Each client need their own user, password and key to access to the server-A. 
3. This rsa-key is not encrypted by password, but will be save on the /root/.ssh folder
4. The tunnel ssh is started as a service
5. Client A has root access to the machine, but is not an advanced user.
6. Client A can reach and have access to the machine via ssh
6. All users on client-a machine can access to the web application on server-b
7. All tools and applications used on client-a and server-a come by default with CentOS 7 and this was the main reason to choose a ssh as a proxy http.

## Configuration
### Access to the machine client-a

Check if you have access to the client-a machine using any ssh client

* [IP Address](https://en.wikipedia.org/wiki/IP_address) or [FQDN](https://en.wikipedia.org/wiki/Fully_qualified_domain_name) of client-a machine.
	* ex: 192.168.0.204 or client-a.local.com
* Root password
* ssh client
	- Mac
		- Launch the Terminal application, Terminal is found in /Applications/Utilities/ 		directory but you can also launch it from Spotlight by hitting Command+Spacebar and 		typing “Terminal” and then ENTER.
		
			Execute the following commando on terminal
		
			``ssh root@client-a.local.com``
			
			or
			
			``ssh root@192.168.0.204`
		
			Accept the key fingerprint writing "yes" and press ENTER, then put your password.
		
			````
			[user@localhost ~]$ ssh root@client-a
			The authenticity of host 'client-a (::1)' can't be established.
			ECDSA key fingerprint is SHA256:V5ZR5hyi/Enl69+0uqqH6rSA0cWHY3Gimkt8sP4WYAY.
			ECDSA key fingerprint is MD5:6b:03:1b:a5:43:1c:73:f9:90:0c:d7:a7:b1:12:e6:b2.
			Are you sure you want to continue connecting (yes/no)? yes
			Warning: Permanently added 'localhost' (ECDSA) to the list of known hosts.
			root@client-a's password: 
			Last login: Sat Jan 19 01:29:41 2019
			[root@client-a ~]# 
			````
		
	- Linux
		- 	Open your preferred terminal emulator and execute the following command.
		
			``ssh root@client-a.local.com``
			
			or
			
			``ssh root@192.168.0.204`
		
			Accept the key fingerprint writing "yes" and press ENTER, then put your password.
		
			````
			[user@localhost ~]$ ssh root@client-a
			The authenticity of host 'client-a (::1)' can't be established.
			ECDSA key fingerprint is SHA256:V5ZR5hyi/Enl69+0uqqH6rSA0cWHY3Gimkt8sP4WYAY.
			ECDSA key fingerprint is MD5:6b:03:1b:a5:43:1c:73:f9:90:0c:d7:a7:b1:12:e6:b2.
			Are you sure you want to continue connecting (yes/no)? yes
			Warning: Permanently added 'localhost' (ECDSA) to the list of known hosts.
			root@client-a's password: 
			Last login: Sat Jan 19 01:29:41 2019
			[root@client-a ~]#
			```` 

		
	- Windows
		- Microsoft Windows has no native ssh client, but you can install putty to access to servers that only accept ssh connections.
		
			To download go to the putty web ([here](https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html)), chose the "MSI (‘Windows Installer’)" according to the architecture of your machine, if your don't know if your machine is 32bit or 64bit, choose 32bit.
			Putty has classic installer, Next , Next , Next and finish. When the process finish, you can find putty on your program list.
			
			Open putty, you can see at the session category on the right side, few blank text box, on text box "Host Name (or IP Address), put the client-a ip or FQDN server name.
			
			

If you can access to the machine, contact your local support to help you, without access you can't continue with the procedure.

### Create RSA keys
Create on /root/.ssh/ a RSA key named "client-a" to be use on ssh login (use root user)

command to use: ``ssh-keygen -t rsa -b 4096``

ssh-keygen is an app to create encrypted keys, in this case we going to create a RSA key (-t rsa) with a length of 4096 bits (-b 4096).

If the execution of this command is successfully, we will get a rsa key pair, one private (client-a), used to configured the service and one public key (client-a.pub), this public key has to be loaded on the server-b (see on the followings steps).

This command ask to you the path to save the key, please introduce /root/.ssh/client-a, after that they ask you for a passphrase, please don't introduce any passphrase, just press ENTER. If everything is ok you can see on your terminal the fingerprint and randomart of your key.

```
[root@client-a ~]# ssh-keygen -t rsa -b 4096
Generating public/private rsa key pair.
Enter file in which to save the key (/root/.ssh/id_rsa): /root/.ssh/client-a
Created directory '/root/.ssh'.
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in /root/.ssh/client-a.
Your public key has been saved in /root/.ssh/client-a.pub.
The key fingerprint is:
SHA256:C7uAVnj9DI2nEGWSmhheZ9DBGlgdWuNK2tskIerCQX8 root@client-a.local
The key's randomart image is:
+---[RSA 4096]----+
|  oo**=          |
|.. o=X.          |
|.=.**.           |
|+.Oooo o         |
|.o.=+E= S        |
|o .+*. O .       |
|.oo...o +        |
|..   . .         |
|      .          |
+----[SHA256]-----+
```

#### Know errors

1. If for any reason the ssh-keygen command is not present on the machine (the app come by default with CentOS 7 minimal) you can install using yum.

	command to use: ``yum install openssh openssh-clients``

### Copy public key to Server A

To copy your public key to the remote server A, you have to run the following command:

``ssh-copy-id -i /root/.ssh/client-a client-a@server-a``

This command ask you at the beginning if you accept the key fingerprint, write "yes" and press ENTER, then you have to insert your password.

If everything is ok, you can see at the end of the command execution a message like 

```
Now try logging into the machine, with:   "ssh 'client-a@server-a'"
and check to make sure that only the key(s) you wanted were added.
```
Now your public rsa key is saved on the server-a and you can login without password.


````
[root@client-a ~]# ssh-copy-id -i /root/.ssh/client-a client-a@server-a
/usr/bin/ssh-copy-id: INFO: Source of key(s) to be installed: "/root/.ssh/client-a.pub"
The authenticity of host 'server-a (192.168.0.204)' can't be established.
ECDSA key fingerprint is SHA256:V5ZR5hyi/Enl69+0uqqH6rSA0cWHY3Gimkt8sP4WYAY.
ECDSA key fingerprint is MD5:6b:03:1b:a5:43:1c:73:f9:90:0c:d7:a7:b1:12:e6:b2.
Are you sure you want to continue connecting (yes/no)? yes
/usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
/usr/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys
client-a@server-a's password: 

Number of key(s) added: 1

Now try logging into the machine, with:   "ssh 'client-a@server-a'"
and check to make sure that only the key(s) you wanted were added.
````
#### Test your access

````
[root@client-a ~]# ssh -i /root/.ssh/client-a client-a@server-a
[client-a@server-a ~]$ exit
logout
Connection to server-a closed.
[root@client-a ~]# 
````

### proxy_ssh Service

#### Create configuration file for proxy_ssh@server-a
Create a file proxy_ssh@server-a on /etc/default/ path. Change to the the directory using the cd command.

`` cd /etc/default ``

Now you have to create the file proxy_ssh@server-a and add the followings values:


````
LOCAL_ADDR=localhost
LOCAL_PORT=8000
REMOTE_PORT=18000
RSA_KEY=/root/.ssh/client-a
REMOTE_ADDR=server-a
CLIENT_USER=client-a
````

To create and edit the file we going to use vi (a simple tex editor). To open vi and start to work on our proxy_ssh@server-a file, run the following command.

`` vi proxy_ssh@server-a``

You can see now a empty file and in the bottom the full path and a tag [New File].
 
``"/etc/default/proxy_ssh@server-a" [New File]``

If you don't see this things maybe you are on the wrong path or file, please press ESC key, then ":" (without the "), now you can give orders to vi, we have to quit the editor, now write "q!" (without the ") and after ENTER. Check the path with pwd command, then check the name of the file.

````
[root@client-a default]# pwd
/etc/default
````

For some people vi is a little bit complicated to use, is you have problems you can use the command cat instead of vi.

``cat > /etc/default/proxy_ssh@server-a``

After run the command, you will see the cursor of the terminal below the prompt of your terminal, now you can paste the configuration values, after past the values, press ENTER and then ctrl+D to exit.

````
[root@client-a default]# cat > /etc/default/proxy_ssh@server-a
LOCAL_ADDR=localhost
LOCAL_PORT=8000
REMOTE_PORT=8000
RSA_KEY=/root/.ssh/client-a
REMOTE_ADDR=server-a
CLIENT_USER=client-a
````
Now check the content of your configuration file using the command more

``more /etc/default/proxy_ssh@server-a``

````
[root@client-a default]# more /etc/default/proxy_ssh@server-a
LOCAL_ADDR=localhost
LOCAL_PORT=8000
REMOTE_PORT=8001
RSA_KEY=/root/.ssh/client-a
REMOTE_ADDR=server-a
CLIENT_USER=client-a
````

#### Create service proxy_ssh@client-a

``cd /etc/systemd/system ``

`` vi proxy_ssh\@.service``

````
[Unit]
Description=Create a proxy ssh to %i
After=network.target

[Service]
EnvironmentFile=/etc/default/proxy_ssh@%i
ExecStart=/usr/bin/ssh -NT -o ServerAliveInterval=60 -o ExitOnForwardFailure=yes -L ${LOCAL_ADDR}:${LOCAL_PORT}:localhost:${REMOTE_PORT} -i ${RSA_KEY} ${CLIENT_USER}@${REMOTE_ADDR}
StandardOutput=syslog
StandardError=syslog
SyslogIdentifier=proxy_ssh


RestartSec=10
Restart=always

[Install]
WantedBy=default.target
````

#### Enabled service on startup

To enable our service at the startup of the server, we have to run the following command.

``systemctl enable proxy_ssh@server-a.service``


````
[root@client-a ~]# systemctl enable proxy_ssh@server-a.service
Created symlink from /etc/systemd/system/default.target.wants/proxy_ssh@server-a.service to /etc/systemd/system/proxy_ssh@.service.
[root@client-a ~]#
````


#### proxy_ssh service management

##### status

````
[root@client-a ~]# systemctl status proxy_ssh@server-a.service
● proxy_ssh@server-a.service - Create a proxy ssh to server-a
   Loaded: loaded (/etc/systemd/system/proxy_ssh@.service; enabled; vendor preset: disabled)
   Active: active (running) since Sat 2019-01-19 03:25:07 EST; 5min ago
 Main PID: 14844 (ssh)
   CGroup: /system.slice/system-proxy_ssh.slice/proxy_ssh@server-a.service
           └─14844 /usr/bin/ssh -NT -o ServerAliveInterval=60 -o ExitOnForwardFailure=yes -L localhost:8000:localhost:18000 -i /root/.ssh/c...

Jan 19 03:25:07 client-a.local systemd[1]: Started Create a proxy ssh to server-a.
[root@client-a ~]# 


````


##### start

````
[root@client-a ~]# systemctl start proxy_ssh@server-a.service
[root@client-a ~]# systemctl status proxy_ssh@server-a.service
● proxy_ssh@server-a.service - Create a proxy ssh to server-a
   Loaded: loaded (/etc/systemd/system/proxy_ssh@.service; enabled; vendor preset: disabled)
   Active: active (running) since Sat 2019-01-19 03:33:00 EST; 1s ago
 Main PID: 14892 (ssh)
   CGroup: /system.slice/system-proxy_ssh.slice/proxy_ssh@server-a.service
           └─14892 /usr/bin/ssh -NT -o ServerAliveInterval=60 -o ExitOnForwardFailure=yes -L localhost:8000:localhost:18000 -i /root/.ssh/c...

Jan 19 03:33:00 client-a.local systemd[1]: Started Create a proxy ssh to server-a.
[root@client-a ~]# 

````

##### stop

````
[root@client-a ~]# systemctl stop proxy_ssh@server-a.service

[root@client-a ~]# systemctl status proxy_ssh@server-a.service
● proxy_ssh@server-a.service - Create a proxy ssh to server-a
   Loaded: loaded (/etc/systemd/system/proxy_ssh@.service; enabled; vendor preset: disabled)
   Active: inactive (dead)

Jan 19 03:25:07 client-a.local systemd[1]: Started Create a proxy ssh to server-a.
Jan 19 03:31:40 client-a.local systemd[1]: Stopping Create a proxy ssh to server-a...
Jan 19 03:31:40 client-a.local systemd[1]: Stopped Create a proxy ssh to server-a.
Jan 19 03:31:44 client-a.local systemd[1]: Started Create a proxy ssh to server-a.
Jan 19 03:32:37 client-a.local systemd[1]: Stopping Create a proxy ssh to server-a...
Jan 19 03:32:37 client-a.local systemd[1]: Stopped Create a proxy ssh to server-a.
[root@client-a ~]#
````

##### restart

````
[root@client-a ~]# systemctl restart proxy_ssh@server-a.service
[root@client-a ~]# systemctl status proxy_ssh@server-a.service
● proxy_ssh@server-a.service - Create a proxy ssh to server-a
   Loaded: loaded (/etc/systemd/system/proxy_ssh@.service; enabled; vendor preset: disabled)
   Active: active (running) since Sat 2019-01-19 03:33:32 EST; 1s ago
 Main PID: 14916 (ssh)
   CGroup: /system.slice/system-proxy_ssh.slice/proxy_ssh@server-a.service
           └─14916 /usr/bin/ssh -NT -o ServerAliveInterval=60 -o ExitOnForwardFailure=yes -L localhost:8000:localhost:18000 -i /root/.ssh/c...

Jan 19 03:33:32 client-a.local systemd[1]: Stopped Create a proxy ssh to server-a.
Jan 19 03:33:32 client-a.local systemd[1]: Started Create a proxy ssh to server-a.
[root@client-a ~]# 
````

#### Check HTTP connection

````
[root@client-a ~]# curl -I localhost:8000
HTTP/1.1 200 OK
Server: nginx/1.12.2
Date: Sat, 19 Jan 2019 08:25:28 GMT
Content-Type: text/html
Content-Length: 3700
Last-Modified: Tue, 06 Mar 2018 09:26:21 GMT
Connection: keep-alive
ETag: "5a9e5ebd-e74"
Accept-Ranges: bytes

[root@client-a ~]# 

````

### Troubleshooting

#### client-a proxy_ssh not started

You try to reach the application, but you get a connections refused from server

````
[root@client-a ~]# curl -v localhost:8000 -o /dev/null
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0* About to connect() to localhost port 8000 (#0)
*   Trying ::1...
* Connection refused
*   Trying 127.0.0.1...
* Connection refused
* Failed connect to localhost:8000; Connection refused
* Closing connection 0
curl: (7) Failed connect to localhost:8000; Connection refused
[root@client-a ~]# 

````

Check the status os the service

````
[root@client-a ~]# systemctl status proxy_ssh@server-a.service
● proxy_ssh@server-a.service - Create a proxy ssh to server-a
   Loaded: loaded (/etc/systemd/system/proxy_ssh@.service; enabled; vendor preset: disabled)
   Active: inactive (dead)
````

If the status is "Active: inactive (dead)", the proxy_ssh@.service is not running for your instance.

Try to start the service.

````
[root@client-a ~]# systemctl start proxy_ssh@server-a.service

````
And check the status again, the next output is the expected

````
[root@client-a ~]# systemctl status proxy_ssh@server-a.service
● proxy_ssh@server-a.service - Create a proxy ssh to server-a
   Loaded: loaded (/etc/systemd/system/proxy_ssh@.service; enabled; vendor preset: disabled)
   Active: active (running) since Sat 2019-01-19 03:40:17 EST; 9s ago
 Main PID: 14945 (ssh)
   CGroup: /system.slice/system-proxy_ssh.slice/proxy_ssh@server-a.service
           └─14945 /usr/bin/ssh -NT -o ServerAliveInterval=60 -o ExitOnForwardFailure=yes -L localhost:8000:localhost:18000 -i /root/.ssh/c...

Jan 19 03:40:17 client-a.local systemd[1]: Started Create a proxy ssh to server-a.
````

Use curl to check the HTTP service

````
[root@client-a ~]# curl -v localhost:8000 -o /dev/null
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0* About to connect() to localhost port 8000 (#0)
*   Trying ::1...
* Connected to localhost (::1) port 8000 (#0)
> GET / HTTP/1.1
> User-Agent: curl/7.29.0
> Host: localhost:8000
> Accept: */*
> 
< HTTP/1.1 200 OK
< Server: nginx/1.12.2
< Date: Sat, 19 Jan 2019 11:01:59 GMT
< Content-Type: text/html
< Content-Length: 3700
< Last-Modified: Tue, 06 Mar 2018 09:26:21 GMT
< Connection: keep-alive
< ETag: "5a9e5ebd-e74"
< Accept-Ranges: bytes
< 
{ [data not shown]
100  3700  100  3700    0     0   168k      0 --:--:-- --:--:-- --:--:--  172k
* Connection #0 to host localhost left intact
[root@client-a ~]# 

````

If the curl output return a 200 HTTP code, you have solve the problem. 

Still have no connection? Maybe the tunnel to server-a is not up

#### Tunnel to server-a is not up, but the proxy_ssh service is running in my machine

Now with the proxy_ssh service up, we have to check if the tunnel to the server-a is up, for this task we going to use netstat as root user. Some installations of CentOS has no installed netstat by default, to install run the following command.

``yum install net-tools -y ``

The service proxy_ssh service can be running ok in your machine, but the tunnel can be down. The proxy_ssh service was configured to restart the session if the tunnel is down, for this reason the service can be up and the tunnel down. Use netstat to check the connections.

``netstat -at | awk '/ssh/ && /SERVER_NAME/'``

``netstat -natup | awk '/ssh/ && /SERVER_IP/``

*** Replace the SERVER_NAME or SERVER_IP for the name/ip of the server-a, if you don't remember, see the file /etc/default/proxy_ssh@server-a

``more /etc/default/proxy_ssh@server-a``

````
[root@client-a ~]# more /etc/default/proxy_ssh@server-a
LOCAL_ADDR=localhost
LOCAL_PORT=8000
REMOTE_PORT=18000
RSA_KEY=/root/.ssh/client-a
REMOTE_ADDR=server-a
CLIENT_USER=client-a

````

````
[root@client-a ~]# netstat -at | awk '/ssh/ && /server-a/'
tcp        0      0 client-a.local:49730    server-a:ssh            ESTABLISHED
[root@client-a ~]#

[root@client-a ~]# netstat -natup | awk '/ssh/ && /192.168.0.204/'
tcp        0      0 192.168.0.205:49730     192.168.0.204:22        ESTABLISHED 15229/ssh           
[root@client-a ~]#
````
If you not see any ESTABLISHED connections, the tunnel to the server-a is not up.

* Possible problems
	- Missing rsa-key file
	- Bad permissions on rsa_key
	- Server-a change and has no configuration for your user

Run the following command to check the tunnel manually

``systemctl status proxy_ssh@server-a.service | grep "/usr/bin/ssh" ``

````
[root@client-a ~]# systemctl status proxy_ssh@server-a.service | grep "/usr/bin/ssh"
           └─15229 /usr/bin/ssh -NT -o ServerAliveInterval=60 -o ExitOnForwardFailure=yes -L localhost:8000:localhost:18000 -i /root/.ssh/client-a client-a@server-a
[root@client-a ~]#

````
Copy the line from ssh to the end.

``ssh -NT -o ServerAliveInterval=60 -o ExitOnForwardFailure=yes -L localhost:8000:localhost:18000 -i /root/.ssh/client-a client-a@server-a``

And run the command manually

````
[root@client-a ~]# ssh -NT -o ServerAliveInterval=60 -o ExitOnForwardFailure=yes -L localhost:8000:localhost:18000 -i /root/.ssh/client-a client-a@server-a
Warning: Identity file /root/.ssh/client-a not accessible: No such file or directory.
client-a@server-a's password: 
Permission denied, please try again.
client-a@server-a's password: 
Permission denied, please try again.
client-a@server-a's password: 
Permission denied (publickey,gssapi-keyex,gssapi-with-mic,password).
[root@client-a ~]#

````

In this case the problem was related do the private rsa key file. To solve this problem recreate the key (ssh-keygen) according to the procedure and re-sync (ssh-copy-id) with your account or recover the key from a backup.

If you recover your key from a backup, maybe the key come with wrong permissions, if you try to open the tunnel manually, you get a different error.


````
[root@client-a ~]# ssh -NT -o ServerAliveInterval=60 -o ExitOnForwardFailure=yes -L localhost:8000:localhost:18000 -i /root/.ssh/client-a client-a@server-a
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
@         WARNING: UNPROTECTED PRIVATE KEY FILE!          @
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
Permissions 0755 for '/root/.ssh/client-a' are too open.
It is required that your private key files are NOT accessible by others.
This private key will be ignored.
Load key "/root/.ssh/client-a": bad permissions
client-a@server-a's password: 
Permission denied, please try again.
client-a@server-a's password: 
Permission denied, please try again.
client-a@server-a's password: 
Permission denied (publickey,gssapi-keyex,gssapi-with-mic,password).
[root@client-a ~]#

````

Lets check the permissions of the private key

````
[root@client-a ~]# ll -h /root/.ssh/client-a
-rwxr-xr-x. 1 root root 3.2K Jan 18 20:43 /root/.ssh/client-a
 
````
The private key can be read and write only for the owner, in this case the root user, to solve, change the permissions with the chmod command.

``chmod 600 /root/.ssh/client-a ``

````
[root@client-a ~]# chmod 600 /root/.ssh/client-a
[root@client-a ~]# ll -h /root/.ssh/client-a
-rw-------. 1 root root 3.2K Jan 18 20:43 /root/.ssh/client-a
[root@client-a ~]# 

````

Now you can restart the service and check the tunnel. After that check the status and tcp connections.

````
[root@client-a ~]# systemctl restart proxy_ssh@server-a.service
[root@client-a ~]# systemctl status proxy_ssh@server-a.service
● proxy_ssh@server-a.service - Create a proxy ssh to server-a
   Loaded: loaded (/etc/systemd/system/proxy_ssh@.service; enabled; vendor preset: disabled)
   Active: active (running) since Sat 2019-01-19 05:51:42 EST; 3s ago
 Main PID: 15521 (ssh)
   CGroup: /system.slice/system-proxy_ssh.slice/proxy_ssh@server-a.service
           └─15521 /usr/bin/ssh -NT -o ServerAliveInterval=60 -o ExitOnForwardFailure=yes -L localhost:8000:localhost:18000 -i /root/.ssh/c...

Jan 19 05:51:42 client-a.local systemd[1]: Stopped Create a proxy ssh to server-a.
Jan 19 05:51:42 client-a.local systemd[1]: Started Create a proxy ssh to server-a.
[root@client-a ~]# netstat -at | awk '/ssh/ && /server-a/' 
tcp        0      0 client-a.local:49762    server-a:ssh            ESTABLISHED
[root@client-a ~]# 

````

Use curl to check the HTTP service.

````
[root@client-a ~]# curl -v localhost:8000 -o /dev/null
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0* About to connect() to localhost port 8000 (#0)
*   Trying ::1...
* Connected to localhost (::1) port 8000 (#0)
> GET / HTTP/1.1
> User-Agent: curl/7.29.0
> Host: localhost:8000
> Accept: */*
> 
< HTTP/1.1 200 OK
< Server: nginx/1.12.2
< Date: Sat, 19 Jan 2019 11:24:50 GMT
< Content-Type: text/html
< Content-Length: 3700
< Last-Modified: Tue, 06 Mar 2018 09:26:21 GMT
< Connection: keep-alive
< ETag: "5a9e5ebd-e74"
< Accept-Ranges: bytes
< 
{ [data not shown]
100  3700  100  3700    0     0   194k      0 --:--:-- --:--:-- --:--:--  200k
* Connection #0 to host localhost left intact
[root@client-a ~]# 

````
If you get a 200 HTTP code, the problem was solved, but if you can't connect with the server. Is time to call for help. Now your sure that the problem is on the server side, please inform to the support team, and share the logs and evidences of your troubleshooting.

````
[root@client-a ~]# curl -v localhost:8000 -o /dev/null 
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0* About to connect() to localhost port 8000 (#0)
*   Trying ::1...
* Connected to localhost (::1) port 8000 (#0)
> GET / HTTP/1.1
> User-Agent: curl/7.29.0
> Host: localhost:8000
> Accept: */*
> 
* Empty reply from server
  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0
* Connection #0 to host localhost left intact
curl: (52) Empty reply from server
[root@client-a ~]# 

````

In both case, server side and tunnel down, you get the same log output on syslog.

````
[root@client-a ~]# systemctl status proxy_ssh@server-a.service
● proxy_ssh@server-a.service - Create a proxy ssh to server-a
   Loaded: loaded (/etc/systemd/system/proxy_ssh@.service; enabled; vendor preset: disabled)
   Active: active (running) since Sat 2019-01-19 06:01:56 EST; 25min ago
 Main PID: 15587 (ssh)
   CGroup: /system.slice/system-proxy_ssh.slice/proxy_ssh@server-a.service
           └─15587 /usr/bin/ssh -NT -o ServerAliveInterval=60 -o ExitOnForwardFailure=yes -L localhost:8000:localhost:18000 -i /root/.ssh/...

Jan 19 06:01:56 client-a.local systemd[1]: Stopped Create a proxy ssh to server-a.
Jan 19 06:01:56 client-a.local systemd[1]: Started Create a proxy ssh to server-a.
Jan 19 06:04:27 client-a.local proxy_ssh[15587]: channel 2: open failed: connect failed: Connection refused
Jan 19 06:11:52 client-a.local proxy_ssh[15587]: channel 2: open failed: connect failed: Connection refused
Jan 19 06:11:54 client-a.local proxy_ssh[15587]: channel 2: open failed: connect failed: Connection refused
Jan 19 06:11:58 client-a.local proxy_ssh[15587]: channel 2: open failed: connect failed: Connection refused
Jan 19 06:12:13 client-a.local proxy_ssh[15587]: channel 2: open failed: connect failed: Connection refused
Jan 19 06:24:37 client-a.local proxy_ssh[15587]: channel 2: open failed: connect failed: Connection refused
Jan 19 06:27:41 client-a.local proxy_ssh[15587]: channel 2: open failed: connect failed: Connection refused
[root@client-a ~]# 
````

For this reason is very important made the troubleshooting according to this procedure to focus on the real problem and resolve quickly.