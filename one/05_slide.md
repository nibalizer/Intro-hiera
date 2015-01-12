!SLIDE commandline incremental


    # ln -s /etc/hiera.yaml /etc/puppet/hiera.yaml



!SLIDE commandline incremental


    # cat /etc/puppet/hiera.yaml
    ---
    :backends:
      - yaml

    :yaml:
      :datadir: /etc/puppet/hieradata

    :hierarchy:
      - "%{clientcert}/common"
      - "osfamily/%{osfamily}/common"
      - common

!SLIDE commandline incremental



    # find /etc/puppet/hieradata
    .
    ./common.yaml
    ./osfamily
    ./osfamily/RedHat
    ./osfamily/RedHat/common.yaml
    ./osfamily/Debian
    ./osfamily/Debian/common.yaml


!SLIDE incremental

# Hiera #

* A place to put your data
* Backend driven
* Function call to lookup on keys



!SLIDE


    @@@puppet
    class { 'jenkins::slave':
      jenkins_ssh_key => 'AAAAB3Nzbu84a....'
    }

!SLIDE commandline incremental


    # cat /etc/puppet/hieradata/common.yaml
    ---
    jenkins_key: AAAAB3NzaC1yc2EAAAADA...
    ...

    # hiera -d jenkins_key
    DEBUG: Hiera YAML backend starting
    DEBUG: Looking up jenkins_key in YAML backend
    DEBUG: Looking for data source common
    DEBUG: Found jenkins_key in common

    AAAAB3NzaC1yc2EAAAADAQAB...

!SLIDE code


    @@@puppet
    $ssh_key = hiera('jenkins_key')
    class { 'jenkins::slave':
        jenkins_ssh_key => $ssh_key,
    }

!SLIDE


    @@@puppet
    class { 'mysql::server':
        root_password => 'hunter2',
    }


!SLIDE commandline incremental


    # cat /etc/puppet/hieradata/common.yaml
    ---
    ...
    mysql_root_password: hunter2
    ...


    # hiera -d mysql_root_password
    DEBUG: Hiera YAML backend starting
    DEBUG: Looking up mysql_root_password in YAML backend
    DEBUG: Looking for data source common
    DEBUG: Found mysql_root_password in common

    hunter2

!SLIDE code


    @@@puppet
    $password = hiera('mysql_root_password')

    class { 'mysql::server':
      root_password => $password,
    }

!SLIDE

# Questions? #

!SLIDE


    @@@puppet
    class graphite {
      if $::osfamily == 'RedHat' {
        $pkgs = [
            'git',
            'python-django',
            'g++',
            'sqlite3',]
      ...
      }
    }



!SLIDE incremental

# Hiera #
* Hierarchy that is facter aware
* Defaults and overrides


!SLIDE commandline incremental


    # cat /etc/puppet/hiera.yaml
    ---
    :backends:
      - yaml

    :yaml:
      :datadir: /etc/puppet/hieradata

    :hierarchy:
      - "%{clientcert}/common"
      - "osfamily/%{osfamily}/common"
      - common

!SLIDE commandline incremental



    # find /etc/puppet/hieradata
    .
    ./common.yaml
    ./osfamily
    ./osfamily/RedHat
    ./osfamily/RedHat/common.yaml
    ./osfamily/Debian
    ./osfamily/Debian/common.yaml



!SLIDE

## Conditional data in code

    @@@puppet
    class { 'graphite':
        if $::osfamily == 'RedHat' {
            $pkgs = [
                'git',
                'python-django',
                'g++',
                'sqlite3',]
        ...
        }
    }



!SLIDE commandline incremental



    # cat osfamily/Debian/common.yaml
    ---
    graphite::pkgs:
      - graphite
      - python-django
      - virtualenv

!SLIDE commandline incremental



    # cat osfamily/RedHat/common.yaml 
    ---
    graphite::pkgs:
      - git
      - python-django
      - g++
      - sqlite3
      - sqlite3-devel
      - python26-virtualenv

!SLIDE commandline incremental

## Hiera data

    # hiera graphite::pkgs  osfamily=RedHat
    ["git",
     "python-django",
     "g++",
     "sqlite3",
     "sqlite3-devel",
     "python26-virtualenv"]

!SLIDE commandline incremental


    # hiera graphite::pkgs  osfamily=Debian
    ["graphite", "python-django", "virtualenv"]


!SLIDE commandline incremental


    # hiera graphite::pkgs
    nil



!SLIDE


    @@@puppet
    class graphite {
        if $::osfamily == 'RedHat' {
            $pkgs = [
                'git',
                'python-django',
                'g++',
                'sqlite3',]
        ...
        }
    }


!SLIDE


    @@@puppet
    class graphite {
      $pkgs = hiera('graphite::pkgs')
      package { $pkgs:
        ensure => latest,
      }
    }



!SLIDE

## Backends

* yaml, json
* file, ldap
* gpg, eyaml
* mysql, postgres, redis

!SLIDE

## Pros

* Separation between data and code
* Secret storage
* Backends, integration with existing datastores
* Some conditional logic irrelevant
* Puppet code sanitized

!SLIDE

## Cons

* hard to figure out where things come from
* hiera-yaml can only support one data directory
* debugging
* public modules + hirea is unsolved


!SLIDE

## In module data:


[puppet-module-data](https://github.com/ripienaar/puppet-module-data/)

!SLIDE incremental

# User issues #

* Complicated hierarchy
* Runaway backends
* Latency/Load
* Architecture

!SLIDE incremental

# Positive note #

* Use hiera, its awesome
* Start with yaml
* Try and experiment, iterate


!SLIDE

# Questions on Hiera

