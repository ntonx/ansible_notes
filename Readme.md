### Instruccions to execute Ansible in Docker containers for demostrative purpose.
This file contains instructions to execute in Docker Desktop and help to simulate  three containers in a network in which is possible to configure and use Ansible.

Execute this command on console:
`#docker run -d --name master1 ubuntu:latest tail -f /dev/null`

Copy this in a Dockerfile and execute the next command on the same directory where the file is saved.
```yaml
FROM ubuntu:latest
RUN apt update && apt install  openssh-server sudo -y
RUN useradd -rm -d /home/ubuntu -s /bin/bash -g root -G sudo -u 1000 test 
RUN  echo 'test:test' | chpasswd
RUN service ssh start
EXPOSE 22
CMD ["/usr/sbin/sshd","-D"]
```

Execute these commands to create an image and run it to deploy servers to be managed by Ansible.

```yaml
#docker build -t server:ansible .
#docker run --rm -d --name server1 server:ansible tail -f /dev/null
#docker run --rm -d --name server2 server:ansible tail -f /dev/null
#docker inspect server1
#docker inspect server2
```

Copy IP from server1 and server2 (see output from the last two commands).

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

##### Commands to execute on master1

Access to container and execute:

```yaml
#docker exec -it master1 bash
#apt-get update
#apt-get install net-tools (just in case you want to check your ip using ifconfig)
#apt install -y iputils-ping (used for making ping to server1 or server2)
#ping <IP-server1>
```

Commands to install Ansible

```yaml
#apt-get install software-properties-common
#apt-get update
#add-apt-repository ppa:ansible/ansible
#<ENTER>
#apt update
#apt install ansible
#ansible --version
```

Edit hosts file according to the IP servers (Inventory: this file contains servers that Ansible is able to manage):
```yaml
#apt-get install -y nano
#nano /etc/ansible/hosts
```
NOTE: 
* It's possible to define each server adding his IP and credentials as:
172.17.0.3  ansible_pass=test ansible_user=test

* It's possible group servers as follows, indicating name of servers with [ ], this means when we use "ansible" command, ansible group a number of servers:
[webservers]
172.17.0.3  ansible_pass=test ansible_user=test
172.17.0.4  ansible_pass=test ansible_user=test
For example: ansible webservers -m ping    <this will make ping to every server declared in [] this section

NOTE:
* Before use module ping, ensure make a ssh connection from master1 to servers (server1 and server2), this is for add a fingersprint in master1 where Ansible is installed.

