# Server side

## Considerations
1. The authentication over the ssh server use a RSA key instead of password to increases the security. 
3. This rsa-key is not encrypted by password, but will be save on the /root/.ssh folder
4. The tunnel ssh is started as a service
7. All tools and applications used on client-a and server-a come by default with CentOS 7 and this was the main reason to choose a ssh as a proxy http.

## Configuration

### Create user for client-a on server-a

### Create service user on server-b

### Create RSA keys
Create on /root/.ssh/ a RSA key named "server-a" to be use on ssh login (use root user)

command to use: ``ssh-keygen -t rsa -b 4096``

ssh-keygen is an app to create encrypted keys, in this case we going to create a RSA key (-t rsa) with a length of 4096 bits (-b 4096).

If the execution of this command is successfully, we will get a rsa key pair, one private (server-a), used to configured the service and one public key (server-a.pub), this public key has to be loaded on the server-b (see on the followings steps).

This command ask to you the path to save the key, please introduce /root/.ssh/server-a, after that they ask you for a passphrase, please don't introduce any passphrase, just press ENTER. If everything is ok you can see on your terminal the fingerprint and randomart of your key.

```
[root@server-a ~]# ssh-keygen -t rsa -b 4096
Generating public/private rsa key pair.
Enter file in which to save the key (/root/.ssh/id_rsa): /root/.ssh/server-a
Created directory '/root/.ssh'.
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in /root/.ssh/server-a.
Your public key has been saved in /root/.ssh/server-a.pub.
The key fingerprint is:
SHA256:C7uAVnj9DI2nEGWSmhheZ9DBGlgdWuNK2tskIerCQX8 root@server-a.remote
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

### Copy public key to Server B

To copy your public key to the remote server B, you have to run the following command:

``ssh-copy-id -i /root/.ssh/server-a proxy_ssh_user@server-b``

This command ask you at the beginning if you accept the key fingerprint, write "yes" and press ENTER, then you have to insert your password.

If everything is ok, you can see at the end of the command execution a message like 

```
Now try logging into the machine, with:   "ssh 'client-a@server-b'"
and check to make sure that only the key(s) you wanted were added.
```
Now your public rsa key is saved on the server-b and you can login without password.

````
[root@server-a ~]# ssh-copy-id -i /root/.ssh/server-a prosy_ssh_user@server-b
/usr/bin/ssh-copy-id: INFO: Source of key(s) to be installed: "/root/.ssh/server-a.pub"
The authenticity of host 'server-b (192.168.0.202)' can't be established.
ECDSA key fingerprint is SHA256:V5ZR5hyi/Enl69+0uqqH6rSA0cWHY3Gimkt8sP4WYAY.
ECDSA key fingerprint is MD5:6b:03:1b:a5:43:1c:73:f9:90:0c:d7:a7:b1:12:e6:b2.
Are you sure you want to continue connecting (yes/no)? yes
/usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
/usr/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys
proxy_ssh_user@server-b's password: 

Number of key(s) added: 1

Now try logging into the machine, with:   "ssh 'proxy_ssh_user@server-b'"
and check to make sure that only the key(s) you wanted were added.
````
#### Test your access

````
[root@server-a ~]# ssh -i /root/.ssh/server-a proxy_ssh_user@server-b
[proxy_ssh_user@server-b ~]$ exit
logout
Connection to server-b closed.
[root@server-a ~]# 
````

### proxy_ssh Service

#### Create configuration file for proxy_ssh@server-b
Create a file proxy_ssh@sever-b on /etc/default/ path. Change to the the directory using the cd command.

`` cd /etc/default ``

Now you have to create the file proxy_ssh@server-b and add the followings values:


````
LOCAL_ADDR=localhost
LOCAL_PORT=8000
REMOTE_PORT=8001
RSA_KEY=/root/.ssh/client-a
REMOTE_ADDR=server-a
CLIENT_USER=proxy_ssh_user
````

To create and edit the file we going to use vi (a simple tex editor). To open vi and start to work on our proxy_ssh@server-b file, run the following command.

`` vi proxy_ssh@server-b``

You can see now a empty file and in the bottom the full path and a tag [New File].
 
``"/etc/default/proxy_ssh@server-b" [New File]``

If you don't see this things maybe you are on the wrong path or file, please press ESC key, then ":" (without the "), now you can give orders to vi, we have to quit the editor, now write "q!" (without the ") and after ENTER. Check the path with pwd command, then check the name of the file.

````
[root@server-a default]# pwd
/etc/default
````

For some people vi is a little bit complicated to use, is you have problems you can use the command cat instead of vi.

``cat > /etc/default/proxy_ssh@server-b``

After run the command, you will see the cursor of the terminal below the prompt of your terminal, now you can paste the configuration values, after past the values, press ENTER and then ctrl+D to exit.

````
[root@server-a default]# cat > /etc/default/proxy_ssh@server-b
LOCAL_ADDR=localhost
LOCAL_PORT=18000
REMOTE_PORT=8000
RSA_KEY=/root/.ssh/server-a
REMOTE_ADDR=server-b
CLIENT_USER=proxy_ssh_user
````
Now check the content of your configuration file using the command more

``more /etc/default/proxy_ssh@server-b``

````
[root@server-a default]# more /etc/default/proxy_ssh@server-b
LOCAL_ADDR=localhost
LOCAL_PORT=8000
REMOTE_PORT=18000
RSA_KEY=/root/.ssh/server-a
REMOTE_ADDR=server-b
CLIENT_USER=proxy_ssh_user
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

To enable our service at the startup of the server, we have to run the following commadnd.

``systemctl enable proxy_ssh@server-b.service ``

````
[root@server-a ~]# systemctl enable proxy_ssh@server-b.service
Created symlink from /etc/systemd/system/default.target.wants/proxy_ssh@server-b.service to /etc/systemd/system/proxy_ssh@.service.
[root@server-a ~]#
````

#### proxy_ssh service management

##### status

````
[root@server-a ~]# systemctl status proxy_ssh@server-b.service
● proxy_ssh@server-b.service - Create a proxy ssh to server-b
   Loaded: loaded (/etc/systemd/system/proxy_ssh@.service; enabled; vendor preset: disabled)
   Active: active (running) since Sat 2019-01-19 03:25:07 EST; 5min ago
 Main PID: 14844 (ssh)
   CGroup: /system.slice/system-proxy_ssh.slice/proxy_ssh@server-b.service
           └─14844 /usr/bin/ssh -NT -o ServerAliveInterval=60 -o ExitOnForwardFailure=yes -L localhost:18000:localhost:8000 -i /root/.ssh/c...

Jan 19 03:25:07 server-a.remote systemd[1]: Started Create a proxy ssh to server-b.
[root@server-a ~]# 


````


##### start

````
[root@server-a ~]# systemctl start proxy_ssh@server-b.service
[root@server-a ~]# systemctl status proxy_ssh@server-b.service
● proxy_ssh@server-b.service - Create a proxy ssh to server-b
   Loaded: loaded (/etc/systemd/system/proxy_ssh@.service; enabled; vendor preset: disabled)
   Active: active (running) since Sat 2019-01-19 03:33:00 EST; 1s ago
 Main PID: 14892 (ssh)
   CGroup: /system.slice/system-proxy_ssh.slice/proxy_ssh@server-b.service
           └─14892 /usr/bin/ssh -NT -o ServerAliveInterval=60 -o ExitOnForwardFailure=yes -L localhost:18000:localhost:8000 -i /root/.ssh/c...

Jan 19 03:33:00 server-a.remote systemd[1]: Started Create a proxy ssh to server-b.
[root@server-a ~]# 

````

##### stop

````
[root@server-a ~]# systemctl stop proxy_ssh@server-b.service

[root@server-a ~]# systemctl status proxy_ssh@server-b.service
● proxy_ssh@server-b.service - Create a proxy ssh to server-b
   Loaded: loaded (/etc/systemd/system/proxy_ssh@.service; enabled; vendor preset: disabled)
   Active: inactive (dead)

Jan 19 03:25:07 server-a.remote systemd[1]: Started Create a proxy ssh to server-b.
Jan 19 03:31:40 server-a.remote systemd[1]: Stopping Create a proxy ssh to server-b...
Jan 19 03:31:40 server-a.remote systemd[1]: Stopped Create a proxy ssh to server-b.
Jan 19 03:31:44 server-a.remote systemd[1]: Started Create a proxy ssh to server-b.
Jan 19 03:32:37 server-a.remote systemd[1]: Stopping Create a proxy ssh to server-b...
Jan 19 03:32:37 server-a.remote systemd[1]: Stopped Create a proxy ssh to server-b.
[root@server-a ~]#
````

##### restart

````
[root@server-a ~]# systemctl restart proxy_ssh@server-b.service
[root@server-a ~]# systemctl status proxy_ssh@server-b.service
● proxy_ssh@server-b.service - Create a proxy ssh to server-b
   Loaded: loaded (/etc/systemd/system/proxy_ssh@.service; enabled; vendor preset: disabled)
   Active: active (running) since Sat 2019-01-19 03:33:32 EST; 1s ago
 Main PID: 14916 (ssh)
   CGroup: /system.slice/system-proxy_ssh.slice/proxy_ssh@server-b.service
           └─14916 /usr/bin/ssh -NT -o ServerAliveInterval=60 -o ExitOnForwardFailure=yes -L localhost:8000:localhost:18000 -i /root/.ssh/c...

Jan 19 03:33:32 server-a.remote systemd[1]: Stopped Create a proxy ssh to server-b.
Jan 19 03:33:32 server-a.remote systemd[1]: Started Create a proxy ssh to server-b.
[root@server-a ~]# 
````

#### Check HTTP connection

````
[root@server-a ~]# curl -I localhost:18000
HTTP/1.1 200 OK
Server: nginx/1.12.2
Date: Sat, 19 Jan 2019 08:25:28 GMT
Content-Type: text/html
Content-Length: 3700
Last-Modified: Tue, 06 Mar 2018 09:26:21 GMT
Connection: keep-alive
ETag: "5a9e5ebd-e74"
Accept-Ranges: bytes

[root@server-a ~]# 

````

### Troubleshooting

