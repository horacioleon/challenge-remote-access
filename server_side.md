
# Server side

## Considerations
1. The authentication over the ssh server use a RSA key instead of password to increases the security. 
3. This rsa-key is not encrypted by password, but will be save on the /root/.ssh folder
4. The tunnel ssh is started as a service
7. All tools and applications used on client-a and server-a come by default with CentOS 7 and this was the main reason to choose a ssh as a proxy http.

## Diagram


````
 ------------------------------                             -----------------------------
|  server-A                    |                           |  server-b                   |
|  http://localhost:18000      |  ---    ssh tunnel   ---> |  http://localhost:8000      |
|  proxy_ssh service           |    tcp 18000 -> tcp 8000  |  web service                |
|  - instance:                 |                           |                             |                              |  client-a_server-b           |                           |                             |
 ------------------------------                             -----------------------------
````

## Configuration

### Create user for client-a on server-a

````
[root@server-a ~]# useradd client-a
[root@server-a ~]# passwd client-a
Changing password for user client-a.
New password: 
Retype new password: 
passwd: all authentication tokens updated successfully.
[root@server-a ~]# 
````


### Create user for server-a on server-b

````
[root@server-b ~]# useradd client-a
[root@server-b ~]# passwd client-a
Changing password for user client-a.
New password: 
Retype new password: 
passwd: all authentication tokens updated successfully.
[root@server-b ~]# 
````
### Create service user on server-b

### Create RSA keys
Create on /root/.ssh/ a RSA key named "client-a" to be use on ssh login (use root user)

command to use: ``ssh-keygen -t rsa -b 4096``

ssh-keygen is an app to create encrypted keys, in this case we going to create a RSA key (-t rsa) with a length of 4096 bits (-b 4096).

If the execution of this command is successfully, we will get a rsa key pair, one private (server-a), used to configured the service and one public key (server-a.pub), this public key has to be loaded on the server-b (see on the followings steps).

This command ask to you the path to save the key, please introduce /root/.ssh/client-a, after that they ask you for a passphrase, please don't introduce any passphrase, just press ENTER. If everything is ok you can see on your terminal the fingerprint and randomart of your key.

```
[root@server-a ~]# ssh-keygen -t rsa -b 4096
Generating public/private rsa key pair.
Enter file in which to save the key (/root/.ssh/id_rsa): /root/.ssh/client-a
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in /root/.ssh/client-a.
Your public key has been saved in /root/.ssh/client-a.pub.
The key fingerprint is:
SHA256:d6h8NZmv1Zv6ppBvjepwj+J/wuRHVcvAyhag+YB/kRM root@server-a.remoto
The key's randomart image is:
+---[RSA 4096]----+
|         E. .    |
|      . o o. o  .|
|     . + +. o o o|
|      . o o= o + |
|       .So+ * .  |
|       ..o +.+ . |
|        o.=+. = .|
|         oo+=B oo|
|        ..+=B==+ |
+----[SHA256]-----+
[root@server-a ~]# 

```

#### Know errors

1. If for any reason the ssh-keygen command is not present on the machine (the app come by default with CentOS 7 minimal) you can install using yum.

	command to use: ``yum install openssh openssh-clients``

### Copy public key to Server B

To copy your public key to the remote server B, you have to run the following command:

``ssh-copy-id -i /root/.ssh/client-a client-a@server-b``

This command ask you at the beginning if you accept the key fingerprint, write "yes" and press ENTER, then you have to insert your password.

If everything is ok, you can see at the end of the command execution a message like 

```
Now try logging into the machine, with:   "ssh 'client-a@server-b'"
and check to make sure that only the key(s) you wanted were added.
```
Now your public rsa key is saved on the server-b and you can login without password.

````
[root@server-a ~]# ssh-copy-id -i /root/.ssh/client-a client-a@server-b
/usr/bin/ssh-copy-id: INFO: Source of key(s) to be installed: "/root/.ssh/client-a.pub"
/usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
/usr/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys
client-a@server-b's password: 

Number of key(s) added: 1

Now try logging into the machine, with:   "ssh 'client-a@server-b'"
and check to make sure that only the key(s) you wanted were added.

[root@server-a ~]# 
````
#### Test your access

