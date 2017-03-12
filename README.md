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

## Usage

You'll need to:

* Set a playbook variable
* Ensure your uberjar gets moved to the correct location
* Ensure you app responds to the commands for starting a server and
  migrating a database

### Playbook Setup

At a minimum, you'll need to create a playbook that uses the Sweet
Tooth roles and sets the _clojure_uberjar_webapp_domain_ variable:

```yaml
---
- hosts: webservers
  become: true
  become_method: sudo
  vars:
    clojure_uberjar_webapp_domain: "localhost"
  roles:
    - "sweet-tooth-clojure.clojure-uberjar-webapp-common"
    - "sweet-tooth-clojure.clojure-uberjar-webapp-app"
    - "sweet-tooth-clojure.clojure-uberjar-webapp-nginx"

- hosts: database
  become: true
  become_method: sudo
  vars:
    clojure_uberjar_webapp_domain: "localhost"
  roles:
    - "sweet-tooth-clojure.clojure-uberjar-webapp-common"
    - "sweet-tooth-clojure.datomic-free"
```

In this example, _clojure_uberjar_webapp_domain_ is set to
`"localhost"`, which is appropriate if you're trying this on a local
VM. For a VPS, you'll want to use your site's domain name. Many other
variables are derived from _clojure_uberjar_webapp_domain_, including
the variable used to set the nginx config's `server_name`. Check out
the READMEs of each of the other repos to see what other vars can get
configged. The roles were designed so that nearly everything can be
customized.

The app deployment task expects your app's uberjar to live at
_files/app.jar_ relative to your Ansible playbook. If your playbook is
under _infrastructure/ansible/sweet-tooth.yml_, then you need to copy
your uberjar to _infrastructure/ansible/files/app.jar_.

### App Commands

These roles create an Upstart script that starts your Clojure uberjar
with, essentially:

```
java -jar /path/to/uberjar.jar server
```

They perform database migrations with:

```
java -jar /path/to/uberjar.jar db/migrate
```

Finally, they perform a post-deployment check with

```
java -jar /path/to/uberjar.jar deploy/check
```


Therefore, your Clojure application's `-main` function must be able to
respond to these command line arguments. Here's the `-main` function
from the character sheet example:

```clojure
(defn -main
  [cmd & args]
  (case cmd
    "server"
    (component/start-system (system))
    
    "db/install-schemas"
    (final
      (let [{:keys [db schema data]} (config/db)]
        (d/create-database db)
        (datb/conform (d/connect db)
                      schema
                      data
                      config/seed-post-inflate)))

    "deploy/check"
    ;; ensure that all config vars are set
    (final (config/full))))
```

Ignore `final`; it just does some error handling. The main thing to
note is that the `-main` function's first argument, `cmd`, corresponds
to the first command line argument. The `-main` function switches on
`cmd` and evaluates the appropriate expression.
