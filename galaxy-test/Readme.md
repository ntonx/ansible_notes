### Instruccions to execute an Ansible example using Galaxy.


Execute this command on console:
`# docker run -d --name master1 ubuntu:latest tail -f /dev/null`

#### 1. Commands to create servers
Run the following commands to create the servers to be managed (see /example1/README.md and complete all commands described in this file).

```yaml
# docker run --rm -d --name server1 server:ansible tail -f /dev/null
# docker run --rm -d --name server2 server:ansible tail -f /dev/null
# docker inspect server1
# docker inspect server2
```

#### 2. Commands to configure Ansible manager server.
 
Copy IP from server1 and server2 (see output from the last two commands).

Access to container and execute:

```yaml
# docker exec -it master1 bash
```

Attach galaxy-test directory inside master container using volumes or create a directory and copy directory content with this structure:
```yaml
- galaxy-test/
    hosts
    ansible.cfg
    - playbooks/
    - vars/
```

Paste servers IP in hosts file. In this case, we use server1 as database and server2 as webserver.
```yaml
[database]
172.17.0.3    ansible_pass=test ansible_user=test  
[web]
172.17.0.4    ansible_pass=test ansible_user=test
```

**Note:**
* Before use module ping, ensure make a ssh connection from master1 to servers (server1 and server2), this is for add a fingersprint in master1 where Ansible is installed.

Execute a ping to every server using ping module.

```
# ansible all -m ping --ask-pass
```

---
### Ansible Galaxy Roles
This is a repository where Ansible objects (roles and collections) are stored in order to shared with community, this objects are described below

* Role:
> A role enables the sharing and reuse of Ansible tasks. It contains Ansible playbook tasks, plus all the supporting files, variables, templates, and handlers needed to run the tasks. A role is a complete unit of automation that can be reused and shared [ https://galaxy.ansible.com/docs/finding/content_types.html#ansible-roles ].

* Collection:
>It is a distribution format for Ansible content that can include playbooks, roles, modules, and plugins [ https://docs.ansible.com/ansible/devel/collections_guide/index.html#collections ].`

#### 1. Using Ansible Galaxy roles
**a)** Open https://galaxy.ansible.com and search mysql server (you can filter results selecting type of search wheter collection or role. At the time this file was written, there were 11 collections and 902 roles).

**b)** Search mysqk server and filter by roles.

**c)** Review geerlingguy.mysql role clicking on Details and ReadMe section.

**d)** Install geerlingguy.mysql role running this command in console:
```yaml
# docker exec -it master1 bash
# cd galaxy-test/
# ansible-galaxy install geerlingguy.mysql
```

**Notes:** 
* In install_database.yml playbook, there is a vars_files section, in which is declared the file where variables, used for playbook, are defined, so check this file.

* In the database.yml file is declared variables as database name (db01), username and password.

**f)** Install and configure database according to playbook, run command:
```
# ansible-playbook playbooks/install_database.yml --ask-pass --ask-become-pass -K
```
* the last command could show an error in output, mysql is installed but is down, restart with the next commands.

**g)** Once installations is completed, verify mysql status accessing to server1:

```yaml
# docker exec -it server1 bash
# service mysql restart
# service status mysql
```

**h)** Once restart is completed, execute again the next command:
```
# ansible-playbook playbooks/install_database.yml --ask-pass --ask-become-pass -K
```

**i)** Access to mysql, run command:
```yaml
# docker exec -it server1 bash
server1# sudo mysql -u linuxhint -p
MariaDB[(none)]> SHOW DATABASES;
```


--------

### Ansible Galaxy Collections

**a)** Search geerlingguy.php_roles collection in Ansible Galaxy.

**b)** Read collection documentation and ReadMe

**c)** Run command to install collection on master1 in /galaxy-test directory:
```
# ansible-galaxy collection install geerlingguy.php_roles
```

**d)** Run command to install php using collection:
```
# ansible-playbook playbooks/install_php.yml --ask-pas --ask-become-pass
```

**Notes:**
* If you check install_php.yml, you will see two roles defined inside this collection:

```yaml
- hosts: web
  user: ansible
  become: yes
  collections:
   - geerlingguy.php_roles
  roles:
    - role: php
    - role: php_versions
      vars:
        php_version: '7.3'
```
* Into this yml file we see a collections section, in which was declared the php_roles collection that will be used.

* In Ansible Galaxy, in the Collection ReadMe section of php_roles, there are describe the different roles included in this collection. Two of those collections are used in this example (php and php_version collections).

* In case of php_version role, this one requires a variable to configure, in this case defined as php_version.

* Exploring Ansible Galaxy, in description of each collection and role, is possible get knowledge how configure a playbook. 

**e)** Once command in **d)** is executed, make a curl request to this server running:

```
# docker exec -it server2 bash
server2# apt install curl
server2# curl localhost:80
```
In this case, we can not access from other containers because port 80 is not expose when we created server2.