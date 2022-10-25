# AUTOMATION WITH ANSIBLE

## Lab 3.1 : Preparation of Lab Environment

**Set hostname using 'hostnamectl'**
- execute on all nodes
```zsh
hostnamectl hostname pod-zaidanmuhammad169-controller
hostnamectl hostname pod-zaidanmuhammad169-managed1
hostnamectl hostname pod-zaidanmuhammad169-managed2
```

**Map host on /etc/hosts**
```zsh
nano /etc/hosts
```
```
...
10.39.39.10     pod-zaidanmuhammad169-controller
10.39.39.20     pod-zaidanmuhammad169-managed1
10.39.39.30     pod-zaidanmuhammad169-managed2
...
```

**Create and Distribute SSH Keygen**
1. create SSH Keygen
```zsh
ssh-keygen -t rsa
```

2. Copy controller public key from regular user to all nodes.
```zsh
ssh-copy-id -i .ssh/id_rsa.pub student@pod-zaidanmuhammad169-controller
ssh-copy-id -i .ssh/id_rsa.pub student@pod-zaidanmuhammad169-managed1
ssh-copy-id -i .ssh/id_rsa.pub student@pod-zaidanmuhammad169-managed2
```

3. Verify login without password

## Lab 3.2 : Installing Ansible

**Install required package.**
```zsh
sudo apt update
sudo apt install -y software-properties-common
```

**configure the PPA on pod-zaidanmuhammad169-controller and install ansible.**
```zsh
sudo add-apt-repository --yes --update ppa:ansible/ansible
sudo apt install ansible=6.4.0-1ppa~jammy
ansible --version
```

**Setup default configuration ansible.**
```zsh
sudo mkdir -p /etc/ansible
sudo vim /etc/ansible/hosts
```
```
...
pod-zaidanmuhammad169-managed1

[webservers]
pod-zaidanmuhammad169-managed2
...
```

**Show Hosts**
- Show all hosts from inventory.
```zsh
ansible all --list-hosts
```

- Show ungrouped hosts from inventory.
```zsh
ansible ungrouped --list-hosts
```

- Show hosts in group webservers.
```zsh
ansible webservers --list-hosts
```

**Check ping to all nodes**
```zsh
ansible all -m ping
```

## Lab 4.1 : Ad-hoc Command

**Show hostname on all node**
```zsh
ansible all -m command -a "hostname"
```

**Show facts from managed1.**
```zsh
ansible pod-zaidanmuhammad169-managed1 -m setup 
```

**Check information localhost.**
```zsh
ansible localhost -m command -a 'id'
ansible localhost -u student -m command -a 'id'
```

**Update content to file /etc/motd.**
```zsh
ansible pod-zaidanmuhammad169-managed1 --become -u student -m copy -a "content='Executed by Ansible\n' dest=/etc/motd"
ansible pod-zaidanmuhammad169-managed1 -u student -m command -a 'cat /etc/motd'
```

**Verify.**
```zsh
ssh pod-zaidanmuhammad169-managed1
```
```
...
Executed by Ansible
```

## Lab 4.2 : Manage Ansible Inventory

**Create a custom inventory file in the working directory.**

Server Inventory Spesifications  