````
[root@server-a ~]# ssh -i /root/.ssh/client-a client-a@server-b
[client-a@server-b ~]$ exit
logout
Connection to server-b closed.
[root@server-a ~]# 
````

### proxy_ssh Service

#### Create configuration file for proxy_ssh@client-a_server-b
Create a file proxy_ssh@sever-b on /etc/default/ path. Change to the the directory using the cd command.

`` cd /etc/default ``
proxy_ssh@client-a_server-b
Now you have to create the file proxy_ssh@client-a_server-b and add the followings values:


````
LOCAL_ADDR=localhost
LOCAL_PORT=18000
REMOTE_PORT=8000
RSA_KEY=/root/.ssh/client-a
REMOTE_ADDR=server-b
CLIENT_USER=client-a
````

To create and edit the file we going to use vi (a simple tex editor). To open vi and start to work on our proxy_ssh@client-a_server-b file, run the following command.

`` vi proxy_ssh@client-a_server-b``

You can see now a empty file and in the bottom the full path and a tag [New File].
 
``"/etc/default/proxy_ssh@client-a_server-b" [New File]``

If you don't see this things maybe you are on the wrong path or file, please press ESC key, then ":" (without the "), now you can give orders to vi, we have to quit the editor, now write "q!" (without the ") and after ENTER. Check the path with pwd command, then check the name of the file.

````
[root@server-a default]# pwd
/etc/default
````

For some people vi is a little bit complicated to use, is you have problems you can use the command cat instead of vi.

``cat > /etc/default/proxy_ssh@client-a_server-b``

After run the command, you will see the cursor of the terminal below the prompt of your terminal, now you can paste the configuration values, after past the values, press ENTER and then ctrl+D to exit.

````
[root@server-a default]# cat > /etc/default/proxy_ssh@client-a_server-b
LOCAL_ADDR=localhost
LOCAL_PORT=18000
REMOTE_PORT=8000
RSA_KEY=/root/.ssh/client-a
REMOTE_ADDR=server-b
CLIENT_USER=client-a
````
Now check the content of your configuration file using the command more

``more /etc/default/proxy_ssh@client-a_server-b``

````
[root@server-a default]# more /etc/default/proxy_ssh@client-a_server-b
LOCAL_ADDR=localhost
LOCAL_PORT=18000
REMOTE_PORT=8000
RSA_KEY=/root/.ssh/client-a
REMOTE_ADDR=server-b
CLIENT_USER=client-a
````

#### Create service proxy_ssh@server-a

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

``systemctl enable proxy_ssh@client-a_server-b.service ``

````
[root@server-a ~]# systemctl enable proxy_ssh@client-a_server-b.service
Created symlink from /etc/systemd/system/default.target.wants/proxy_ssh@client-a_server-b.service to /etc/systemd/system/proxy_ssh@.service.
[root@server-a ~]#
````

#### proxy_ssh service management

##### status

````
[root@server-a ~]# systemctl status proxy_ssh@client-a_server-b.service
● proxy_ssh@client-a_server-b.service - Create a proxy ssh to server-b
   Loaded: loaded (/etc/systemd/system/proxy_ssh@.service; enabled; vendor preset: disabled)
   Active: active (running) since Sat 2019-01-19 03:25:07 EST; 5min ago
 Main PID: 14844 (ssh)
   CGroup: /system.slice/system-proxy_ssh.slice/proxy_ssh@client-a_server-b.service
           └─14844 /usr/bin/ssh -NT -o ServerAliveInterval=60 -o ExitOnForwardFailure=yes -L localhost:18000:localhost:8000 -i /root/.ssh/c...

Jan 19 03:25:07 server-a.remote systemd[1]: Started Create a proxy ssh to server-b.
[root@server-a ~]# 


````


##### start

````
[root@server-a ~]# systemctl start proxy_ssh@client-a_server-b.service
[root@server-a ~]# systemctl status proxy_ssh@client-a_server-b.service
● proxy_ssh@client-a_server-b.service - Create a proxy ssh to server-b
   Loaded: loaded (/etc/systemd/system/proxy_ssh@.service; enabled; vendor preset: disabled)
   Active: active (running) since Mon 2019-01-21 04:23:08 EST; 6s ago
 Main PID: 4094 (ssh)
   CGroup: /system.slice/system-proxy_ssh.slice/proxy_ssh@client-a_server-b.service
           └─4094 /usr/bin/ssh -NT -o ServerAliveInterval=60 -o ExitOnForwardFailure=yes -L localhost:18000:localhost:8000 -i /root/.ssh/s...

