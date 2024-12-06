# Ansible-Handbook
* ### What is Ansible:
Ansible is an open source automation tool for tasks like configuration management, application deployment, and orchestration.
It is agentless uses pushed based mechanism, uses SSH for linux and WinRM for windows to manage systems

- ### Why use Ansible:
Ansible simplifies complex IT workflows, it is agentless no need to install software on managed nodes just share your authentication key with them.
It is Scalable, secure and easy to learn because of yaml based syntax to write playbooks.

- ### How Ansible works:
- #### Key concepts:
#### 1: Control Node:  The system where ansible is installed and working, this node is used to run ansible commands, modules, and playbooks i.e your local or a dedicated server.
#### 2: Managed Nodes: The systems that ansible will manages e.g servers, network devices, or containers.
#### 3: Inventory: A file where details of managed nodes are stored and organized into groups for easier management.
#### 4: Modules: Predefined scripts for specific tasks e.g installing packages, copying files or restarting services.
#### 5: Playbook: YAML files that contain a series of tasks for ansible to execute on managed devices.


- ### Full installation of ansible on Ubuntu:
##### First Create a user for ansible and add that user into sudo group:
`sudo adduser` <br>
`sudo usermod –aG sudo ansible`     &rarr; here ansible is user that I have created.

#### Update your OS:
`sudo apt update`

#### Install Prerequisites:
`sudo apt install -y software-properties-common`

#### Add Ansible PPA:
`sudo add-apt-repository --yes --update ppa:ansible/ansible`

#### Install Ansible:
`sudo apt install -y ansible`

#### Verify Installation:
`ansible –version`

#### Configure ansible.cfg file (etc/ansible/ansible.cgf):
`sudo nano /etc/ansible/ansible.cfg`  &rarr; you can check the location of cgf file by (ansible –version)

#### Add below configurations after Default:
`Inventory = /etc/ansible/hosts`    &rarr; Create a file named hosts and give the location <br>
`Become_user=root`   &rarr; define that become user is root so you can install packages on managed nodes.

`/ ansible all –i <inventory location> -m ping` &rarr; if you are using other inventory file

#### Add Hosts:
`sudo nano /etc/ansible/hosts` &rarr; open hosts file and create groups or add host
#### Example:
<pre>[production] &rarr; name of group <br>
192.168.0.1 <br>
192.168.0.2<br>
192.168.0.3<br>
[webservers]<br>
192.168.0.4<br>
192.168.0.5</pre>

#### Create a SSH key:
`ssh-keygen -t rsa` &rarr; create a SSH key

#### Copy SSH key into host machine:
`ssh-copy-id username@192.168.0.1` &rarr; here username is the user with whom you want to share the key and IP address of host machine.


#### Check the connection:
`ansible production –m ping` &rarr; if the ping is successful everything is good to go.

- ### Ansible Architecture:
 ![WhatsApp Image 2024-12-06 at 11 21 03 AM](https://github.com/user-attachments/assets/2a6c44f8-d571-4cb5-b428-cf4058a9b8e0)

- ### Inventory Types:
#### 1.	Static Inventory
A static inventory is a manually created file that lists all the managed hosts (nodes) and their groupings. It's typically written in INI, YAML, or JSON format.
#### 2.	Dynamic Inventory
A dynamic inventory is generated programmatically, often from cloud providers (e.g., AWS, Azure) or external systems, and provides a list of managed hosts dynamically at runtime.


#### Comparison:
|    Aspect    |    Static Inventory                  |     Dynamic Inventory                             |
|--------------|--------------------------------------|---------------------------------------------------|
|Use Case      |Small, static infrastructure          |Large, dynamic, or cloud-based infrastructure      |
|Ease of Setup |Easy, manual entry                    |Requires setup for external systems/plugins        |
|Maintenance   |Needs manual updates                  |Automatically updated                              |
|Examples      |File-based (INI/YAML/JSON)            |AWS EC2, Azure, GCP, or custom scripts             |

- ### Configuration Methods:
#### 1.	Ad-hoc commands
A quick and straightforward way to execute single or simple tasks across managed hosts without creating a playbook.
#### Example: / ansible all –m ping    &rarr; Ping all hosts
#### 2.	Modules
Modules are pre-built scripts in Ansible that perform specific tasks like installing software, copying files, managing users, and more.
#### Example: Copy, apt, user, file and many more to check run this command `(ansible –doc)`
`ansible all -m apt –a “name:VLC state:present”`   &rarr;will install VLC on all hosts
Note: -b (used to become root), -m (indicates that it is a module), -a (passing arguments)
#### 3.	Playbooks
Playbooks are YAML files containing a series of tasks to be executed on managed hosts. They are reusable and designed for complex automation workflows.
Playbooks should be save as YAML or YML
#### Structure:
1.	Target:
2.	Variable
3.	Task
4.	Handler & Dependency
#### Example:
<pre> 
---
- Name: My playbook
Host: all
Become: yes
Gather_facts: yes
Tasks:
  - name: install vlc
    Copy:
      Src:File location
      Dest: where to copy location 
</pre>
#### Comparison:
|Aspect              |Ad-hoc Commands      |Modules                  |Playbooks                       |
|--------------------|---------------------|-------------------------|--------------------------------|
|Purpose             |Quick, one-time tasks|Building blocks for tasks|Complex workflows and automation|
|Ease of Use         |Very simple          |Simple                   |Requires YAML knowledge         |
|Reusability         |Not reusable         |Partially reusable       |Fully reusable                  |
|Complexity Supported|Low                  |Medium                   |High                            |

# Bonus
- ## Adding windows Host
- #### Create a user named ansible: In windows host

- #### Enable WinRM on the Windows Host:

#### 1: Open PowerShell as Administrator and run below commands:
<pre>
Enable-PSRemoting -Force
Set-Item wsman:\localhost\client\trustedhosts -Value '*'
Enable-WSManCredSSP -Role Server -Force
Enable-WSManCredSSP -Role Client -DelegateComputer "*"
</pre>

#### 2: Configure WinRM Basic Authentication:
<pre>
winrm set winrm/config/service/auth ‘@{Basic="true"}’
winrm set winrm/config/service ‘@{AllowUnencrypted="true"}’
</pre>
#### 3: Open Firewall Ports:
`New-NetFirewallRule -Name "WinRM HTTP" -DisplayName "WinRM HTTP" -Protocol TCP -LocalPort 5985 -Action Allow`

#### 4: Install the required Python libraries:
`pip install pywinrm`

#### 5: Configure the Ansible Inventory:
<pre>
[windows]
windows_host ansible_host=Windows_IP

[windows:vars] 
ansible_user=username
ansible_password=password
ansible_connection=winrm 
ansible_winrm_transport=basic 
ansible_port=5985
</pre>
#### 6: Test the Connection on control node:
`ansible win -m win_ping`

