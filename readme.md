# AUTOMATION WITH ANSIBLE

## Lab 3.1 : Preparation of Lab Environment

#### Set hostname using 'hostnamectl'
- execute on all nodes
```zsh
hostnamectl hostname pod-zaidanmuhammad169-controller
hostnamectl hostname pod-zaidanmuhammad169-managed1
hostnamectl hostname pod-zaidanmuhammad169-managed2
```

#### Map host on /etc/hosts
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

#### Create and Distribute SSH Keygen
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

#### Install required package.
```zsh
sudo apt update
sudo apt install -y software-properties-common
```

#### configure the PPA on pod-zaidanmuhammad169-controller and install ansible.
```zsh
sudo add-apt-repository --yes --update ppa:ansible/ansible
sudo apt install ansible=6.4.0-1ppa~jammy
ansible --version
```

#### Setup default configuration ansible.
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

#### Show Hosts
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

#### Check ping to all nodes
```zsh
ansible all -m ping
```

## Lab 4.1 : Ad-hoc Command

#### Show hostname on all node
```zsh
ansible all -m command -a "hostname"
```

#### Show facts from managed1.
```zsh
ansible pod-zaidanmuhammad169-managed1 -m setup 
```

#### Check information localhost.
```zsh
ansible localhost -m command -a 'id'
ansible localhost -u student -m command -a 'id'
```

#### Update content to file /etc/motd.
```zsh
ansible pod-zaidanmuhammad169-managed1 --become -u student -m copy -a "content='Executed by Ansible\n' dest=/etc/motd"
ansible pod-zaidanmuhammad169-managed1 -u student -m command -a 'cat /etc/motd'
```

#### Verify.
```zsh
ssh pod-zaidanmuhammad169-managed1
```
```
...
Executed by Ansible
```

## Lab 4.2 : Manage Ansible Inventory

#### Create a custom inventory file in the working directory.

- Server Inventory Spesifications
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

#### Verify the managed hosts or groups in the custom the inventory file.

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

#### Create a new directory
```zsh
mkdir -p deploy-review
cd deploy-review
```

#### Create ansible configuration.
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

#### Create inventory.
```zsh
vim inventory
```
```
...
[servers]
pod-zaidanmuhammad169-managed1
pod-zaidanmuhammad169-managed2
```

#### Run ansible with ad-hoc command.
```zsh
ansible all -m command -a 'id'
ansible all -m copy -a "content='This server is managed by Ansible. \n' dest=/etc/motd" --become
ansible all -m command -a 'cat /etc/motd'
```

#### Verify
```zsh
ssh pod-zaidanmuhammad169-managed1 "whoami; cat /etc/motd"
ssh pod-zaidanmuhammad169-managed2 "whoami; cat /etc/motd"
```