![Server Inventory Spesifications](https://course.adinusa.id/media/markdownx/a7245250-d68b-4975-8f39-3b00078b92bb.png)

1. Create the working directory, and change into it.
```zsh
mkdir managing-inventory
cd managing-inventory
```

2. Create an inventory file in the working directory. Use the Server Inventory Specifications table as a guide. In addition, create a new group called Indonesia from the combined location group and add pod-zaidanmuhammad169-contoller as ungrouped host.
```zsh
vim inventory
```
```
pod-zaidanmuhammad169-controller

[Bogor]  
pod-zaidanmuhammad169-managed1  

[Jakarta]  
pod-zaidanmuhammad169-managed2

[WebServers]  
pod-zaidanmuhammad169-managed[1:2] 

[Testing]  
pod-zaidanmuhammad169-managed1 

[Development]  
pod-zaidanmuhammad169-managed2

[Indonesia:children]
Jakarta
Bogor
```

**Verify the managed hosts or groups in the custom the inventory file.**

- Check the list of all hosts.
```zsh
ansible all -i inventory --list-hosts
```

- Check the list of ungrouped hosts.
```zsh
ansible ungrouped -i inventory --list-hosts
```

- Check host pod-zaidanmuhammad169-managed1 in the list of hosts in the inventory file.
```zsh
ansible pod-zaidanmuhammad169-managed1 -i inventory --list-hosts
```

- Check the list of hosts in the Development group.
```zsh
ansible Development -i inventory  --list-hosts
```

- Check the list of hosts in the Testing group.
```zsh
ansible Testing -i inventory  --list-hosts
```

- Check the list of hosts in the Indonesia group.
```zsh
ansible Indonesia -i inventory  --list-hosts
```

## Lab 4.3 : Managing Ansible Configuration Files

**Create a new directory**
```zsh
mkdir -p deploy-review
cd deploy-review
```

**Create ansible configuration.**
```zsh
vim ansible.cfg
```
```
...
[defaults]
inventory = ./inventory
remote_user = student
host_key_checking = False
...
```

**Create inventory.**
```zsh
vim inventory
```
```
...
[servers]
pod-zaidanmuhammad169-managed1
pod-zaidanmuhammad169-managed2
```

**Run ansible with ad-hoc command.**
```zsh
ansible all -m command -a 'id'
ansible all -m copy -a "content='This server is managed by Ansible. \n' dest=/etc/motd" --become
ansible all -m command -a 'cat /etc/motd'
```

**Verify**
```zsh
ssh pod-zaidanmuhammad169-managed1 "whoami; cat /etc/motd"
ssh pod-zaidanmuhammad169-managed2 "whoami; cat /etc/motd"
```

## Lab 4.4 : Writing and Running Playbooks

**Create directory.**
```zsh
mkdir -p playbook-basic/files
cd playbook-basic
```

**Create ansible configuration.**
```zsh
vim ansible.cfg
```
```
...
[defaults]
inventory = ./inventory
remote_user = student
...
```

**Create inventory.**
```zsh
vim inventory
```
```
...
[web]
pod-zaidanmuhammad169-managed1
...
```

**Create playbook.**
```zsh
echo "This is a test page." > files/index.html
```
```zsh
vim site.yml
```
```
...
- name: Install and start Apache 2.
  hosts: web
  become: true
  tasks:
    - name: apache2 package is present
      apt:
        update_cache: yes
        force_apt_get: yes
        name: apache2
        state: present

    - name: correct index.html is present
      copy:
        src: files/index.html
        dest: /var/www/html/index.html

    - name: Apache 2 is started
      service:
        name: apache2
        state: started
        enabled: true
...
```

**Run playbook.**
```zsh
ansible-playbook site.yml
```

**Verify webserver.**
```zsh
curl pod-zaidanmuhammad169-managed1
```

## Lab 4.5 : Managing Variables

**Create a directory.**
```zsh
mkdir data-variables/
cd data-variables/
```

**Create ansible local configuration.**
```zsh
vim ansible.cfg
```
```
...
[defaults]
inventory = ./inventory
remote_user = student
host_key_checking = False
...
```

**Create an inventory.**
```zsh
vim inventory
```

```
...
[webserver]
pod-zaidanmuhammad169-managed2
```

**Create playbook.**
```zsh
vim playbook.yml
```
```
...
- name: Deploy and start Apache 2 service
  hosts: webserver
  become: true
  vars:
    web_pkg: apache2
    web_service: apache2
    python_pkg: python3-urllib3
  tasks:
    - name: Required packages are installed and up to date
      apt:
        update_cache: yes
        force_apt_get: yes
        name:
          - "{{web_pkg}}"
          - "{{python_pkg}}"
        state: latest
    - name: The {{web_service}} service is started and enabled
      service:
         name: "{{web_service}}"
         enabled: true
         state: started
    - name: Web content is in place
      copy:
        content: "Example web content"
        dest: /var/www/html/index.html
- name: Verify the apache service
  hosts: localhost
  tasks:
    - name: Ensure the webserver is reacheable
      uri:
        url: http://pod-zaidanmuhammad169-managed2
        status_code: 200
        return_content: yes
      register: Result
    - name: Print Ouput Webserver
      debug:
        var: Result.content
```

**Running playbook.**
```zsh
ansible-playbook --syntax-check playbook.yml
ansible-playbook playbook.yml
```

**Verify webserver.**
```zsh
curl pod-zaidanmuhammad169-managed2
```

## Lab 4.6 : Using Jinja 2 Template

**Create directory.**
```zsh
mkdir jinja2-template
cd jinja2-template
```

**Create inventory.**
```zsh
vim inventory
```

```
...
[webservers]
pod-zaidanmuhammad169-managed1
...
```

**Create playbook.**
```zsh
vim site.yml
```

```
...
- name: install and start apache2
  hosts: webservers
  become: true

  tasks:
    - name: apache2 package is present
      apt:
        name: apache2
        state: present
        update_cache: yes
        force_apt_get: yes

    - name: restart apache2 service
      service: name=apache2 state=restarted enabled=yes

    - name: copy index.html
      template: src=zaidanmuhammad169.html.j2 dest=/var/www/html/zaidanmuhammad169.html
...
```

**Create Jinja 2 template.**
```zsh
vim zaidanmuhammad169.html.j2
```
```
...
Hello World!
This is zaidanmuhammad169 site.
...
```

**Run playbook.**
```zsh
ansible-playbook -i inventory site.yml
```

**Verify.**
```zsh
curl pod-zaidanmuhammad169-managed1/zaidanmuhammad169.html
```

## Quiz 1.1 : Playbook

**(Instructions)**
1. Create a new folder named quiz-001-1 for working directory.
2. Create a new file ansible.cfg, define the location of inventory on that file. also, create inventory that stored the pod-zaidanmuhammad169-managed2.
3. Create a new playbook named quiz-1-1_playbook.yml. Add the necessary entries to start a first play named Quiz Playbook. define pod-zaidanmuhammad169-managed2 as target host and student as the remote user ,also add require privilege escalation.
4. Add the tasks that installs the latest versions of apache2, mariadb-server, php, and php-mysql packages.
5. Add the tasks to ensure the apache2 and mariadb services are enabled and running.
6. Add the necessary entries to define the final task for generating web content for testing. Use the copy module to copy the text 'Adinusa quiz Playbook - zaidanmuhammad169' on the content parameter to /var/www/html/index.php on pod-zaidanmuhammad169-managed2.
7. Define another play for the task to be performed on the control node. This play will test access to the apache2 web server that should be running on the pod-zaidanmuhammad169-managed2 host. This play does not require privilege escalation, and will run on the localhost.
8. Add a task that tests the web service running on http://pod-zaidanmuhammad169-managed2/index.php from the control node using the uri module. Check for a return status code of 200. After that, save and run the playbook.

**(Verification)**
1. make sure you have ansible.cfg, inventory, and quiz-1-1_playbook.yml file in the quiz-001-1 directory.
2. Make sure apache2, mariadb-server, php, and php-mysql packages are installed.
3. Make sure the apache2 and mariadb service is running.
4. Make sure the /var/www/html/index.php file is exist in the managed pod-zaidanmuhammad169-managed2.
5. Test webserver on pod-zaidanmuhammad169-managed2, make sure the 'Adinusa quiz Playbook - zaidanmuhammad169' text its appears.

**Create directory.**
```zsh
mkdir quiz-001-1
cd quiz-001-1
```

**Create ansible configuration.**
```zsh
vim ansible.cfg
```
```
[defaults]
inventory = ./inventory
remote_user = student
```

**Create inventory.**
```zsh
vim inventory
```
```
[webserver]
pod-zaidanmuhammad169-managed2
```

**Create playbook.**
```zsh
vim quiz-1-1_playbook.yml
```
```
...
- name: Quiz Playbook
  hosts: webserver
  become: true
  become_user: root
  tasks:
    - name: Installs the latest versions of apache2, mariadb-server, php, and php-mysql packages.
      apt:
        update_cache: yes
        force_apt_get: yes
        name:
          - apache2
          - mariadb-server
          - php
          - php-mysql
        state: latest
    - name: The Apache2 is enabled and running
      service:
        name: apache2
        enabled: true
        state: started
    - name: The MariaDB is enabled and running
      service:
        name: mariadb
        enabled: true
        state: started
    - name: Web content
      copy:
        content: "Adinusa quiz Playbook - zaidanmuhammad169"
        dest: /var/www/html/index.php

- name: Verify the apache service
  hosts: localhost
  tasks:
    - name: Test Access
      uri:
        url: http://pod-zaidanmuhammad169-managed2
        status_code: 200
        return_content: yes
      register: Result
...
```

**Running playbook.**
```zsh
ansible-playbook --syntax-check quiz-1-1_playbook.yml
ansible-playbook quiz-1-1_playbook.yml
```

**Verify webserver.**
```zsh
curl pod-zaidanmuhammad169-managed2
```

## Quiz 1.2 : Variables

**(Instructions)**
1. Create new folder quiz-001-2 for working directory.
2. Create a new file ansible.cfg, define the location of inventory on that file. also, create inventory that stored the pod-zaidanmuhammad169-managed2.
3. Create the playbook named quiz-1-2_variables.yml and define the following variables in the vars section.
   List of vars:
 - required_Pkg:
 - apache2
 - python3-urllib3
 web_Sevice: apache2
 content_File: "adinusa lab quiz variable - zaidanmuhammad169"
 dest_File: /var/www/html/index.html

4. Create an task block that uses those vars. Create the first task that installed required packages.
5. Create the tasks to make sure that the services are started and enabled.
6. Add a task that ensures specific content exists in pod-zaidanmuhammad169-managed2. Use copy module to copy the content_File to dest_File variable.
7. Define another play for the task to be performed on the localhost. This play will test access to the web server that should be running on the pod-zaidanmuhammad169-managed2 host. This play does not require privilege escalation.
8. Add a task that tests the web service running on http://pod-zaidanmuhammad169-managed2/index.html from the control node using the uri module. Check for a return status code of 200. After that, save and run the playbook.

**(Verification)**
1. Make sure you have ansible.cfg, inventory, and quiz-1-2_variables.yml file in the quiz-001-2 directory.
2. Make sure apache2, and python3-urllib3 packages are installed.
3. Make sure the apache2 service is running.
4. Make sure the /var/www/html/index.html file is exist in the managed pod-zaidanmuhammad169-managed2.
5. Test webserver on pod-zaidanmuhammad169-managed2. make sure the 'adinusa quiz Variable - zaidanmuhammad169' text its appears.

**Create directory.**
```zsh
mkdir quiz-001-2
cd quiz-001-2
```

**Create ansible configuration.**
```zsh
vim ansible.cfg
```
```
[defaults]
inventory = ./inventory
remote_user = student
```

**Create inventory.**
```zsh
vim inventory
```
```
[target]
pod-zaidanmuhammad169-managed2
```

**Create playbook.**
```zsh
vim quiz-1-2_variables.yml
```
```
- name: Quiz Playbook
  hosts: target
  become: true
  vars:
    required_Pkg: 
      - apache2
      - python3-urllib3
    web_Service: apache2
    content_File: "adinusa lab quiz variable - zaidanmuhammad169"
    dest_File: /var/www/html/index.html
  tasks:
    - name: Required packages are installed and up to date
      apt:
        update_cache: yes
        force_apt_get: yes
        name: "{{required_Pkg}}"
        state: latest
    - name: The {{web_Service}} service is started and enabled
      service:
        name: "{{web_Service}}"
        enabled: true
        state: started
    - name: Web content
      copy:
        content: "{{content_File}}"
        dest: "{{dest_File}}"
- name: Verify the apache service
  hosts: localhost
  tasks:
    - name: Test Access
      uri:
        url: http://pod-zaidanmuhammad169-managed2
        status_code: 200
        return_content: yes
      register: Result
```

**Running playbook.**
```zsh
ansible-playbook --syntax-check quiz-1-2_variables.yml
ansible-playbook quiz-1-2_variables.yml
```

**Verify webserver.**
```zsh
curl pod-zaidanmuhammad169-managed2
```