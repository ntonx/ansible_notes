#### Execute this commands to create a server able to receive ssh and ping requests from a master in which is installed Ansible

```
docker build -t server:ansible .

#### Command to execute in order to scan vulnerabilities on docker images
```
docker scan --dependency-tree server:ansible
docker scan name:version
```


Deploy a ubuntu server with this command
```
docker run --rm -d --name server1 server:ansible tail -f /dev/null
docker run --rm -d --name server2 server:ansible tail -f /dev/null
```

##### Commands to execute on server1 and server2
These commands allow execute a ssh client on these servers.

Access to servers and execute:
```yaml
#docker exec -it server1 bash
#apt-get update && apt-get install -y nano
#nano /etc/ssh/sshd_config
```
On this file, change this line:
```
#PermitRootLogin prohibit-password
```
to:
```
PermitRootLogin yes
```
Restart service using command:
```
#service ssh restart
```