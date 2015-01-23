---
layout: post
published: false
title: "Ansible with AWS: from EC2 to Autoscale"
mathjax: false
featured: true
comments: false
categories: 
  - devops
tags: ec2 aws cloudwatch autoscaling ansible
---

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

I had a run down on this subject in a previous post: [Ansible with Multiple Inventory Files](http://allandenot.com/devops/2015/01/16/ansible-with-multiple-inventory-files.html), but I will copy/paste the most important bits here with an extended explanation for the context of this article.



## Creating EC2 hosts

## Running Playbooks on just created EC2 hosts

## Creating Cloudwatch alarms dynamically

## Baking AMIs

## Autoscaling

## Autoscaling lifecycle
