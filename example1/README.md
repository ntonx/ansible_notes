#### Execute this commands to create a server able to receive ssh and ping requests from a master in which is installed Ansible

```
docker build -t server:ansible .
```
Deploy a ubuntu server with this command
```
docker run --rm -d --name server1 server:ansible tail -f /dev/null
docker run --rm -d --name server2 server:ansible tail -f /dev/null
```