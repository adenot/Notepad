---
layout: post
published: false
title: Faux Service Discovery with Ansible and AWS
mathjax: false
featured: false
comments: false
categories: 
  - devops
---

In a dynamic environment, servers are created and destroyed all the time. When a new server comes to life, depending on its purpose, you might need to update configuration files from other servers to tell how to access it.
A simple example would be updating your load balancer configuration by adding a new webserver or removing a webserver that was destroyed.

This article will demonstrate how to achieve this with ansible and AWS - using dynamic inventory. Updating a haproxy load balancer with information of webservers (EC2 hosts).

## Setup Dynamic Inventory

The setup will require a dynamic inventory to retrieve information about EC2 hosts. 

If you haven't setup already, take your time and check [Ansible documentation on this subject](http://docs.ansible.com/intro_dynamic_inventory.html#example-aws-ec2-external-inventory-script). Also check [my post](http://allandenot.com/devops/2015/01/16/ansible-with-multiple-inventory-files.html) where I describe how to use static and dynamic inventories together.

To test you can run ./ec2.py and should see a json with all your EC2 infrastructure there.

## Identifying Instances

To identify which EC2 hosts are going to be automatically added to haproxy, let's use EC2 tagging. 
Add a tag to your webservers called: Type = webserver

![ansiblediscovery-tag.png](/images/ansiblediscovery-tag.png)

## HAproxy configuration

Now we need a role which will deploy /etc/haproxy/haproxy.cfg

Role is compose by 2 files:
roles/haproxy/tasks/main.yml:
{% highlight yaml %}
{% raw %}
---
 - template: src=haproxy.cfg.j2 dest=/etc/haproxy/haproxy.cfg
{% endraw %}
{% endhighlight %}

roles/haproxy/templates/haproxy.cfg.j2:
{% highlight yaml %}
{% raw %}

{% endraw %}
{% endhighlight %}

When deploying to webservers, need to find the database IP and write to a file “/etc/webenv” with format: “DATABASE={database_ip_address}”, so the web application will know where to find its db.

