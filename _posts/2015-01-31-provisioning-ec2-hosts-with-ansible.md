---
layout: post
published: false
title: Provisioning EC2 Hosts with Ansible
mathjax: false
featured: false
comments: false
categories: 
  - devops
tags: ansible ec2 aws
---

Looking to build EC2 hosts with more consistency? Using Ansible you can easily provision EC2 hosts and put some logic on it to adjust EC2 parameters based on the type of host you are building.

The easiest way to start is to create a playbook calling the *ec2* module with the parameters you want to pass to AWS to create your host. In this post I will show a little more scalable way to do this, where the parameters are variables and you can easily have multiple types of hosts sharing the same playbook and role.

The solution is organized in 3 parts:
1. A generic Ansible role that uses *ec2* module to provision
2. Yaml files with variables that will be used as parameters for each type of EC2 host
3. Playbook that combines the variables file with the role

All code is in a GitHub repository: https://github.com/adenot/ansible-blog-provision-ec2

## Ansible Role

Create a folder for the role:
    mkdir -p roles/provision-ec2/tasks

Name the file below as *main.yml* and add to the folder *roles/provision-ec2/tasks/main.yml*

{% highlight yaml %}
{% raw %}
---
 - name: Provision EC2 Box
   local_action:
     module: ec2
     key_name: "{{ ec2_keypair }}"
     group_id: "{{ ec2_security_group }}"
     instance_type: "{{ ec2_instance_type }}"
     image: "{{ ec2_image }}"
     vpc_subnet_id: "{{ ec2_subnet_ids|random }}"
     region: "{{ ec2_region }}"
     instance_tags: '{"Name":"{{ec2_tag_Name}}","Type":"{{ec2_tag_Type}}","Environment":"{{ec2_tag_Environment}}"}'
     assign_public_ip: yes
     wait: true
     count: 1
     volumes: 
     - device_name: /dev/sda1
       device_type: gp2
       volume_size: "{{ ec2_volume_size }}"
       delete_on_termination: true
   register: ec2

 - debug: var=item
   with_items: ec2.instances

 - add_host: name={{ item.public_ip }} >
             groups=tag_Type_{{ec2_tag_Type}},tag_Environment_{{ec2_tag_Environment}}
             ec2_region={{ec2_region}} 
             ec2_tag_Name={{ec2_tag_Name}}
             ec2_tag_Type={{ec2_tag_Type}}
             ec2_tag_Environment={{ec2_tag_Environment}}
             ec2_ip_address={{item.public_ip}}
   with_items: ec2.instances

 - name: Wait for the instances to boot by checking the ssh port
   wait_for: host={{item.public_ip}} port=22 delay=60 timeout=320 state=started
   with_items: ec2.instances

{% endraw %}
{% endhighlight %}

## Variables Files

These are YAML files that will be included by the playbook before calling the role above. It needs to fill all variables used in the *provision-ec2* role otherwise it will fail.

Create a folder for the variables:
    mkdir ec2-vars
    
In this example we will have a webservers.yml file to simulate provisioning a webserver host in AWS.

ec2-vars/webservers.yml:

{% highlight yaml %}
{% raw %}


{% endraw %}
{% endhighlight %}