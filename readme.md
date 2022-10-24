# AUTOMATION WITH ANSIBLE

## Lab 3.1 : Preparation of Lab Environment

1. Set hostname using 'hostnamectl'
	- execute on all nodes
```shell
hostnamectl hostname pod-zaidanmuhammad169-controller
hostnamectl hostname pod-zaidanmuhammad169-managed1
hostnamectl hostname pod-zaidanmuhammad169-managed2
```

2. Map host on /etc/hosts
```shell
nano /etc/hosts
```
```
...
10.39.39.10     pod-zaidanmuhammad169-controller
10.39.39.20     pod-zaidanmuhammad169-managed1
10.39.39.30     pod-zaidanmuhammad169-managed2
...
```

3. Create and Distribute SSH Keygen
	- create SSH Keygen
	```shell
	ssh-keygen -t rsa
	```

	- Copy controller public key from regular user to all nodes.
	```shell
	ssh-copy-id -i ~/.ssh/id_rsa.pub student@pod-zaidanmuhammad169-controller
	ssh-copy-id -i ~/.ssh/id_rsa.pub student@pod-zaidanmuhammad169-managed1
	ssh-copy-id -i ~/.ssh/id_rsa.pub student@pod-zaidanmuhammad169-managed2
	```

	- Verify login without password

## Lab 3.2 : Installing Ansible

1. Install required package.
```shell
sudo apt update
sudo apt install -y software-properties-common
```

2. configure the PPA on pod-zaidanmuhammad169-controller and install ansible.
```shell
sudo add-apt-repository --yes --update ppa:ansible/ansible
sudo apt install ansible=6.4.0-1ppa~jammy
ansible --version
```

3. Setup default configuration ansible.
```shell
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

4. Show Hosts
	- Show all hosts from inventory.
	```shell
	ansible all --list-hosts
	```

	- Show ungrouped hosts from inventory.
	```shell
	ansible ungrouped --list-hosts
	```

	- Show hosts in group webservers.
	```shell
	ansible webservers --list-hosts
	```

5. Check ping to all nodes
```shell
ansible all -m ping
```

## Lab 4.1 : Ad-hoc Command

1. Show hostname on all node
```shell
ansible all -m command -a "hostname"
```

2. Show facts from managed1.
```shell
ansible pod-zaidanmuhammad169-managed1 -m setup 
```

3. Check information localhost.
```shell
ansible localhost -m command -a 'id'
ansible localhost -u student -m command -a 'id'
```

4. Update content to file /etc/motd.
```shell
ansible pod-zaidanmuhammad169-managed1 --become -u student -m copy -a "content='Executed by Ansible\n' dest=/etc/motd"
ansible pod-zaidanmuhammad169-managed1 -u student -m command -a 'cat /etc/motd'
```

5. Verify.
```shell
ssh pod-zaidanmuhammad169-managed1
```
```
...
Executed by Ansible
```

## Lab 4.2 : Manage Ansible Inventory

1. Create a custom inventory file in the working directory.

	- Server Inventory Spesifications
	![Server Inventory Spesifications](https://course.adinusa.id/media/markdownx/a7245250-d68b-4975-8f39-3b00078b92bb.png)


	1) Create the working directory, and change into it.
	```shell
	mkdir managing-inventory
	cd managing-inventory
	```

	2) Create an inventory file in the working directory. Use the Server Inventory Specifications table as a guide. In addition, create a new group called Indonesia from the combined location group and add pod-zaidanmuhammad169-contoller as ungrouped host.
	```shell
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

2. Verify the managed hosts or groups in the custom the inventory file.

	- Check the list of all hosts.
	```shell
	ansible all -i inventory --list-hosts
	```

	- Check the list of ungrouped hosts.
	```shell
	ansible ungrouped -i inventory --list-hosts
	```

	- Check host pod-zaidanmuhammad169-managed1 in the list of hosts in the inventory file.
	```shell
	ansible pod-zaidanmuhammad169-managed1 -i inventory --list-hosts
	```

	- Check the list of hosts in the Development group.
	```shell
	ansible Development -i inventory  --list-hosts
	```

	- Check the list of hosts in the Testing group.
	```shell
	ansible Testing -i inventory  --list-hosts
	```

	- Check the list of hosts in the Indonesia group.
	```shell
	ansible Indonesia -i inventory  --list-hosts
	```

