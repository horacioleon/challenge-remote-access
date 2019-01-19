#Client side

##Considerations
1. The autentication over the ssh server use a RSA key instead of password to increasse the security.
2. Each client need their own user, password and key to access to the server-A. 
3. This rsa-key is not encrypted by password, but will be save on the /root/.ssh folder
4. The tunnel ssh is started as a service
5. Client A has root access to the machine, but is not an advanced user.
6. Client A has can reach an have access to the machine via ssh
6. All users on client-a machine can access to the web application on server-b
7. All tools and aplications used on cliente-a and server-a com by default with CentOS 7 and this was the main reason to chose a ssh as a proxy http.

##Configuration
###Access to the machine client-a

Check if you have access to the client-a machine ussing any ssh client

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
		
			Accept the key fingerprint wrinting "yes" and press ENTER, then put your password.
		
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
		- 	Open your prefered terminal emulator and execute the following command.
		
			``ssh root@client-a.local.com``
			
			or
			
			``ssh root@192.168.0.204`
		
			Accept the key fingerprint wrinting "yes" and press ENTER, then put your password.
		
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
		- Microsft Windows has no native ssh client, but you can install putty to access to servers that only accept ssh conections.
		
			To dowload go to the putty web ([here](https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html)), chose the "MSI (‘Windows Installer’)" acording to the arquitecture of your machine, if your don't know if your machine is 32bit or 64bit, choose 32bit.
			Putty has clasic instaler, Next , Next , Next and finish. When the processs finish, you can find putty on your program list.
			
			Open putty, you can see at the session category on the rigth side, few blank text box, on text box "Host Name (or IP Address), put the client-a ip or FQDN server name.
			
			

<mark>If you can access to the machine, contact your local support to help you, without access you can't continue with the procedure.</mark>

###Create RSA keys
Create on /root/.ssh/ a RSA key named "client-a" to be use on ssh login (use root user)

command to use: ``ssh-keygen -t rsa -b 4096``

ssh-keygen is an app to create encrypted keys, in this case we going to create a RSA key (-t rsa) with a length of 4096 bits (-b 4096).

If the execution of this command is successfully, we will get a rsa key pair, one private (client-a), used to configured the service and one public key (client-a.pub), this public key has to be loaded on the server-b (see on the followings steps).

This command ask to you the path to save the key, please introduce /root/.ssh/client-a, after that they ask you for a passphrase, please dont introduce any passphrase, just press ENTER. If everything is ok you can see on your terminal the fingerprint and randomart of your key.

####Example
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

####Know errors

1. If for any reason the ssh-keygen command is not present on the machine (the app come by default with CentOS 7 minimal) you can install using yum.

	command to use: ``yum install openssh openssh-clients``

###Copy public key to Server A

To copy your public key to the remote server A, you have to run the following command:

``ssh-copy-id -i /root/.ssh/client-a client-a@server-a``

This command ask you at the begining if you accept the key fingerprinr, write "yes" and press ENTER, then you have to insert your password.

If everything is ok, you can see at the end of the command execution a message like 

```
Now try logging into the machine, with:   "ssh 'client-a@server-a'"
and check to make sure that only the key(s) you wanted were added.
```
Now your public rsa key is saved on the server-a and you can login without password.

####Example
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
####Test your access

````
[root@client-a ~]# ssh -i /root/.ssh/client-a client-a@server-a
[client-a@server-a ~]$ exit
logout
Connection to server-a closed.
[root@client-a ~]# 
````

###proxy_ssh Service

####Create configuration file for proxy_ssh@cliente-A
Create a file proxy_ssh@client-a on /etc/default/ path. Change to the the directory using the cd command.

`` cd /etc/default ``

Now you have to create the file proxy_ssh@client-a and add the followings values:


````
LOCAL_ADDR=localhost
LOCAL_PORT=8000
REMOTE_PORT=8000
RSA_KEY=/root/.ssh/client-a
REMOTE_ADDR=server-a
````

To create and edit the file we going to use vi (a simple tex editor). To open vi and start to work on our proxy_ssh@client-a file, run the following command.

`` vi proxy_ssh@client-a``

You can see now a empty file and in the bottom the full path and a tag [New File].
 
``"/etc/default/proxy_ssh@client-a" [New File]``

If you don't see this things maybe you are on the wrong path or file, please press ESC key, then ":" (without the "), now you can give orders to vi, we have to quit the editor, now write "q!" (without the ") and after ENTER. Check the path with pwd commnad, then check the name of the file.

````
[root@client-a default]# pwd
/etc/default
````

For some people vi is a little bit complicated to use, is you have problems you can use the command cat instead of vi.

``cat > /etc/default/proxy_ssh@client-a``

After run the command, you will see the cursor of the terminal below the promt of your terminal, now you can paste the configuration values, after past the values, press ENTER and then ctrl+D to exit.

````
[root@client-a default]# cat > /etc/default/proxy_ssh@client-a
LOCAL_ADDR=localhost
LOCAL_PORT=8000
REMOTE_PORT=8000
RSA_KEY=/root/.ssh/client-a
REMOTE_ADDR=server-a
````
Now check the content of your configuration file using the command more

``more /etc/default/proxy_ssh@client-a``

````
[root@client-a default]# more /etc/default/proxy_ssh@client-a
LOCAL_ADDR=localhost
LOCAL_PORT=8000
REMOTE_PORT=8000
RSA_KEY=/root/.ssh/client-a
REMOTE_ADDR=server-a
````

####Create service proxy_ssh@client-a

````
[Unit]
Description=Create a proxy ssh to %I
After=network.target

[Service]
EnvironmentFile=/etc/default/proxy_ssh@%i
ExecStart=/usr/bin/ssh -NT -o ServerAliveInterval=60 -o ExitOnForwardFailure=yes -L ${LOCAL_ADDR}:${LOCAL_PORT}:localhost:${REMOTE_PORT} -i ${RSA_KEY} ${REMOTE_ADDR}

RestartSec=10
Restart=always

[Install]
WantedBy=default.target
````