Jan 21 04:23:08 server-a.remoto systemd[1]: Started Create a proxy ssh to server-b.
[root@server-a ~]# 
````

##### stop

````
[root@server-a ~]# systemctl stop proxy_ssh@client-a_server-b.service

[root@server-a ~]# systemctl status proxy_ssh@client-a_server-b.service
● proxy_ssh@client-a_server-b.service - Create a proxy ssh to server-b
   Loaded: loaded (/etc/systemd/system/proxy_ssh@.service; enabled; vendor preset: disabled)
   Active: inactive (dead) since Mon 2019-01-21 04:23:38 EST; 1min 6s ago
  Process: 4094 ExecStart=/usr/bin/ssh -NT -o ServerAliveInterval=60 -o ExitOnForwardFailure=yes -L ${LOCAL_ADDR}:${LOCAL_PORT}:localhost:${REMOTE_PORT} -i ${RSA_KEY} ${CLIENT_USER}@${REMOTE_ADDR} (code=exited, status=0/SUCCESS)
 Main PID: 4094 (code=exited, status=0/SUCCESS)

Jan 21 04:23:08 server-a.remoto systemd[1]: Started Create a proxy ssh to server-b.
Jan 21 04:23:38 server-a.remoto systemd[1]: Stopping Create a proxy ssh to server-b...
Jan 21 04:23:38 server-a.remoto systemd[1]: Stopped Create a proxy ssh to server-b.
[root@server-a ~]# 
````

##### restart

````
[root@server-a ~]# systemctl restart proxy_ssh@client-a_server-b.service
[root@server-a ~]# systemctl status proxy_ssh@client-a_server-b.service
● proxy_ssh@client-a_server-b.service - Create a proxy ssh to server-b
   Loaded: loaded (/etc/systemd/system/proxy_ssh@.service; enabled; vendor preset: disabled)
   Active: active (running) since Mon 2019-01-21 04:25:34 EST; 1s ago
 Main PID: 4134 (ssh)
   CGroup: /system.slice/system-proxy_ssh.slice/proxy_ssh@client-a_server-b.service
           └─4134 /usr/bin/ssh -NT -o ServerAliveInterval=60 -o ExitOnForwardFailure=yes -L localhost:18000:localhost:8000 -i /root/.ssh/s...

Jan 21 04:25:34 server-a.remoto systemd[1]: Stopped Create a proxy ssh to server-b.
Jan 21 04:25:34 server-a.remoto systemd[1]: Started Create a proxy ssh to server-b.
[root@server-a ~]# 
````

#### Check HTTP connection

````
[root@server-a ~]# curl -I localhost:18000
HTTP/1.1 200 OK
Server: nginx/1.12.2
Date: Mon, 21 Jan 2019 09:26:04 GMT
Content-Type: text/html
Content-Length: 3700
Last-Modified: Tue, 06 Mar 2018 09:26:21 GMT
Connection: keep-alive
ETag: "5a9e5ebd-e74"
Accept-Ranges: bytes

[root@server-a ~]# 

````

### Troubleshooting

#### server-a proxy_ssh not started

You try to reach the application, but you get a connections refused from server

