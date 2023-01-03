### Brief review of geerlingguy.mysql role defined in install-database.yml playbook


When the following command is executed on master1, ansible download files from Ansible Galaxy to local computer.
```
# ansible-galaxy install geerlingguy.mysql
```

The files are stored in ``/root/.ansible/roles/`` in an specific folder, in this case in ``geerlingguy.mysql/``

If you access to this folder, you will see the next organization:

```
-- LICENSE
-- README.md
-- defaults/
   -- main.yml
-- handlers/
   -- main.yml
-- meta/
   -- main.yml
-- molecule/
   -- default/
       -- converge.yml
       -- molecule.yml
-- tasks/
   -- configure.yml
   -- databases.yml
   -- main.yml
   -- replication.yml
   -- secure-installation.yml
   -- setup-Archlinux.yml
   -- setup-Debian.yml
   -- setup-RedHat.yml
   -- users.yml
   -- variables.yml
-- templates/
   -- my.cnf.j2
   -- root-my.cnf.j2
   -- user-my.cnf.j2
-- vars/
    -- Archlinux.yml
    -- Debian-10.yml
    -- Debian-11.yml
    -- Debian.yml
    -- RedHat-7.yml
    -- RedHat-8.yml
    -- RedHat-9.yml
```

In task/ folder there is main.yml file, in which is declared task to do in order to install mysql on the specific server. 

Analizing task/main.yml, we look there are defined yml dependencies (configure.yml, databases.yml, user.yml, etc) in which there are specified configuration set up.
```
### main.yml file
---
# Variable configuration.
- include_tasks: variables.yml

# Setup/install tasks.
- include_tasks: setup-RedHat.yml
  when: ansible_os_family == 'RedHat'

- include_tasks: setup-Debian.yml
  when: ansible_os_family == 'Debian'

- include_tasks: setup-Archlinux.yml
  when: ansible_os_family == 'Archlinux'

- name: Check if MySQL packages were installed.
  set_fact:
    mysql_install_packages: "{{ (rh_mysql_install_packages is defined and rh_mysql_install_packages.changed)
      or (deb_mysql_install_packages is defined and deb_mysql_install_packages.changed)
      or (arch_mysql_install_packages is defined and arch_mysql_install_packages.changed) }}"

# Configure MySQL.
- include_tasks: configure.yml
- include_tasks: secure-installation.yml
- include_tasks: databases.yml
- include_tasks: users.yml
- include_tasks: replication.yml
```

In default/main.yml is stored default configuration for MySQL service, we can configure MySQL modifying this file.

More information could be read in README.md file