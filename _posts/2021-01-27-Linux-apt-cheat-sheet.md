---
layout: post
title: Linux APT cheat sheet
subtitle: It can be hard to remember everything
tags: [linux, apt]
comments: true
---

## The basics
Linux uses **packages** to install **software**.

*What is a package?*  
A package is a program to be installed as well as its **dependencies**, i.e. the libraries (which are also programs) needed to run the software.
So when you install a software (from a package), you will also install all its dependencies.

*OK, but how do I find a package?*  
Packages are indexed in **repositories**. So, when you want to install a software, you have to query a repository to find the desired pacakge.  
This is automatically handled by your *package manager* that uses your list of repositories `/etc/apt/sources.list` to search for packages.

*Package manager? I'm lost*  
Package managers are programs that allow to manage the packages of your OS. With package managers, you can install, update and remove packages.  
Linux provides several package managers, the most common are `apt-get` and `apt`.

## Cheat sheet

#### APT
x

| Name | Command |
| :------ |:--- |
| Update list of available packages in repositories | `sudo apt update` |
| Upgrade installed packages | `sudo apt upgrade` |
| Remove useless packages | `sudo apt autoremove` |
| Search for a package | `sudo apt search <package_name>` |
| Install a package | `sudo apt install <package_name>` |
| Install multiple packages | `sudo apt install <package_name1> <package_name2>` |
| Remove a package | `sudo apt remove <package_name>` |
| List installed packages | `sudo apt list --installed` |
| Find installed package | `sudo apt list --installed | grep <package_name>` |
| View package information | `sudo apt show <package_name>` |