```yaml
#ssh test@IP-SERVER1 
#WRITE test
#WRITE yes
```
---
### Commands ad-hoc using in Ansible.
Ansible allows execute specific task on every server declared in hosts file (inventory) using the console and declaring task and params on the console where Ansible is installed. For that purpose, Ansible uses modules which are declared in console using -m flag (there are many modules you can use, they are declared on https://docs.ansible.com/ansible/2.9/modules/modules_by_category.html ).

For this purpose in the next section are written ansible commands to configure servers using specific modules.

#### Ping module
When a playbook (a yml file) is not defined, we can use commands defining a server or group of them, for example, ping command using the module ping.

Access to master1 and execute:
```
#ansible 172.17.0.3 -m ping --ask-pass
```
If servers are declared as part of a group [ GroupName ] then execute:
```
#ansible <GroupName> -m ping --ask-pass
```
#### Shell module

```
#ansible -m shell -a '<A command to execute in shell inside a server>'
```
This module can be skipped in console, as follow:
```yaml
#ansible <GroupName o IP> -a 'echo hola' --ask-pass
#ansible webservers -a 'echo hola' --ask-pass
#ansible <IP> -a 'true' --ask-pass
#ansible <IP> -a 'false' --ask-pass
#ansible <IP> -a 'ls /' --ask-pass
#ansible <IP> -a 'uname' --ask-pass
#ansible webservers -a 'nano --version' --ask-pass
#ansible webservers -a 'vim --version' --ask-pass
```

#### Apt module

In this case apt module is used because servers are Ubuntu. Depending on your distro this module can change, for example apk, pacman, brew, etc.

```
#ansible webservers -m apt -a 'name=vim state=present' --ask-pass -b -K
```
NOTE: -b -K are flags to set as superoot privileges.

We can use also this command:
```
#ansible webservers -m apt -a 'name=vim state=present' --ask-pass --ask-become-pass
```

In this case -a flag is used to define params, each module has his ows params, so check documentation on https://docs.ansible.com/ansible/2.9/modules/modules_by_category.html

---
### Using a playbook with Ansible
We can use a yml file to declare tasks, each task need metadata as: 
* name: used to see which section is executed by Ansible on console   
* module: used to define the module and configure with params

This yml is a json format file as follow:

```yaml
#hosts: webservers
#hosts: IP
#become : true is used to execute superroot commands
- hosts: all
  tasks:
    - name: Vim version verification
      shell: vim --version
    - name: Install VIM
      apt: name=vim state=present
      become: true
#    - name: Start a service
#      service: name=nginx state=stopped
#      become: true
#    - name: Start a service
#      service: name=nginx state=started
#      become: true
```
Note:
* #define a comment in yml file, so every line that start with it, is skipped.
* become allows execute task as super root user, so in apt task and services task is neccesary declared it as true.
* We can declare all tasks in playbook as become: true if we declare at the beginning, just after hosts.
* name allows to see in which part of the execution, the configuration is. It's like a legend show in console.
* After declare a name flag, is important define the module by its name, in the last file there was used the next modules: shell, apt and service (commented).
* After declared the module name, we need to declare params, for example in the file 'name=vim state=present'.

Save the last file as vim-playbook.yml and execute using:
```
#ansible-playbook vim-playbook.yml --ask-pass -K
```
The console output of the last command is as follow:
```yaml
PLAY [all] ****************************************************************

TASK [Gathering Facts] ****************************************************
ok: [172.17.0.4]
ok: [172.17.0.3]

TASK [Vim version verification] *******************************************
changed: [172.17.0.4]
changed: [172.17.0.3]

TASK [Install VIM] ********************************************************
ok: [172.17.0.4]
ok: [172.17.0.3]

PLAY RECAP ****************************************************************
172.17.0.3  : ok=3    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
172.17.0.4  : ok=3    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

---
### Other important flags in Ansible
To execute a connection in other servers where a user and pass is configured (in the
previous examples servers containers have test as user and password) we can specify user using flag -u, for example:


```
#ansible webservers -m ping -u <user> --ask-pass
```
___
### Configuration file for Ansible
As in the last section we need to specify a user in the command line, we can skipped this if we create a ansible.cfg file in /etc/ansible/ directory

Access to master1
```yml
#docker exec -it master1 bash
#nano /etc/ansible/ansible.cfg
```
Write this lines in the file and save it:
```yaml
[defaults]
remote_user = <write-user-here>
```
So we can skipped user flag in the command:
```
#ansible webservers -m ping --ask-pass
```

We can define the remote_user in the playbook also, adding the next line just after the 'hosts: all' line (in case the server in which the task is executed has different user)
```
remote_user: <user>
```
---
### Handlers in Ansible
A handler is a task that required a condition to be executed, it means that according to a notification the task is executed. To see the functionality of this element we need to define a playbook as /example2/apache2-playbook.yml

Once the file is saved, execute this command:
```
ansible-playbook apache2-playbook.yml --ask-pass -K
```
The servers did not have installed apache2 so for that reason a notification was set up (Restart server). This last task (service apache2 restart) was defined as a handler, so in case of execute the command again, this task (restart) will not be executed. Execute this again to see output:

```
ansible-playbook apache2-playbook.yml --ask-pass -K
```
