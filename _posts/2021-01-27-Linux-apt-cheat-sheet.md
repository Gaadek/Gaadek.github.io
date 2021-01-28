---
layout: post
title: Linux APT cheat sheet
subtitle: It can be hard to remember everything
tags: [linux, apt]
---

## What is a APT?
Apt stands for **Advanced Packaging Tool**. This software allows to manage the installation, updating and removal of packages in a Debian based Linux system. Such a tool is called a **package manager** and there are plenty of them, the most well known being `apt-get` and `apt`.

Behind the hood, apt interacts with **dpkg**, the Debian packaging system whose purpose is to pack software in an easy to install entity. Apt offers a user friendly tool for interacting with dpkg.

## What is a package?
A package is a software to be installed as well as its **dependencies**, i.e. the neeed libraries.

## How do I find packages?
Packages are indexed in **repositories**. When you want to install a software, you have to query a repository to find the needed package.  
A **package manager** handles this for you by using a list of repositories (stored here: `/etc/apt/sources.list`) .

## How do I use apt?

Below a list of the most common apt commands 

| I want to... | ...run the folowing command |
| :------ |:--- |
| Update list of available packages in repositories | `sudo apt update` |
| Upgrade all installed packages | `sudo apt upgrade` |
| Remove useless packages | `sudo apt autoremove` |
| Search for a package | `sudo apt search <package_name>` |
| Install a package | `sudo apt install <package_name>` |
| Install multiple packages | `sudo apt install <package_name1> <package_name2>` |
| Remove a package | `sudo apt remove <package_name>` |
| List installed packages | `sudo apt list --installed` |
| Filter installed package | `sudo apt list --installed | grep <package_name>` |
| View package information | `sudo apt show <package_name>` |
