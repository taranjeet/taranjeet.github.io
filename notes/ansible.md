## Ansible 101

* Infrastructure as a code

### Features

* Agent Less - does not install any agent on the remote host. Uses ssh to manage and provision systems

* Push based approach to reflect changes on system

* Idempotent - run the same command twice and effect remains the same

### Important Concepts

#### Inventory

Used to specify meaningful group of hosts

#### Modules

It is a way to accomplish a task. Use available context to determing what actions to execute on the state of host machine Eg apt for Debian, yum for centos.

#### Playbook

This is what Ansible speaks. It is used to run multiple tasks. It can be termed as Ansible's configuration, deployment and orchestration language.

#### Roles

Organize a playbook into resuable files.

#### Variables

Holds certain value during program execution

#### Templates

To accesss variables and enable dynamic expression. Uses Jinja templating language

#### Conditionals

Execute a certain task only when some condition is met. 

* when - eg: create mysql user if mysql is installed and running
* loop - eg: create several databases in mysql

#### Handlers

Just like task. But event based. Tasks notifies a handler. Eg: Start nginx after it is installed.

#### Facts

Gather info about the system like no of cores, linux distribution, etc.

#### Vault

Store sensitive data like database password.

### References

* [An Ansible Tutorial](https://serversforhackers.com/c/an-ansible-tutorial)
* [Getting Started with Ansible](https://scotch.io/tutorials/getting-started-with-ansible)
