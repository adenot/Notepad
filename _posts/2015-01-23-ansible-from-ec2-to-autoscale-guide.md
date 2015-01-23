---
layout: post
published: false
title: "Ansible with AWS: from EC2 to Autoscale"
mathjax: false
featured: false
comments: false
categories: 
  - devops
tags: ec2 aws cloudwatch autoscaling ansible
---

**ATTENTION: THIS ARTICLE STILL BEING WRITTEN.**

## Intro

If you are here, you probably use Ansible already, or is thinking about using it. 
So let me tell you: Ansible is great, mainly because it's simple and still powerful enough that even after working with it for a year, I always find new tricks.

One of these tricks I will describe here and will help you to:
- Get rid of your static hosts file and move to a dynamic based one
- Create EC2 hosts directly from Ansible
- Setup monitoring on these hosts automatically based on any criteria
And for the grand finalle:
- Autoscale

## Dynamic Inventory

I had a run down on the subject of inventories in a previous post: [Ansible with Multiple Inventory Files](http://allandenot.com/devops/2015/01/16/ansible-with-multiple-inventory-files.html), but I will copy/paste the most important bits here with an extended explanation for the context of this article.

### Start with an inventory folder

First tip is to create a folder called *inventory* and move your hosts file inside. When you run playbooks you can point the inventory to the folder instead of the hosts file. 

Ansible automatically merge all inventory files inside the folder into one big inventory. This is perfect when you are switching from a static inventory to a dynamic one.

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
    
### Dynamic and static inventories together

Here's the trick, add both your static and dynamic inventories to the folder and you can run playbooks targeting both as same time.

#### Copying EC2.py to inventory folder

Add your *ec2.py* to the inventory folder and make sure it's executable.

    cp /usr/local/lib/python2.7/dist-packages/ansible/module_utils/ec2.py inventory/
    chmod +x inventory/ec2.py

*[Check ansible documentation for help to setup ec2.py](http://docs.ansible.com/intro_dynamic_inventory.html#example-aws-ec2-external-inventory-script)*

It's very important to add execute permission to ec2.py. That's how Ansible knows it has to **execute** the inventory file and get the results as JSON instead of read it and parse as an INI-like file.

It means that you can put any script inside the inventory folder and give execute permission (*ex.: chmod +x myscript.php*), Ansible will execute, grab the JSON results and use it as your inventory.

## Creating EC2 hosts

The generic role I use to create EC2 hosts is:

To run the role from a playbook, in this example, a webserver:

provision-webserver.yml:

---

{% highlight yaml %}
{% raw %}
---
 - hosts: localhost
   connection: local
   gather_facts: false
   vars:
     - ec2_keypair: "XX-KEYPAIR-NAME-XX" # To be replaced
     - ec2_security_group: "sg-XXXXXXXX" # To be replaced
     - ec2_instance_type: "t2.micro"
     - ec2_image: "ami-1711732d"    # Ubuntu 14.04
     - ec2_subnet_ids: [ 'subnet-XXXXXXXX', 'subnet-XXXXXXXX' ] # To be replaced
     - ec2_region: "ap-southeast-2" # or us-east-1 for Virginia US
     - ec2_tag_Name: "Webserver"    # Tag Name
     - ec2_volume_size: "8"         # Size in GB
   tasks:
    - name: Provision EC2 Box
      local_action:
        module: ec2
        key_name: "{{ ec2_keypair }}"
        group_id: "{{ ec2_security_group }}"
        instance_type: "{{ ec2_instance_type }}"
        image: "{{ ec2_image }}"
        vpc_subnet_id: "{{ ec2_subnet_ids|random }}"
        region: "{{ ec2_region }}"
        instance_tags: '{"Name":"{{ec2_tag_Name}}"}'
        assign_public_ip: yes
        wait: true
        count: 1
        volumes:
         - device_name: /dev/sda1
           device_type: gp2
           volume_size: "{{ ec2_volume_size }}"
           delete_on_termination: true
      register: ec2
{% endraw %}
{% endhighlight %}

## Running Playbooks on just created EC2 hosts

## Creating Cloudwatch alarms dynamically

## Baking AMIs

## Autoscaling

## Autoscaling lifecycle
