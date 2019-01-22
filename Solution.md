# Solution

To solve this problem, I used two ssh tunnel as a proxy service.

The first ssh tunnel is enable from the client-a to server-a,  using the port 8000 to create a tunnel to 18000 tcp port on server-a.  Due to 8000 tcp port is on use on server-a, to keep some name relationship between the services on both sides I decided to use the 18000 tcp port.   

The second tunnel is established on server-a to server-b, using the tcp port 18000 to publish the web application on server-a from server-b.

The solution uses the same service on both sides (client and server) this service is managed by system and can restart the ssh connections if some problems happen. The ssh connection use a RSA certificate to avoid enter a password to give more security.

The name of the service is proxy_ssh and they receive the instance of the new tunnel to be enabled, with this kind of configuration we can configure more than one instance and tunnel. 
To identify the services an create an easy way to understand the process the name on client side has the ssh destiny server in the instance name and from the internal server ssh tunnel I added the name os the client to identify each tunnel by clients.

proxy_ssh@server-a.service 

proxy_ssh@client-a_server-b.service

Only root user on client-a can get access to the server-a using the RSA key, but all users can access to the HTTP service using http://localhost:8000 as endpoint.

Client side

````
 ------------------------------                             -----------------------------
|  client-A                    |                           |  server-a                   |
|  http://localhost:8000       | ---    ssh tunnel.   ---> |  http://localhost:18000     |
|  proxy_ssh service           |    tcp 8000 -> tcp 18000  |  proxy_ssh service          |
|  - instance: client-a        |                           |   - instance:               |                              |                              |                           |  client-a_server-b.         |
 ------------------------------                             -----------------------------
````


Server side

````
 ------------------------------                             -----------------------------
|  server-A                    |                           |  server-b                   |
|  http://localhost:18000      |  ---    ssh tunnel   ---> |  http://localhost:8000      |
|  proxy_ssh service           |    tcp 18000 -> tcp 8000  |  web service                |
|  - instance:                 |                           |                             |                              |  client-a_server-b           |                           |                             |
 ------------------------------                             -----------------------------
````

Please see below the configuration documents for each environment.

[Client configuration](client_side.md)

[Server configuration](server_side.md)