# Systemd

[![Puppet Forge](http://img.shields.io/puppetforge/v/camptocamp/systemd.svg)](https://forge.puppetlabs.com/camptocamp/systemd)
[![Build Status](https://travis-ci.org/camptocamp/puppet-systemd.png?branch=master)](https://travis-ci.org/camptocamp/puppet-systemd)

## Overview

This module declares exec resources to create global sync points for reloading systemd.

## Usage and examples

There are two ways to use this module.

### unit files

Let this module handle file creation and systemd reloading.

```puppet
::systemd::unit_file { 'foo.service':
 source => "puppet:///modules/${module_name}/foo.service",
}
```

Or handle file creation yourself and trigger systemd.

```puppet
include ::systemd::systemctl::daemon_reload

file { '/usr/lib/systemd/system/foo.service':
  ensure => file,
  owner  => 'root',
  group  => 'root',
  mode   => '0644',
  source => "puppet:///modules/${module_name}/foo.service",
}
~> Class['systemd::systemctl::daemon_reload']
```

### tmpfiles

Let this module handle file creation and systemd reloading

```puppet
::systemd::tmpfile { 'foo.conf':
  source => "puppet:///modules/${module_name}/foo.conf",
}
```

Or handle file creation yourself and trigger systemd.

```puppet
include ::systemd::tmpfiles

file { '/etc/tmpfiles.d/foo.conf':
  ensure => file,
  owner  => 'root',
  group  => 'root',
  mode   => '0644',
  source => "puppet:///modules/${module_name}/foo.conf",
}
~> Class['systemd::tmpfiles']
```

### service limits

Manage soft and hard limits on various resources for executed processes.

```puppet
::systemd::service_limits { 'foo.service':
  limits => {
    LimitNOFILE => 8192,
    LimitNPROC  => 16384,
  }
}
```

Or provide the configuration file yourself. Systemd reloading and restarting of the service are handled by the module.

```puppet
::systemd::service_limits { 'foo.service':
  source => "puppet:///modules/${module_name}/foo.conf",
}
```

### network

systemd-networkd is able to manage your network configuration. We provide a
defined resource which can write the interface configurations. systemd-networkd
needs to be restarted to apply the configs. The defined resource can do this
for you:

```puppet
::systemd::network{'eth0.network':
  source          => "puppet:///modules/${module_name}/eth0.network",
  restart_service => true,
}
```

### Services

Systemd provides multiple services. Currently you can manage `systemd-resolved`
and `systemd-networkd` via the main class:

```puppet
class{'::systemd':
  $manage_resolved => true,
  $manage_networkd => true,
```

$manage_networkd is required if you want to reload it for new
`::systemd::network` resources. Setting $manage_resolved will also manage your
`/etc/resolv.conf`.
