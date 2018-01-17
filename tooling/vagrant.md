# Typo 3 Dev Environment with Vagrant and Homestead

With dev environments there are basically two choices:

- install everything you need on your local dev machine and run the dev sites on your machine (e.g. MAMP, XAMPP)
- use some kind of virtual machine that has everything in it and reproduces the environment the live site will run in

This guide focuses on the latter.

## Used Software

- [VirtualBox](https://www.virtualbox.org/), provides the virtual machine infrastructure
- [Vagrant](https://www.vagrantup.com/), a command line utility for managing headless virtual machines
- [Homestead](https://laravel.com/docs/homestead), a pre-packaged vagrant box that provides everything needed to work with PHP projects

## Setup

First install VirtualBox and Vagrant as per the instructions on their respective websites.