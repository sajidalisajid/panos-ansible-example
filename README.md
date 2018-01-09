### Prerequisites 
This tutorial will take around 15 minutes to complete, once all components are setup and all dependencies have been installed. For this tutorial we will need:

##### Palo Alto Firewall
This can be a physical or virtual appliance, it makes no difference. You can download a virtual appliance from the link here: https://docs.gns3.com/appliances/pan-vm-fw.html#appliance_supported. We won't need any licenses to complete this tutorial, so once we have downloaded this file and installed it as a virtual machine, no further setup is required.
	
##### Ansible

On a Unix machine, this is wonderfully easy to setup, and can be done on a Debian distro with the following commands:
``` $ sudo apt-get update
	$ sudo apt-get install software-properties-common
	$ sudo apt-add-repository ppa:ansible/ansible
	$ sudo apt-get update
	$ sudo apt-get install ansible  	
```
To see installation instruction for other distros (and Mac OSX), you can use the official instructions: http://docs.ansible.com/ansible/latest/intro_installation.html

For windows clients, it is also possible. But it involves installing Cygwin. Speaking from experience, having written too much C code on a windows client via. Cygwin, I recommend installing a virtual box Ubuntu and saving yourself the trouble and eventual tears. However, if you insist, instructions for installing Ansible through Cygwin can be found here: https://www.jeffgeerling.com/blog/running-ansible-within-windows

##### Your Favourite IDE / Text Editor
I use Visual Studio. It's free, it's pretty damn good and easy to install: https://code.visualstudio.com/

All in all, it doesn't really matter which editor you use, just use whatever you like the most.

### Setting up the Playbook

So, know we have ansible installed and a completely blank Palo Alto. In the next few steps, we will write a file, that will define define a new rule for our palo alto. We will write these definitions in YAML (YAML Ain't Markup Language), which is a very simple markup language, not unlike JSON or XML. These YAML definition files are, for Ansible, called 'Playbooks'. Each CM (Configuration Manager), seem to have their own word for this, Chef calls it a 'Recipe' and Puppet calls it a 'Manifest'. I'm sure there are as many names for it, as there are CM's... Either way, it doesn't actually matter, all we need to know is that it is a document which defines what a configuration should look like (or which operations should be taken). 

So let's start by writing a small 'Playbook' for creating a rule on our Palo Alto firewall:

```ruby
# addrule.yml

# All Ansible playbooks start with: ---
---
# This defines which machines the playbook will run on
- hosts: localhost
  
  # This defines which libraries are used for the playbook
  roles:
    - role: PaloAltoNetworks.paloaltonetworks

  # This defines the different actions of the playbook
  tasks:
  - name: include variables (free-form)
    include_vars: vars.yml
    no_log: "no"
    
  # This is the rule we will add to our palo alto
  - name: permit ssh to 1.1.1.1
    panos_security_rule:
      operation: "add"
      ip_address: "{{ mgmt_ip }}"
      password: "{{ admin_password }}"
      rule_name: 'SSH permit'
      description: 'SSH rule test'
      source_zone: ['untrust']
      destination_zone: ['trust']
      source_ip: ['any']
      source_user: ['any']
      destination_ip: ['1.1.1.1']
      category: ['any']
      application: ['ssh']
      service: ['application-default']
      hip_profiles: ['any']
      action: 'allow'
```

In this document we have defined that the playbook will run on 'localhost'. Usually, this is where you would specify which ansible server (or group of servers) you want to execute this script. But in our case, we are fine with this running on our local machine.

Secondly, we import the `PaloAltoNetworks.paloaltonetworks` role, which is like a module or library for playbooks.

Lastly, we define the tasks that we wish to perform, whenever this playbook is run. In this case, we import a variable file (don't worry we will get to what this means in just 2 seconds) and then define a task to 'add' a rule that allows ssh to the destionation ip 1.1.1.1

Before we can run this document, we will ensure that we have the Palo Alto role (module), installed on our Ansible host, with the following command:

`$ ansible-galaxy install PaloAltoNetworks.paloaltonetworks`

Aaaaaaand, we will need to explain the variables file. So, essentially a variables files is just a file that has defined some... variables. There can be many reasons to have a variables file. For example, we can re-use these variables across multiple playbooks, ensuring that if the variables are changed, they are changed across all playbooks (rather than having to individually change these variables manually for every playbook we have written). It could also be beneficial in situations where you want to reuse playbooks, but with different variables (perhaps due to differences between offices of locations). Either way, let's make a variables file, and while we are at it, we will even encrypt it to ensure that all our secrets are kept safe.

We will create this encrypted file, with Ansible's built-in 'Vault':

`$ ansible-vault create vars.yml`

This will open your default editor (which by default I believe is vi). If you aren't familiar with vi and just want to get through this tutorial, I recommend switching the default editor to nano (temporarily of course, vi is life) by running the following command before running the ansible-vault command: `export EDITOR=nano'

Once you have given your vars.yml file a password, enter in the following information:

```ruby
---
mgmt_ip: <IP Address of Palo Alto Management Interface>
admin_password: <Password for admin user>
```
If you don't care about encryption right now. You can also omit using ansible-vault and instead just create a normal vars.yml file with the same content as above. If you need to edit the variables file, use the following command:

`$ ansible-vault edit vars.yml`

Great! Now we can finally run our playbook! Wohooo:

`$ ansible-playbook testbook.yml`

