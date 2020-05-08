+++
title = "Ldap Triggers"
date = 2020-05-09T00:32:37+02:00
draft = false
author = "Marc Benedi"
authorTwitter = "marcbenedi"
cover = ""
tags = ["Project", "Linux"]
keywords = ["LDAP", "Trigger"]
description = "LDAP Triggers is a python package which allows executing an arbitrary script when an LDAP entity modification is detected."
showFullContent = false
+++

> **For up to date information check the project's repository: [LDAP Triggers][2] (https://github.com/marcbenedi/ldap-triggers)**

# Abstract

[LDAP Triggers][2] (https://github.com/marcbenedi/ldap-triggers) is an Open Source python package under MIT license which triggers an action every time an LDAP action is done. The current supported actions are **creation/deletion** of **users/groups**.

It comes with very clear documentation and it's very easy to set up. 

# What is LDAP?

[LDAP (Lightweight Directory Access Protocol)][1] is a protocol for accessing and managing a ditributed directory information over a network. The most common use case could be managing users and groups of a company over multiple machines. It would be impractical creating all users in all machines because each time an addition/modification/deletion of a user or machine would be required, it would be to be done multiple times. That is were LDAP joing the game. 

# Why would I need a trigger?

Executing an arbitrary script every time an entity modification is detected can be very useful: The range of applications goes from sending an email when their account is created to deleting their home directories when their account is deleted. 

By default LDAP does not come with this feature. It could be that some private vendors offer it in their custom implementation but this comes with the drawback of using propietary software. [LDAP Triggers][2] is Open Source (MIT License) so you are free to use it and extend.


# LDAP Triggers

## Motivation and approach

Since I could not find any solution for this problem, I decided to start this Open Source project so more people can benefit from it. 

The package is very lightweight and does not have boilerplate features. It has a single purpose: Execute scripts placed in a specific path.

## Installation and usage

Please, check the [repository][2] for up to date information and detailed instructions. I decided not to write this here because it would become obsolete. 

## Flow

{{< figure src="ldap-triggers-flow-background.png" alt="Visualization of LDAP Triggers execution flow" position="center" style="border-radius: 8px;" captionPosition="center" caption="Visualization of LDAP Triggers execution flow">}}

## Triggers

The name and the path where the scripts are stored is very important because it's where the package will search for them. The path is `/etc/ldaptriggers/triggers/` and the file name should follow the next pattern: 

**\[action\]\_\[entity\]\_\[description\]**.bash

*The extension is not relevant.*

For example, this would be a valid trigger name: `add_people_send_email.bash`. Of course, it is necessary that the script is executable by root. This is because the deamon is run by root.

## Configuration options

The configuration file is stored in `/etc/ldaptriggers/config.yaml`. It is recommended to execute `$ ldaptriggers --init` after installing the package because it will ask you the configuration parameters:

- ldap_uri: The LDAP server uri
- ldap_secret: The path with the LDAP admin password
- org: Directory
- admin: Admin user
- people: Users
- groups: Groups
- timeout: Wait time between fetches

## Examples

In the [repository][2], in the `examples` folder you can find examples of **triggers**, and **configuration file** among others.

<!-- # References -->

[1]: https://en.wikipedia.org/wiki/Hobbit#Lifestyle
[2]: https://github.com/marcbenedi/ldap-triggers