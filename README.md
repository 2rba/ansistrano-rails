# ansistrano-rails
Setup and deploy Rails to Ubuntu server via [Ansistrano](https://github.com/ansistrano/deploy)

## Description
[Ansistrano](https://github.com/ansistrano/deploy) is a Ansible playbooks to perform deploys in a Capistrano like way.

This repo includes everything required to setup and then deploy Rails on Ubuntu server.   

## Features

### List of installed software
- Nginx
- Puma
- Ruby
- Delayed Job service

## Installation

- Install ansible
```bash
brew install ansible
``` 

- Clone this repo
```bash
cd <your project>/config/
git clone git@github.com:2rba/ansistrano-rails.git deploy
```

- Download ansistrano and rvm roles (By default it will be stored to `~/.ansible`)

```bash
cd <your project>/config/deploy/
ansible-galaxy install -r galaxy.yml
```

- To allow ssh key forwarding (to access private git repo from server) update `~/.ansible.cfg` as:  
```.env
[defaults]
# human-readable stdout/stderr results display
stdout_callback = yaml

[ssh_connection]
ssh_args = -o ForwardAgent=yes -o ControlMaster=auto -o ControlPersist=60s
```

## Configure

This repo root folder has 3 playbooks and 1 inventory file (production.yml)

Playbooks:
- `database.yml` to install mysql to Ubuntu server
- `site.yml` to prepare Rails env at Ubuntu server
- `deploy.yml` to deploy Rails to prepared server 

Each of these files is intended to run like `ansible-playbook -i production.yml deploy.yml`

Inventory file `production.yml` contains environment specific information: hosts, rails_env, etc ...
Pay attention to the group names (oldweb, database), these names are referenced from playbooks,
if are you going to change them do it in both places. 

Update `group_vars/all.yml` file with project specific variables.
Review and update each playbook you are going to use.

## Playbooks features
### database.yml
This playbook is designed to install mysql server

- Creates unprivileged user with sudo access 
- Installs mysql server
- Adds custom config (`templates/mysql.custom.j2`), feel free to review it before deploy
  

### site.yml
This playbook is designed to prepare Rails env at Ubuntu server

- Installs Ruby (check and update version in the file `site.xml`)
- Installs Nginx, mysql libs
- Creates unprivileged user with sudo access
- Creates puma service
- Creates delayed job service (feel free to remove related lines if you dont need it)
- Creates logrotate config file 

### deploy.yml
This playbook is designed to deploy Rails to prepared server. it contains ansistrano config values.
It also reference 2 tasks files `prepare.yml` and `restart.yml`
- The `prepare.yml` perform shell commands to prepare rails env (bundle, assets:precompile, etc). 
`prepare.yml` also has task to copy .env file for [dotenv gem](https://github.com/bkeepers/dotenv). 
Feel free to remove this task (`Copy env file`) if you dont need it.
- The `restart.yml` executes after `current` symlink changed. It restarts services (Puma, delayed job) and updates [whenever](https://github.com/javan/whenever) cron jobs   

## Contribution
PRs, Issues are welcome

## License
MIT

