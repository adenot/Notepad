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

## Dynamic and static inventory files together

Here's the trick, add both your static and dynamic inventories to the folder and you can run playbooks targeting both as same time.

#### Example