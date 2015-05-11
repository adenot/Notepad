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

This article will demonstrate how to achieve this with ansible and AWS - using dynamic inventory. Updating a haproxy load balancer with information of EC2 hosts.

## Setup Dynamic Inventory

The setup will require a dynamic inventory to retrieve information about EC2 hosts. 

If you haven't setup already, take your time and follow Ansible documentation. Also check [my post](http://allandenot.com/devops/2015/01/16/ansible-with-multiple-inventory-files.html) where I describe how to use static and dynamic inventories together.

We tag the database server in EC2 with a tag Type=database. 

When deploying to webservers, need to find the database IP and write to a file “/etc/webenv” with format: “DATABASE={database_ip_address}”, so the web application will know where to find its db.

