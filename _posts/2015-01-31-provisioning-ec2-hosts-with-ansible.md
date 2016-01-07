---
layout: post
published: true
title: Provisioning EC2 Hosts with Ansible
mathjax: false
featured: true
comments: true
categories: 
  - devops
tags: ansible ec2 aws
description: ""
headline: ""
modified: ""
imagefeature: ""
---


Looking to build EC2 hosts with more consistency? Using Ansible you can easily provision EC2 hosts and put some logic on it to adjust EC2 parameters based on the type of host you are building.

The easiest way to start is to create a playbook calling the *ec2* module with the parameters you want to pass to AWS to create your host. In this post I will show a little more scalable way to do this, where the parameters are variables and you can easily have multiple types of hosts sharing the same playbook and role.

The solution is organized in 3 parts:

1. A generic Ansible role that uses *ec2* module to provision
2. Yaml files with variables that will be used as parameters for each type of EC2 host
3. Playbook that combines the variables file with the role

All code is in a GitHub repository: [https://github.com/adenot/blog-ansible-provision-ec2](https://github.com/adenot/blog-ansible-provision-ec2)


## Setup Environment

Ansible's EC2 module uses python-boto library to call AWS API, and boto needs AWS credentials in order to function.

There are many ways to set your AWS credentials. One of them is to create a file under your user home folder:

    touch ~/.boto

Then edit the file and add the following:

    [Credentials]
    aws_access_key_id = REDACTED
    aws_secret_access_key = REDACTED

For more information, [check Boto documentation](http://boto.readthedocs.org/en/latest/boto_config_tut.html).
To learn how to create AWS credentials, [check this documentation](http://docs.aws.amazon.com/IAM/latest/UserGuide/ManagingCredentials.html#Using_CreateAccessKey).

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
ec2_keypair: "REDACTED"
ec2_security_group: "REDACTED"
ec2_instance_type: "m3.medium"
ec2_image: "ami-9eaa1cf6"
ec2_subnet_ids: ['subnet-REDACTED','subnet-REDACTED','subnet-REDACTED']
ec2_region: "us-east-1"
ec2_tag_Name: "Webserver"
ec2_tag_Type: "webserver"
ec2_tag_Environment: "production"
ec2_volume_size: 16
{% endraw %}
{% endhighlight %}

Change the **REDACTED** values above to your AWS account ones. You can easily find by inspecting a EC2 host (using AWS console) that you want to automate it's provisioning.

You can have multiple variable files, one for each type of EC2 host.

## Playbook

Create a playbook inside ansible playbooks root folder called *provision-ec2.yml*, with the contents:

{% highlight yaml %}
{% raw %}
---
 - hosts: localhost
   connection: local
   gather_facts: false
   user: root
   pre_tasks:
    - include_vars: ec2_vars/{{type}}.yml
   roles:
    - provision-ec2 
{% endraw %}
{% endhighlight %}

Notice that the *type* variable above is not defined. Depending on the value of the parameter, Ansible will include different a variables file, thus populating the parameters used in the *provision-ec2* role.

The type will be defined at run time.

## Running

Call *ansible-playbook* passing the *type* parameter as an argument:

    ansible-playbook -vv -i localhost, -e "type=webservers" provision-ec2.yml
    
If your variables are correct, you should see a new host at your AWS console.

## GitHub

All code is available at:

[https://github.com/adenot/blog-ansible-provision-ec2](https://github.com/adenot/blog-ansible-provision-ec2)
