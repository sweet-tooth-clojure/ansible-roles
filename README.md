# Sweet Tooth Ansible Roles

This is the documentation for the Sweet Tooth Ansible Roles, a
collection of tools that allow you to **easily deploy your Clojure web
site**. Here, you'll find concise instructions on using the tools. If
you're new to Ansible or to server management/devops,
[_Deploying Your First Clojure App ...From the Shadows_](http://www.braveclojure.com/quests/deploy/)
is a 70-page book for beginners that explains everything you need to
know to get your site online.

## What is this?

The purpose of this project is to allow developers to easily and
reliably set up machines to run their Clojure web sites. I want it to
be possible for devs to set up a VM on a service like
[Digital Ocean](https://m.do.co/c/7c5112400186) with only a couple
commands.

The project includes four
[Ansible roles](http://www.braveclojure.com/quests/deploy/ansible-tutorial/#Ansible_Roles)
for setting up a server for Clojure- and Datomic-based web apps, and
for actually doing deployments. These roles are:

* [clojure-uberjar-webapp-common](https://github.com/sweet-tooth-clojure/ansible-role-clojure-uberjar-webapp-common)
  a lightweight role that contains variable definitions common to the
  rest of the roles
* [clojure-uberjar-webapp-app](https://github.com/sweet-tooth-clojure/ansible-role-clojure-uberjar-webapp-app):
  a role that configures a server to run a standalone java program as
  a web server
* [clojure-uberjar-webapp-nginx](https://github.com/sweet-tooth-clojure/ansible-role-clojure-uberjar-webapp-nginx):
  a role that installs and configures an nginx server to reverse-proxy
  an application server
* [datomic-free](https://github.com/sweet-tooth-clojure/ansible-role-datomic-free)
  a role that installs Datomic Free and configures it to run as an
  Upstart service

## Requirements

You must have
[Ansible installed](http://docs.ansible.com/ansible/intro_installation.html). If
you want to try the scripts out on a local VM, consider using
[Vagrant](https://www.vagrantup.com/). If you're unfamiliar with these
tools,
[_Deploying Your First Clojure App ...From the Shadows_](http://www.braveclojure.com/quests/deploy/)
explains how to use them.

## Installation

Run the command `ansible-galaxy install
sweet-tooth-clojure.clojure-uberjar-webapp-common` for each role you'd
like to install, replacing `clojure-uberjar-webapp-common` for each
role. You'll need to use at least `-common`, `-app`, and `-nginx`.

## Demo

See the instructions for
[downloading an example app, provisioning, building, and deploying](http://www.braveclojure.com/quests/deploy/set-up-a-server-and-deploy-a-clojure-app-to-it/#Provision_and_Deploy_to_Prod)
for a short list of commands that you can run to see these tools in
action. This uses the
[character sheet example Clojure application](https://github.com/sweet-tooth-clojure/character-sheet-example),
a fully-functioning Clojure app that's been configured to use the
Ansible roles.

## Basic Usage

