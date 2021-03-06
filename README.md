Bitlancer Strings Documentation
=====================

We built our industry experience into an open source project that manages cloud infrastructure and code deployment. Strings is primarily a point-and-click control panel targeted towards those searching for easy infrastructure management "out of box" for Rackspace Cloud and OpenStack. It manages the entire lifecycle of infrastructure, including creation, "puppetization", code deployment, ongoing maintenance, and deprovisioning. Sound useful to you? Continue reading.

## Table of Contents
- [Overview](#overview)
- [Screenshots & Data Sheet](#screenshots--data-sheet)
- [Disclaimer](#disclaimer)
- [Feedback & Contributions](#feedback--contributions)
- [Getting Started](#getting-started)
- [Devices](#devices)
  - [Device Names](#device-names)
  - [Public DNS Entries](#public-dns-entries)
- [Formations](#formations)
  - [Blueprints](#blueprints)
- [Applications](#applications)
  - [Application DNS](#application-dns)
  - [Application Deployment](#application-deployment)
    - [Jump Server](#jump-server)
    - [Deploy User](#deploy-user)
    - [Using the Strings Deploy Toolkit](#using-the-strings-deploy-toolkit)
    - [Using a Custom or 3rd Party Deploy Framework](#using-a-custom-or-3rd-party-deploy-framework)
- [Configuration Management](#configuration-management)
  - [Puppet](#puppet)
  - [Hiera](#hiera)
  - [Roles, Profiles, & Components](#roles--profiles--components)
- [User Management](#user-management)
  - [Control Panel Privileges](#control-panel-privileges)
  - [Infrastructure Privileges](#infrastructure-privileges)
    - [SSH Key Management](#ssh-key-management)
    - [Sudo Privileges & Sudo Roles](#sudo-privileges--sudo-roles)
    - [Granting Privileges](#granting-privileges)
    - [Credential & Sudo Privilege Caching](#credential--sudo-privilege-caching)
- [Troubleshooting](#troubleshooting)
  - [I can't login to a server](#i-cant-login-to-a-server)

## Overview

### Flexible Integration

Strings integrates with Rackspace and (theoretically) OpenStack and uses Puppet, OpenLDAP, PowerDNS, and other open source tools to make launching, configuring, managing, and destroying servers a piece of cake. Automate package installation and configuration for almost anything, letting you and your Engineers focus on building and shipping code!

### Central Authentication

No more shell scripts to update and replace SSH keys. Control who can access your infrastructure and what permissions they have. The level of complexity is up to you! Manage users easily right from the (actually simple) web interface.

### Automation

Managing a lot of different configurations? Strings comes with out-of-the-box support for Apache, PHP, Tomcat, Java, Node.js, MySQL, MongoDB, Redis, RabbitMQ, and Postfix! We'll even train your Engineers on how to build integration themselves!

### Maintain total control of your virtual machines and data

Strings lets you keep things “in-house” while providing the automation you need and the support of whatever technical stack you’ve chosen to use.

## Screenshots & Data Sheet

Want to see Strings in action? We've included some [great screenshots](/screenshots) as part of this repository. This [data sheet](datasheet.pdf) might also be quite useful.

## Disclaimer

Strings originally started as a product of Bitlancer and has since been converted to an open source project. Please keep this in mind when reviewing documentation, as there may be some legacy sales pitches and reference to a hosted service ;-) You will likely need our assistance setting Strings up and making it useful for your team. We're happy to help.

## Feedback & Contributions

We welcome feedback and contributions from the community! If you would like to contribute or leave feedback, please open an issue or issue a pull request with the edits or additions.

## Getting Started

New to Strings? This section will define some terms and concepts to help you get started.

## Devices

A device represents the lowest-level entity that can be managed. Currently, devices are restricted to virtual machines and load-balancers but we may add support for other devices like containers in the future.

### Device Names

Device names are assigned automatically and randomly from one or more dictionaries created by your organization. If you did not supply a dictionary when your account was created, it was generated randomly.

### Public DNS Entries

With the excpetion of load-balancers, Strings will automatically add public DNS entries for all of your devices when they are spun up. The DNS entries are created based on the following formula: device-name.target.organization-infra.net. 

Ex: python.dfw01.bitlancer-infra.net

## Formations

A formation represents a logical grouping of tightly coupled devices that are managed together. Think cluster. A formation does not need to contain more then one device however.

Two good examples of formations are:

* A MySQL formation composed of a MySQL master and a MySQL slave
* A web formation composed of a load-balancer and multiple application servers

## Applications

An application is composed of one or more formations and represents the full stack of services that are required to run the larger service you are offering. For example, a website built on Wordpress would require Apache, PHP, and MySQL and therefore might be composed of an Apache-PHP application server formation and a MySQL formation.

### Application DNS

Strings allows you to setup DNS records within the context of your application to serve as a simple mechanism for service discovery. To demonstrate this feature, lets look at an example.

* You've setup a wordpress application, www, which is composed of two formations: app server formation, database formation. The database formation is composed of two mysql servers, longo & jago, in a master-master replication mode.

* You setup application dns entries for each of the mysql servers as follows: db01.www.example-infra.net -> longo, db02.www.example-infra.net -> jago.

* You configure wordpress to use the dns entries setup in the previous step.

This configuration is advantageous because you can easily replace your database nodes, in the event of a node failure, upgrade, etc, without altering your application.

### Application Deployment

Putting application code and data in place is handled via *Deploy Scripts*.

#### Deploy Script

A *Deploy Script* is an executable that handles setting up your application. This can and often does include setting up your application code, pulling in libraries, running schema upgrades, and a slew of other tasks.

Currently Strings requires you to host your deploy script in a version control system. As a result, your deploy script can be written in any language or around any framework you would like. In order for us to support this kind of flexibility without compromising security we run your deploy script in your environment on a *Jump Server*.

#### Jump Server

A *Jump Server* is a server living in your environment within a particular region/target that has local access to all your devices within that region/target. The *Jump Server* serves as a staging area where the deploy script can orchestrate the larger deploy process.

#### Deploy User

A *Deploy User* is a special user account that the deploy script is run under. This account is required to have a username of `remoteexec`.

This user account is accompanied by a team, since privileges can only be granted on teams, and a sudo role, which controls what the deploy user can execute as root during a deploy.

#### Using the Strings Deploy Toolkit

The Strings Deploy Toolkit provides a framework for deploying applications within the Bitlancer Strings PaaS. For details on utilizing the toolkit visit its [repository](https://github.com/Bitlancer/strings-deploy-toolkit).

#### Using a Custom or 3rd Party Deploy Framework

As mentioned above Strings can execute code of your choosing during deploy. For it to be useful however, you will need to parse and utilize the parameters Strings passes to the deploy script. These parameters include information about the application such as its name and the list of servers and their roles. Below is a example of what Strings will pass to the deploy script. In addition, Strings will pass any user supplied parameters that were specified with the *Deploy Script* when it was created within the control panel.

```
--exec-id 836 --type Application --name test --server-list python.dfw01.int.example-infra.net,exampleorg::role::lamp_server,stringed::profile::apache_phpfpm,stringed::profile::mysql;anaconda.dfw01.int.example-infra.net,exampleorg::role::lamp_server,stringed::profile::apache_phpfpm,stringed::profile::mysql --verbosity 4 --repo git@github.com:Bitlancer/strings-sample-app.git
```

These parameters are custom to Strings. If you're using a 3rd part deploy framework, like Capistrano, you will likely need to create a wrapper that handles converting the Strings parameters into something useful for your deploy framework.

## Configuration Management

### Puppet 

Puppet is a framework for automating the configuration of a server.

### Hiera

### Roles, Profiles, & Components

Ex:

```puppet
class bitlancerorg::role::www_webserver inherits bitlancerorg::role {
  include bitlancerorg::profile::drupal
}
```

```puppet
class bitlancerorg::profile::drupal (
  $apache_listen = ['80','443'],
  $apache_name_virtual_hosts = ['*:80','*:443'],
  $apache_modules = ['fastcgi','ssl'],
  $apache_fastcgi_servers = {
    'www' => {
      host => '127.0.0.1:9000',
      timeout => 60,
      flush => false,
      faux_path => '/var/www/php.fcgi',
      alias => '/php.fcgi',
      file_type => 'application/x-httpd-php'
    }
  },
  $phpfpm_pools = {
    'www' => {
      listen  => '127.0.0.1:9000',
      user => 'apache',
      pm_max_requests => 500,
      catch_workers_output => 'no',
      php_admin_values => {},
      php_values => {}
    }
  },
  $php_modules = ['pdo','mysql'],
  $firewall_rules = {},
  $backup_jobs = {},
  $cron_jobs = {}
) {
  
  class { ::bitlancerorg::wrapper::apache_phpfpm:
    apache_listen => $apache_listen,
    apache_name_virtual_hosts => $apache_name_virtual_hosts,
    apache_modules => $apache_modules,
    apache_fastcgi_servers => $apache_fastcgi_servers,
    phpfpm_pools => $phpfpm_pools,
    php_modules => $php_modules
  }

  create_resources(::firewall, $firewall_rules)
  create_resources(::duplicity::job, $backup_jobs) 
  create_resources(::cron::job, $cron_jobs)
}
```

```puppet
class bitlancerorg::wrapper::apache_phpfpm (
  $apache_listen = ['80'],
  $apache_name_virtual_hosts = ['*:80'],
  $apache_modules = ['fastcgi'],
  $apache_fastcgi_servers = {},
  $phpfpm_pools = {},
  $php_modules = []
) {

  include ::apache
  ::apache::listen { $apache_listen: }
  ::apache::namevirtualhost { $apache_name_virtual_hosts: }
  ::apache::mod { $apache_modules: }
  create_resources(::apache::fastcgi::server, $apache_fastcgi_servers)
  
  include ::php::fpm::daemon
  create_resources(::php::fpm::conf, $phpfpm_pools)
  ::php::module { $php_modules: } ~> Service['php-fpm']
  
  # Create the apache user before the php-fpm
  # service is started
  Package['apache'] -> Package['php-fpm']
}
```

## User Management

Users and Teams can be managed within the Strings control panel.

### Control Panel Privileges

These privileges affect what a user can do within the Strings control panel. When a new user is created, that user is assigned either *Administrator* or *User* privileges within the Strings control panel. *Administrators* are permitted to execute any action whereas *Users* are restricted from executing any action beyond those specific to themselves such as changing a password.

### Infrastructure Privileges

These privileges control whether a user can log onto a device, *Login Privileges*, and what they can execute once they've logged on, *Sudo Privileges*. 

#### Granting Privileges

Privileges can be granted on an Application, Formation, Device, or Role with the *Unix Privileges* action menu item. When privileges are granted on an Application, Formation, or Role, any device that falls within those criteria will be affected. For example, if you grant yourself *Login Privileges* to a application, you are effectively granting yourself login privileges to any device that is a member that application.

#### Sudo Privileges & Sudo Roles

Sudo allows an administrator the ability to delegate a user privileges to run certain commands as root or another user. To learn more about sudo, checkout its [man page](http://www.sudo.ws/sudo/sudo.man.html).

Within Strings, when you grant sudo privileges you must specify one or more *Sudo Roles*. A *Sudo Role* encapsulates a sudo privilege and consists of a user and series of commands.

#### Credential & Sudo Privilege Caching

The underlying authentication system Strings is using will cache credentials and sudo privileges. This is configurable but it currently defaults to 15 minutes. **Any change you make could take up to 15 minutes before it takes effect.**

## Troubleshooting

### I can't login to a server

1) Did you grant your team login privileges?

You must grant your team, or any team you are a member of, login privileges via *Unix Privileges* on the individual device, or the Formation or Application the device is a member of. See [Granting Privileges](#granting-privileges).

2) Did you recently change your password or the *Unix Privileges* of the device?

See [Credential & Sudo Privilege Caching](#credential--sudo-privilege-caching)

3) Did you verify you are a member of the team you granted privileges?

We all make mistakes :)

4) I tried everything above, what now?

Contact us at support@bitlancer.com, or open an issue on Github, and we'll try to help resolve your problem.
