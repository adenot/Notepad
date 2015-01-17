---
layout: post
published: true
title: Ansible with multiple inventory files
mathjax: false
featured: false
comments: true
modified: "2015-01-16"
categories: 
  - devops
tags: ansible
---

The power of Ansible always surprises me. In this post I will be sharing the way you can use multiple inventory files when running your playbooks and how to organize them.

**TL;DR: Inventory can be a folder. Create a folder, add as many inventory files inside this folder and instruct Ansible to use this folder as the inventory (with -i folder_name or in your ansible.cfg file). All inventory files inside the folder will be merged into one (including scripts like ec2.py).**

## Inventory as a folder

Inside your playbooks folder, create a folder called *inventory*.

    mkdir inventory
    
Now move your current inventory file (often called *hosts*) into this folder

    mv hosts inventory/
    
To tell ansible to use the new inventory, there are 2 options:

#### Using ansible.cfg

Create a file ansible.cfg within the same folder as your playbooks and add the lines:

    [defaults]
    hostfile = inventory

#### Command line argument

Next time you run your playbook, simply add *-i inventory*:

    ansible-playbook -i inventory my_playbook.yml

And that's it to start. 
Now the fun part.

## Dynamic and static inventories together

Here's the trick, add both your static and dynamic inventories to the folder and you can run playbooks targeting both as same time.

#### Example: Group EC2 hosts by country

EC2 hosts from both us-east-1 and us-west-1 need to belong to a **USA** group, and both eu-west-1 and eu-central-1 to belong to **Europe** group.

Add your *ec2.py* to the inventory folder and make sure it's executable.

    cp /usr/local/lib/python2.7/dist-packages/ansible/module_utils/ec2.py inventory/
    chmod +x inventory/ec2.py

*[Check ansible documentation for help to setup ec2.py](http://docs.ansible.com/intro_dynamic_inventory.html#example-aws-ec2-external-inventory-script)*

Create new file *countries* and add to inventory folder.

    touch inventory/countries
    
*countries* file contents:

    # Declaring country groups
    [USA]
    [Europe]
    
    # Attaching EC2 groups as children
    [USA:children]
    us-east-1
    us-west-1
    
    [Europe:children]
    eu-west-1
    eu-central-1
    
Now when you run:

    ansible-playbook -i inventory --limit "Europe" my_playbook.yml
    
Will run *my_playbook.yml* only to your european servers.