````
[root@server-a ~]# curl -v localhost:18000 -o /dev/null
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0* About to connect() to localhost port 18000 (#0)
*   Trying ::1...
* Connection refused
*   Trying 127.0.0.1...
* Connection refused
* Failed connect to localhost:18000; Connection refused
* Closing connection 0
curl: (7) Failed connect to localhost:18000; Connection refused
[root@server-a ~]# 

````

Check the status os the service

````
[root@server-a ~]#  systemctl status proxy_ssh@client-a_server-b.service
● proxy_ssh@client-a_server-b.service - Create a proxy ssh to client-a_server-b
   Loaded: loaded (/etc/systemd/system/proxy_ssh@.service; enabled; vendor preset: disabled)
   Active: inactive (dead) since Tue 2019-01-22 01:01:35 EST; 59s ago
  Process: 3283 ExecStart=/usr/bin/ssh -NT -o ServerAliveInterval=60 -o ExitOnForwardFailure=yes -L ${LOCAL_ADDR}:${LOCAL_PORT}:localhost:${REMOTE_PORT} -i ${RSA_KEY} ${CLIENT_USER}@${REMOTE_ADDR} (code=exited, status=0/SUCCESS)
 Main PID: 3283 (code=exited, status=0/SUCCESS)

Jan 21 21:33:59 server-a.remoto systemd[1]: Started Create a proxy ssh to client-a_server-b.
Jan 22 01:00:39 server-a.remoto proxy_ssh[3283]: channel 2: open failed: connect failed: Connection refused
Jan 22 01:01:35 server-a.remoto systemd[1]: Stopping Create a proxy ssh to client-a_server-b...
Jan 22 01:01:35 server-a.remoto systemd[1]: Stopped Create a proxy ssh to client-a_server-b.
[root@server-a ~]# 

````

If the status is "Active: inactive (dead)", the proxy_ssh@.service is not running for your instance.

Try to start the service.

````
[root@server-a ~]#  systemctl start proxy_ssh@client-a_server-b.service

````

And check the status again, the next output is the expected

````
[root@server-a ~]#  systemctl status proxy_ssh@client-a_server-b.service
● proxy_ssh@client-a_server-b.service - Create a proxy ssh to client-a_server-b
   Loaded: loaded (/etc/systemd/system/proxy_ssh@.service; enabled; vendor preset: disabled)
   Active: active (running) since Tue 2019-01-22 01:03:16 EST; 22s ago
 Main PID: 3710 (ssh)
   CGroup: /system.slice/system-proxy_ssh.slice/proxy_ssh@client-a_server-b.service
           └─3710 /usr/bin/ssh -NT -o ServerAliveInterval=60 -o ExitOnForwardFailure=yes -L localhost:18000:localhost:8000 -i /root/.ssh/s...

Jan 22 01:03:16 server-a.remoto systemd[1]: Started Create a proxy ssh to client-a_server-b.
[root@server-a ~]# 
````
Use curl to check the HTTP service

````
[root@server-a ~]# curl -v localhost:18000 -o /dev/null
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0* About to connect() to localhost port 18000 (#0)
*   Trying ::1...
* Connected to localhost (::1) port 18000 (#0)
> GET / HTTP/1.1
> User-Agent: curl/7.29.0
> Host: localhost:18000
> Accept: */*
> 
< HTTP/1.1 200 OK
< Server: nginx/1.12.2
< Date: Tue, 22 Jan 2019 06:07:31 GMT
< Content-Type: text/html
< Content-Length: 3700
< Last-Modified: Tue, 06 Mar 2018 09:26:21 GMT
< Connection: keep-alive
< ETag: "5a9e5ebd-e74"
< Accept-Ranges: bytes
< 
{ [data not shown]
100  3700  100  3700    0     0   248k      0 --:--:-- --:--:-- --:--:--  258k
* Connection #0 to host localhost left intact
[root@server-a ~]# 

````

If the curl output return a 200 HTTP code, you have solve the problem. 

Still have no connection? Maybe the tunnel client_a-server-b is not up.

#### Tunnel client_a-server-b is not up, but the proxy_ssh service is running in my machine


Now with the proxy_ssh service up, we have to check if the tunnel to the server-a is up, for this task we going to use netstat as root user. Some installations of CentOS has no installed netstat by default, to install run the following command.

``yum install net-tools -y ``

The service proxy_ssh service can be running ok in your machine, but the tunnel can be down. The proxy_ssh service was configured to restart the session if the tunnel is down, for this reason the service can be up and the tunnel down. Use netstat to check the connections.

``netstat -at | awk '/ssh/ && /SERVER_NAME/'``

``netstat -natup | awk '/ssh/ && /SERVER_IP/``

*** Replace the SERVER_NAME or SERVER_IP for the name/ip of the server-a, if you don't remember, see the file /etc/default/proxy_ssh\@client-a_server-b

``more /etc/default/proxy_ssh\@client-a_server-b``

````
[root@server-a ~]# more /etc/default/proxy_ssh\@client-a_server-b
LOCAL_ADDR=localhost
LOCAL_PORT=18000
REMOTE_PORT=8000
RSA_KEY=/root/.ssh/client-a
REMOTE_ADDR=server-b
CLIENT_USER=client-a
[root@server-a ~]# 

````


````
[root@server-a ~]# netstat -at | awk '/ssh/ && /server-b/'
tcp        0      0 server-a.remoto:39190   server-b:ssh            ESTABLISHED
[root@server-a ~]#

[root@server-a ~]# netstat -natup | awk '/ssh/ && /192.168.0.209/'
tcp        0      0 192.168.0.206:39190     192.168.0.209:22        ESTABLISHED 3772/ssh            
[root@server-a ~]# 

````


If you not see any ESTABLISHED connections, the tunnel to the server-a is not up.

* Possible problems
 	- Port is in use
	- Missing rsa-key file
	- Bad permissions on rsa_key
	- Server-a change and has no configuration for your user

Run the following command to check the tunnel manually

``systemctl status proxy_ssh@server-a.service | grep "/usr/bin/ssh" ``


````
[root@server-a ~]# systemctl status proxy_ssh@client-a_server-b.service | grep "/usr/bin/ssh" 
           └─3837 /usr/bin/ssh -NT -o ServerAliveInterval=60 -o ExitOnForwardFailure=yes -L localhost:18000:localhost:8000 -i /root/.ssh/client-a client-a@server-b
[root@server-a ~]# 

````

Copy the line from ssh to the end.

``ssh -NT -o ServerAliveInterval=60 -o ExitOnForwardFailure=yes -L localhost:18000:localhost:8000 -i /root/.ssh/client-a client-a@server-b``

And run the command manually

**bind: Address already in use**

If you get this error when you run the command manually, your system has another application using 18000 tcp port. 



````
[root@server-a ~]# ssh -NT -o ServerAliveInterval=60 -o ExitOnForwardFailure=yes -L localhost:18000:localhost:8000 -i /root/.ssh/client-a client-a@server-b
bind: Address already in use
channel_setup_fwd_listener_tcpip: cannot listen to port: 18000
Could not request local forwarding.
[root@server-a ~]# 
````

To know who process is runnig on your server on port 18000 use netstat to discover.

````
[root@server-a ~]# netstat -natup  | grep 18000
tcp        0      0 0.0.0.0:18000           0.0.0.0:*               LISTEN      3903/python                     
[root@server-a ~]# 
````

In the last column of the output, you can see the PID (in this case 3903) and the application (python) who is used the port 18000, use the command ps to check the process.

````
[root@server-a ~]# ps -p 3903 -f
UID        PID  PPID  C STIME TTY          TIME CMD
root      3903  3890  0 02:42 pts/1    00:00:00 python -m SimpleHTTPServer 18000
[root@server-a ~]# 
````

Now you discover the app who is using the port 18000, talk with your team an check if you can kill this process.


**Warning: Identity file /root/.ssh/client-a not accessible: No such file or directory**

````
[root@server-a ~]# ssh -NT -o ServerAliveInterval=60 -o ExitOnForwardFailure=yes -L localhost:18000:localhost:8000 -i /root/.ssh/client-a client-a@server-b
Warning: Identity file /root/.ssh/client-a not accessible: No such file or directory.
client-a@server-b's password: 
Permission denied, please try again.
client-a@server-b's password: 
Permission denied, please try again.
client-a@server-b's password: 
Permission denied (publickey,gssapi-keyex,gssapi-with-mic,password).
[root@server-a ~]#

````

In this case the problem was related do the private rsa key file. To solve this problem recreate the key (ssh-keygen) according to the procedure and re-sync (ssh-copy-id) with your account or recover the key from a backup.

**WARNING: UNPROTECTED PRIVATE KEY FILE!**

If you recover your key from a backup, maybe the key come with wrong permissions, if you try to open the tunnel manually, you get a different error.


````
[root@server-a ~]# ssh -NT -o ServerAliveInterval=60 -o ExitOnForwardFailure=yes -L localhost:18000:localhost:8000 -i /root/.ssh/client-a client-a@server-b
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
@         WARNING: UNPROTECTED PRIVATE KEY FILE!          @
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
Permissions 0755 for '/root/.ssh/client-a' are too open.
It is required that your private key files are NOT accessible by others.
This private key will be ignored.
Load key "/root/.ssh/client-a": bad permissions
client-a@server-b's password: 
Permission denied, please try again.
client-a@server-b's password: 
Permission denied, please try again.
client-a@server-b's password: 
Permission denied (publickey,gssapi-keyex,gssapi-with-mic,password).
[root@server-a ~]#

````

Lets check the permissions of the private key

````
[root@server-a ~]# ll -h /root/.ssh/client-a
-rwxr-xr-x. 1 root root 3.2K Jan 18 20:43 /root/.ssh/client-a
 
````
The private key can be read and write only for the owner, in this case the root user, to solve, change the permissions with the chmod command.

``chmod 600 /root/.ssh/client-a ``

````
[root@server-a ~]# chmod 600 /root/.ssh/client-a
[root@server-a ~]# ll -h /root/.ssh/client-a
-rw-------. 1 root root 3.2K Jan 18 20:43 /root/.ssh/client-a
[root@server-a ~]# 

````

Now you can restart the service and check the tunnel. After that check the status and tcp connections.

````
[root@server-a ~]# systemctl status proxy_ssh@client-a_server-b.service 
● proxy_ssh@client-a_server-b.service - Create a proxy ssh to client-a_server-b
   Loaded: loaded (/etc/systemd/system/proxy_ssh@.service; enabled; vendor preset: disabled)
   Active: active (running) since Tue 2019-01-22 03:00:58 EST; 10s ago
 Main PID: 3956 (ssh)
   CGroup: /system.slice/system-proxy_ssh.slice/proxy_ssh@client-a_server-b.service
           └─3956 /usr/bin/ssh -NT -o ServerAliveInterval=60 -o ExitOnForwardFailure=yes -L localhost:18000:localhost:8000 -i /root/.ssh/c...

Jan 22 03:00:58 server-a.remoto systemd[1]: Stopped Create a proxy ssh to client-a_server-b.
Jan 22 03:00:58 server-a.remoto systemd[1]: Started Create a proxy ssh to client-a_server-b.
[root@server-a ~]# netstat -at | awk '/ssh/ && /server-b/'
tcp        0      0 server-a.remoto:39216   server-b:ssh            ESTABLISHED
[root@server-a ~]# 

````

Use curl to check the HTTP service.


````
[root@server-a ~]# curl -v localhost:18000 -o /dev/null
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0* About to connect() to localhost port 18000 (#0)
*   Trying ::1...
* Connected to localhost (::1) port 18000 (#0)
> GET / HTTP/1.1
> User-Agent: curl/7.29.0
> Host: localhost:18000
> Accept: */*
> 
< HTTP/1.1 200 OK
< Server: nginx/1.12.2
< Date: Tue, 22 Jan 2019 08:02:36 GMT
< Content-Type: text/html
< Content-Length: 3700
< Last-Modified: Tue, 06 Mar 2018 09:26:21 GMT
< Connection: keep-alive
< ETag: "5a9e5ebd-e74"
< Accept-Ranges: bytes
< 
{ [data not shown]
100  3700  100  3700    0     0   225k      0 --:--:-- --:--:-- --:--:--  240k
* Connection #0 to host localhost left intact
[root@server-a ~]# 

````

If you get a 200 HTTP code, the problem was solved, but if you can't connect with the server. Is time to check the application server. Now your sure that the problem is on the server-a, please inform to the application support team, and share the logs and evidences of your troubleshooting.

````
[root@server-a ~]# curl -v localhost:18000 -o /dev/null
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0* About to connect() to localhost port 18000 (#0)
*   Trying ::1...
* Connected to localhost (::1) port 18000 (#0)
> GET / HTTP/1.1
> User-Agent: curl/7.29.0
> Host: localhost:18000
> Accept: */*
> 
* Empty reply from server
  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0
* Connection #0 to host localhost left intact
curl: (52) Empty reply from server
[root@server-a ~]# 

````